---
layout: post
title: "AFNetworking代码阅读（一）"
date: 2016-06-16 15:55:52 +0800
categories:
---

# AFURLSessionManager阅读
本笔记为scfhao阅读AFNetworking-AFURLSessionManager类源代码时记录，当前AFNetworking版本3.0.4，笔记创建时间：2016-6-14。这篇笔记中，用“其”代表AFURLSessionManager类。为方便稍后阅读，本篇笔记的记录顺序尽量选择更容易理解的顺序，尽量不多贴代码。
## AFURLSessionManagerTaskDelegate
AFURLSessionManager为其中每个请求任务都创建了一个任务代理对象，这个代理对象，这些代理对象可以管理其对应的任务的请求进度，如上传任务的上传进度、下载任务的下载进度，还实现了几个请求任务代理方法（NSURLSessionDelegate）。这个类设计的很小巧、精妙，这个类使AFURLSessionManager对上传、下载进度的监控更灵活。其属性列表如下：

* manager 对所属AFURLSessionManager的一个弱引用。
* mutableData 存储对应请求的响应数据。
* uploadProgress 上传进度，NSProgress对象。
* downloadProgress 下载进度，也是NSProgress对象。
* downloadFileURL 下载文件的目标位置。
* downloadTaskDidFinishDownloading 下载请求下载完成的回调。
* uploadProgressBlock 上传回调，上传进度改变时会调用这个回调。
* downloadProgressBlock 下载回调，下载进度改变时会调用这个回调。
* completionHandle 请求完成回调。

从上述的属性列表也可以看出这个类的作用，这个类的初始化方法也比较简单，创建了请求的上传下载进度对象。

这个的方法不多，首先是监控进度的两个方法，这两个方法一个是为和进度相关的对象添加观察者，另一个移除上个方法添加的观察者，这是iOS开发比较常用的KVO，所以不要再说一般的开发中用不到KVO了，你没用到只是说明你自己有待提升：

* `- setupProgressForTask:`先对上传下载进度对象进行了一些常规的设置，比如总进度、取消处理、暂停处理、恢复处理，设置完这些以后，用户就可以通过调用进度对象（NSProgress）的cancel、pause、resume方法来使请求任务对象（NSURLSessionTask）做出相应的处理了。然后对请求任务和进度相关的属性（countOfBytesReceived、countOfBytesExpectedToReceive、countOfBytesSent、countOfBytesExpectedToSend）和进度对象的进度（fractionCompleted）添加了观察者，观察请求任务和进度相关的属性是为了更新NSProgress对象，而观察NSProgress对象的fractionCompleted则是为了调用进度回调方法。
* `- cleanUpProgressForTask:`请求任务结束后通过调用此方法清除上面添加的观察者。

除了进度监控外，任务代理类还分别实现了NSURLSessionTaskDelegate、NSURLSessionDataTaskDelegate、NSURLSessionDownloadTaskDelegate协议的一个方法，如下：

* `- URLSession:task:didCompleteWithError:`用responseSerializer处理返回数据，调用completionHandle回调，发送AFNetworkingTaskDidCompleteNotification通知。
* `- URLSession:dataTask:didReceiveData:`将接收到的数据添加到mutableData属性中。
* `- URLSession:downloadTask:didFinishDownloadingToURL:`在这个方法中调用了该类的downloadTaskDidFinishDownloading回调，然后将下载到的文件移动到目标下载位置，上述回调的返回值就是目标下载位置，如果移动文件失败，发送了AFURLSessionDownloadTaskDidFailToMoveFileNotification通知。

### 初始化

## AFURLSessionManager
### 初始化
其指定初始化方法为：`- initWithSessionConfiguration:`，在这个方法里，其初始化了自身的一些属性。初始化的属性列表：

* sessionConfiguration 初始化为本方法的参数，如果未指定，则初始化为NSURLSessionConfiguration.defaultSessionConfiguration。
* session 使用sessionConfiguration初始化的NSURLSession，用于创建请求任务NSURLSessionTask。
* operationQueue 为上述session属性的delegateQueue。这个OperationQueue的maxConcurrentOperationCount被设置为1，保证NSURLSession的代理方法不会同时执行。
* responseSerializer 被初始化为AFJSONResponseSerializer对象。
* securityPolicy 被初始化为AFSecurityPolicy.defaultPolicy，本类中有关安全策略的内容将在AFSecurityPolicy类的阅读笔记中注释。
* reachabilityManager 被初始化为AFNetworkReachabilityManager.sharedManager。这个属性在本类中并没有被使用。
* mutableTaskDelegatesKeyedByTaskIdentifier 这是一个NSMutableDictionary类型的属性，用来保存请求任务（NSURLSessionTask）到任务代理对象（AFURLSessionManagerTaskDelegate）的映射关系。
* lock 线程锁NSLock类型。

