---
date: '2025-08-06T18:02:24+08:00'
draft: false
title: '发现 debtap 中一个“删库跑路”级 Bug'
summary: '一篇对 Arch Linux 工具 `debtap` 的 Bug 调查，揭示了一个看似无害的拼写错误修复，是如何意外触发了删除当前目录下所有文件的“rm -rf”致命缺陷。'
tags: [ 'debtap', 'bug', 'rm-rf', 'shell-script', 'bash', 'arch-linux', 'pkgbuild', 'dirname', 'debugging', 'git', 'git-blame', 'open-source', 'data-loss', 'linux', 'command-line', 'software-development', 'code-review', 'typo' ]
---

故事的开头平淡无奇。我需要安装一个名为 `SwashbucklerDiary` 的软件，官方只提供了 `.deb` 包。这对于 Arch Linux 用户来说不是问题，
`debtap` 正是为此而生。

我像往常一样，创建了一个临时目录来处理这次转换，以免污染我的下载文件夹。

```bash
> pwd
/home/myusername/Downloads/tmp

> ls
SwashbucklerDiary-1.17.0-linux-x64.deb
```

一切正常。目录里只有我刚刚下载的 `.deb` 文件。我首先执行了标准的 `debtap` 命令，它顺利地生成了我想要的 `pkg.tar.zst` 包。

```bash
> debtap SwashbucklerDiary-1.17.0-linux-x64.deb
... (省略转换过程输出) ...
==> Package successfully created!
==> Removing leftover files...

> ls -alh
total 106M
drwxr-xr-x 2 myusername myusername 116 Aug  6 12:34 .
drwxr-xr-x 4 myusername myusername  41 Aug  6 12:34 ..
-rw-r--r-- 1 myusername root        58M Aug  6 12:34 com.yucore.swashbucklerdiary-1.17.0-1-x86_64.pkg.tar.zst
-rw-r--r-- 1 myusername myusername 49M Aug  4 18:14 SwashbucklerDiary-1.17.0-linux-x64.deb
```

完美。转换后的包和原始的 `.deb` 包都在这里。但接着，我出于好奇和学习的目的，想看看 `debtap` 生成的 `PKGBUILD` 文件是什么样的。
`debtap` 提供了 `-p` 或 `-P` 标志来生成 `PKGBUILD`。于是，我删除了刚刚生成的文件，重新执行了命令，这次带上了 `-p` 标志。

```bash
> debtap -p SwashbucklerDiary-1.17.0-linux-x64.deb
... (同样的交互式提问) ...
==> Package successfully created!
==> Generating PKGBUILD file...
mv: cannot stat 'PKGBUILD': No such file or directory
==> PKGBUILD is now located in "/home/myusername/Downloads/tmp" and ready to be edited
==> Removing leftover files...
```

输出看起来有点奇怪，有一个 `mv: cannot stat 'PKGBUILD': No such file or directory` 的错误，但最后它依然提示我 `PKGBUILD`
已经生成在当前目录了。我没有多想，习惯性地敲下了 `ls -alh`。

然后，我看到了让我脊背发凉的一幕。

```bash
> ls -alh
total 0
```

空空如也。

我的第一反应是震惊。不仅是预期的 `PKGBUILD` 目录没有出现，连原始的 `.deb` 文件也消失了！整个 `tmp` 目录被清空了。

更诡异的事情还在后面。我尝试 `cd ..` 然后再 `cd tmp` 回来，我的 shell 提示符（我用的是 Oh My Zsh +
Powerlevel10k）显示出了一些奇怪的现象，似乎连目录本身的元数据都受到了影响。当我再次 `ls -alh` 时，我看到了更让我困惑的输出：

```bash
> ls -alh
total 0
drwxr-xr-x 2 myusername myusername  6 Aug  6 12:35 .
drwxr-xr-x 4 myusername myusername 41 Aug  6 12:35 ..
```

注意看 `.` 目录的大小，只有 6 字节。一个正常的、刚刚被清空的目录，在 XFS 文件系统上大小通常是 4096 字节。这太不寻常了。

