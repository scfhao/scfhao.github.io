---
layout: post
title: "基于AFNetworking实现的下载管理工具SODownloader"
date: 2016-09-14 11:48:03 +0800
categories:
---

在客户端应用中，多文件下载及下载监控的功能是很常见的，比如音乐类App中的下载音乐列表的功能，视频类App中下载视频列表等，可以说只要涉及到下载文件的功能都会有类似的需求。

对于下载功能的实现方式，你可以尝试用NSURLConnection、用NSURLSession+NSURLSessionDownloadTask或者用我没用过的CFNetwork实现，但如果你看过一些AFNetworking的源码后，你一定会觉得基于AFNetworking来实现是最佳的方式，因为AFURLSessionManager已经帮你把NSURLSession部分所有的功能都帮你封装好了，甚至连下载进度回调都是现成的。

于是，我就基于AFNetworking 3.x实现了一个简单的下载管理工具，支持如下的特性：

* 方便的监控每个下载文件的下载进度和下载状态
* 可以方便的为每个下载文件指定下载位置
* 指定可同时下载的文件个数
* 具备常用下载管理方法（下载、暂停、恢复、全部暂停、取消等）
* 方便的创建多个下载管理类实例
* 支持后台下载
* 可指定接收文件的MIME类型，不符合则认为失败
* 使用简单（原谅我自卖自夸一下）

## SODownloader 介绍

如果你看完了上面说的这些，还没关闭这篇博客的话，下面我就深入的介绍一下我写的[SODownloader](https://github.com/scfhao/SODownloader)（这就是上面说的下载工具类，是不是很逊？）。

SODownloader 的核心为 SODownloader 和 SODownloadItem，其中SODownloader 为下载管理类，SODownloadItem 则代表下载项。

### 创建 SODownloader 对象

创建/获取 SODownloader 对象特别简单，代码如下：

```
+ (instancetype)musicDownloader {
    return [SODownloader downloaderWithIdentifier:@"music" completeBlock:^(id<SODownloadItem>  _Nonnull item, NSURL * _Nonnull location) {
        SODebugLog(@"%@ 下载成功！", item);
        // 这个block每下载成功一个文件时被调用，这个block在后台线程中调用，不建议在这里做更新UI的操作
        // 你可以在这里对下载成功做特别的处理，例如：
        // 1. 把下载完成的 item 的信息存入数据库
        // 2. 把下载完成的文件从 location 位置移动到你想要保存到的文件夹
        // 3. 其他处理，如解析下载文件等
    }];
}

```

SODownloader 建议为不同类型的下载列表创建不同的 SODownloader 对象，例如要下载音乐，则 identifier 参数指定为“music”，在一次应用生命周期内，对相同的 identifier 会返回同一个 SODownloader 对象。如果要下载视频，identifier 传“video”即可返回另一个专门用于视频下载的 SODownloader 对象。至于第二个block参数，上面代码注释里应该算是很清楚了。

### SODownloadItem 可下载项

可下载项是指你的应用中要下载的数据类型，例如音乐App中可能用 SOMusic 来代表一首歌曲，这里歌曲就是可下载项。让自己应用中的 model 成为 SODownloader 的可下载项有两种途径：

1. 继承 SODownloadItem 类，如果你的 model 的直接父类为NSObject，建议选择继承 SODownloadItem 类。
2. 实现 SODownloadItem 协议，如果你的 model 的需要继承其他父类，就可以选择实现 SODownloadItem 协议，比直接继承 SODownloadItem 稍微麻烦的一点就是需要在你的 model 类的实现中为 `downloadProgress`和`downloadState`属性合成访问器，可以直接写`@synthesize downloadProgress, downloadState;`搞定，如果有特殊需求也可以自己为这两个属性实现 setter 和 getter。
3. 要成为可下载项，最重要的一点就是在这个 model 类中实现`- (NSURL *)downloadURL;`方法，在这个方法中返回 model 对应的文件的下载地址。

### 进行下载

到这里，就可以很方便的开始下载了，SODownloader 提供了功能齐全的下载控制方法：

```
/// 下载
- (void)downloadItem:(id<SODownloadItem>)item;
- (void)downloadItems:(NSArray<SODownloadItem>*)items;
/// 暂停
- (void)pauseItem:(id<SODownloadItem>)item;
- (void)pauseAll;
/// 继续
- (void)resumeItem:(id<SODownloadItem>)item;
- (void)resumeAll;
/// 取消／删除
- (void)cancelItem:(id<SODownloadItem>)item;
- (void)cancenAll;

/// 删除所有已下载
- (void)removeAllCompletedItems;

```

### 监控下载状态和下载进度

SODownloadItem 为下载模型增加了两个属性，downloadState（下载状态）、downloadProgress（下载进度）。这两个属性的值由 SODownloader 管理，不建议在 SODownloader 外部自己设置这两个属性的值，如果非要这么做的话，请使用 SODownloader 提供的这个方法来修改下载状态的值：

```
- (void)setDownloadState:(SODownloadState)state forItem:(id<SODownloadItem>)item;

```

### 通知

SODownloader 每下载成功一个文件时，会发送 SODownloaderCompleteItemNotification 通知，当你收到这个通知时，可以做刷新界面等操作。这个通知的 object 为 SODownloader 对象，这样就可以为指定的 SODownloader 对象注册这个通知，也可以在通知对象的 userInfo 字典中通过 SODownloaderCompleteDownloadItemKey 拿到当前下载成功的 SODownloadItem。

### MIME 类型检测

如果服务器支持 MIME-type 的话，可以设置 SODownloader 对象的 acceptableContentTypes 属性来设置接收的文件类型。例如，你创建了一个 SODownloader 对象 videoDownloader，用它来下载你的应用中用到的视频文件，假设视频格式为 mp4（对应的 MIME-type 为“video/mpeg4”） 或 avi（对应的 MIME-type 为“video/avi”），这时你就可以通过下面的代码来为 videoDownloader 指定接收数据的类型了，当服务器发生错误，返回一个非期望的数据（例如服务器上没有对应的视频文件，所以服务器返回了一个404 html页面，这时的 MIME-type 可能为“text/html”），SODownloader就可以自动认为这个视频文件下载失败了：

```
videoDownloader.acceptableContentTypes = [[NSSet alloc]initWithObjects:@"video/mpeg4", @"video/avi", nil];

```

## 推广

如果你觉得 SODownloader 的功能还不错，欢迎在自己的项目中使用，也欢迎把 SODownloader 推荐给更多的人。

## 建议&反馈&改进

因本人能力有限，所以 SODownloader 肯定会有很多不足之处。

* 如果觉得 SODownloader 还缺少什么下载通用的功能，欢迎评论提出。
* 如果发现 SODownloader 存在 Bug，欢迎在评论提出，或者在[Issue](https://github.com/scfhao/SODownloader/issues)中讨论，或者提交pull request。
* 如果有可以改进的代码，欢迎提交 pull request。
* 欢迎各种拍砖！
* 如果觉得还不错，欢迎 Star！

