# slackbuild-guidelines

## 关于

这里是一份Slackware 中文社区的SlackBuild 规范。

阅读这份规范之前，默认你已经阅读、理解并遵守[SlackBuilds.org](https://slackbuilds.org/) 的[Guidelines](https://slackbuilds.org/guidelines/)。

## 模板

你可以在[emplates](templates) 目录查看我们正在使用的SlackBuild 模板，请按照模板的格式编写SlackBuild。

## 规范

### Place you build at The Slackware Linux CN Community SlackBuilds

每一个SlackBuild 都使用Git 仓库的形式存放，你需要成为[slackwarecn-slackbuilds](https://github.com/slackwarecn-slackbuilds) 的成员，在这里新建项目，并将你的SlackBuilds 推送上来。

### PRGNAM is repository name

你的Git 项目名称必须和SlackBuild 脚本中的`$PRGNAM` 一致（包括大小写）。

### Use space instead of tab

脚本中请不要出现任何TAB 字符。因为在不同编辑器设置下，TAB 字符的长度是不统一的，而空格的长度是统一的。

### Every file encode with UTF-8

请在所有的文件中使用UTF-8 编码，不要使用其他编码方案例如CP936。

### Only English is allowed

在你的所有文件中避免使用非英文单词，因为你的SlackBuild 会被世界各地的人阅读。

> 所以也请写可读性高的代码。

### Use bash as shebang

你需要将shebang 设置为bash。因为`sh` 是一个软连接，通常来说它指向`bash`，但是`sh` 是不可靠的，它可以被用户修改，而`bash` 是可靠的。

```bash
#!/usr/bin/env bash
```

### Add executable permission to you SlackBuild

你需要为你的SlackBuild 脚本增加可执行权限，否则shebang 将没有任何意义。

### Avoid to use root

请确保你的Build 脚本**不需要使用**root 权限即可成功构建软件包，否则你的Build 将不会被收录于[repo](https://github.com/slackwarecn/repo)。

### Sign your commit with GPG as far as possible

请尽量使用你的GPG 密钥环签名你的提交，你需要在项目中额外增加`fingerprint` 文件来写明你的主钥指纹。一个例子如下。

```
4444 9495 01B3 CC2A 7657  9C08 25FF D92A B66C C194
```

> 关于如何签名提交可以参考[这里](http://arondight.me/2016/04/17/%E4%BD%BF%E7%94%A8GPG%E7%AD%BE%E5%90%8DGit%E6%8F%90%E4%BA%A4%E5%92%8C%E6%A0%87%E7%AD%BE/)。

### Use master branch

必须使用master 分支来存放你的SlackBuild。

### Add your name to Copyright

你需要在脚本的源文件中显式写明版权信息并在其中加入你的名字。一个例子如下。

```bash
# Copyright (c) 2016 Arondight <shell_way@foxmail.com>,
# All rights reserved.
```

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

### Use `set -e` at the very beginning of script

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
REQUIRES="packageA packageB"
```

### Test your SlackBuild and packages

你需要自己测试一遍SlackBuild 是否能够正确生成软件包，以及测试软件包是否能够正常工作。

