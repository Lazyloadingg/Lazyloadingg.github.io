---
title: SDWebImage链接重定向问题处理
date: 2022-03-14 20:01:58
tags: iOS
categories: 开发
---


#### Round 1

最近公司的文件服务器进行了改造，即使是图片的加载请求也要携带`token`，否则无法加载，而我们项目中图片加载用的是`SDWebImage`,当时听到这个需求我内心毫无波动，心里已经....你懂得，不过该做还是要做，三下五除二写完了代码如下：
```objectivec
[[SDWebImageDownloader sharedDownloader] setValue:@"你的token" forHTTPHeaderField:@"Authorization"];
```
`SDWebImage`的下载处理是由`SDWebImageDownloader`单例类实现，所以在你项目中合适的地方加上这句代码，项目中所有用`SDWebImage`做图片加载的地方就都会携带上`token`了

#### Round 2
这样修改完后确实本来不能加载的地方变得正常了，直到那一天，那是一个春天...
项目中要添加一个需求，需要引用公司的一个私有`Pod`功能库，又是一顿操作，集成完毕，逻辑编写完成，run，诶，图片居然没加载出来，我草这什么情况，我再次确认了一下，我的`token`设置完成了的

我去询问编写这个`Pod`的同事，是不是我哪里没配置完成，他略微沉思一下两秒钟后说，你添加完`token`还要在`SDWebImageDownloader`修改下源码，我：？？？，随后他找到了这个源码，代码如下：
```objectivec
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task willPerformHTTPRedirection:(NSHTTPURLResponse *)response newRequest:(NSURLRequest *)request completionHandler:(void (^)(NSURLRequest * _Nullable))completionHandler {

    // Identify the operation that runs this task and pass it the delegate method
    NSOperation<SDWebImageDownloaderOperation> *dataOperation = [self operationWithTask:task];
    if ([dataOperation respondsToSelector:@selector(URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:)]) {
        [dataOperation URLSession:session task:task willPerformHTTPRedirection:response newRequest:request completionHandler:completionHandler];
    } else {
        if (completionHandler) {
            completionHandler(request);
        }
    }
}
```

他解释说道，这个功能模块里的一些图片链接中携带了一些参数，并不是直接指向资源，所以请求会进行重定向，所以需要在这里进行处理，处理后的代码如下：
```objectivec
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task willPerformHTTPRedirection:(NSHTTPURLResponse *)response newRequest:(NSURLRequest *)request completionHandler:(void (^)(NSURLRequest * _Nullable))completionHandler {
    
    // Identify the operation that runs this task and pass it the delegate method
    NSOperation<SDWebImageDownloaderOperation> *dataOperation = [self operationWithTask:task];
    NSMutableURLRequest *customRequest = [[NSMutableURLRequest alloc] initWithURL:request.URL];
    customRequest.allHTTPHeaderFields = self.HTTPHeaders;
    if ([dataOperation respondsToSelector:@selector(URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:)]) {
        [dataOperation URLSession:session task:task willPerformHTTPRedirection:response newRequest:customRequest completionHandler:completionHandler];
    } else {
        if (completionHandler) {
            completionHandler(customRequest);
        }
    }
}
```

我将代码修改后，run，确实，问题解决了，但是不对啊，我们的`SDWebImage`是通过`Pod`的方式集成的，这样直接在`Pod`文件夹下修改三方的源码，那么下次更新后，岂不是被覆盖了？这是一个新的问题，于是我开始思考怎么解决重定向问题的同事不修改三方库的源码，脑海中瞬间就想到了`AOP`

iOS开发中优秀的`AOP`库那必须有`Aspects`名字，接下来我开始思考具体步骤
首先，通过同事提供的解决问题的代码来看，他是把参数`request`给改为了一个自定义的`customRequest`，这两个的区别，然后重新设置了`allHTTPHeaderFields`

```objectivec
NSMutableURLRequest *customRequest = [[NSMutableURLRequest alloc] initWithURL:request.URL];
customRequest.allHTTPHeaderFields = self.HTTPHeaders;
```
那么我想，问题主要就是在`allHTTPHeaderFields`这里了，我打印了`request`和`customRequest`的`allHTTPHeaderFields`后发现，前者比后者少了`token`,怪不得无法加载，这里要提一下下边这个方法