初始化完上述属性后，调用了session属性的getTasksWithCompletionHandle:方法，为已存在的task设置了任务代理，这里我有点**疑惑**，刚创建的session，还没有创建任何task，能获取到task吗？

### 创建请求任务
这里的请求任务指NSURLSessionTask及其子类（NSURLSessionDataTask、NSURLSessionUploadTask、NSURLSessionDownloadTask）。初始化了AFURLSessionManager对象后，就可以调用其创建请求任务的方法了，请求任务的方法如下：

* `- dataTaskWithRequest:completionHandle:`
* `- dataTaskWithRequest:uploadProgress:downloadProgress:completionHandle:`

---

* `- uploadTaskWithRequest:fromFile:progress:completionHandle:`
* `- uploadTaskWithRequest:fromData:progress:completionHandle:`
* `- uploadTaskWithStreamedRequest:progress:completionHandle:`

---

* `- downloadTaskWithRequest:progress:destination:completionHandle:`
* `- downloadTaskWithResumeData:progress:destination:completionHandle:`

### 为请求对象设置代理

上述创建请求任务的方法也很简单，先调用NSURLSession类的对应方法创建了task，然后分别为不同类型的task对象调用对应的设置任务代理的方法设置了前面说的任务代理（AFURLSessionManagerTaskDelegate），这些设置任务代理的相关方法如下：

* `- addDelegateForDataTask:uploadProgress:downloadProgress:completionHandle:`
* `- addDelegateForUploadTask:progress:completionHandle:`
* `- addDelegateForDownloadTask:progress:destination:completionHandle`

上面3个方法中分别为3种不同类型的请求任务创建了请求代理对象，设置好代理对象的属性。

### 请求任务代理对象的存取

在为请求任务创建好任务代理对象后，这些对象存到了AFURLSessionManager类的mutableTaskDelegatesKeyedByTaskIdentifier字典中，这个字典中task的identifier为key，任务代理对象为value，存取这个字典时，使用了线程锁保证在多个线程中访问这个字典的线程安全性。相关方法如下：

* `setDelegate:forTask:`将代理对象存入字典，调用代理方法的`setupProgressForTask:`方法，注册监听task的挂起、恢复通知。
* `delegateForTask:`从字典中取到代理对象并返回。
* `removeDelegateForTask:`从字典中移除代理对象，移除`setDelegate:forTask:`方法中注册的通知。

### 请求的挂起与恢复

AFNetworking通过*method swizzle*在NSURLSessionTask类的resume和suspend方法中分别发送了AFNSURLSessionTaskDidResumeNotification和AFNSURLSessionTaskDidSuspendNotification通知。

AFURLSessionManager在收到对应的通知后，判断通知对应的task如果是自己（指AFURLSessionManager对象）创建的task，则继续分发了AFNetworkingTaskDidResumeNotification和AFNetworkingTaskDidSuspendNotification通知，见`taskDidResume:`和`taskDidSuspend:`两个方法。

### 同步地调用异步方法

这个类中封装了几个方便调用方获取当前AFURLSessionManager中的请求任务数组的方法，为：`tasks`、`dataTasks`、`uploadTasks`、`downloadTasks`，这几个方法都是通过调用`tasksForKeyPath:`方法获取对应的数组，在这个方法中，通过调用session对象的异步方法`getTasksWithCompletionHandle:`获取session下的请求任务，如果不做任何处理，外层方法返回时内层的异步方法还未执行结束，此时返回值是nil，AFURLSessionManager通过使用GCD提供的信号量，达到了同步地获取异步函数的返回值的效果：

