---
layout: post
title: "基于AFNetworking实现的下载管理工具SODownloader"
date: 2016-09-14 11:48:03 +0800
categories:
---

## 谈谈下载功能

文件下载是一种很常见的功能，在 Foundation 层可以使用 NSURLConnection 或 NSURLSession+NSURLSessionDownloadTask 进行实现，这是两种最简单的实现下载功能的方式。如果你熟悉 AFNetworking，你一定会觉得基于用 AFNetworking 来实现下载才是最简单的姿势，因为AFURLSessionManager 类已经封装了完整的下载功能，如果你在一个界面中要下载一个文件，而且这个下载操作和别的界面没有任何关系，我建议你实现下载功能时直接用 AFNetworking，比起 NSURLConnection 和 NSURLSession 的一大堆代理方法，AFNetworking 简洁的不要不要的，比如下面这段代码使用 AFNetworking 进行下载，并实现了完整的进度回调，暂停、继续等操作也是特别方便。

```
NSString *url = @"http://test.daqing.net/300M.rar";
NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:url]];
__weak __typeof__(self) weakSelf = self;
// 指定目标下载地址
NSURL *(^destinationBlock)(NSURL *targetPath, NSURLResponse *response) = ^(NSURL *targetPath, NSURLResponse *response) {
    NSString *fileName = [targetPath lastPathComponent];
    NSString *destinationPath = [NSTemporaryDirectory() stringByAppendingPathComponent:fileName];
    return [NSURL fileURLWithPath:destinationPath];
};
// 指定进度回调
void (^progressBlock)(NSProgress *downloadProgress) = ^(NSProgress *downloadProgress) {
    dispatch_async(dispatch_get_main_queue(), ^{
        __strong __typeof__(weakSelf) strongSelf = weakSelf;
        strongSelf.progressView.progress = downloadProgress.fractionCompleted;
    });
};
// 创建下载任务
self.downloadTask = [self.downloadManager downloadTaskWithRequest:request progress:progressBlock destination:destinationBlock completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nullable filePath, NSError * _Nullable error) {
    if (error) {
        NSLog(@"下载失败");
    } else {
        NSLog(@"下载成功");
    }
}];
// 开始下载
[self.downloadTask resume];
```

简单吧？如果要实现更加复杂的下载，比如有一个下载列表然后同时可以下载N个文件的，比如要在多个界面中访问下载任务的，上面的代码就有点“捉襟见肘”了。于是，我就基于 AFNetworking 3 实现了一个简单的下载管理工具 SODownloader，支持如下的特性：

* 支持下载进度和下载状态管理
* 支持下载列表并可为每个文件分别指定保存位置
* 支持设置同时下载任务数量
* 具备常用下载管理方法（下载、暂停、恢复、全部暂停、取消等）
* 最简单的代码即可支持后台下载
* 支持判断下载文件的MIME类型，不符合则认定失败

## SODownloader 介绍

