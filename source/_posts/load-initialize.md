---
title: load和initialize区别
tags: iOS
categories: 开发
abbrlink: a696ae61
date: 2022-04-29 09:47:41
---
[TOC]

最近在群里突然有群友问load和initialize区别，我心想这不简单吗，脑海中瞬间冒出两条，然后就想不起来了，然后看到群友七嘴八舌一人一个说法，我就寻思重新回顾下这个知识点，遂做个回顾。

这篇文章是我结合网上资料，官方文档，和runtime源码整理得出的，在说明结论时我会在结论前附带官方文档解释并会在（）附带自己的理解，如果理解有误还请指正；你也可以写个demo去验证，我顺手在项目中验证了下所以就不贴代码了。
### 调用方式
> - OBJC_EXPORT void _objc_load_image(HMODULE image, header_info *hinfo)
{
    prepare_load_methods(hinfo);
    call_load_methods();
}
可以下载源码(objc4-818.2源码)点`objc-os`文件进去查看`_objc_load_image -> call_load_methods -> call_load_methods -> call_class_loads -> (*load_method(cls,@selector(load));`

- load通过函数地址调用
> - The runtime sends initialize to each class in a program just before the class, or any class that inherits from it, is sent its first message from within the program. Superclasses receive this message before their subclasses.
- initialize通过objc_msgSend（runtime消息发送）调用

### 调用时机
> - Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.
- load是runtime加载类、分类的时候调用(只会调用一次)
> - Initializes the class before it receives its first message.
> - The superclass implementation may be called multiple times if subclasses do not implement initialize—the runtime will call the inherited implementation—or if subclasses explicitly call [super initialize]. 
- initialize是类第一次接收到消息的时候调用（可以在main前也可以在后，看什么时候第一次接收消息）, 每一个类只会initialize一次(如果子类没有实现initialize方法, 会调用父类的initialize方法, 所以父类的initialize方法可能会调用多次)

**以上的调用一次指的是程序生命周期内默认只调用一次，但是如果你手动调用的话还是会调用多次**

### 调用顺序
> - A class’s +load method is called after all of its superclasses’ +load methods.
> - A category +load method is called after the class’s own +load method.
- load
  1. 先调用类的load, 再调用分类的load
  2. 先调用父类的load，再调用子类的load
  3. 先编译的类（分类），先调用
 > - The runtime sends initialize to each class in a program just before the class, or any class that inherits from it, is sent its first message from within the program. Superclasses receive this message before their subclasses.
 > - The superclass implementation may be called multiple times if subclasses do not implement initialize—the runtime will call the inherited implementation—or if subclasses explicitly call [super initialize].
- initialize
    1. 先调用父类，再调用子类（即使你先给子类发消息，子类默认也会先调用父类）
    2. 如果子类没实现initialize，会向上调用父类
    3. 如果分类实现initialize，则会覆盖本类的initialize（因为是objc_msgSend消息发送调用，分类方法加载时会插入在本类前）
    
 ### 资料   
- 如果你不清楚怎么查看官方文档:
`  Xcode -> Help -> Developer Documentation`
或者你也可以下载`Dash`查阅
-  如果你想查看源码：
 [runtime源码下载](https://opensource.apple.com/tarballs/objc4/)
