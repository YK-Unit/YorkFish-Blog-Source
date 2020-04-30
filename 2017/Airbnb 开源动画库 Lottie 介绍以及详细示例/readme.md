---
title: Airbnb 开源动画库 Lottie 介绍以及详细示例
date: 2017-08-21 12:34:56
categories: ["技术"]
tags: ["2017", "教程", "iOS"]
comments: true
---

## 前言

该文章主要介绍了 Lottie是什么，如何为 Lottie 制作动画，以及 Lottie的应用场景。适合设计师和开发者阅读以及结对实践。

<!-- more -->


## Lottie 介绍

Lottie 是 Airbnb 开源的一个动画渲染库，同时支持 Android、iOS、React Native 平台。Lottie 目前只支持渲染播放 After Effects 动画。 [Lottie](http://airbnb.design/lottie/) 使用从 [bodymovin](https://github.com/bodymovin/bodymovin) (开源的 After Effects 插件)导出的json数据来作为动画数据。所以从动画制作到动画使用的整个工作流程如下：

![此图引用自http://cdn.trojx.me/blog_pic/lottie_sum.png](http://upload-images.jianshu.io/upload_images/1655773-f0d4b7bd03ad3dcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 设计师使用 After Effects 制作动画，并导出json文件给开发者
2. 各端的开发者通过 Lottie 渲染播放动画

截止目前，各平台的 Lottie 支持的 After Effects 特性可从下面网页获得：
> http://airbnb.io/lottie/supported-features.html

**所以，设计师在使用 After Effects 制作动画时，建议先预览上述网页，以知道应该使用哪些特性制作动画，因为若使用 Lottie 还不支持的特性，如3D图层，则 Lottie 会无法正确渲染。**

为了推广Lottie，Airbnb 还建立了一个Lottie动画网站，供网友分享自己制作的动画。网站地址为：
>https://www.lottiefiles.com


## 为 Lottie 制作动画

为 Lottie 制作动画，需要： After Effects + bodymovin。After Effects制作好动画后，通过插件 bodymovin 导出一份 json文件，然后使用 Lottie 进行渲染播放。下面将会介绍如何安装该插件以及如何导出json文件。

#### 安装 After Effects
PS： 已经安装好 After Effects 的童鞋可以忽略此环节

After Effects 可以从Adobe官网下载安装试用，其目前售价为：3499￥/年，相对来说还是很贵的。对于负担不起的童鞋来说，也可以考虑破解版本。以下是Mac 的破解版本的下载链接：

 >百度云盘：https://pan.baidu.com/s/1eRMCL26 
 >提取密码：xyu5

下载的文件夹中包含安装文件`After Effects CC 2017.dmg`以及破解文件`Adobe Zii cc2017.app`压缩包。安装好 After Effects 后，解压运行Adobe Zii cc2017.app 即可免费使用 After Effects 。但是，建议负担的起的童鞋还是购买正版服务，始终可以得到各种升级服务。

#### 安装 After Effects 插件 bodymovin
**1. 下载插件 `bodymovin.zxp`**
   - 下载 [bodymovin压缩文件](https://codeload.github.com/bodymovin/bodymovin/zip/master)
   - 解压文件，在目录 '/build/extension' 找到 `bodymovin.zxp`

**2. 安装插件**
   - 下载 After Effects 插件安装器 [ZXP Installer](http://aescripts.com/learn/zxp-installer/)（有 Windows 和 Mac 版本）
   - 运行 `ZXP Installer`，按照指示拖动`bodymovin.zxp` 到其窗口，即可安装完成
          
       ![拖动安装bodymovin.zxp.png](http://upload-images.jianshu.io/upload_images/1655773-dfef77e49b677d90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
        
     安装成功后，如图所示：

  ![bodymovin.zxp安装成功.png](http://upload-images.jianshu.io/upload_images/1655773-e2560d76514b1c8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3. 重启 After Effects，然后修改 AE 的设置，在 'After Effects CC -> Preferences ->  General' 中打开
 'Allow Scripts to Write Files and Access Network'**
    
   ![打开
 'Allow Scripts to Write Files and Access Network'.png](http://upload-images.jianshu.io/upload_images/1655773-9d5b0b3fc25eeb37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

   现在可以开始制作你的动画了，制作完毕后，需要使用 bodymovin 时，前往 'window -> extensions' 即可找到 bodymovin：

   ![bodymovin.png](http://upload-images.jianshu.io/upload_images/1655773-713b81ae8fa8d7b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 使用 After Effects 制作动画

![此处请开始你的表演~](http://upload-images.jianshu.io/upload_images/1655773-d40516b8747c9a27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 使用 bodymovin 导出 json文件
当动画制作完毕后，运行 bodymovin，选择你要导出的动画，以及保存json文件的目录，点击 'Render' 即可导出，具体流程如图所示：


![导出 json文件流程.png](http://upload-images.jianshu.io/upload_images/1655773-43186d06673eb373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 在线预览动画效果
制作好 After Effects 动画，导出json文件，当然要验证一下 Lottie 能否正确渲染播放了。

Airbnb 提供了  [iOS APP](https://www.lottiefiles.com/ios) 、[ Android APP](https://www.lottiefiles.com/android) 以及 [Lottie 动画在线预览网站](https://www.lottiefiles.com/preview) 供设计师进行动画预览。

在网站预览的话，设计师只要把导出后的 json 文件，拖动到网页的预览框，即可在线看到 Lottie 渲染播放的动画效果。

使用 APP 预览的话，则需要上传 json 文件到服务端，通过链接进行预览。建议设计师上传文件到 Airbnb 建立的分享网站 [lottiefiles.com]( https://www.lottiefiles.com)。


## 哪些场景适宜使用 Lottie？

Lottie 作为一个动画渲染库，在探索过程中，笔者认为其比较适宜解决以下场景的问题：

- 启动(splash)动画：典型场景是APP logo动画的播放
- 上下拉刷新动画：所有APP都必备的功能，利用 Lottie 可以做的更加简单酷炫了
- 加载(loading)动画：典型场景是网络请求的loading动画
- 提示(tips)动画：典型场景是空白页的提示
- 按钮(button)动画：典型场景如switch按钮、编辑按钮、播放按钮等按钮的切换过渡动画
- 礼物(gift)动画：典型场景是直播类APP的高级动画播放
- 视图转场动画

各场景的示例如下：（以iOS平台为例）

![启动(splash)动画.gif](http://upload-images.jianshu.io/upload_images/1655773-cbbf046e5ce137f8.gif?imageMogr2/auto-orient/strip)


![上下拉刷新动画.gif](http://upload-images.jianshu.io/upload_images/1655773-f51232f3f0432910.gif?imageMogr2/auto-orient/strip)


![加载(loading)动画+提示(tips)动画.gif](http://upload-images.jianshu.io/upload_images/1655773-048ca0f8ab4fae95.gif?imageMogr2/auto-orient/strip)


![按钮(button)动画+礼物(gift)动画.gif](http://upload-images.jianshu.io/upload_images/1655773-988f1b7aa35e6bb0.gif?imageMogr2/auto-orient/strip)


![转场动画.gif](http://upload-images.jianshu.io/upload_images/1655773-148e39c8fcf3aa74.gif?imageMogr2/auto-orient/strip)

## 接入 Lottie 

制作好动画，导出json文件后，iOS、Android、React Native的开发者们就可以像使用静态资源一样使用动画了。接入教程可以看官网教程或者 各平台的 Lottie 项目的github：
- [官网教程](http://airbnb.io/lottie/)
- [lottie-ios-git](https://github.com/airbnb/lottie-ios)
- [lottie-android-git](https://github.com/airbnb/lottie-android)
- [lottie-react-native-git](https://github.com/airbnb/lottie-react-native)

> lottie-iOS 的应用示例（包括上述所有例子）可访问：
https://github.com/YK-Unit/LottieExample

## lottie-ios 极速上手手册

#### 安装 Lottie
可通过 Cocoapods 或者 Carthage 导入 Lottie。
- Cocoapods：` pod 'lottie-ios' `
- Carthage： ` github "airbnb/lottie-ios" "master" `

#### 加载 Lottie 动画
Lottie 动画支持从本地或者服务器的json文件加载。

``` objc
//从本地json加载
LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"Lottie"];
//从本地指定的bundle的json加载
LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"Lottie" inBundle:[NSBundle YOUR_BUNDLE]];
//从服务器的json加载
LOTAnimationView *animationView = [[LOTAnimationView alloc] initWithContentsOfURL:[NSURL URLWithString:URL_TO_JSON]];

animationView.frame = CGRectMake(20, 20, 400, 300);
[self.view addSubview:animationView];
```

#### 播放 Lottie 动画
Lottie 动画的播放控制，除了常规的控制，还支持进度播放、帧播放。

- 播放、暂停、停止

	``` objc
	LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"Lottie"];
	
	//从上一次的动画位置开始播放
	[animationView play];
	
	...
	//暂停动画播放
	[animationView pause];
	
	....
	//停止动画播放，此时动画进度重置为0
	[animationView stop];
	```
	
- 控制进度播放：可参考示例的上下拉刷新动画

	``` objc
	LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"RefreshHeaderAnim"];
	
	//直接播放到指定进度
	[animationView playToProgress:0.8 withCompletion:^(BOOL animationFinished) {
	// do something
	}];
	
	//从进度A播放到进度B
	[animationView playFromProgress:0 toProgress:0.8 withCompletion:^(BOOL animationFinished) {
	// do something
	}];
	
	//直接设置当前进度
	animationView.animationProgress = currentProgress;
	```

- 控制帧播放：可参考示例的switch按钮动画

	``` objc
	LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"Switch"];
	
	//直接播放到指定帧
	[animationView playToFrame:@(40) withCompletion:^(BOOL animationFinished) {
	
	 }];
	
	//从A帧播放到B帧
	[animationView playFromFrame:@(20) toFrame:@(40) withCompletion:^(BOOL animationFinished) {
	
	}];
	```
	
- 循环播放动画：可参考示例的Play-Pause按钮动画

	``` objc
	LOTAnimationView *animationView = [LOTAnimationView animationNamed:@"Play-Pause"];
	//设置循环播放
	animationView.loopAnimation = YES;
	//设置自动倒退播放
	animationView.autoReverseAnimation = YES;
	[animationView playFromFrame:@(90) toFrame:@(180) withCompletion:^(BOOL animationFinished) {
	
	}];
	```

- 编辑某帧的动画对象的属性：可参考示例的switch按钮动画

	``` objc
    [self.switchButton setValue:[UIColor orangeColor] forKeypath:@"Background 2.Shape 1.Fill 1.Color" atFrame:@(0)];
    [self.switchButton setValue:[UIColor blueColor] forKeypath:@"Background 2.Shape 1.Fill 1.Color" atFrame:@(13)];	
	```
	
	要修改对象的属性，需要知道属性的路径（Keypath）。获取属性的路径的方法有：

	- 直接打印对象的所有层级属性，从日志中获取：
        `[animationView logHierarchyKeypaths];` 

         ![logHierarchyKeypaths日志.png](http://upload-images.jianshu.io/upload_images/1655773-d64edaa7ec935147.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

	- 通过AE文件获得：`Background 2.Shape 1.Fill 1.Color`
  
        ![Keypath.png](http://upload-images.jianshu.io/upload_images/1655773-313a26f1555c701a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 视图控制器转场动画（View Controller Transitions）：可参考示例的转场动画
  Lottie 提供了 `LOTAnimationTransitionController`生成 `id <UIViewControllerAnimatedTransitioning>` 对象。

   ```objc
  #pragma mark - UIViewControllerTransitioningDelegate
  - (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source {
		
	    LOTAnimationTransitionController *animationController = [[LOTAnimationTransitionController alloc] initWithAnimationNamed:@"vcTransition1"
	                                                                                                              fromLayerNamed:@"outLayer"
	                                                                                                                toLayerNamed:@"inLayer"
	                                                                                                     applyAnimationTransform:NO];
	    return animationController;
	}
		
	- (nullable id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed {
	    LOTAnimationTransitionController *animationController = [[LOTAnimationTransitionController alloc] initWithAnimationNamed:@"vcTransition2"
	                                                                                                              fromLayerNamed:@"outLayer"
	                                                                                                                toLayerNamed:@"inLayer"
	                                                                                                     applyAnimationTransform:NO];
	    return animationController;
	}
		
   ```

#### 添加视图到 Layer 层：可参考“添加 View 到 Layer 示例”
Lottie 除了支持动画播放，还支持添加自定义的视图到指定的 Layer ：

![添加视图到Layer层.gif](http://upload-images.jianshu.io/upload_images/1655773-92ec6921402fa14a.gif?imageMogr2/auto-orient/strip)

```objc
NSArray *layerNames = @[@"Green Solid 1",@"Shape Layer 1",@"Shape Layer 2",@"Shape Layer 3",@"Shape Layer 4"];
for (NSString *layerName in layerNames) {
    CGRect subRectViewFrame = CGRectMake(0, 0, 15, 15);
    UIView *subRectView = [[UIView alloc] initWithFrame:subRectViewFrame];
    subRectViewFrame = [self.rectView convertRect:subRectViewFrame toLayerNamed:layerName];
    subRectView.frame = subRectViewFrame;
    subRectView.backgroundColor = [UIColor whiteColor];
    [self.rectView addSubview:subRectView toLayerNamed:layerName applyTransform:YES];
}
```
在AE中，我们一般会用到2种类型的 Layer 来制作动画：Solid Layer（固态图层）和 Shape Layer（形状图层）。图中，绿色的视图元素就是Solid Layer，蓝色、红色、黄色、棕色的视图元素就是 Shape Layer。

细心的读者可以发现，添加同一个 subRectView 到 不同的 Layer ，subRectView 的绘制位置不一样，这是因为不同类型的 Layer 其坐标系统不一样：

- Solid Layer 的坐标原点在左上角位置，向右是X轴正方向，向下是Y轴正方向

- Shape Layer 的坐标原点由`ShapeLayer-Contents-layer-AnchorPoint`（内容图层的锚点位置）决定，X轴和Y轴的正方向则取决于其`ShapeLayer-Transform-Scale`的值的正负，具体如图所示：


![Shape Layer坐标系统.jpg](http://upload-images.jianshu.io/upload_images/1655773-5dd76b9de9fbfcf4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上面演示的Gif图中，`Shape Layer 1`、`Shape Layer 2`、`Shape Layer 3`、`Shape Layer 4`的坐标原点都在图层中心位置，不同的是其X轴和Y轴的正方向位置。感兴趣的同学，可以下载打开 Demo 里的 [RectComp.aep](https://github.com/YK-Unit/LottieExample/tree/master/AE-Files) 文件，查看对应 Layer 的坐标系统数据，然后你也可以尝试编辑修改对应图层的坐标原点位置和XY轴方向，导出动画数据进行试验。

总得来说，若你要添加视图到Layer，在添加前，最好打开AE文件，查看对应的 Layer 的坐标系统信息。当然，更好的做法还是和AE设计师结对开发动画，这样可以更方便知道各个Layer的信息。

