---
layout: post
title: "《Pro Git》速查笔记"
date: 2017-02-03 21:14:33 +0800
categories:
---

## 前言

Git是程序员的好伙伴，不懂 Git 一定会被同行鄙视的。更多关于Git的内容可以在[Git官网](https://git-scm.com)看看，快速了解Git可以试一下[Code School的十五分钟教程“Try Git“](https://try.github.io/levels/1/challenges/1)，系统地了解Git可以看看开源的 Git 电子书[《Pro Git》](https://git-scm.com/book/en/v2)，这本电子书很不错，提供了简体中文版，你可以免费下载pdf、epub、mobi版本，或者直接在网页上阅读。

这里是我阅读《Pro Git》前三章时做的笔记。惭愧的是，使用 Git 进行版本管理的时间已经满两年了，才开始系统的看这本书。嗯，我刚开始使用 Git 时，也是以此书入门的，只可惜只学会了最简单的几个命令就丢开了。所以，在这里我总结了一些我觉得我可能并没有真正理解或者我可能忘记的东西，所以这里的内容可能“不全”，所以请不要把本文当作教程，去读《Pro Git》吧，相信你会有更大的收获！

## 第一章

### 安装

#### 在 Linux 上安装

* 如果基础软件包管理工具是 Fedora，使用 yum 安装：

```
$ sudo yum install git
```

* 如果是基于 Debian 的发行版，使用 apt-get:

```
$ sudo apt-get install git
```

* 访问[Git 官网](http://git-scm.com/download/linux)

#### 在 Mac 上安装

* 安装 Xcode Command Line Tools。
* 或者直接使用git命令，会提示安装 Xcode Command Line Tools。（要不要这么贴心）
* 或者从[官网](http://git-scm.com/download/mac)下载安装程序。
* 或者从[GitHub](http://mac.github.com)安装 GitHub for Mac。

#### 在 Windows 上安装

* 或者从[官网](http://git-scm.com/download/win)下载安装程序。
* 或者从[GitHub](http://windows.github.com)下载 GitHub for Windows。

### 配置

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

#### 查看配置信息

1. 可以使用`git config --list`命令列出所有 Git 当时能找到的配置。
2. 可以使用`git config <key>`命令查看 Git 某一项的配置。

#### 配置账号

如果不想每一次推送时都输入用户名与密码，你可以设置一个“credential cache”。最简单的方式就是将其保存在内存中几分钟，更多关于不同验证缓存的可用选项，查看凭证存储章节：

```
$ git config --global credential.helper cache
```

### 帮助信息

有三种查看 Git 命令使用手册的方法：

```
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

## 第二章

### 基础操作

```bash
$ git init
# git clone -o remote-name
$ git clone
$ git add
$ git commit
$ git pull
# git push origin localbranchname:remotebranchname
$ git push
```

### 状态简览。

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

### 忽略文件

可以在`.gitignore`文件中指定不希望被 Git 纳入版本控制的文件，要养成一开始就设置好`.gitignore`文件的好习惯。推荐一个可以自动生成该文件的[网址](https://www.gitignore.io)，这个网址配合 Linux 的`curl`命令使用效果更佳哦！

文件`.gitignore`的格式规范如下：

* 所有空行或者以`#`开头的行都会被 Git 忽略。
* 可以使用标准的 glob 模式匹配。
* 匹配模式可以以`/`开头防止递归。
* 匹配模式可以以`/`结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号`!`取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（*) 表示匹配任意中间目录，比如a/**/z 可以匹配 a/z, a/b/z 或 a/b/c/z等。

看一个 .gitignore 文件的例子：

```
# 不跟踪 .a 文件
*.a

# 跟踪 lib.a，虽然之前已经设置忽略 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，但不忽略子目录下的 TODO 文件
/TODO

# 忽略 build/ 目录下的所有文件
build/

# 忽略 doc/notes.txt，但不会忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录下所有的 .pdf 文件
doc/**/*.pdf
```

> github 有一个十分详细的针对数十种项目及语言的 .gitignore 文件列表，你可以在 [链接](https://github.com/github/gitignore) 找到它。

### 查看修改

不加参数的`git diff`命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

若要查看已暂存的将要添加到下次提交里的内容，可以使用`git diff --cached`命令。(Git 1.6.1 及更高版本还允许使用`git diff --staged`，效果是相同的)

可以使用`git difftool`命令来用 Araxis、emerge 或 vimdiff 等软件输出 diff 分析结果。使用`git difftool --tool-help`命令来看你的系统支持哪些 Git Diff 插件。

### 移除文件

使用`git rm`命令从 Git 中移除某个文件，如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项`-f`。

如果你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。为达到这一目的，使用`--cached`选项。

```
$ git rm log/\*.log

# 注意到星号 * 之前的反斜杠 \，因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。此命令删除 log/ 目录下扩展名为 .log 的所有文件。

$ git rm \*~

# 该命令杀出以`~`结尾的所有文件。
```

### 移动文件

不像其他的 VCS 系统，Git 并不显式跟踪文件移动操作。如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。要在 Git 中对文件改名，可以这么做：

```
$ git mv file_from file_to
```

它会恰如预期般正常工作。运行`git mv`就相当于运行了下面三条命了：

```
$ mv README.md README
$ git rm README.md
$ git add README
```

如此分开操作，Git 也会意识到这是一次改名。

### 查看提交历史

默认不用任何参数的话，`git log`会按提交时间列出所有的更新，最近的更新排在最上面。这个命令列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

一个常用的选项是 `-P`，用来显示每次提交的内容差异。你也可以加上`-2`来仅显示最近两次提交。该选项除了显示基本信息之外，还在附带了每次 commit 的变化。用于代码审查非常有用。

如果你想看到每次提交的简略的统计信息，你可以使用`--stat`选项。这个选项在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是增加了。

另外一个常用的选项是`--pretty`。这个选项可以指定使用不同于默认的格式的方式展示提交历史。这个选项有一些内建的子选项供你使用。比如使用`oneline`将每个提交放在一行显示，查看的提交数据很大时非常有用。另外还有`short`、`full`和`fuller`可以用，展示的信息多少有些不同，如：

```
$ git log --pretty=oneline
```

但最有意思的是`format`，可以定制要显示的记录格式。

```
$ git log --pretty=format:"%h - %an, %ar : %s"
```

表 2-1.`git log --pretty=format`常用格式

选项 | 说明
----|-----
%H | 提交对象（commit）的完整哈希字串
%h | 提交对象的简短哈希字串
%T | 数对象（tree）的完整哈希字串
%t | 树对象的简短哈希字串
%P | 父对象（parent）的完整哈希字串
%p | 父对象的简短哈希字串
%an | 作者（author）的名字
%ae | 作者的电子邮件地址
%ad | 作者修订日期（可以用`--date=`选项定制格式
%ar | 作者修订日期，按多久以前的方式显示
%cn | 提交者（committer）的名字
%ce | 提交者的电子邮件地址
%cd | 提交日期
%cr | 提交日期，按多久以前的方式显示
%s | 提交说明

当`oneline`或`format`与另一个 log 选项`--graph`结合使用时尤其有用。这个选项添加了一些 ASCII 字符串来形象地展示你的分支、合并历史：

```
$ git log --pretty=format:"%h %s" --graph
```

表 2-2.`git log`的常用选项

选项 | 说明
----|-----
-p | 按补丁格式显示每个更新之间的差异
--stat | 显示每次更新的文件修改统计信息
--shortstat | 只显示 --stat 中最后的行数修改添加移除统计
--name-only | 仅在提交信息后显示已修改的文件清单
--name-status | 显示新增、修改、删除的文件清单
--abbrev-commit | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符
--relative-date | 使用较短的相对时间显示（比如：“2 weeks ago”）
--graph | 显示 ASCII 图形表示的分支合并历史
--pretty | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller和format（后跟指定格式）。

#### 限制输出长度

git log 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。

`-<n>`选项，其中的 n 可以是任何整数，表示仅显示最近的若干条提交。不过实践中我们是不太用这个选项的，Git 在输出所有提交时会自动调用分页程序，所以你一次只会看到一页的内容。

另外还有按照时间作限制的选项，比如`--since`和`--until`也很有用。例如，下面的命令列出所有最近两周内的提交：

```
$ git log --since=2.weeks
```

这个命令可以在多种格式下工作，比如说具体的某一天"2008-01-15"，或者是相对地多久以前"2 years 1 day 3 minutes ago"。

还可以给出若干搜索条件，列出符合的提交。用`--author`选项显示指定作者的提交，用`--grep`选项搜索提交说明中的关键字。**如果要得到同时满足这两个选项搜索条件的提交，就必须用`--all-match`选项。否则，满足任意一个条件的提交都会被匹配出来**

另一个非常有用的筛选选项是`-S`，可以列出那些添加或移除了某些字符串的提交。比如说，你想找出添加或移除了某一个特定函数的引用的提交，你可以这样使用：

```
$ git log -Sfunction_name
```

最后一个很实用的 git log 选项是路径（path），如果只关心某些文件或者目录的历史提交，可以在 git log 选项的最后指定它们的路径。因为是放在最后位置上的选项，所以用两个短划线`--`隔开之前的选项和后面限定的路径名。

表 2-3. 限制 git log 输出的选项

选项 | 说明
----|-----
-(n) | 仅显示最近的 n 条提交
--since, --after | 仅显示指定时间之后的提交
--until, --before | 仅显示指定时间之前的提交
--author | 仅显示指定作者相关的提交
--committer | 仅显示指定提交者相关的提交
--grep | 仅显示含指定关键字的提交
-S | 仅显示添加或移除了某个关键字的提交

运行`git log --oneline --decorate --graph --all`，它会输出你的提交历史、各个分支的指向以及项目的分支分叉情况。

### 撤销操作

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。此时，可以运行带有`--amend`选项的提交命令尝试重新提交：

```
$ git commit --amend
```

这个命令会将暂存区中的文件提交。如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你修改的只是提交信息。这个命令相当于撤销最后一次提交操作。

#### 撤销暂存

```
$ git reset HEAD <file>...
```

> 虽然在调用时加上`--hard`选项可以令`git reset`成为一个危险的命令（可能导致工作目录中所有当前进度丢失），但本例中工作目录内的文件并不会被修改。不加选项地调用`git reset`并不危险--它只会修改暂存区域。

#### 撤销未暂存的修改

```
$ git checkout -- <file>...
```

> 你需要知道`git checkout -- [file]`是一个危险的命令。你对那个文件做的任何修改都会消失-你只是拷贝了另一个文件来覆盖它。除非你确实清楚不想要那个文件来，否则不要使用这个命令。

### 远程仓库

```
$ git remote
origin
$ git remote add pb
https://github.com/paulboone/ticgit
$ git remote -v
origin https://github.com/schacon/ticgit(fetch)
origin https://github.com/schacon/ticgit(push)
pb     https://github.com/paulboone/ticgit(fetch)
pb     https://github.com/paulboone/ticgit(push)
```

现在可以在本地通过`pb/master`访问pb服务器上的master分支，这里的这个‘pb/master’就叫远程跟踪分支。

#### 从远程仓库中抓取与拉取

```
$ git fetch [remote-name]
$ git fetch --all
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。执行完成后，你将拥有那个远程仓库中所有分支的引用，可以随时合并或查看。执行 fetch 命令后，会得到远程仓库分支的远程跟踪分支（如origin/branchname），这个分支指针是只读的，你可以将其合并到本地分支，也可以用如下命令在本地建立对应的本地分支（fetch不会自动建立）。

```
$ git checkout -b serverfix origin/serverfix
# 本地的分支名称 serverfix 可以随便叫的哦。
```

如果你的协作者推送了自己的分支到服务器上，你执行 fetch 命令后本地没有那个分支，你可以通过上面这个命令在本地建立对应的分支，Git 还提供了另一个命令做相同的事：

```
$ git checkout --track origin/serverfix
```

> `git fetch`命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。当准备好时你必须手动将其合并入你的工作。

运行`git pull`通常会从最初克隆的服务器上抓取数据并自动尝试合并到**当前**所在的分支。

#### 查看远程仓库

如果想要查看某一个远程仓库的更多信息，可以使用`git remote show [remote-name]`命令。这个命令列出来当你在特定的分支上执行`git push`会自动地推送到哪一个远程分支。它同样地列出来哪些分支不在你的本地，哪些远程分支已经从服务器上移除了，还有当你执行`git pull`时哪些分支会自动合并。

#### 远程仓库的移除与重命名

```
$ git remote rename pb paul
$ git remote rm paul
```

### 标签（Tag）

Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

一个轻量标签很像一个不会改变的分支 - 它只是一个特定提交的引用。

然而，附注标签是存储在 Git 数据库中的一个完整对象。 它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。 通常建议创建附注标签，这样你可以拥有以上所有信息；但是如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的。

#### 列出标签

```
$ git tag
```

也可以使用特定的模式查找标签(注意后面的通配符)：

```
$ git tag -l 'v1.8.5*'
```

#### 创建标签

1. 创建附注标签

```
$ git tag -a v1.4 -m 'my version 1.4'
```

2. 创建轻量标签

```
$ git tag v1.4-lw
```

3. 事后打标签

```
$ git tag -a v1.2 9fceb02
```

要在那个提交上打标签，你需要在命令的末尾指定提交的校验和（或部分校验和）

#### 输出标签信息

```
$ git show v1.4
```

#### 共享标签

默认情况下，git push 命令并不会传送标签到远程仓库服务器上。

```
# 将tagname标签推送到共享服务器上
$ git push origin [tagname]
# 将不再远程仓库服务器上的标签全部传送到那里
$ git push origin --tags
```

#### 检出标签

```
$ git checkout -b [branchname] [tagname]
```

### Git 别名

可以通过 git config 文件来轻松地为每一个命令设置一个别名：

```
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.unstage 'reset HEAD --'
$ git config --global alias.last 'log -1 HEAD'
```

这意味着，当要输入`git commit`时，只需要输入`git ci`。

你可能想要执行外部命令，而不是一个 Git 子命令。这种情况，可以在命令前面加入`!`符号。

```
$ git config --global alias.visual '!gitk'
```

## 第三章 Git 分支

分支原理在《Pro Git》中讲的相当清楚，如有所疑惑，请前往查看。

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

你可以简单地使用`git log`命令查看各个分支当前所指的对象。提供这一功能的参数是`--decorate`。

虽然后面不加`-v`也可以，但加个`-v`就可以顺便列出每个分支最后一次提交的内容。

有两个有用的选项`--merged`和`--no-merged`可以过滤这个列表中已经合并或尚未合并到当前分支的分支。

### 删除本地分支

```
$ git branch -d branchName
```

在`git branch`后面加`-d`再加分支名称即可删除本地的分支，Git在这里做了件特别安全的事情，如果你要删除的分支里有尚未合并到其他分支的内容，这条删除命令会执行失败，避免了工作内容的丢失；要强制删除这样的分支，把`-d`选项改为`-D`即可。

### 查看远程分支

你可以通过`git ls-remote (remote)`来显式地获得远程引用的完整列表，或者通过`git remote show (remote)`获得远程分支的更多信息。

### 删除远程分支

```
$ git push origin --delete branchName
```

基本上这个命令做的只是从服务器上移除这个指针。Git服务器通常会保留数据一段时间直到垃圾回收运行，所以如果不小心删除掉了，通常是很容易恢复的。

### 分支合并冲突

基本的冲突解决方式不在赘述。使用`git merge`合并分支，解决冲突后，使用`git add`将冲突标记为解决。

如果你想使用图形化工具来解决冲突，你可以运行`git mergetool`，该命令会为你启动一个合适的可视化合并工具，并带领你一步一步解决这些冲突。（如果你需要更加高级的工具来解决复杂的合并冲突，我们会在“高级合并”介绍更多关于分支合并的内容。）等你退出合并工具之后，Git会询问刚才的合并是否成功。如果你回答是，Git会暂存那些文件以表面冲突已解决。

### 变基

使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像重新播放一样，在 Git 中，这种操作就叫做变基。使用变基方式合并的例子：

```
$ git checkout experiment
# 将 experiment 分支的修改应用到 master 分支上
$ git rebase master
$ git checkout master
$ git merge experiment
```

使用变基和merge这两种整合方式的最终结果没有任何区别，但是变基使得提交历史更加整洁。

一个涉及到三个分支的变基示例：

```
$ git rebase --onto master server client
```

以上命令的意思是：“取出 client 分支，找出处于 client 分支和 server 分支的共同祖先之后的修改，然后把它们在 master 分支上重演一遍”

更实用的命令（不需要关系当前在哪个分支）：

```
# 将 server 分支的修改应用到 master 分支上
$ git rebase master server
```

**不要对在你的仓库外有副本的分支执行变基**

变基的缺点可参考《Pro Git》，当你先推送一个分支到服务器，后对此分支进行变基操作，这致使此分支在你的版本库中消失，而其他人如果基于此分支进行了工作，那将带来麻烦。如果发生了这样的事，可以使其他协作者使用`git pull --rebase`进行补救。如果你习惯使用`git pull`，同时又希望默认使用选项`--rebase`，你可以执行这条语句`git config --global pull.rebase true`来更改`pull.rebase`的默认设置。

```
$ git pull --rebase
# 相当于
$ git fetch
$ git rebase teamone/master
```

### 设置跟踪

设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，你可以在任意时间使用`-u`或`--set-upstream-to`选项运行`git branch`来显式地设置。

```
$ git branch -u origin/serverfix
```

如果想要查看设置的所有跟踪分支，可以使用`git branch`的`-vv`选项。

设置好跟踪关系后，就可以直接使用`git pull`和`git push`而不用再加任何参数了。

当设置好跟踪分支后，可以通过 @{upstream} 或 @{u} 快捷方式来引用它。所以在 master 分支时并且它正在跟踪 origin/master 时，如果愿意的话可以使用`git merge @{u}`来取代`git merge origin/master`。

## 容易产生的误解

* 使用git时要先更新（pull）、再提交（push）。这是没有必要的，这也是git的一个优越性所在，你可以尽管放心的直接push，如果别人往仓库中同样提交过代码，你的push操作将会失败，真他娘的贴心。

* 在 Git 中任何已提交的东西几乎总是可以恢复的。甚至那些被删除的分支中的提交或使用`--amend`选项覆盖的提交也可以恢复。
