---
layout: post
title: "打印AFNetworking请求及响应内容"
date: 2016-07-28 15:04:02 +0800
categories:
---

在 iOS 开发中，对服务器提供的API进行调试是很常见的场景，每个人都不会陌生。有的人可能是通过一些浏览器插件（比如post man），有的人可能是拼装好URL在浏览器里访问，还有的人可能是使用`curl`这个很实用的命令。

如果程序中使用 AFNetworking 进行请求，我觉得更好的方法是注册 AFNetworkingTaskDidCompleteNotification 消息监听，因为 AFNetworking 在每个请求结束后都会发送这个消息，并包含请求任务、发生的错误、响应内容等信息。基于这些信息，我封装了一个 AFNetworkingLogger，只需要一句代码开启打印功能，然后所有通过 AFNetworking 的请求及响应内容就会在标准输出中打印，其使用过程如下：

```
// 1. 只打印请求失败的内容：
[AFNetworkingLogger setupLoggerUseOptions:AFNetworkingLogFailure];
// 2. 成功和失败请求都要打印：
[AFNetworkingLogger setupLoggerUseOptions:AFNetworkingDefaultLogOption];
// 3. 关闭日志功能：
[AFNetworkingLogger setupLoggerUseOptions:AFNetworkingLogClose];
```

可以在 [SOKit](https://coding.net/u/scfhao/p/SOKit/git) 中找到 AFNetworkingLogger。
