---
layout: post
title: "探讨“iOS 多任务下载”"
date: 2016-09-14 11:48:03 +0800
categories:
---

这是一个老掉牙的标题，从 iOS 入行到现在，你或许看过很多介绍下载功能的博客，光简书上就已经有一大片了。但我还是要再写一篇！我也看过很多这个主题的文章，也读过很多人封装的下载代码，但没看到过比较让人满意的。如果你也有这种感觉，不妨接着往下看，我表述一些我的观点，希望与诸君共同探讨这个话题，共同探索一个最优解。

## 用哪种技术方案？

1. NSURLConnection: 如果要支持 iOS 7 以下的版本，NSURLConnection 肯定会用到，但现在还要支持 iOS 6 或以下系统的应该很少了。由于 NSURLSession 比 NSURLConnection 有天生的优势，此方案后面不再考虑。
2. NSURLSession: 从 iOS 7 开始被支持，功能比 NSURLConnection 更强大。
3. AFNetworking: iOS 界久负盛名的网络框架，内部实现也是基于 NSURLSession 或 NSURLConnection。

## 普通网络请求用 AFNetworking，下载也可以用吗？

就我之前查到的各种下载封装来看，直接用 NSURLSession 实现的比较多，而用 AFNetworking 实现的很少。

先想一下为什么有那么多项目中网络请求要用 AFNetworking ？答案很简单：简单呗，不用自己管 NSURLSession（或NSURLConnection）那一大堆代理方法了，直接处理 AFNetworking 提供的success & failure block回调就行了。

既然 AFNetworking 有这种好处，那我们再想，下载也可以直接用 AFNetworking 吗？答案是可以！回顾一下 AFNetworking 中的两个核心类：AFURLSessionManager 封装了对 NSURLSession 的处理，尤其是将 NSURLSession 的那些代理方法都通过封装转换成了 AFNetworking 中简单的 block 了；AFHTTPSessionManager 将常用的 HTTP 请求方法（GET、POST等）进行了封装。

一起看下 AFURLSessionManager 类的两个方法：

```
- (NSURLSessionDownloadTask *)downloadTaskWithRequest:(NSURLRequest *)request
                                             progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                                          destination:(nullable NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                    completionHandler:(nullable void (^)(NSURLResponse *response, NSURL * _Nullable filePath, NSError * _Nullable error))completionHandler;

- (NSURLSessionDownloadTask *)downloadTaskWithResumeData:(NSData *)resumeData
                                                progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                                             destination:(nullable NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                       completionHandler:(nullable void (^)(NSURLResponse *response, NSURL * _Nullable filePath, NSError * _Nullable error))completionHandler;
```

毫无疑问，通过 AFNetworking 实现下载功能会比直接使用 NSURLSession 简单很多，在使用 AFNetworking 的项目中，使用 AFNetworking 实现下载功能是最佳的方案。

## 需要支持的功能

上面讨论了实现的方案，这里再说说下载的封装需要具备哪些“通用”的下载功能，适应各种各样的下载需求呢？

* 既然叫“多任务”下载，同时下载多个是必然需要的。
* 再进一步，需要可以控制同时下载的数目。
* 状态更新，比如在 UITableViewCell 上更新下载进度、下载状态等。
* 再进一步，不能只在下载的界面观察下载状态，应该是所有关心下载状态的地方都可以实时更新状态的改变。
* 后台下载，现在下载的标配。
* 其他细碎的需求，比如下载速度、是否允许蜂窝网络、是否可以定制下载请求的请求头等。
* 在具备这些功能的基础上，最重要的是需要使用简单，最好像 AFNetworking 那么简单，降低学习成本，这样用的人才会爽。

## 我的实现

如果你需要一个如上所说的下载封装来在自己的项目中实现多任务下载这样的功能，可以尝试一下我写的 [SODownloader](https://github.com/scfhao/SODownloader)，可以通过 CocoaPods 很方便的集成到自己项目中，还有其[用法](https://github.com/scfhao/SODownloader/wiki/SODownloader-使用介绍)。

虽然 SODownloader 已经在我自己的项目中“试用”了一年多了，但由于我个人能力的局限性，这依然不可能是一个最佳的方案。如果你也认同我的思路，认为可以创造出一个最佳方案，欢迎和我一起完善 SODownloader！

另外，好想感受被 Star 狂砸的感觉的说～

## 一起探讨

我写这篇博客表达我自己对下载功能的理解，抛砖引玉。如果阁下有不一样的见解，欢迎提出，我们一起探讨。
