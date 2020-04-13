---
layout: post
title: "IDEDebugSessionErrorDomain: could not attach to pid"
date: 2020-04-13 14:17:58 +0800
categories:
---

这几天用 iOS 模拟器调试的时候，总遇到一个问题，程序更新到模拟器以后程序自动就退出了，Xcode会报一个错，内容如下：

```
Details

Could not attach to pid : “15612”
Domain: IDEDebugSessionErrorDomain
Code: 3
Failure Reason: Error 1
--
Error 1
Domain: IDEDebugSessionErrorDomain
Code: 3
--
```

先说一下我当前的软件环境：macOS Catalina 10.15.2、Xcode 11.3.1，这类问题很容器过时。网上能查到的解决方案有以下几种：

* 重启Xcode、模拟器、电脑。
* Edit Schema->Run->Uncheck "Debug executable"。
* `sudo DevToolsSecurity -enable`

第一个方案和第三个方案效果都不明显，第二个方案虽然管用但那样操作就不能调试了。偶然之间发现个可以解决我电脑上这个问题的方法，我这边网络环境比较复杂，模拟器要连的服务器在公司内网，而本公司内网默认是访问不了外网的，但这个问题貌似和网络有关：当电脑可以访问外网时，没这个问题，当电脑不能访问外网时，这个问题出现的几率很大。所以我这边只有同时让 Mac 连接公网后这个问题就不再出现了。
