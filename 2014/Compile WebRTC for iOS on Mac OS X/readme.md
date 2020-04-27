---
layout: post
title: "Compile WebRTC for iOS on Mac OS X"
date: 2014-03-25 14:40:18 +0800
categories: ["技术"]
tags: ["2014", "iOS"]
comments: true
---

## 我的编译环境：

Mac OS X 10.9 + Command Line Tools（XCode 4.6.3）

## 编译步骤：
#### 一、安装依赖工具

1. 安装 git、svn（不过，一般Xcode自带安装好了）

2. 安装depot_tools  

	a. 在指定目录下执行（我选择在 `/Developer` ）：  
  
   ``` shell
	svn co http://src.chromium.org/svn/trunk/tools/depot_tools
   ```
	
	b. 把depot_tools 添加到 PATH（环境变量）中：
	
    ``` shell
	sudo vim /private/etc/paths
    ```
	
    然后往打开的文件添加 `depot_tools` 的路径（我添加的是：`/Developer/depot_tools ）

#### 二、下载源代码以及生成项目文件 
1.下载源代码

```shell
mkdir webrtc_src
cd webrtc_src
gclient config http://webrtc.googlecode.com/svn/trunk
giclaient sync —force	//如果中途中断了，再次执行该命令
```

2.生成 iOS 项目文件

```shell
cd trunk
export GYP_GENERATORS="xcode"	//设置环境配置
./build/gyp_chromium --depth=.  -DOS=ios -Dtarget_arch=arm -Dinclude_tests=0 -Denable_protobuf=0 all.gyp
```

（其它生成例子：`./build/gyp_chromium --depth=.  -DOS=ios -Dtarget_arch=arm -Dinclude_tests=0 -Denable_protobuf=0 -Denable_video=1 webrtc/webrtc.gyp`）

<!-- more -->

## 其它知识：

1. 关于 gclient 

	http://blog.csdn.net/doon/article/details/9287693  

2. 关于Configuring gyp

	https://code.google.com/p/chromium/wiki/LinuxBuildInstructions

	>Configuring gyp
	>
	>See Configuring the Build for details; most often you'll be changing the GYP_DEFINES options, which is discussed here.
	>
	>gyp supports a minimal amount of build configuration via the -D flag.
	>
	>`build/gyp_chromium -Dflag1=value1 -Dflag2=value2`
	>
	>You can store these in the GYP_DEFINES environment variable, separating flags with spaces, as in:
	>
	>`export GYP_DEFINES="flag1=value1 flag2=value2"`



## 参考资料：  

1. http://www.cnblogs.com/ProbeStar/p/3411510.html
2. https://bitbucket.org/dashboard/overview

