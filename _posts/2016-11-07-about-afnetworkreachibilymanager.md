---
layout: post
title: "AFNetworking判断网络连接是否存在及离线网络缓存的使用"
date: 2016-11-07 20:07:28 +0800
categories:
---

# 判断网络连接是否存在

## 从哪里来

要说在iOS中如何判断是否存在网络连接，大家肯定会想到 Reachability，Reachability 是 Apple 为我们提供的判断网络连接是否存在的 Demo。大部分读者应该也都留意过 AFNetworking 中有个类叫 AFNetworkReachabilityManager 的类。对！这两个类都可以用来判断设备是否存在网络连接，可是我们到底该用哪个呢？

AFNetworking 在 iOS 项目中的普及率是很高的，而凡是集成了这个网络库的项目，肯定会包含了 AFNetworkReachabilityManager，对于这种情况来说，用AFNetworkReachabilityManager 来判断是否存在网络连接是最好的选择，这个时候肯定会有一些人跳出来说：“AFNetworkReachabilityManager好像不准哎”，于是在使用了 AFNetworking 的同时，又把 Reachability 集成进项目中…… 而且这样的人还不在少数！直到有一天，一个同事对我说了同样的话，我就觉得有必要写点什么了，于是就有了这篇博客，如果你也有相同的认知，那你便是本文的读者。

对于这样的读者，我想问如下几个问题：

1. 不用 AFNetworking 中已有的，专门添加一个Reachability，心里觉得爽吗？反正我大天秤座是很不爽。
2. 既然觉得 AFNetworkReachabilityManager 不太准确，有没有去看一下为什么不准确？

## 会觉得AFNetworkReachabilityManager 不准确的原因

先看一下 AFNetworkReachabilityManager 的用法：

```
[[AFNetworkReachabilityManager sharedManager]startMonitoring];
BOOL reachability = [AFNetworkReachabilityManager sharedManager].reachable; // reachability will be NO.

```

这可能是大部分说AFNetworkReachabilityManager不准的同行的测试方法，设备明明有网络访问，但reachability却是NO，这不是不准是什么？

但如果调用Reachability的currentReachabilityStatus方法，则会返回正确的网络状态，所以很多人判断是否存在网络连接时，就用Reachability判断，而把AFNetworkReachabilityManager当作废柴。

## 我们亲眼见到的也并步一定就是真的

不管是AFNetworkReachabilityManager还是Reachability，都是通过SystemConfiguration框架中的SCNetworkReachability来判断是否存在网络连接的，其中有个关键函数叫SCNetworkReachabilityGetFlags，获取当前的网络状态就是通过调用这个函数得到的，唯一的区别就是Reachability是在当前的线程调用这个函数的，所以currentReachabilityStatus方法返回时，这个函数已经把正确的网络状态返回了，而AFNetworkReachabilityManager在startMonitoring方法中将SCNetworkReachabilityGetFlags函数放到其他线程中异步执行了，所以当startMonitoring方法返回时，SCNetworkReachabilityGetFlags函数可能在其他线程正在执行，所以此时AFNetworkReachabilityManager对象的networkReachabilityStatus属性值还是初始值AFNetworkReachabilityStatusUnknown，直到异步线程中的SCNetworkReachabilityGetFlags函数返回，该属性值才能正确反映当前的网络状态（这段的描述比较长，如果不是很理解，结合AFNetworking的源码一看就明白了）。

那么，怎么避免当我们想要判断当前网络状况时，AFNetworkReachabilityManager的status还是Unknow的情况呢？

1. 将AFNetworkReachabilityManager的startMonitoring方法在一个比较早的时机执行，比如放到某个类的load方法里，或者放到AppDelegate的finishLaunch方法中执行。
2. 尽量不在程序刚启动的时候判断是否存在网络访问，例如尽量不要在finishLaunch方法或初始视图控制器的viewDidLoad方法中判断，因为这时异步线程的SCNetworkReachabilityGetFlags很可能还没有返回。
3. 没有3了，如果你的应用中不需要在程序一启动就判断网络是否存在，那么你应该选择使用AFNetworkReachabilityManager，因为AFNetworkReachabilityManager是AFNetworking中精心封装的一个工具，而Reachability仅仅是一个教你如何检测是否存在网络连接的Demo！想想你每次调用currentReachabilityStatus方法，它都会阻塞当前线程直到耗时的SCNetworkReachabilityGetFlags返回就明白了。

