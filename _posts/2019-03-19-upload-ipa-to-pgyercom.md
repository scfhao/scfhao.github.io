---
layout: post
title: "往蒲公英上传 iPA 的两种方式"
date: 2019-03-19 07:38:23 +0800
categories:
---

想必大多数 iOS 开发者对于[蒲公英](http://www.pgyer.com/)这个应用分发网站都不会陌生。打开这个网站，点击上传按钮就可以把自己的 ipa 或者 apk 文件上传上去，供他人下载，这没啥好说的，我讲一下我的另外两种往这个网站上传 ipa 的方式：一种是使用 shell 脚本，另一种是使用自动操作（Automator App）。

## 脚本上传

不知道从什么时候起，我开始喜欢使用命令行，不常用命令行的人可能会觉得去记那么多命令多麻烦啊，但当你真正去使用命令行的时候就会发现其实很简单，尤其是如果能用好命令行的自动补全功能，会发现命令行比鼠标操作是多么的快捷。

回忆一下通过网页往蒲公英上传安装包的过程：

1. 打开浏览器
2. 输入网址 pgyer.com
3. 点击上传按钮
4. 在文件拾取器中选择要上传的 ipa 文件
5. 等待上传完成
6. 输入更新内容

看吧，至少需要 6 个步骤，而且这 6 个步骤中除了第 5 步是需要我们去等待外，其他步骤都是需要我们亲自操作的。

如果是用脚本上传就简单多了，只需要输入一个命令就能与上面全部的操作等效。

```Bash
$ ./pgyer-uploader.sh /path/to/app.ipa "更新内容"
```

> `pgyer-uploader.sh`是上传脚本的名称，`/path/to/app.ipa`是要上传的 ipa 的路径，"更新内容"是上传的这个 ipa 的更新内容。

一个命令，不需要任何等待，用节约的时间喝口茶吧。

这是脚本`pgyer-uploader.sh`的内容：

```Shell
#!/bin/sh
#=====================================
# 将 ipa 上传到蒲公英
# Author: scfhao
# Usage: ./uploadIPA ipa或apk文件路径 更新说明
#=====================================
# PGY_UKEY和PGY_API_KEY需要配置成自己的，每个蒲公英的注册用户都对应这两个参数，可以从这个网站查看自己的具体参数：https://www.pgyer.com/doc/api#commonParams
# 对应蒲公英 api 中的 uKey 参数
PGY_UKEY=""
# 对应蒲公英 api 中的 _api_key 参数
PGY_API_KEY=""


# 蒲公英上传地址
PGY_UPLOAD_URL="http://www.pgyer.com/apiv1/app/upload"

if [ ! -n "$PGY_UKEY" ] || [ ! -n "$PGY_API_KEY" ]; then
	echo "请将脚本中的PGY_UKEY、PGY_API_KEY替换成你自己的账号的uKey和_api_key"
	exit
fi

if [[ ! -n "$1" ]]; then
	echo "Usage: uploadIPA.sh /path/to/ipa 'update message'"
	exit
fi


if [ "${1##*.}" != "ipa" ] && [ "${1##*.}" != "apk" ]; then
	echo "${1##*.}"
	echo "只能上传ipa或apk文件哦"
	exit
fi

if [[ ! -f "$1" ]]; then
	echo "找不到要上传到ipa|apk文件"
	exit 
fi

if [[ ! -n "$2" ]]; then
	$2 = "无"
fi

curl -F "file=@$1" -F "uKey=$PGY_UKEY" -F "_api_key=$PGY_API_KEY" -F "updateDescription=$2" $PGY_UPLOAD_URL | jq '.'
```

说明：

这个脚本中用到了`jq`命令，用`curl`命令上传安装包到蒲公英后，蒲公英会返回一个 json 响应。`jq`命令可以将蒲公英返回的 json 字符串格式化成方便我们阅读的形式，如果你的 Mac 上没有 jq 命令，可以把脚本最后的` | jq '.'`删掉。

你可以用`Home brew`安装`jq`命令(如果你的 Mac 上没安装 Home brew，还需要安装一下 Home brew，当然这和上传没关系，只是为了让蒲公英返回的信息更方便我们阅读)：

```Bash
$ brew install jq
```

## 使用自动操作（Automator App）

Automator(现在叫自动操作)是 macOS 上自带的一个软件，昨天在看这个软件的使用帮助时看到一句：“应用程序：打开后或者将文件或文件夹拖放到上面后，便开始运行的单独工作流程。”看到这句话后，我就想是不是可以用 Automator 制作一个应用程序，用于往蒲公英上上传 ipa，于是就用 Automator 制作了一个应用程序，名称就叫“pgyer.com”，放到了桌面上，每次要往蒲公英上传 ipa 时，执行下面的操作：

1. 用鼠标拖动 ipa 文件到桌面上的 pgyer.com 应用程序。

嗯，只需要这一步，非常方便呢。

Automator 中用到的脚本就是上一节中的上传脚本稍加修改做的，现在还不太完善，所以先不往博客上贴，再增加输入更新内容和错误提示后再更新到本博客。
