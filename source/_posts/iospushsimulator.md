---
title: iOS模拟器推送测试及踩坑
tags: iOS
categories: 开发
abbrlink: b766a676
date: 2021-12-19 15:43:04
---

**前言：** 现在的App几乎都有推送功能，开发推送功能的时候我们肯定要进行测试，但是之前推送功能只有在真机上才能测试，在`Xcode11.4`之后，模拟器也支持推送测试，具体操作如下：

### 1.创建推送文件
内容类似如下格式
```
{
    "aps":{
        "alert":{
            "title":"标题",
            "body":"内容辱与共产主义不容辞"
        },
        "content-available" : 1,
        "mutable-content" : 1
    }
}
```
具体格式根据你们的产品要求，接入极光或者个推的可以在控制台发一条推送打印出具体格式内容查看,将文件保存后缀为**apns**，待会要用到

### 2.执行如下命令进行测试

1. 查看已启动模拟器
```
xcrun simctl list devices | grep Booted
```
会看到类似下面信息，如果没有请先启动模拟器
```
iPhone 12 Pro (1BEE4182-C934-431E-BCBF-F7676C4C2BFC) (Booted)
```

2. 运行项目在模拟器上后执行相应命令`simctl push <device> [<bundle identifier>] (<json file> | -)`

示例如下
```
xcrun simctl push Booted com.app.test /Users/lazyloading/Desktop/payload.apns
```

### 3.另外还有一种方式是直接使用推送文件
将第一步创建的json文件内容稍加修改,具体就是添加了`    "Simulator Target Bundle": "com.app.test"你项目的包名
`
```
{
    "Simulator Target Bundle": "com.app.test",
    "aps":{
        "alert":{
            "title":"标题",
            "body":"内容辱与共产主义不容辞"
        },
        "content-available" : 1,
        "mutable-content" : 1
    }
}
```
然后直接拖动文件到模拟器上，出现绿色➕后松手，这样也可以进行推送测试
### 踩坑
到这里按照网上你查看的其他教程应该已经收到测试的推送了，但是我没有>_<!

原因很简单，没有注册，我的项目中集成的是极光推送，已经存在注册步骤，但是这里依然要我注册，如果你遇到类似的问题，添加如下代码

```objective
[UNUserNotificationCenter.currentNotificationCenter requestAuthorizationWithOptions:UNAuthorizationOptionAlert | UNAuthorizationOptionSound  completionHandler:^(BOOL granted, NSError * _Nullable error) {
        NSLog(@"%d",granted);
}];
```

这是应该就可以正常收到推送了，但是依然存在几个问题，推送相关的代理方法没有执行，iOS15真机可以，模拟器不行，例如
```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler 
```
另外就是前台没法收到推送，退到后台才可以，至少我这里遇到的情况是这样，虽然模拟器支持推送测试，但是依然无法和真机进行比较，最好还是以真机测试为准