# 缓存网络请求

**这里只探讨在本地缓存网络请求的响应数据，以备没有网络的情况使用(本文中出现的“缓存”都是指这个意思)，这里说的缓存有点狭隘，因为缓存本应是服务器和客户端双向交流、相互作用的过程，所以本文中的缓存都应加引号**，不展开论述缓存原理，关于缓存原理的博客，网络上已经有很多了，建议不了解缓存原理的读者，在接受完本文的误导后，先去了解一下缓存机制的原理。

我见过很多人写的缓存功能（包括我自己），大体的可以分为如下几类：

1. 使用数据库，将请求到的内容保存到数据库中，如果判断到没有网络或请求失败则在本地数据库中读取内容。
2. 将响应内容保存到本地目录，例如把服务器返回的二进制数据（或者解析得到的Dictionary或Array存到本地一个路径）存到本地一个路径下，在没有网络或请求失败的情况下读取对应的文件。

大家都应该注意过NSURLRequest有个属性叫cachePolicy，NSURLSessionConfiguration有个属性叫requestCachePolicy，这两个属性都是指缓存策略，即你的应用如何使用缓存数据。在默认的情况下，缓存策略为NSURLRequestUseProtocolCachePolicy，系统（URL Loading System）会为你缓存每个请求的信息，这些信息也是保存在一个sqlite数据库中，这个数据库的位置为App's bundle path/Library/Cache/{bundle identifier}/Cache.db，有心的人如果去搜索引擎搜一下cache.db这个关键字，可以搜到很多威锋论坛关于这个文件的讨论，例如这里有一篇[来自威锋技术组的](http://bbs.feng.com/read-htm-tid-5860622.html)。

嗯，系统已经默默地为我们把每个请求的响应缓存下了，然后我们自己在写一些所谓的在本地保持“缓存数据”的逻辑，相当于同样的数据存了两份，大天秤座又受不了了。

那么，如何使用系统保存的这一份缓存呢？答案是使用NSURLCache，这是官方提供的缓存类，你即使没亲自用过也一定见过这个类，例如AFNetworking在AFImageDownloader中创建图片缓存的代码如下：

```
+ (NSURLCache *)defaultURLCache {
    return [[NSURLCache alloc] initWithMemoryCapacity:20 * 1024 * 1024
                                         diskCapacity:150 * 1024 * 1024
                                             diskPath:@"com.alamofire.imagedownloader"];
}

```

所以，你不要再说什么AFNetworking的图片缓存只是在内存中缓存的屁话了，如果你亲自测试这个缓存没有起到作用（例如没有网络的时候，之前请求过的图片没有展示出来），那你应该去找篇将缓存原理的文章看看，然后拉上你们公司的后台一起讨论一下“如何使用缓存”。（好的后台不多了，如果你不是在大公司，公司的服务器还支持完整的缓存功能，一定要好好珍惜！）

## 离线环境的缓存利用

回到正题上来，如何在离线环境使用系统为我们提供的缓存功能呢？

要在离线的情况下使用本地（Cache.db中）的缓存，就要用到前面提到的缓存策略了，当把NSURLRequest的cachePolicy或NSURLSessionConfiguration的requestCachePolicy设置为NSURLRequestReturnCacheDataElseLoad或NSURLRequestReturnCacheDataDontLoad时（NSURLRequest中设置的缓存策略的优先级高于NSURLSessionConfiguration中的requestCachePolicy），之前缓存的响应会直接被返回，如果存在的话。