我的大脑开始飞速运转。我的 `/home` 目录建立在两块 SSD 组成的 RAID0 上，文件系统是 XFS。我第一个念头是：“完了，RAID0
是不是出问题了？难道这么快就坏了一块盘？” RAID0 带来的高性能是以零冗余为代价的，任何一块硬盘的故障都会导致整个阵列的数据丢失。我开始检查
`dmesg` 和系统日志，但没有发现任何 I/O 错误或文件系统损坏的迹象。

排除了硬件和文件系统问题后，我冷静下来，开始怀疑 `debtap` 本身。既然第一次不带 `-p` 的运行是正常的，而第二次带了 `-p`
就出事了，那么问题很可能就出在这个参数上。

我决定复现这个问题，但这次要在一个绝对安全的环境下。我新建了一个测试目录，放了几个无关紧要的触摸文件和一个 `.deb` 包的副本。

```bash
mkdir ~/safe-test
cd ~/safe-test
touch fileA.txt fileB.log
cp ~/Downloads/SwashbucklerDiary-1.17.0-linux-x64.deb .
ls -l
# total 49264
# -rw-r--r-- 1 myusername myusername 50442386 Aug  4 18:14 SwashbucklerDiary-1.17.0-linux-x64.deb
# -rw-r--r-- 1 myusername myusername        0 Aug  6 13:00 fileA.txt
# -rw-r--r-- 1 myusername myusername        0 Aug  6 13:00 fileB.log
```

接着，我屏住呼吸，再次执行了那条“魔鬼”命令：

```bash
debtap -p SwashbucklerDiary-1.17.0-linux-x64.deb
```

走完流程后，我再次 `ls`。

```bash
ls -l
# total 0
```

结果一模一样。目录被清空了。

至此，案情明朗了。这不是什么灵异事件，也不是硬件故障，而是 `debtap` 在使用 `-p` 或 `-P` 参数时，存在一个极其危险的
Bug，它会删除当前工作目录下的所有文件。

确定了问题所在，下一步就是找出原因。`debtap` 是一个 shell 脚本，这让源码分析变得非常直接。我打开了 `/usr/bin/debtap` 文件，版本号是
`3.6.2`。这是一个长达三千多行的庞大脚本，直接通读显然不现实。

我的调查思路很明确：

1. Bug 与 `-p`/`-P` 参数强相关。
2. 这个参数的功能是生成 `PKGBUILD`。
3. 最终现象是当前目录被删除。

所以，我需要在脚本中找到处理 `-p`/`-P` 参数，并最终生成、移动 `PKGBUILD` 文件的相关代码块。我直接在代码中搜索关键字
`pkgbuild`。

很快，我在脚本的末尾部分，找到了生成和处理 `PKGBUILD` 的逻辑。

```bash
# ... (前面是生成 PKGBUILD 内容的代码) ...

# Moving PKGBUILD (and .INSTALL, if it exists) and announcing its creation
pkgname="$(grep '^pkgname=' PKGBUILD | sed s'/^pkgname=//')"
if [[ $output == set ]]; then
    pkgbuild_location="$(dirname "$outputdirectory/$pkgname-PKGBUILD")"
    rm -rf "$pkgbuild_location" 2> /dev/null
    mkdir "$pkgbuild_location" 2> /dev/null
    # ... (错误处理和移动文件的代码) ...
else
    pkgbuild_location="$(dirname ""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD")"
    rm -rf "$pkgbuild_location" 2> /dev/null
    mkdir "$pkgbuild_location" 2> /dev/null
    # ... (错误处理和移动文件的代码) ...
fi
```

我的目光立刻被 `rm -rf "$pkgbuild_location"` 这行代码吸引住了。这无疑是最大的嫌疑犯。脚本在这里执行了一个强制递归删除操作。现在的问题是，
`$pkgbuild_location` 这个变量的值到底是什么？

我们来看 `else` 分支中的这行关键代码，因为我没有使用 `-o` 输出目录选项，所以程序会走到这里：

```bash
pkgbuild_location="$(dirname ""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD")"
```

这行代码看起来有些复杂，嵌套了两层 `dirname` 命令。让我们来庖丁解牛，一步步分析它的执行过程。

`dirname` 是一个基础的 shell 命令，它的作用是去除文件名，返回其所在的目录路径。例如：

- `dirname /usr/bin/ls` 会返回 `/usr/bin`
- `dirname /home/user/file.txt` 会返回 `/home/user`

