---
layout: post
title: "基于AFNetworking实现的下载管理工具SODownloader"
date: 2016-09-14 11:48:03 +0800
categories:
---

## 谈谈下载功能

文件下载是一种很常见的功能，在 Foundation 层可以使用 NSURLConnection 或 NSURLSession+NSURLSessionDownloadTask 进行实现，这是两种最简单的实现下载功能的方式。如果你熟悉 AFNetworking，你就会觉得基于用 AFNetworking 来实现下载才是最简单的姿势，因为AFURLSessionManager 类已经封装了下载的功能，而且 AFNetworking 简洁的 API 将本来散落在很多代理方法中的处理都聚合到了几个 block 中。所以你可以像下面这段代码中使用 AFNetworking 来实现一个简单的下载需求，与 NSURLConnection 和 NSURLSession 的一大堆代理方法相比，AFNetworking 简洁的不要不要的。

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

但要实现更加复杂的下载，比如有一个下载列表并需要设置同时可以执行几个下载任务，比如要在多个界面中访问下载任务，上面的代码就有点“捉襟见肘”了。于是，我就基于 AFNetworking 3 封装了一个下载管理工具 [SODownloader](https://github.com/scfhao/SODownloader)，力求封装通用的下载需求。

* 支持下载列表及设置同时进行的下载任务的数量
* 支持下载状态、下载进度、及下载列表变化的 KVO 观察
* 支持下载完成后的多样性处理
* 具有完备的下载控制方法
* 支持后台下载
* 支持判断下载文件的MIME类型，不符合则认定失败
* 支持设置是否允许使用

## SODownloader 介绍

为了便于大家使用 SODownloader 时更快的完成下载功能，写了一篇[使用简介](https://github.com/scfhao/SODownloader/wiki)。

## 推广

如果你觉得 SODownloader 的功能还不错，欢迎在自己的项目中使用，也欢迎把 SODownloader 推荐给更多的人。

## 建议&反馈&改进

因本人能力有限，所以 SODownloader 肯定会有很多不足之处。

* 欢迎[Issue](https://github.com/scfhao/SODownloader/issues)、欢迎 pull request。
* 如果觉得还不错，欢迎 Star！
