---
title: OCLint静态代码校验初探
date: 2019-08-19 21:04:41
tags: OCLint
categories: 开发
---
# OCLint静态代码校验初探

**“OCLint是一个能够让我们的OC，C，C++代码变得更加优雅的检测分析工具--鲁迅”**


#### OCLint的目的
&emsp;&emsp; 在开发过程中，特别是团队协作开发中，规范的重要性不言而喻，他可以降低沟通成本，增加代码可读性与可维护性提高可靠性与健壮性，作为一名优秀的开发人员，我们应该不断完善并严格遵守相关规范，但是实际情况是，由于我们的疏忽，或者工期的紧张等因素，导致在某些时候开发过程中变得随心所欲，代码变得为所欲为，进而对后期的维护造成不良影响，甚至原地crash>_<!，而`OCLint`可以帮助我们来检查代码是否遵守了某些规范，是否存在一些潜在的问题，降低review成本。

#### OCLint能检测哪些问题
附官方文档[OCLint](http://oclint.org)
* 代码长度，过长或过短，包含方法，变量等
* 未使用的代码，包括变量和方法
* 代码复杂度，多重循环以及多重判断等
* 语法错误

#### OCLint安装
&emsp;&emsp; OCLint有多种安装方式，此处采取`Homebrew`安装，so 默认你的电脑已经安装了`Homebrew`,如果没安装，请先去安装[Homebrew](https://brew.sh/index_zh-cn.html),一次就够，你会爱上它的😊

```sh
brew tap oclint/formulae
brew install oclint
```
&emsp;&emsp;为什么要执行第一句？因为要先安装`OCLint`的依赖，否则会报错，安装结束执行：
```sh
oclint --version
```
&emsp;&emsp;如果出现类似下面的信息，说明安装成功
```sh
LLVM (http://llvm.org/):
LLVM version 5.0.0svn-r313528
Optimized build.
Default target: x86_64-apple-darwin18.6.0
Host CPU: skylake

OCLint (http://oclint.org/):
OCLint version 0.13.
Built Sep 18 2017 (08:58:40).
```
&emsp;&emsp;紧接着安装`xcpretty`这个东西是什么呢？它可以格式化`xcodebuild`的输出，增加可读性并可以生成报告,执行下面命令安装

```sh
sudo gem install xcpretty
```
&emsp;&emsp; 安装结束检查是否安装成功
```sh
xcpretty -v
```
&emsp;&emsp; 如果出现类似下面的版本号信息说明安装成功
```sh
0.3.0
```

#### OCLint使用
&emsp;&emsp; OCLint作为静态代码检测工具是可以直接在`Xcode`中使用的，也可以在终端进行操作，本文介绍的是终端操作方式，`Xcode`使用后续补充；首先进入项目根目录执行如下命令：
```sh
xcodebuild clean 
xcodebuild analyze | tee xcodebuild.log 
```
&emsp;&emsp;这一步会对项目进行分析并会在根目录生成一个`build`目录，并将分析日志输出在`xcodebuild.log`文件中

```sh
xcodebuild |xcpretty -r json-compilation-database -o compile_commands.json 
```
&emsp;&emsp;这一步是将编译结果输出在`compile_commands.json`文件中,这里有一个点要注意的就是，如果你已经编译过并将结果输出在`compile_commands.json`中，那么再次编译时已编译过的内容是不会被覆盖的，如果希望每次都重头操作，那么可以使用`xcodebuild clean`命令清除缓存，这也是第一步执行此命令的原因。

&emsp;&emsp;最后一步是将分析结果生成报告，这中间就会用到各种校验规则了，OCLint默认有些规则，包含了可能出现的大部分场景，当然也可以根据OCLint提供的方法自定义规则,执行如下命令生成报告

```sh
oclint-json-compilation-database -e Pods -- -report-type html 
-rc CYCLOMATIC_COMPLEXITY=5 
-rc TOO_MANY_PARAMETERS=8
-rc NESTED_BLOCK_DEPTH=5
-rc LONG_LINE=200 
-rc NCSS_METHOD=50
-rc LONG_VARIABLE_NAME=30 
-rc SHORT_VARIABLE_NAME=2
-rc LONG_CLASS=1500
-rc LONG_METHOD=150
-disable-rule ShortVariableName 
-max-priority-1=10000 
-max-priority-2=10000 
-max-priority-3=10000 > report.html
```

&emsp;&emsp;最终会在根目录生成一个`report.html`文件，点击打开可查看分析结果，大概是下面这个样子
![](https://s1.ax1x.com/2020/03/19/8yHn8U.md.png)
&emsp;&emsp;另外上述出现的操作命令为了简便写了一个[脚本](https://github.com/shengguangdaren/OCLint.git)，仅做参考，需要的自取
&emsp;&emsp;因为我此处只是为了展示，所以创建了一个空项目，并没有写内容，所以分析报告一片空白😂😂😂

&emsp;&emsp;本篇文章只做OCLint使用的初步介绍，实际上我在安装使用的过程中遇到了很多必然或偶然的坑，另外对于`OCLint`也并没有做详细的介绍，比如`OCLint`默认规则的介绍，如何自定义规则，相关套件的作用，以及`OCLint`的缺点等等，随后有时间我会一一整理补充上来。嗯，就酱~








