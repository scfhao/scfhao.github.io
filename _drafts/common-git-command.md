# 常用的Git命令

## 前言

Git是程序员的好伙伴，不懂Git一定会被同行鄙视的。更多关于Git的内容可以在[Git官网](https://git-scm.com)看看，快速了解Git可以试一下[Code School的十五分钟教程“Try Git“](https://try.github.io/levels/1/challenges/1)，系统地了解Git可以看看开源的Git电子书[《Pro Git》](https://git-scm.com/book/en/v2)，这本电子书很不错，提供了简体中文版，你可以免费下载pdf、epub、mobi版本，或者直接在网页上阅读。本文算是我读《Pro Git》所做的一点笔记，把我觉得有用的Git命令记录一下，所以此文的作用仅作为速查，更全面的内容建议看专业书籍，很多人不愿意去学习的原因是觉得自己已掌握的技能“已经够用了”，井底之蛙的即视感。我的建议是对于“工具”的使用，一定要读读官方的文档，而不是简单的百度一篇“xx工具使用方法”读一下就觉得够用了，先不说你百度到的内容的正确性如何（自己理解不充分，就写出来误导他人的博客真是太多了，这可能和我们的学习方式有关），即使内容正确无误，别人嚼过的东西，还是原来的味道吗？这样的人，即使倚天剑在手，也只会用来劈柴。（貌似今天的扯淡内容有点多，请见谅:-p）

## 安装

### 在 Linux 上安装

* 如果基础软件包管理工具是 Fedora，使用 yum 安装：

```
$ sudo yum install git
```

* 如果是基于 Debian 的发行版，使用 apt-get:

```
$ sudo apt-get install git
```

* 访问[Git 官网](http://git-scm.com/download/linux)

### 在 Mac 上安装

* 安装 Xcode Command Line Tools。
* 或者直接使用git命令，会提示安装 Xcode Command Line Tools。（要不要这么贴心）
* 或者从[官网](http://git-scm.com/download/mac)下载安装程序。
* 或者从[GitHub](http://mac.github.com)安装 GitHub for Mac。

### 在 Windows 上安装

* 或者从[官网](http://git-scm.com/download/win)下载安装程序。
* 或者从[GitHub](http://windows.github.com)下载 GitHub for Windows。

## 配置

配置分三个级别：

1. 系统级，配置文件位于：`/etc/gitconfig`，使用`git config --system`配置此文件。
2. 用户级，配置文件位于：`~/.gitconfig`或`~/.config/git/config`，使用`git config --global`配置此文件。
3. 仓库级，配置文件位于：`repo path/.git/config`，在`repo path`下直接使用不带选项的`git config`配置此文件。

在 Windows 系统中，Git 会查找 $HOME 目录下（一般情况下是 C:\Users\$USER）的 .gitconfig 文件。 Git 同样也会寻找 /etc/gitconfig 文件，但只限于 MSys 的根目录下，即安装 Git 时所选的目标位置。

示例：

```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global core.editor emacs
```

### 查看配置信息

1. 可以使用`git config --list`命令列出所有 Git 当时能找到的配置。
2. 可以使用`git config <key>`命令查看 Git 某一项的配置。

## 帮助信息

有三种查看 Git 命令使用手册的方法：

```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

## 基础操作

如果你略微使用过 Git，你应该已经使用过几个最基础的命令，如果你没有使用过这几个命令，建议先去熟悉一下这几个命令。

```
$ git init
$ git clone
$ git add
$ git commit
$ git pull
$ git push
```

1. 状态简览。

```
$ git status -s

 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

这里的`-s`(也可以使用`--short`)可以使`git status`命令进行一个比较简洁输出，比直接用`git status`爽多了，你一定会喜欢。

新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。 你可能注意到了 M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区。

## 忽略文件

可以在`.gitignore`文件中指定不希望被 Git 纳入版本控制的文件，要养成一开始就设置好`.gitignore`文件的好习惯。推荐一个可以自动生成该文件的[网址](https://www.gitignore.io)，这个网址配合 Linux 的`curl`命令使用效果更佳哦！

文件`.gitignore`的格式规范如下：

* 所有空行或者以`#`开头的行都会被 Git 忽略。
* 可以使用标准的 glob 模式匹配。
* 匹配模式可以以`/`开头防止递归。
* 匹配模式可以以`/`结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号`!`取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（\*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（\*) 表示匹配任意中间目录，比如a/\*\*/z 可以匹配 a/z, a/b/z 或 a/b/c/z等。



## 分支

### 创建分支

```
$ git checkout -b newBranchName
```

用`git branch newBranchName`也可以创建一个名称为 newBranchName 的新分支，但这个命令创建新分支后工作目录还是在当前分支，并没有切换到新分支上（仅创建而已），而`git checkout -b newBranchName`创建新分支后会把当前的分支也设置成新分支，所以我觉得这个命令会更常用。

### 本地分支查看：

```
$ git branch -v
---
  iss53   93b412c fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```

虽然后面不加`-v`也可以，但加个`-v`就可以顺便列出每个分支最后一次提交的内容。

有两个有用的选项`--merged`和`--no-merged`可以过滤这个列表中已经合并或尚未合并到当前分支的分支。

### 删除本地分支

```
$ git branch -d branchName
```

在`git branch`后面加`-d`再加分支名称即可删除本地的分支，Git在这里做了件特别安全的事情，如果你要删除的分支里有尚未合并到其他分支的内容，这条删除命令会执行失败，避免了工作内容的丢失；要强制删除这样的分支，把`-d`选项改为`-D`即可。

### 删除远程分支

```
$ git push origin --delete branchName
```

基本上这个命令做的只是从服务器上移除这个指针。Git服务器通常会保留数据一段时间直到垃圾回收运行，所以如果不小心删除掉了，通常是很容易恢复的。


## 容易产生的误解

* 使用git时要先更新（pull）、再提交（push）。这是没有必要的，这也是git的一个优越性所在，你可以尽管放心的直接push，如果别人往仓库中同样提交过代码，你的push操作将会失败，真他娘的贴心。



