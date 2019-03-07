---
layout: post
title: "删除 PersonTeam 中的 App ID"
date: 2019-03-07 10:13:46 +0800
categories:
---

用 Xcode 创建 iOS 项目时，默认情况下 Xcode 会为我们自动管理签名，这样 Xcode 会自动在你的账号里为新创建的项目创建 App ID。

App ID 也是一种稀缺资源，在一个账号中创建了一个 App ID，那在别的账号里就不能创建一个同样的 App ID 了。如果非要在另一个账号中创建一个相同的 App ID，需要先把前一个账号中对应的 App ID 删掉。

如果项目设置里的 Develop Team 是开发者账号，这种情况可以在开发者网站中自助删除 App ID。如果是免费的 PersonTeam 呢？开发者网站目前并不支持 PersonTeam 创建与删除 App ID。

但这种需求是存在的，有人说需要打客服电话联系 Apple 进行删除。今天介绍一种自助的方式：使用[Impactor](http://www.cydiaimpactor.com)，Impactor 是越狱社区的一个工具，可以帮你把没有签名的 ipa 安装到手机上。

用这个工具删除 PersonTeam 里的 App ID 是很方便的，打开 Impactor 后，在 Impactor 的菜单中选择“Xcode->Delete App ID”，按照提示输入你的 Apple ID 的账号和密码（如果要登录的 Apple ID 开启了两步验证，这里需要输专用密码，专用密码可以在[Apple ID管理](https://appleid.apple.com/account/manage)网站生成），输入密码后，Impactor 会列出你的账号里的 App ID，选中要删除的那个，点击删除即可。
