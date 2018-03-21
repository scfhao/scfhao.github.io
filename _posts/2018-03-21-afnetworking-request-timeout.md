---
layout: post
title: "AFNetworking 设置请求超时时间"
date: 2018-03-21 11:26:34 +0800
categories:
---

在 iOS 中为了一个 HTTP(S) 请求设置超时一般有以下几个途径：

1. 设置`NSURLRequest`对象的`timeoutInterval`属性。
2. 设置初始化`NSURLSession`对象时用到的`NSURLSessionConfiguration`对象的`timeoutIntervalForRequest`属性。

AFNetworking 封装了 NSURLSession，并未对外暴漏可以为 NSURLRequest 对象设置 timeoutInterval 的接口，但在创建 AFHTTPSessionManager 时，提供了`- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration`初始化方法，就是说，你可以通过传入一个设置好超时时间的 NSURLSessionConfiguration 对象来初始化 AFHTTPSessionManager 对象，但这么做有个明显的缺陷：所有通过此 AFHTTPSessionManager 发送的网络请求都将使用这里设置的这个超时时间。

但在实际开发中，不同的请求服务器处理所需要的时间是不同的，在不同的情景中，使用不同的超时时间也是特别常见的需求。在这种情况下，如果 AFHTTPSessionManager 提供的 GETXXX、POSTXXX等方法中有参数可以直接控制当前这个请求的超时时间就好了，可惜 AFNetworking 并没有提供这样的便利。

当然我们可以通过修改 AFNetworking 源码的方式为每个 GETXXX、POSTXXX 增加设置超时时间的参数来获得这种便利性。如果不想修改 AFNetworking，我们还可以通过下面的方式为某个请求指定一个单独的请求超时时间。

```
// yourManager 是你自己创建的 AFHTTPSessionManager 对象
// 假设想将 api/example 这个接口的超时时间设置为 6s，其他接口超时时间为默认的 60s。

// 下面这行代码通过 requestSerializer 设置设置一个特殊的 6 秒
[AFHTTPSessionManager yourManager].requestSerializer.timeoutInterval = 6;

NSLog(@"请求开始");
[[AFHTTPSessionManager yourManager]GET:@"api/example" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable response) {
	// ...
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    NSLog(@"请求结束:%@", error);
}];

// 将默认超时时间恢复为 60 秒，这样不会影响其他请求的超时时间
[AFHTTPSessionManager yourManager].requestSerializer.timeoutInterval = 60;
```

可以用 [Charles](https://www.charlesproxy.com) 上的断点功能测试请求超时的情况。
