---
layout: post
title: "通过Xcode在命令行进行编译的常见问题"
date: 2016-08-17 17:15:49 +0800
categories:
---
# 通过Xcode在命令行进行编译的常见问题

原文是：Technical Note TN2339 Building from the Command Line with Xcode FAQ

## 命令行工具包是什么？

命令行工具包是一个轻小的、可以与Xcode分开下载的、允许你在OS X上进行命令行开发的工具包。它由两部分组成：OS X SDK和类似Clang等安装在/usr/bin下的命令行工具。

## 在OS X 10.9上的Xcode上不再支持下载命令行工具了，那该如何下载他们呢？

在OS X 10.9上，Xcode首选项的下载面板中不再支持命令行工具的下载了。可以用如下的几个方法在你的系统中安装命令行工具：

- 使用Xcode

   如果你的机器上安装了Xcode，那就没必要再安装命令行工具了，Xcode里面已经捆绑了所有的命令行工具。OS X 10.9包含了可执行文件的垫片（shims）和包装（warp）。垫片安装于/usr/bin，可以转换一些/usr/bin下的工具到对应的Xcode中自带的工具。xcrun就是这样的一个垫片，它可以允许你在命令行中找到或者运行任何Xcode中包含的工具。
          
   在终端中运行dwarfdump
   
   ```
   $ xcrun dwarfdump —uuid MySample.app/MySample
   ```

- 使用终端应用

	你可以通过运行`xcode-select —-install`命令或尝试使用别的一个命令（如git）来安装。
          
    OS X自带`xcode-selecta`，这是一个安装于`/usr/bin`中的命令行工具。它允许你管理Xcode或其他的BSD开发工具的开发者路径。
          
    当你尝试使用这几个命令时，OS X会弹出一个对话框，选择Install来在你的系统中的`/Library/Developer/CommandLineTools`初始化命令行工具。