```objectivec
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task willPerformHTTPRedirection:(NSHTTPURLResponse *)response newRequest:(NSURLRequest *)request completionHandler:(void (^)(NSURLRequest * _Nullable))completionHandler 
```
这其实不是`SDWebImageDownloader`的方法，是`NSURLSessionTaskDelegate`的里的协议方法，`SDWebImageDownloader`实现协议方法后在里边做了自己的重定向处理

那么说一下我一开始的想法，既然问题出在重定向时`request`里未携带我们手动添加的`token`，并且重定向这里肯定是要做处理的，那我们直接把相关参数设置给`request`，没必要创建一个新的`customRequest`实例

```objectivec
[request setValue:self.HTTPHeaders forKey:@"allHTTPHeaderFields"];
```
因为`request`是一个`NSURLRequest`对象，它的`allHTTPHeaderFields`是一个`readonly`属性，我们不能直接修改，所以我想当然的用`KVC`去操作，
然后 run ，然后 我草crash了，报错信息如下

```objectivec
"[<NSURLRequest 0x2800efa50> setValue:forUndefinedKey:]: this class is not key value coding-compliant for the key allHTTPHeaderFields."
```
看描述信息是说`NSURLRequest`没有对应的`allHTTPHeaderFields`这个key，有那么一瞬间我愣了下，这不科学啊怎么可能没有，我点进`NSURLRequest`类确认了下，有啊，什么情况，但是本着求知的态度，我心想是不是`NSURLRequest`内部使用的是不是不叫`allHTTPHeaderFields`,但是不对啊，这个属性明明在的啊，即使是别的也应该是内部重新赋值我这里不应该报错啊，不过我还是用通过`runtime`将他的属性列表打印了一下，再次确认了，他确实有`allHTTPHeaderFields`这个属性，于是我搜索了下相关问题，发现了一个关键词
```objectivec
+ (BOOL)accessInstanceVariablesDirectly
```
详细信息自行检索，我这里说下结果，这个是针对`KVC`的，总的来说，当一个类实现了这个方法并且返回了`YES`他就可以通过`KVC`(这个说法不完全准确，因为本文不是针对KVC的故简要说明)赋值，如果返回`NO`就不可以用`KVC`赋值

看到这里后我猜测`NSURLRequest`里这个方法应该是返回了`NO`,那完了，走不通了，还是要实例化一个新的对象



#### Round 3
搞了半天想省事看来不行啊，那拉倒，直接开始`AOP`修改：
```objectivec
    NSError * error ;
    [[SDWebImageDownloader sharedDownloader] aspect_hookSelector:@selector(URLSession: task: willPerformHTTPRedirection: newRequest: completionHandler:) withOptions:AspectPositionInstead usingBlock:^(id<AspectInfo> aspectInfo, NSString *num){
        
        NSArray * param = aspectInfo.arguments;
        NSURLRequest * request = param[3];
        NSURLSessionTask * task = param[1];
        NSMutableURLRequest *customRequest = [[NSMutableURLRequest alloc] initWithURL:request.URL];
        customRequest.allHTTPHeaderFields = task.currentRequest.allHTTPHeaderFields;
        request = customRequest;
        NSInvocation * invocation = aspectInfo.originalInvocation;
        [invocation setArgument:&request atIndex:5];
        [invocation invoke];
        
    } error:&error];
```
这里边`aspectInfo`就是被hook方法的信息，可以通过它取到原方法的参数，调用对象等等，我们这里要添加我们的token，因此取出参数进行修改
* arguments是原方法的入参列表，是一个数组
* invocation是一个消息对象，包含了一个方法的所有信息
通过`URLSession: task: willPerformHTTPRedirection: newRequest: completionHandler:`方法我们可以知道`request`的索引是3，`task`的索引是1（取出它是因为我们要获取原header信息，这个不能丢弃），之后对`request`重新进行赋值，完成修改，然后重新调用方法
```objectivec
 [invocation setArgument:&request atIndex:5];
 [invocation invoke];
```
因为我们只需要修改`request`一个对象，因此只重新设置这一个方法入参，至于这里为什么在赋值的时候索引是5,因为前两个分别被该方法的`self`与`_cmd`占用，所以我们设置参数的时候索引是从2开始

再次run，嗯，图片顺利加载，问题解决。

这样一来，我们就不需要修改三方的源码就解决了问题，否则修改源码的话每次更新`Pod`我们的修改就会被覆盖掉，如果哪次发版没注意，测试也没回归覆盖，很容易将问题带到线上

