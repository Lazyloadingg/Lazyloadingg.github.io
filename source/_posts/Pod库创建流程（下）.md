---
title: Pod库创建流程（下）
tags: Cocoapods
categories: 开发
abbrlink: fcb7297c
date: 2019-07-28 15:41:41
---
<meta name="referrer" content="no-referrer" />
 [TOC]
 
# 创建pod库流程(下)
前边讲了pod库默认工程模板的创建，文件添加存放路径，代码编写位置等基本信息，今天简单说下spec文件的编写及推送。

## 第一步
首先我们打开pod库的spec文件,以`DYTest`项目为例：
![屏幕快照 2019-07-28 上午11.55.39](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8A%E5%8D%8811.55.39.png)
其中红色箭头所指的就是项目的spec文件，红色方框内就是pod默认的spec文件的内容，里边有很多字段分别对应不同的配置信息，但一般来说我们**只需要**配置其中的几项就可以，其他的可以暂时删掉。

删完之后大概如下图：

```ruby
Pod::Spec.new do |s|
  s.name             = 'DYTest'
  s.version          = '0.1.0'
  s.summary          = 'A short description of DYTest.'
  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/lazyloading@163.com/DYTest'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'lazyloading@163.com' => 'lazyloading@163.com' }
  s.source           = { :git => 'https://github.com/lazyloading@163.com/DYTest.git', :tag => s.version.to_s }
  s.ios.deployment_target = '8.0'
  s.source_files = 'DYTest/Classes/**/*'

  # s.dependency 'AFNetworking', '~> 2.3'
end

```
其中
- s.name : 你所创建的库的名字，比如Masonry，AFNNetworking等等

- s.version ： 库的版本号
- s.summary ： 简短的描述信息
- s.description ： 比简短描述长一点的描述信息
- s.homepage ： pod库远程仓库主页地址 我们此次的示例工程的主页就是 `http://git.zhcs.com/iOS_Group/DYTest` 
- s.license ： 遵守的协议
- s.author ： 创建人
- s.ios.deployment_target ： 支持的系统版本版本此次示例工程支持到`iOS 8.0`
- s.source ： 你的pod库的远程仓库路径，我们此次的示例工程的路径就是`http://192.168.120.30/iOS_Group/DYTest.git`
- s.source_files = 要加入库中的代码在工程中的路径，此次示例工程编写的代码就在之前强调的`'DYTest/Classes/**/*'`路径下

## 第二步

当库每个版本代码编写完成后我们需要将pod库代码提交到对应的远程仓库，同时对应的spec文件也要推送到spec文件远程仓库。

第一步我们完成了spec文件的编写，第二步我们就要推送spec文件，此次示例工程我们的spec文件远程仓库就是我们前文所创建的`http://192.168.120.30/iOS_Group/DYTestSpecs.git`

我们先对spec文件做一个本地校验看看是否有配置错误：

进入spec文件所在目录
![屏幕快照 2019-07-28 下午2.34.53](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%882.34.53.png)

执行命令
```
pod lib lint 
```
当你在spec文件所在目录并且只有一个spec文件时候你可以这么写，否则你需要指定你要校验的spec文件用如下写法
```
pod lib lint DYTest.podspec
```
执行结果如下：

![屏幕快照 2019-07-28 下午2.39.19](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%882.39.19.png)

第一次执行`pod lib lint `未成功，报出了⚠️其实这些警告可以忽略，我们只需要加上`--allow-warnings`参数，可以看到我们第二次执行成功了，如果你想看校验过程还可以在后边添加`--verbose`参数

但是只是做本地校验还不够，毕竟我们的库并不是自己玩玩，是需要让其他人也用的，这就需要推送的远端，远端推送之前就需要进行远端校验，不仅查看语法是否正确，也查看你在远端的代码版本和spec文件中的是否匹配。

我们先给代码打个`tag`,然后把tag推送到远程，tag版本需要和spec文件中的`s.version`一致，此处前边我们spec文件中`s.version`写的是`0.1.0`

![屏幕快照 2019-07-28 下午2.51.27](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%882.51.27.png)
![屏幕快照 2019-07-28 下午2.51.40](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%882.51.40.png)

至此tag已经打完并推送成功，接下来就是对spec文件进行远程校验
操作和本地校验类似只不过命令不同，执行命令
```
pod spec lint --allow-warnings
```
结果如下
![屏幕快照 2019-07-28 下午2.56.27](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%882.56.27.png)

或者我们很自信spec文件绝对没问题，这时候也可以不校验直接推
```
pod repo push DYTestSpecs *.podspec --sources=http://192.168.120.30/iOS_Group/DYTestSpecs.git,https://github.com/CocoaPods/Specs.git --use-libraries --allow-warnings --verbose
```
结果如下:
![屏幕快照 2019-07-28 下午3.03.05](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%883.03.05.png)


一次成功，完美，这里忍不住给自己鼓掌，整个操作行云流水一气呵成如德芙版丝滑中途没出现一点意外和错误，嘴角不自觉扬起了迷人的弧度
![2334192-98329dbf4ae5a5bd](media/15642856190114/2334192-98329dbf4ae5a5bd.jpg)


不过这句命令很长有必要给大家解释下每一部分的含义不然你只会`知其然不知其所以然`，看看看看，出口成章，小伙子学着点以后要多读书啊。
其中`DYTestSpecs`是我们第一篇里讲的，添加进本地repo目录下的示例工程对应的spec工程。

`*.podspec`中*指通配符，这么写指以`.podspec`为后缀的文件,你也可以指定对应的spec文件，比如我们的示例工程就可以写成 `DYTest.podspec`。

`--sources`指的是要将本地repo目录下`DYTestSpecs`工程推送的远端路径，如果你只想公司内部使用就可以不写后边的`https://github.com/CocoaPods/Specs.git`

## 第三步
创建，编写，校验，推送都完成了，接下来就是使用了，我们先`search`试一下看能不能搜到我们的库
```
pod search DYTest
```

![屏幕快照 2019-07-28 下午3.27.17](media/15642856190114/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-07-28%20%E4%B8%8B%E5%8D%883.27.17.png)

嗯，放心了，搜索的到，那么，接下来，就不用我多说了吧，开始愉快的使用吧~



**后记：**这两篇文章主要说的是私有pod库的创建，编写及推送流程，实际具体使用过程中可能还会遇到其他问题导致你用的头大，比如操作顺序出错导致配置了不可逆的错误环境，操作的天时不对，坐的位置风水不好等等，总之就是很多不可控，玄学问题，这个在这篇文章就不一一细说，后边有问题可以直接问，或者有时间转门写一下。