那么问题来了，如果我们将每个请求的缓存策略都设置为优先使用本地缓存，那服务器上更新过的内容就请求不到了（因为每次都是得到之前返回的缓存到本地的旧数据），所以就需要分情况使用了，结合本文前半部分提到的判断网络连接是否存在，要当没有网络的情况下才返回已缓存的数据（即缓存逻辑设置为NSURLRequestReturnCacheDataDontLoad或NSURLRequestReturnCacheDataElseLoad），而在有网络的情况下请求服务器上最新的数据（即缓存逻辑设置为NSURLRequestUseProtocolCachePolicy）。

听起来好麻烦，幸运的是，AFNetworking中的AFNetworkReachabilityManager提供了当网络环境切换时的回调方法（当然也可以通过网络环境切换的通知实现），分别如下：

```
- (void)setReachabilityStatusChangeBlock:(nullable void (^)(AFNetworkReachabilityStatus status))block;

AFNetworkingReachabilityDidChangeNotification

```

这样，我们就可以分别为在线和离线两种环境创建不同的AFHTTPSessionManager对象，然后在不同的环境下使用对应的manager请求数据：

```

@implementation AFHTTPSessionManager (example)

+ (void)initialize {
    [[AFNetworkReachabilityManager sharedManager]setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        if (status == AFNetworkReachabilityStatusNotReachable) {
            [self setJSONManager:[self jsonManagerOffline]];
        } else {
            [self setJSONManager:[self jsonManagerOnline]];
        }
    }];
}

static AFHTTPSessionManager *_jsonManager = nil;

+ (void)setJSONManager:(AFHTTPSessionManager *)manager {
    _jsonManager = manager;
}

+ (instancetype)managerForJSON {
    return _jsonManager;
}

// 示例请求调用方法
[AFHTTPSessionManager managerForJSON]GET:@"api.example.com/apiName" param:apiParam...

```
创建离线AFHTTPSessionManager的方法如下(在线版本的代码类似，不贴重复的代码了，唯一的区别就是configuration.requestCachePolicy不同)：

```
+ (NSURLSessionConfiguration *)sessionConfigurationForOffline {
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    configuration.timeoutIntervalForRequest = 10;
    configuration.requestCachePolicy = NSURLRequestReturnCacheDataElseLoad;
    return configuration;
}

+ (instancetype)jsonManagerOffline {
    static AFHTTPSessionManager *_managerForJSON = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _managerForJSON = [[AFHTTPSessionManager alloc]initWithBaseURL:[NSURL URLWithString:@"http://api.gkbbapp.com/api/"] sessionConfiguration:[self sessionConfigurationForOffline]];
        _managerForJSON.responseSerializer = [AFJSONResponseSerializer serializerWithReadingOptions:NSJSONReadingAllowFragments];
        _managerForJSON.responseSerializer.acceptableContentTypes = [self defaultAcceptableContentTypes];
    });
    return _managerForJSON;
}

```

## 停用缓存

并不是所有的请求的响应都应该缓存下来，这里举个登录的例子，在线时调用登录接口并缓存了数据，稍后修改了密码，离线登录时用原密码竟然登录成功了，这显然是不合适的。不仅仅是登录，凡是对数据实时性要求比较高的接口数据都不应该被缓存下来。

登录接口不能缓存的一个更重要的还原是安全问题，试想默认情况（没有进行任何和缓存有关的设置），系统把你的登录请求（里面包含用户名、密码）保存到了Cache.db文件中，即使你的信息加密过了，用Cache.db中缓存的请求也可以继续用于登录。这点也是在写本文时想到的，还有其他敏感接口，如果不管不顾缓存的使用，细思恐极。

要在AFNetworking中禁止对某个请求缓存数据调用AFHTTPSessionManager的setDataTaskWillCacheResponseBlock方法，如下：

```
[_managerForJSON setDataTaskWillCacheResponseBlock:^NSCachedURLResponse * _Nonnull(NSURLSession * _Nonnull session, NSURLSessionDataTask * _Nonnull dataTask, NSCachedURLResponse * _Nonnull proposedResponse) {
            NSString *path = dataTask.originalRequest.URL.path;
            if (path对应的接口需要缓存) {
                return proposedResponse;
            } else {
                return nil;
            }
        }];

```

本文探讨的问题比较初级，如果文中有错误或让你觉得不舒服的内容，请在评论中指出。