- 使用开发者网站的“Download for Apple Developers”页面

	可以在[Download for Apple Developers](https://developer.apple.com/downloads/)网页下载命令行工具。登录上你的Apple ID，然后下载命令行工具包。

## 如果卸载我的命令行工具？

- Xcode包括所有的命令行工具。如果你的系统中安装了Xcode，删除它就可以了。
- 如果你的工具是单独下载的，它们会在你的系统的/Library/Developer/CommandLineTools。删除这个文件夹来卸载它们。

## 电脑上安装了多个版本的Xcode，当前的命令行工具使用的是哪一个呢？

可以在终端中运行下面的命令来确定命令行使用的Xcode的版本：

```
$ xcode-select —-print-path
```

## 如何为命令行工具设置默认使用的Xcode版本？

在终端运行如下的命令：

```
$ sudo xcode-select -switch <path/to/>Xcode.app
```

将<path/to/>替换为你想要使用的Xcode.app包的路径。

## 如何在命令行中编译我的工程？

xcodebuild是一个命令行工具允许你从命令行对你的Xcode工程或工作空间执行编译，查询，分析，测试和打包等操作。它可以操作一个或多个你工程或工作空间中包含的target或scheme。xcodebuild为这些操作提供了一些选项，参照man page。xcodebuild命令的输出保存在你的Xcode的Locations首选项面板中设置的位置。

下面有几个xcodebuild命令的使用。使用下面命令时确保先将路径切换到包含你的project或workspace文件的路径。

- 在终端运行如下的命令，列出工程中所有的target，编译配置和scheme。

	```
	$ xcodebuild -list -project <your_project_name>.xcodeproj
	```

	其中的 `<your_project_name>` 是你的工程的名字。

- 编译工程中的一个scheme，在终端运行下面的命令：

	```
	$ xcodebuild -scheme <your_scheme_name> build
	```

	其中 `<your_scheme_name>` 和 `build` 分别对应要编译的scheme的名称和要在该scheme上要执行的操作。

	注意：xcodebuild 支持多个构建操作如 build、analyze 和 archive 可以在你的target或scheme上执行，如果没有指定操作，默认为build。

- 使用一个配置文件来编译你的target，在终端运行如下命令：

```
$ xcodebuild -target <your_target_name> -xcconfig <your_configuration_file>.xcconfig
```

其中 `<your_target_name>` 和 `<your_configuration_file>` 分别对应你要编译的target的名字和你的配置文件的名字。


- 要修改xcodebuild命令的输出地址，使用SYMROOT（编译结果路径）和DSTROOT（可安装产品路径）编译设置分别指定了你的debug products和.dSYM文件和一个发布位置。参见Xcode Build Setting Reference查找更多信息。

	两个示例：
	
{% highlight PowerShell %}
// 为MyiOSApp的debug版本设置位置
$ xcodebuild -scheme MyiOSApp SYMROOT="/Users/username/DebugLocation"
// 为MyiOSApp设置archive位置
$ xcodebuild -scheme MyiOSApp DSTROOT="/Users/username/ReleaseLocation" archive
{% endhighlight %}

## 应用有多套编译配置，如何为xcodebuild设置默认的编译配置？

Xcode的工程的info面板中的Configuration区有一个弹出菜单，在这里可以设置使用xcodebuild编译一个target时的默认编译配置。

## 如何使用命令行运行 OS X 和 iOS 的单元测试

要在命令行中运行单元测试，在终端中执行如下的命令：

```
xcodebuild test -scheme <your_scheme_name> -destination destinationspecifier

```

xcodebuild 使用 test 编译操作来运行单元测试。这个编译操作要求指定一个 scheme 和 destination。-destination 选项允许你为你的单元测试指定一个目的地。参数 -destinationspecifier 描述了使用真机、模拟器或Mac来作为目的地。它由一些逗号分隔的 key=value 对的集合组成，根据要用真机、模拟器或Mac。

更多关于 scheme 和 destination 相关的信息见 Xcode Scheme 和 Choose a Destination to Run Your App。

* 对于 OS X 应用，-destinationspecifier 支持平台和构建key见下表。执行 OS X中的单元测试时，两个key都是必需的。


key | Description | Value
----|-------------|------
platform | 你的单元测试支持的目标平台 | OS X
arch | 运行单元测试使用的构建平台 | i386 或 x86_64

实例：测试 MyMacApp scheme 在64位 OS X上

```
xcodebuild test -scheme MyMacApp -destination 'platform=OS X, arch=x86_64'
```

* 对于iOS 应用，-destinationspecifier支持platform、name和id等关键字如下表：

Key | Description | Value
----|-------------|------
platform | 单元测试支持的目标平台 | iOS
name | 单元测试要用的设备的全名 | 设备在Xcode中显示的名称
id | 单元测试使用的设备的标识符 | 

name 和 id 关键字二选一和platform构成必要的关键字：
示例：通过给定的 id 在设备上运行 MyiOSApp scheme 的单元测试

```
xcodebuild test -scheme MyiOSApp -destination 'platform=iOS,id=998058a1c30d845d0dcec81cd6b901650a0d701c'
```

示例：在一台iPod touch上测试MyiOSApp Scheme

```
xcodebuild test -scheme MyiOSApp -destination 'platform=iOS,name=iPod touch'
```

* 对于iOS模拟器应用，-destinationspecifier支持platform、name和 OS 等关键字如下：

Key | Description | Value 
----|-------------|------
platform | 单元测试支持的目标平台 | iOS Simulator
name | 单元测试运行的目标iOS模拟器的全称 | iPhone Retina (3.5-inch), iPhone Retina (4-inch), iPhone Retina (4-inch 64-bit), iPad, iPad Retina, or iPad Retina (64-bit.
OS | iOS模拟器的版本如 7.1 | 支持的最新的iOS版本

platform和name是必选的关键字，OS可选。

示例：在iPad模拟器上运行MyiOSApp scheme的单元测试

```
xcodebuild test -scheme MyiOSApp -destination 'platform=iOS Simulator,name=iPad
```

示例：在iOS 7.1 Retina (4-inch 64-bit)模拟器运行MyiOSApp scheme的单元测试

```
xcodebuild test -scheme MyiOSApp -destination 'platform=iOS Simulator,name=iPhone Retina (4-inch 64-bit),OS=7.1'
```

-destination选项也允许在多个目标平台上运行同一个单元测试。只需要在xcodebuild命令后面多加几次就可以了，如下：

```
xcodebuild test -scheme MyiOSApp -destination 'platform=iOS Simulator,name=iPhone Retina(4-inch 64-bit),OS=7.1' -destination 'platform=iOS,name=iPod touch'
```

注意，xcodebuild会按照循序一个一个运行单元测试，上面的例子中先在模拟器上测试，然后在iPod touch上测试。
