# 常用的Git命令

## 前言

Git是程序员的好伙伴，不懂Git一定会被同行鄙视的。更多关于Git的内容可以在[Git官网](https://git-scm.com)看看，快速了解Git可以试一下[Code School的十五分钟教程“Try Git“](https://try.github.io/levels/1/challenges/1)，系统地了解Git可以看看开源的Git电子书[《Pro Git》](https://git-scm.com/book/en/v2)，这本电子书很不错，提供了简体中文版，你可以免费下载pdf、epub、mobi版本，或者直接在网页上阅读。本文算是我读《Pro Git》所做的一点笔记，把我觉得有用的Git命令记录一下，所以此文的作用仅作为速查，更全面的内容建议看专业书籍，很多人不愿意去学习的原因是觉得自己已掌握的技能“已经够用了”，井底之蛙的即视感。我的建议是对于“工具”的使用，一定要读读官方的文档，而不是简单的百度一篇“xx工具使用方法”读一下就觉得够用了，先不说你百度到的内容的正确性如何（自己理解不充分，就写出来误导他人的博客真是太多了，这可能和我们的学习方式有关），即使内容正确无误，别人嚼过的东西，还是原来的味道吗？这样的人，即使倚天剑在手，也只会用来劈柴。（貌似今天的扯淡内容有点多，请见谅:-p）

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

