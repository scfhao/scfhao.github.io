---
layout: post
title: "打印AFNetworking从服务器获取的数据内容"
date: 2016-07-28 15:04:02 +0800
categories:
---

在一些网络请求中，AFNetworking处理服务器的响应数据时产生了错误，然后对应的失败回调将被调用。这种情况是很常见的，比如当我们使用的AFHTTPSessionManager对象的responseSerializer设置为AFJSONResponseSerializer（这是默认设置），而服务器返回了HTML、XML、格式错误无法解析的JSON或者响应的MIME type不是"application/json"、"text/json"、"text/javascript"中的一种时，AFNetworking就会产生一个错误，然后调用失败回调。

上面说的这几种情况大家或多或少都遇到过，我想说一点，对于最后那种MIME type不对应的情况，很多人都是通过修改AFJSONResponseSerializer对象的acceptableContentTypes数组来“委曲求全”的。我并不建议大家这么做，因为响应的数据本身为JSON时，数据类型你给我返回"text/html"这本身就是错误的！遇到这种问题，我们应该向后台开发反映这个问题，让服务器那边修改。

下面进入本篇的正题，遇到这类问题时，为了确定问题原因，我们需要打印一下服务器返回的数据内容，但失败回调中的2个参数dataTask、error并没有包含服务器返回的数据，所以有人就在这里束手无策了。有两种比较简单的方法容易想到：

- 在失败回调中打印`task.originalRequest.URL.absoluteString`，然后把打印的URL复制到浏览器的地址栏里用浏览器请求一下，这种方式简单易用，但很多情况并不适应，例如提交方式不是GET而是POST时，参数就不能放到浏览器的地址栏了吧？（我仿佛听到有人在说：你可以往浏览器里装个测试post提交的插件啊）
- 打开AFNetworking的源代码，在请求完成的地方打印响应数据的内容，这种情况当然可行，缺点就是如果每个用到AFNetworking的工程，都进去这么改一下，或者每次pod更新AFNetworking后，进去找到对应的地方改一下。

其实AFNetworking提供了可以很方便的获取响应数据的方式，每个请求完成时，AFNetworking就会向NSNotificationCenter发送一条消息通知。这个通知的userInfo字典中就包含了我们需要的数据。

为伸手党准备了一点代码：

```
// 注册通知
- (void)setupLog {
	[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(taskDidComplete:) name:AFNetworkingTaskDidCompleteNotification object:nil];
}

- (void)taskDidComplete:(NSNotification *)notification {
    NSDictionary *userInfo = [notification userInfo];
    NSURLSessionDataTask *task = notification.object;
    NSError *error = userInfo[AFNetworkingTaskDidCompleteErrorKey];
    NSData *receivedData = userInfo[AFNetworkingTaskDidCompleteResponseDataKey];
    NSString *receivedDataAsString = [[NSString alloc]initWithData:receivedData encoding:NSUTF8StringEncoding];
    if (error) {
        [self printInfoForTask:task recievedData:receivedDataAsString error:error];
    } else {
        [self printInfoForTask:task recievedData:receivedDataAsString error:nil];
    }
}

- (void)printInfoForTask:(NSURLSessionTask *)task recievedData:(NSString *)receivedData error:(NSError *)error {
    static NSString * logHeader = @"\n<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<\n";
    static NSString * logFooter = @">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>";
    static NSString * line = @"---\n";
    NSMutableString *log = [NSMutableString string];
    NSURLRequest *request = task.originalRequest;
    [log appendString:logHeader];
    [log appendFormat:@"使用AFNetworking调用<%@>请求[%@]\n", request.HTTPMethod, error == nil ? @"成功": @"失败"];
    [log appendString:line];
    [log appendFormat:@"请求地址:%@\n", [request.URL absoluteString]];
    if ([request.HTTPMethod isEqualToString:@"POST"]) {
        [log appendFormat:@"POST内容:%@\n", [[NSString alloc]initWithData:request.HTTPBody encoding:NSUTF8StringEncoding]];
    }
    [log appendString:line];
    if (error) {
        [log appendFormat:@"错误详情:%@\n", error];
        [log appendString:line];
    }
    [log appendFormat:@"收到数据:%@\n", receivedData];
    [log appendString:logFooter];
    SODebugLog(@"%@", log);
}

```