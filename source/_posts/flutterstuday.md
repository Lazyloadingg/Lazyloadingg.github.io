---
title: 初识Flutter总结
date: 2021-07-22 22:33:55
tags: Flutter
---
<meta name="referrer" content="no-referrer" />
[toc]

## Flutter是什么
* Flutter是一个基于`Dart`语言的跨平台**UI**框架，目前支持的平台有`iOS`，`android`，`Web`，`Mac OS`，`Windows`，`Linux`，强调这个是因为有部分初学者没接触的时候认为Flutter是一门语言，这是不正确的

## 环境搭建
* Mac和Linux系统设备建议使用[Homebrew](https://brew.sh/index_zh-cn)执行如下命令安装，Windows的话，去看教程
```
brew install flutter
```

* 安装完默认路径一般为`/usr/local/Caskroom/flutter/2.2.0/flutter`这个样子
* 网上有一大堆的教程去叫你怎么搭建环境，这里重复的原因是因为，如果你去按照网上的教程去搭建环境，会感觉，很繁琐，真的很繁琐，特别是iOS开发人员，为什么搭建环境一定要那么多步骤？而Homebrew让这一切变得很简单，只需要一行命令，然后等待网络下载完成就是了

## 平台支持
* 已Android studio为例，创建项目时，默认支持iOS和Android，但是可勾选支持Web，Linux和Windows ，Mac OS为不可选，如果需要支持可手动进行配置，命令如下，之后再创建项目就可以勾选相应的其他平台
```
 flutter config --enable-windows-desktop
 flutter config --enable-macos-desktop
 flutter config --enable-linux-desktop
```

* 如果想要在已有的项目中添加新的平台，进入项目根目录后执行如下命令
```
flutter create .
```


## 组件
### 概括
* Flutter里的组件都是Widget，基本可以分为两大类`StatefulWidget`（可变）和`StatelessWidget`（不可变）,区别就在于前者可以通过对应的State类去改变内容，所以每一个StatefulWidget都要对应一个State，而后者初始化后就不能再进行改变

* Flutter框架本身的组件风格分为两种`Material`和`Cupertino`，前者是Google的设计风格，看一下安卓设备大概就能了解到是什么样子，后者是iOS风格也就是Apple的设计风格，网上的教程几乎都是以`Material`，我在学习的过程中也是使用`Material`风格的组件，不过这些只是Flutter内置的供你使用的组件，同一个UI你完全可以用这两种风格分别实现

### 常用Widget
Flutter中内置了几百个Widget，包含显示类的文字(Text)，图片(Image)等，也包含不直接显示的容器类Container，Row，Column等，当你入门后，对Flutter有了一定的认知，后续的使用中就是经验的积累，去识记更多的Widget，记得越多，开发越快，下面介绍几种常用的Widget，记住这几个后基本开发一个简单的App就没什么问题了
1. **MaterialApp：** 一般用作根Widget，App启动后，main（入口）函数中执行的方法中需要提供

2. **Scaffold：** 顾名思义，他是Flutter提供的一个脚手架Widget，自带抽屉，导航等，方便你快速搭起App的结构
* **Container：** 容器类Widget的一种，也是最常用的Widget之一，可以理解为iOS中的UIView
* **Row：** 横向布局容器，加入其中的子Widget会水平排列布局
* **Column：** 纵向布局容器，加入其中的子Widget会纵向排列布局
* **Stack：** 叠加布局布局容器，加入其中的子Widget会叠加起来排列布局
* **ListView：** 市面上几乎所有的App都会使用的，列表Widget
* **GridView：** 卡片，瀑布流布局
* **Text：** 文字展示
* **Image：** 图片展示
* **TextButton：** 文字按钮
* **GestureDetector：** 如果某个Widget想要响应点击等交互事件，即使它本身不具备交互功能，只要用此Widget包括就可以了

**示例：**
[![效果](https://upload-images.jianshu.io/upload_images/2334192-c74109cf6db91e49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://imgtu.com/i/W0q3Ss)
上面这样的一个布局，觉得是怎样实现的?

[![结构](https://upload-images.jianshu.io/upload_images/2334192-a4ad346ecbc02782.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://imgtu.com/i/W0qs61)
结构是这个样子的
![层级](https://upload-images.jianshu.io/upload_images/2334192-68609f3a7c8cd261.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

层级是这个样子的

## 响应式
* 在iOS中，iOS的框架本身是不支持响应式编程的，如果需要，则要借助第三方框架，比如OC的RAC，Swift的RxSwift等，这些框架由于不是官方提供，只是社区为了实现响应式而基于官方框架开发的应用框架，有相当高的额外学习成本

* 而Flutter本身就是支持响应式的，这为我们的开发提供的很大的便利

## 原生交互
通过前面的介绍我么你知道Flutter本身是一个UI框架，如果涉及到一些非UI的比如相机拍照，必须要和Native交互调用硬件能力，那我们就必须和Native进行通信，Flutter中提供了三种通信方式
1. **MethodChannel：** Flutter 与 Native 端相互调用，调用后可以返回结果，可以 Native 端主动调用，也可以Flutter主动调用，属于双向通信。此方式为最常用的方式， Native 端调用需要在主线程中执行。

2. **BasicMessageChannel：** 用于使用指定的编解码器对消息进行编码和解码，属于双向通信，可以 Native 端主动调用，也可以Flutter主动调用。
3. **EventChannel：** 用于数据流（event streams）的通信， Native 端主动发送数据给 Flutter，通常用于状态的监听，比如网络变化、传感器数据等。

**MethodChannel示例：**
Flutter端
```dart
//Flutter端 初始化消息通道
  static const MethodChannel _channel = const MethodChannel('com.methodChannel');
  
  //Flutter调用
  final Map test = await _channel.invokeMethod('methodChannelSend');
```
iOS端
```swift
//swift端 初始化消息通道
    let channel = FlutterMethodChannel(name: "com.methodChannel", binaryMessenger: registrar.messenger())
    
//swift端 接收消息
channel.setMethodCallHandler { [weak vc](call,  result) in
            if call.method == "methodChannelSend" {
                print("flutter给native发消息\(call.arguments ?? (Any).self)")
            }
        }
```

## 库和插件
Flutter是一个UI框架，但是一个完整的App不只有UI展示，还有其他的能力，比如上边提到的硬件调用，录音，定位，网络请求，状态管理，数据持久化等等...这些能力中Flutter本身是不提供支持的，但是我们我们可以借助响应的库来实现，首先说一下库和插件的关系
* 库是提供特定能力的开发工具包，可以是官方提供，也可以是三方提供

* 插件是特殊的库，举个例子，比如仅提供iOS和Android调用的库，要知道Flutter支持的平台不止这两个

引入库和插件需要在Flutter项目中的`pubspec.yaml`中进行配置，下面提供一个我的配置截图：
[![W0LKnx.png](https://upload-images.jianshu.io/upload_images/2334192-71d51f9ca94b6d32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://imgtu.com/i/W0LKnx)

这里要注意`dev_dependencies`和`dependencies`两者的区别，前者指的是，只在开发阶段生效，不参与编译的库，后者指的是，参与编译，运行需要依赖的库，他们是有区别的，举个例子`build_runner`,`json_serializable`配合使用的话，可以通过注解，结合如下命令，生成额外的`xxx.g.dart`文件来帮助我们快速的生成JSON转Model方法，但是`xxx.g.dart`这个文件只在开发过程中生成和更新，这个库的作用也仅仅是生产这个文件，所以`build_runner`它本身不参与编译
```
flutter packages pub run build_runner build
```
虽然都写在`dependencies`中项目也可以正常运行，但是这样的话打出来的包就会变得大一点，可以，但没必要

下面举例一些开发常用的库:
* `dio` 三方网络库
* `webview_flutter`官方维护的webview库
* `cached_network_image`图片加载库
* `sqflite`基于sqlite的轻量级数据库框架
* `pull_to_refresh`下拉刷新上拉加载
* `shared_preferences`本地数据存储，适合少量数据



## 图片资源
* 我们在开发过程中不免要导入一些静态资源，比如图片，文本文件，音频文件等，我目前只用到了图片资源这里也只介绍图片资源的导入和使用，下面看一下我的图片引入截图，同样是在`pubspec.yaml`

[![W0LRuq.png](https://upload-images.jianshu.io/upload_images/2334192-21710e0666ce8475.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](https://imgtu.com/i/W0LRuq)
注意`pubspec.yaml`文件有着严格的格式要求，对齐有问题都会导致你导入失败，我当时就吃了系统模板的亏
## 混编
以上介绍的都是纯Flutter项目，什么意思，就是说整个项目的主体是Flutter编写，可能部分功能涉及到和Native交互，但是实际上，很多成熟的项目想要体验Flutter，但是又不敢将整个项目使用Flutter重构，毕竟这中间是有很大风险的，所以我们可以在原生项目中，引入Futter模块，将Native中的某个小模块用Flutter开发来进行体验，这样做的风险我们是可以接受的，下面举一个iOS和Flutter混编并用cocoapods引入的示例
* 假设`fluttermoduledemo`是一个iOS原生项目

```rb
//进入到fluttermoduledemo目录的上一级，执行如下命令
flutter create --template module flutter_module
```
* 其中`flutter_module`为需要导入iOS的Flutter模块名称，这个模块的目录结构和纯Fluter项目的结构几乎一样，只不过有几个文件夹变成了隐藏文件夹，编写Flutter代码和在纯Flutter项目中没有区别

* 下面来看一下iOS项目如何引入

```rb
//在iOS项目的Podfile文件中添加如下配置，其中第一行最后的flutter_module，就是上一步创建的Flutter模块的名称
  flutter_application_path = '../flutter_module'
  load File.join(flutter_application_path,".ios","Flutter","podhelper.rb")
  install_all_flutter_pods(flutter_application_path);
```
* iOS使用

```swift
    //在Delegate中初始化Flutter引擎
    lazy var flutterEngine = FlutterEngine(name: "ZMEngine")

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        flutterEngine.run()
        GeneratedPluginRegistrant.register(with: self.flutterEngine);

        return true
    }
    
    //在需要的地方，跳转Flutter模块
   let vc = FlutterViewController(engine:self.flutterEngine, nibName: nil, bundle: nil) 
   self.present(vc,animated: true, completion: nil);

```


## 未完内容
* 另外还有一些内容也很重要，但是我这里没有展开说的原因是因为，我现在自己也没有搞得很明白，仅仅处于会用的阶段，比如`Future`涉及到Dart的线程机制，比如`状态管理`等，我听过一句话`唯有深入，才能浅出`，而现今我还没有非常深入，所以干脆就不讲，以免误人子弟，只做一个印象级概括


## 学习参考文档
[Flutter学习文档](https://flutterchina.club)
[Dart学习文档](https://dart.cn/samples)
[Flutter Package](https://pub.flutter-io.cn)


## 结尾
这个文档只是我去学Flutter后遇到的一些问题的理解，以及对Flutter整个框架粗略的概括，并非完整教程，类似于你去看电视剧，你需要一集一集的追下去，然后直到结局才能窥见全貌，而这个文档相当于一种剧透，或者说剧情的概括，看完之后脑海中与一个大体的轮廓，这个Flutter是个什么东西，然后你可以去详细的，认真的跟着教程去学习，丰富轮廓内的内容。