如果你看完了上面说的这些，还没关闭这篇博客的话，下面我就深入的介绍一下我写的[SODownloader](https://github.com/scfhao/SODownloader)

SODownloader 的核心为 SODownloader 和 SODownloadItem，其中SODownloader 为下载管理类，SODownloadItem 则代表下载项。

### 创建 SODownloader 对象

可以这样创建一个 SODownloader 对象：

```
+ (instancetype)musicDownloader {
    static SODownloader *downloader = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        downloader = [[SODownloader alloc]initWithIdentifier:@"music" completeBlock:^(id<SODownloadItem>  _Nonnull item, NSURL * _Nonnull location) {
            SODebugLog(@"%@ 下载成功！%@", item, location);
            // 这个block每下载成功一个文件时被调用，这个block在后台线程中调用，不建议在这里做更新UI的操作
            // 你可以在这里对下载成功做特别的处理，例如：
            // 1. 把下载完成的 item 的信息存入数据库
            // 2. 把下载完成的文件从 location 位置移动到你想要保存到的文件夹
            // 3. 其他处理，如解析下载文件等
        }];
    });
    return downloader;
}
```

建议为不同类型文件的下载创建不同的 SODownloader 对象，例如要下载音乐，创建一个 identifier 为“music”的下载器；如果还要下载视频，在创建一个 identifier 为“video”的另一个 SODownloader 对象。至于第二个block参数，上面代码注释里应该算是很清楚了。

### SODownloadItem 可下载项

可下载项是指你要下载的数据类型，例如音乐App中可能用 SOMusic 来代表一首歌曲，这里歌曲就是可下载项。让自己应用中的 model 成为 SODownloader 的可下载项有两种途径：

1. 继承 SODownloadItem 类，如果你的 model 的直接父类为NSObject，建议选择继承 SODownloadItem 类。
2. 实现 SODownloadItem 协议，如果你的 model 的需要继承其他父类，就可以选择实现 SODownloadItem 协议，比直接继承 SODownloadItem 稍微麻烦的一点就是需要在你的 model 类的实现中为 `so_downloadProgress`和`so_downloadState`属性合成访问器，可以直接写`@synthesize so_downloadState, so_downloadProgress;`搞定，如果有特殊需求也可以自己为这两个属性实现 setter 和 getter。
3. 要成为可下载项，最重要的一点就是在这个 model 类中实现`- (NSURL *)so_downloadURL;`方法，在这个方法中返回 model 对应的文件的下载地址。

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

SODownloadItem 为下载模型增加了几个属性，so_downloadState（下载状态）、so_downloadProgress（下载进度），你可以使用`KVO`观察这两个属性或重写这两个属性的 setter 方法来做相应的处理。这两个属性的值由 SODownloader 管理，不建议在 SODownloader 外部自己设置这两个属性的值，如果非要这么做的话，请使用 SODownloader 提供的这个方法来修改下载状态的值：

```
- (void)setDownloadState:(SODownloadState)state forItem:(id<SODownloadItem>)item;
```

### 通知

SODownloader 每下载成功一个文件时，会发送 SODownloaderCompleteItemNotification 通知，当你收到这个通知时，可以做刷新界面等操作。这个通知的 object 为 SODownloader 对象，这样就可以为指定的 SODownloader 对象注册这个通知，也可以在通知对象的 userInfo 字典中通过 SODownloaderCompleteDownloadItemKey 拿到当前下载成功的 SODownloadItem。

### 支持后台下载

在 AppDelegate 类中实现`application:handleEventsForBackgroundURLSession:completionHandler:`方法，这个方法的实现位置没有限制，可以你的 AppDelegation.m 中实现，也可以在 AppDelegate 类的 category 中实现，例如写在 AppDelegate+SODownloader.m 中(但不要同时在多个地方实现这个方法，否则只有一处会生效)：

```
- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler {
	// 根据 identifier 判断后台下载会话是否属于某个 SODownloader 对象，如果属于则交由该 SODownloader 对象处理。
    if ([identifier isEqualToString:@"music"]) {
        SODownloader *downloader = [SODownloader musicDownloader];
        [downloader setDidFinishEventsForBackgroundURLSessionBlock:^(NSURLSession * _Nonnull session) {
            completionHandler();
        }];
    }
}
```

### MIME 类型检测

如果服务器支持 MIME-type 的话，可以设置 SODownloader 对象的 acceptableContentTypes 属性来设置接收的文件类型。例如，你创建了一个 SODownloader 对象 videoDownloader，用它来下载你的应用中用到的视频文件，假设视频格式为 mp4（对应的 MIME-type 为“video/mpeg4”） 或 avi（对应的 MIME-type 为“video/avi”），这时你就可以通过下面的代码来为 videoDownloader 指定接收数据的类型了，当服务器发生错误，返回一个非期望的数据（例如服务器上没有对应的视频文件，所以服务器返回了一个404 html页面，这时的 MIME-type 可能为“text/html”），SODownloader就可以自动认为这个视频文件下载失败了：

```
videoDownloader.acceptableContentTypes = [[NSSet alloc]initWithObjects:@"video/mpeg4", @"video/avi", nil];
```

### 对 AFNetworking 2.x 的支持

如果项目中使用的 AFNetworking 的版本号为 2.x 且不能轻易升级到 AFNetworking 3，可以在 SODownloader 项目的`baseOnAF2`分支找到支持 AFNetworking 2.x 的 SODownloader 版本。

## 推广

如果你觉得 SODownloader 的功能还不错，欢迎在自己的项目中使用，也欢迎把 SODownloader 推荐给更多的人。

## 建议&反馈&改进

因本人能力有限，所以 SODownloader 肯定会有很多不足之处。

* 如果觉得 SODownloader 还缺少什么下载通用的功能，欢迎评论提出。
* 如果发现 SODownloader 存在 Bug，欢迎在评论提出，或者在[Issue](https://github.com/scfhao/SODownloader/issues)中讨论，或者提交pull request。
* 如果有更好的代码，欢迎提交 pull request。
* 欢迎各种拍砖！
* 如果觉得还不错，欢迎 Star！
