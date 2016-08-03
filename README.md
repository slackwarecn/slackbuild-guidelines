# slackbuild-guidelines

## 关于

这里是一份Slackware 中文社区的SlackBuild 规范。

阅读这份规范之前，默认你已经阅读、理解并遵守[SlackBuilds.org](https://slackbuilds.org/) 的[Guidelines](https://slackbuilds.org/guidelines/)。

## 规范

### Use space instead of tab

脚本中请不要出现任何TAB 字符。因为在不同编辑器设置下，TAB 字符的长度是不统一的，而空格的长度是统一的。

### Every file encode with UTF-8

请在所有的文件中使用UTF-8 编码，不要使用其他编码方案例如CP936。

### Only English is allowed

在你的所有文件中避免使用非英文单词，因为你的SlackBuild 会被世界各地的人阅读。

> 所以也请写可读性高的代码。

### Add "The Slackware Linux CN Community" to Copyright

你需要在脚本的源文件中显式写明版权信息并在其中加入`The Slackware Linux CN Community`，一个例子如下。

```bash
# Copyright (c) 2016 nnnewb <weak_ptr>,
#                    Arondight <shell_way@foxmail.com>,
#                    The Slackware Linux CN Community (https://github.com/slackwarecn)
# All rights reserved.
```

### Use bash as shebang

你需要将shebang 设置为bash。

```bash
#!/usr/bin/env bash
```

因为`sh` 是一个软连接，通常来说它指向`bash`，但是`sh` 是不可靠的，它可以被用户修改，而`bash` 是可靠的。

### Avoid to use root

请确保你的Build 脚本不需要使用root 权限，否则你的Build 将不会被收录于[repo](https://github.com/slackwarecn/repo)。

因为在软件的Build 过程中一定是可以避免使用root 权限的，并且使用root 权限的SlackBuild 通常被用户拒绝执行。

### Write `doinst.sh` even it is useless

即使你不准备在`doinst.sh` 中做什么事情，你也需要创建这个文件，一个什么也不做的`doinst.sh` 如下。

```bash
#!/usr/bin/env bash
( : )
```

> 安装软件包时，这个文件将会被复制到`/var/log/scripts/PRGNAM`。

### Use environment variable instead of argument

如果你的SlackBuild 允许用户控制行为，你不应当使用参数来完成这一功能，而应当使用环境变量来进行。一个例子如下。

```bash
# In you SlackBuild
PKGTYPE=${PKGTYPE:-txz}
TAG=${TAG:-_SBo}
```

这样一来用户可以手动通过环境变量控制SlackBuild 的行为。

```
$ PKGTYPE=tgz TAG="_$(whoami)" ./PRGNAM.SlackBuild
```

这是一个约定俗成的规范。

### Use "set -e" at the very beginning of script

当你的脚本有指令执行失败时，脚本应该停止执行。你可以在你脚本的最开始使用它。

```bash
# shebang line
# Copyright and other information
set -e
# Script begin
```

### Show verbose output

指令应该尽量打印详细输出，有些指令接受`-v` 或者`--verbose` 参数，用来打印更多的信息。下面是接受`-v` 参数的常用指令列表。

1. `ar`
+ `chmod`
+ `chown`
+ `cp`
+ `cpio`
+ `intall`
+ `ln`
+ `mkdir`
+ `mv`
+ `rm`
+ `rmdir`
+ `tar`

### Write requires to `.info` file

你需要计算好所有需要的包，无论是构建还是运行时需要的，并将其写入到`${PRGNAM}.info` 文件中。一个例子如下。

```bash
REQUIRES="alsa-lib gst-libav gst-plugins-base gst-plugins-good gst-plugins-ugly kconfig libcue libxkbcommon mozilla-nss qt5"
```

### `slack-desc` with at least 11 lines

你需要将软件包的说明写入`slack-desc` 文件中，这个文件以`PRGNAM:` 的行会在安装包时被打印，在`slack-desc` 中应该至少有11 行可以被打印的内容（即使是空行）。

在其中你需要写明包名、软件名称、软件说明和主页。一个例子如下。

```
       |-----handy-ruler------------------------------------------------------|
extra-cmake-modules: extra-cmake-modules (Extra modules and scripts for CMake)
extra-cmake-modules:
extra-cmake-modules: The Extra CMake Modules package, or ECM, adds to the modules provided by CMake,
extra-cmake-modules: including ones used by find_package() to find common software,
extra-cmake-modules: ones that can be used directly in CMakeLists.txt files to
extra-cmake-modules: perform common tasks and toolchain files that must be specified on
extra-cmake-modules: the commandline by the user.
extra-cmake-modules:
extra-cmake-modules:
extra-cmake-modules: Homepage: https://community.kde.org/Frameworks
extra-cmake-modules:
```

### Set source file from URL

你需要显式在SlackBuild 中指定URL，然后根据URL 设置源文件的文件名。一个例子如下。

```bash
DOWNLOADS=(
  "http://download.kde.org/stable/frameworks/${VERSION%.*}/${PRGNAM}-${VERSION}.tar.xz"
)
SOURCES=(
  $(basename ${DOWNLOADS[0]})
  './pri-install-dir.patch'
)
```

### Download source file if `PREBUILD=yes`

你需要在脚本中根据环境变量`PREBUILD` 的值来控制是否由脚本主动下载源文件。一个例子如下。

```bash
# 这个过程应该在MD5 校验之前
if [[ 'yes' == $PREBUILD ]]
then
  test -n ${DOWNLOADS[0]}
  wget -N ${DOWNLOADS[0]}
fi
```

这样一来用户可以显式指定自动下载源文件。

```
$ PREBUILD=yes ./PRGNAM.SlackBuild
```

### Verify MD5 during build

你需要在构建软件包的时候额外验证所需要文件的MD5 是否正确，这些你需要在脚本中显式写定。一个例子如下。

```bash
MD5SUMS=(
  'cd3b0c844234ad29cfdba89d63ccb2ae'  # extra-cmake-modules-5.24.0.tar.xz
  '76ec20005b7a78be8ac88265b436c13a'  # pri-install-dir.patch
)

for ((index = 0; index < ${#MD5SUMS[@]}; ++index))
do
  expectmd5=${MD5SUMS[$index]}
  curmd5=$(md5sum ${SOURCES[$index]} | awk '{print $1}')

  echo -n "Verify file \"${SOURCES[$index]}\" ... "
  if [[ $expectmd5 != $curmd5 ]]
  then
    echo "failed"
    ( echo "Error \"${SOURCES[$index]}\":"
      echo -e "\texpect md5 \"${expectmd5}\""
      echo -e "\tcurrent md5 \"${curmd5}\""
    ) >&2
  else
    echo "ok"
  fi
done
```

因为固然`.info` 文件中已经明确了MD5，但是少有用户手动校验。

> MD5 数组中MD5 值的顺序需要和源文件数组一一对应。

### Test your SlackBuild and packages

你需要自己测试一遍SlackBuild 是否能够正确生成软件包，以及测试软件包是否能够正常工作。