现在，我们把实际的变量值代入进去。

1. **`$package_with_full_path`**：这个变量在脚本开头被定义为输入 `.deb` 文件的绝对路径。在我的例子中，它的值是
   `/home/myusername/Downloads/tmp/SwashbucklerDiary-1.17.0-linux-x64.deb`。
2. **`$pkgname`**：这个变量是从临时生成的 `PKGBUILD` 文件中提取的包名。根据我的日志，转换后的包名是
   `com.yucore.swashbucklerdiary-1.17.0-1`。

现在，我们来解析那个嵌套的命令：

**第一步：执行内层的 `dirname`**

```bash
"$(dirname "$package_with_full_path")"
# 等价于
"$(dirname "/home/myusername/Downloads/tmp/SwashbucklerDiary-1.17.0-linux-x64.deb")"
```

这步的输出是 `.deb` 文件所在的目录：`/home/myusername/Downloads/tmp`。

**第二步：拼接字符串**

上一步的结果会和后面的字符串拼接起来，形成一个更长的路径字符串：

```bash
"/home/myusername/Downloads/tmp/$pkgname-PKGBUILD"
# 代入 $pkgname 的值
"/home/myusername/Downloads/tmp/com.yucore.swashbucklerdiary-1.17.0-1-PKGBUILD"
```

这个字符串的含义是：在 `.deb` 文件所在的目录中，创建一个名为 `包名-PKGBUILD` 的……等等，这似乎是一个**文件**路径，而不是**目录
**路径。作者的意图应该是创建一个名为 `包名-PKGBUILD` 的**目录**，然后把 `PKGBUILD` 文件放进去。

**第三步：执行外层的 `dirname`**

现在，最关键的一步来了。脚本对上一步生成的整个字符串执行了外层的 `dirname`：

```bash
"$(dirname "/home/myusername/Downloads/tmp/com.yucore.swashbucklerdiary-1.17.0-1-PKGBUILD")"
```

这个命令的输出是什么？正是 `/home/myusername/Downloads/tmp`！

**真相大白**

经过这三步分析，我们得到了 `pkgbuild_location` 变量的最终值：`/home/myusername/Downloads/tmp`，也就是我执行 `debtap` 命令时所在的
**当前工作目录**。

现在再回看那几行致命的代码：

```bash
pkgbuild_location="/home/myusername/Downloads/tmp"
rm -rf "$pkgbuild_location"  # 相当于执行 rm -rf "/home/myusername/Downloads/tmp"
mkdir "$pkgbuild_location" # 相当于执行 mkdir "/home/myusername/Downloads/tmp"
```

谜底揭晓了。脚本先是计算出了一个错误的路径——当前工作目录，然后毫不犹豫地执行了 `rm -rf`，将这个目录连同其内部所有文件（包括原始的
`.deb` 包）一并删除。紧接着，`mkdir` 命令又重新创建了这个目录，这就是为什么我最后看到了一个空空如也的 `tmp`
目录，连目录本身的元数据都像是“初始化”了。

这真是一个经典而又可怕的逻辑错误。作者的本意可能是想确保目标目录是一个干净的新目录，所以先删除后创建。但他错误地使用了两次
`dirname`，导致删除的目标从“预想中的子目录”变成了“整个当前目录”。

找到问题根源后，我冒出了一个新的想法：这么严重的 Bug，不太可能是 `debtap` 一直以来就有的，否则早就被发现了。它很可能是近期才被引入的。

我决定去 `debtap` 的 GitHub 仓库进行“代码考古”，看看这个 Bug 的前世今生。通过 `git blame` 和翻阅提交历史，我很快锁定了一个可疑的提交：
**commit `27a9ff5`**。

这个提交的信息很简单，就是一次代码更新。我们来看看它的 `diff`：

```diff
diff --git a/debtap b/debtap
index 4518a7a..71aea20 100755
--- a/debtap
+++ b/debtap
@@ -3458,8 +3458,8 @@ if [[ $output == set ]]; then
    fi
 else
    pkgbuild_location="$(dirname ""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD")"
-   rm -rf "$pkgbuilt_location" 2> /dev/null
-   mkdir "$pkgbuilt_location" 2> /dev/null
+   rm -rf "$pkgbuild_location" 2> /dev/null
+   mkdir "$pkgbuild_location" 2> /dev/null
    if [[ $? != 0 ]]; then
        echo -e "${red}Error: Cannot create PKGBUILD directory to the same directory as .deb package, permission denied. Removing leftover files and exiting...${NC}"
        rm -rf "$working_directory"
```