```
- (NSArray *)tasksForKeyPath:(NSString *)keyPath {
    __block NSArray *tasks = nil;
    // 创建了初始信号为0的信号量
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    // 下面方法参数中的block会异步执行即不在当前线程中执行
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        // ...
		// 异步的block执行结束，发送一个信号
        dispatch_semaphore_signal(semaphore);
    }];
	// 当前线程执行到这里时，因为信号量的初始信号数是0，所以线程在这里会阻塞，直到接收到上面的block在其他线程执行结束后发送的信号，这里才继续往下执行。
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

    return tasks;
}
```

### 销毁session
调用AFURLSessionManager对象的`invalidateSessionCancelingTasks:`方法可以使session失效，如果改方法的参数为YES，会同时取消未完成的请求任务。

### 回调
AFURLSessionManager实现了NSURLSession的代理方法，为了让用户方便得定制这些代理方法的实现，AFURLSessionManager提供了一批回调属性。有点多，不过这个类马上就看完了，再忍一下。

* sessionDidBecomeInvalid
* sessionDidReceiveAuthenticationChallenge
* didFinishEventsForBackgroundURLSession

---
* taskNeedNewBodyStream
* taskWillPerformHTTPRedirection
* taskDidReceiveAuthenticationChallenge
* taskDidSendBodyData
* taskDidComplete

---
* dataTaskDidReceiveResponse
* dataTaskDidBecomeDownloadTask
* dataTaskDidReceiveData
* dataTaskWillCacheResponse

---
* downloadTaskDidFinishDownloading
* downloadTaskDidWriteData
* downloadTaskDidResume

### Session 代理方法实现

AFURLSessionManager最主要的功能就是实现其session属性的众多代理方法，包括NSURLSessionDelegate、NSURLSessionTaskDelegate、NSURLSessionDataTaskDelegate、NSURLDownloadTaskDelegate协议的方法。虽然有点多，但这些方法的实现可如下分类：

#### 仅调用外部传进来的回调
在这些方法中仅仅是调用外部设置的对应的回调，或进行了少许处理，如果没有设置回调，什么都不会执行，甚至重写了`respondsToSelector:`，对没有设置对应的回调的代理方法返回NO。这样的方法有：

* `- URLSession:didBecomeInvalidWithError`调用回调外，还发送了AFURLSessionDidInvalidateNotification通知。
* `- URLSession:task:willPerformHTTPRedirection:newRequest:completionHandle:`仅调用回调。
* `- URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`先对totalUnitCount为NSURLSessionTransferSizeUnknown的情况做了容错，然后调用了回调。
* `- URLSession:task:needNewBodyStream:`如果未设置回调，使用原始请求的的HTTPBodyStream。
* `- URLSession:dataTask:didReceiveResponse:completionHandle:`仅调用回调。
* `- URLSession:dataTask:didBecomeDownloadTask:`为task重新设置了一个任务代理，然后调用回调。
* `- URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`仅调用回调。
* `- URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`这里仅仅是调用了回调，AFURLSessionManager并未利用该方法计算下载任务的下载进度，而是通过KVO观察了下载任务的相关属性来计算的进度。
* `- URLSessionDidFinishEventsForBackgroundURLSession:`在主线程调用回调。
* `- URLSession:dataTask:willCacheResponse:completionHandle:`仅调用回调。

#### 转发给请求任务代理对象执行
任务代理对象实现了3个代理方法，这3个代理方法除了自己执行一些操作，调用回调外还调用了任务代理对象对应的方法：

* `- URLSession:task:didCompleteWithError:`先调用了任务代理对象对应的方法，然后调用自身的removeDelegateForTask方法清理了代理对象，最后调用了外部设置的回调。
* `- URLSession:dataTask:didReceiveData:`先调用了任务代理对象的对应方法，然后调用了外部设置的回调。
* `- URLSession:downloadTask:didFinishDownloadingToURL:`先调用回调方法得到下载任务的目标位置，将下载完成的文件移动到目标位置，后调用任务代理对象的对应方法。任务代理实现的这个方法中也有移动文件的操作，所以如果同时设置了AFURLSessionManager的downloadTaskDidFinishDownloading回调并在创建下载任务时传入了destination回调时，下载完成后，任务代理对象的移动文件的操作将会失败。

#### 其他代理方法
剩余的这两个代理方法将在AFSecurityPolicy类的阅读笔记中说明。

* `- URLSession:didReceiveChallenge:completionHandle:`
* `- URLSession:task:didReceiveChallenge:completionHandle:`