看到这里，我恍然大悟，甚至有点哭笑不得。

在这次提交之前，代码是这样的：

```bash
rm -rf "$pkgbuilt_location" 2> /dev/null
mkdir "$pkgbuilt_location" 2> /dev/null
```

注意看变量名：`pkgbuilt_location`。而上面定义变量时用的是 `pkgbuild_location`。这是一个**拼写错误**！

在 shell 脚本中，如果引用一个不存在的变量（比如因为拼写错误），它会扩展成一个空字符串。所以，在 `27a9ff5` 这个提交之前，实际执行的命令是：

```bash
rm -rf "" 2> /dev/null
mkdir "" 2> /dev/null
```

`rm -rf ""` 和 `mkdir ""` 都不会产生任何效果，也不会报错。因此，那个有逻辑缺陷的 `dirname` 虽然计算出了错误的路径，但由于这个拼写错误，它从未被用在
`rm -rf` 命令中。这个拼写错误，就像一个保险丝，阴差阳错地保护了无数用户的数据安全。

而 **commit `27a9ff5`** 的作者，很可能是在代码审查时发现了这个拼写错误，本着“修正代码”的好意，将 `pkgbuilt_location` 改成了正确的
`pkgbuild_location`。他“修复”了这个拼写错误，却无意中接通了那根引爆炸弹的引线。

这是一个教科书级别的案例，告诉我们一个看似微不足道的、善意的修改，如果没有完全理解其上下文和潜在影响，也可能导致灾难性的后果。

发现了问题的来龙去脉后，我意识到必须尽快将此问题报告给项目维护者，以防更多用户遭殃。我立刻在 `debtap` 的 GitHub 仓库创建了一个新的
Issue。

Issue 提交后，很快得到了社区的回应。有其他用户证实他们也遇到了同样的问题，其中一位用户庆幸自己没有在 `$HOME`
目录下运行这个命令。这再次凸显了问题的严重性。

项目维护者 helixarch 很快注意到了这个问题，并在几天后发布了修复。我们来看一下修复这个 Bug 的核心 `diff`：

```diff
--- a/debtap
+++ b/debtap
@@ -3486,7 +3486,7 @@ if [[ $output == set ]]; then
        echo -e "${lightgreen}==>${NC} ${bold}PKGBUILD is now located in${normal} ${lightblue}\"$pkgbuild_location\"${NC} ${bold}and ready to be edited${normal}"
    fi
 else
-   pkgbuild_location="$(dirname ""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD")"
+   pkgbuild_location=""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD"
    rm -rf "$pkgbuild_location" 2> /dev/null
    mkdir "$pkgbuild_location" 2> /dev/null
    if [[ $? != 0 ]]; then
```

修复方案非常直接、优雅。维护者**移除了外层的 `dirname`**。

现在，`pkgbuild_location` 的计算方式变成了：

```bash
pkgbuild_location=""$(dirname "$package_with_full_path")"/$pkgname-PKGBUILD"
```

我们再来走一遍流程：

1. `dirname "$package_with_full_path"` 仍然是 `/home/myusername/Downloads/tmp`。
2. 拼接后的字符串是 `/home/myusername/Downloads/tmp/com.yucore.swashbucklerdiary-1.17.0-1-PKGBUILD`。

这个值现在被直接赋给了 `pkgbuild_location`。于是，后续的命令变成了：

```bash
rm -rf "/home/myusername/Downloads/tmp/com.yucore.swashbucklerdiary-1.17.0-1-PKGBUILD"
mkdir "/home/myusername/Downloads/tmp/com.yucore.swashbucklerdiary-1.17.0-1-PKGBUILD"
```

这正是我们期望的行为！脚本现在会正确地在当前目录下创建一个新的、干净的子目录，用来存放 `PKGBUILD` 文件，而不会再对当前目录本身造成任何威胁。

随着 `debtap 3.6.3` 版本的发布，这个惊心动魄的 Bug 终于被修复了。