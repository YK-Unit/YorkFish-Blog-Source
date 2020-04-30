---
title: YKPageControllerScrollView设计总结
date: 2017-02-19 12:34:56
categories: ["技术"]
tags: ["2017", "iOS"]
comments: true
---

# 概述
[YKPageControllerScrollView](https://github.com/YK-Unit/YKPageControllerScrollView) 是一个 `UIViewController` 容器类的滚动视图，支持 `UIViewController` 重用机制。`YKPageControllerScrollView ` 类的设计参考了 `UICollectionView` 类，所以你会发现，其接口以及代理方法和 `UICollectionView` 的是很相似的，使用上也是相似的。

# 如何与『容器内容』交互？
容器有一个特性在我看来是很重要的，那就是『容器』和『容器内容』之间的交互：『容器』告知『容器内容』其状态的变更。
对于`YKPageControllerScrollView `而言，这交互就是：告知『VC实例』的显示状态（将出现 or 已出现 or 已消失在视图中）以及生命状态（被容器回收了）的变更。

那么怎么达到以上的目的呢？此处是设计了一个协议 `YKPageControllerScrollViewLifeCycleProtocol` ，每个要放置入容器内的 `UIViewController` 类都应该去实现这么一个协议。协议内容如下：

``` objc
@protocol YKPageControllerScrollViewLifeCycleProtocol <NSObject>

@optional

- (void)controllerWillAppearInPageControllerScrollView;

- (void)controllerDidAppearInPageControllerScrollView;

- (void)controllerDidDisappearInPageControllerScrollView;

- (void)controllerDidBeReclaimedByPageControllerScrollView;

@end
```

实现了上述协议的 `UIViewController` 在状态有变更时，会得到来自容器的通知。


# 怎么通知VC实例显示状态的变更
`YKPageControllerScrollView ` 继承自 `UIView`，那在其内部，到底是谁真正装载了 `UIViewController` 的视图内容呢？

答案是：`UICollectionView`。

而 `YKPageControllerScrollView` 是怎么获取到VC实例的显示状态（将出现 or 已出现 or 已消失在视图中）呢？正是借助了`UICollectionView` 的 `UICollectionViewDelegate` 里的相关回调方法：

```objc
- (void)collectionView:(UICollectionView *)collectionView willDisplayCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath;

- (void)collectionView:(UICollectionView *)collectionView didEndDisplayingCell:(UICollectionViewCell *)cell forItemAtIndexPath:(NSIndexPath *)indexPath
```

但是上述的回调只是帮助 `YKPageControllerScrollView` 通知应用层哪些VC实例  『将出现在视图中』 和 『已消失在视图中』 而已（包括通知对应的VC实例），另外一个『已出现在视图中』的状态，`YKPageControllerScrollView` 怎么通知应用层呢？

方法其实也很简单——当`YKPageControllerScrollView` 里的视图滑动
停止后，获取当前的VC实例，即可告知应用层哪个VC实例已出现在视图中（包括通知当前VC实例）。

`YKPageControllerScrollView` 的滑动的产生，一个源自用户手动滑动，一个源自程序接口 `[UICollectionView setContentOffset:animated:]`。

若是用户手动滑动视图，则在 `- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView` 回调中，执行上述通知逻辑即可。

若是通过 `[UICollectionView setContentOffset:animated:]` 滑动视图，则需要进一步区分 `animated` 为 YES 和 NO 的情况：

- 当`animated` 为 YES  时：
  
  此时在 `- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView` 回调中，执行上述通知逻辑即可。

- 当`animated` 为 NO  时：
  
    此时在 `- (void)scrollViewDidScroll:(UIScrollView *)scrollView` 回调中，判断当前的滑动是非用户手动滑动且`animated` 为 NO的情况下，才执行上述通知逻辑，具体代码如下：

  ``` objc
	- (void)scrollViewDidScroll:(UIScrollView *)scrollView
	{
	    if (!self.isScrollWithAnim && !scrollView.isTracking && !scrollView.isDragging && !scrollView.isDecelerating) {
	        self.currentIndex = (NSInteger)(scrollView.contentOffset.x / self.frame.size.width);
	        
	        UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *currentVC = [self currentViewController];
	        //如果当前VC还没生成，则推迟发送通知
	        if (currentVC) {
	            [self sendDidDisplayNotificationToViewController:currentVC];
	            
	            //停止滚动后，回收可回收的VC
	            [self recycleViewController];
	        }else{
	            [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(scrollViewDidEndScrollingAnimation:) object:scrollView];
	            [self performSelector:@selector(scrollViewDidEndScrollingAnimation:) withObject:scrollView afterDelay:1.0];
	        }
	    }
	        
	}
  ```

# UIViewController 重用机制

`YKPageControllerScrollView` 支持的 `UIViewController` 重用机制类似于 `UICollectionView` 的 `cell` 重用机制。

在应用层面上，二者的使用是近似的：
1. 通过 `[YKPageControllerScrollView registerClassForController:class]` 注册可重用的 `ViewController` 类。
2. 通过 ` [YKPageControllerScrollView dequeueReusableViewControllerWithReuseClass:class forIndex:index]` 返回可重用的VC实例。若返回的实例为 nil，则由应用层生成一个新的VC实例

那么，在 `YKPageControllerScrollView` 内，该机制是如何实现的呢？

重用机制，总体来说，涉及3个方面：
- VC是怎么得到的？
- VC是怎么重新利用的？
- VC是怎么回收的？

在解答上述3个问题前，先了解一下`YKPageControllerScrollView` 的辅助属性：

``` objc
@property (nonatomic,strong) NSMutableDictionary *dict4ReusableArray; 
@property (nonatomic,strong) NSMutableDictionary *dict4ActiveController; 
@property (nonatomic,strong) NSMutableArray *array4PendingControllerIndex; 
```

- `dict4ReusableArray` 用于保存可重用的VC实例（没加载到容器上的VC实例）数组
- `dict4ActiveController` 用于保存正在使用的VC实例（加载到容器上的VC实例）
- `array4PendingControllerIndex` 用于保存那些不可见的VC实例（加载到容器上，但是没在可视区域的VC实例）的索引

### VC是怎么得到的？
在回调 `- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
` 中，通知应用层返回一个VC实例，并存放到`dict4ActiveController` 字典中：

``` objc
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    UICollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:YKPageControllerScrollViewCellIdentifier forIndexPath:indexPath];
    
    NSInteger index = indexPath.row;
    self.currentIndex = index;
    
    if (self.delegate && [self.delegate respondsToSelector:@selector(pageControllerScrollView:controllerForItemAtIndex:)]) {
        UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *vc = [self.delegate pageControllerScrollView:self controllerForItemAtIndex:index];
        
        vc.view.frame = cell.contentView.bounds;
        [cell.contentView addSubview:vc.view];
        [self.containerViewController addChildViewController:vc];
        [self.dict4ActiveController setObject:vc forKey:@(index)];
        [self.array4PendingControllerIndex removeObject:@(index)];
    }
    
    //NSLog(@"cellForItemAtIndex:%d",index);
    
    return cell;
}

```

### VC是怎么重新利用的？
在应用层上，当 `YKPageControllerScrollView` 通知返回一个 VC实例时，应用层首先调用 `YKPageControllerScrollView` 的`dequeueReusableViewControllerWithReuseClass:forIndex:`  方法获取一个可重用的VC实例，若为nil，才生成一个VC实例给`YKPageControllerScrollView`。VC能够重新利用的重点就在于 `dequeueReusableViewControllerWithReuseClass:forIndex:`的实现：

``` objc
- (nullable UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *)dequeueReusableViewControllerWithReuseClass:(nonnull Class)reuseClass forIndex:(NSInteger)index
{
    UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *reusableVC = nil;
    
    NSString *identifier = [reuseClass description];
    NSMutableArray *reusableArray = [self.dict4ReusableArray objectForKey:identifier];
    
    //若reusableArray为nil，说明没有注册reuseClass（执行[YKPageControllerScrollView registerClassForController:reuseClass]）
    if (reuseClass && reusableArray) {
        UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *vc = [self.dict4ActiveController objectForKey:@(index)];
        if (vc) {
            reusableVC = vc;
        }else{
            vc = [reusableArray firstObject];
            if (vc) {
                reusableVC = vc;
                [reusableArray removeObject:vc];
            }
        }
    }
    
    return reusableVC;
}
```
`YKPageControllerScrollView` 根据 `class`，从 `dict4ReusableArray` 字典中获取对应的VC重用数组，然后从数组中取出一个可用的VC实例。若数组为空，则返回 nil，由应用层自己生成一个VC实例。

### VC是怎么回收？
 在`YKPageControllerScrollView` 滑动过程中，会把显示的VC实例的索引从 `array4PendingControllerIndex` 中移除，把消失的VC实例的索引添加到 `array4PendingControllerIndex` 中。
当 `YKPageControllerScrollView` 停止滑动后，执行回收操作 `[YKPageControllerScrollView  recycleViewController]`：把消失的且距离当前索引的距离大于3的VC实例从 `dict4ActiveController` 字典中回收到对应重用数组中。回收操作具体如下：

 ``` objc
- (void)recycleViewController
{
    NSArray *tempArray = [NSArray arrayWithArray:self.array4PendingControllerIndex];
    for (NSNumber *indexNum in tempArray) {
        NSInteger index = [indexNum integerValue];
        
        if (labs(self.currentIndex - index) >= 3 ) {
            UIViewController<YKPageControllerScrollViewLifeCycleProtocol> *vc = [self.dict4ActiveController objectForKey:@(index)];
            
            if (vc) {
                [vc.view removeFromSuperview];
                [vc removeFromParentViewController];
                
                if ([vc respondsToSelector:@selector(controllerDidBeReclaimedByPageControllerScrollView)]) {
                    [vc controllerDidBeReclaimedByPageControllerScrollView];
                }
                
                [self.dict4ActiveController removeObjectForKey:@(index)];
                [self.array4PendingControllerIndex removeObject:@(index)];
                
                Class reuseClass = [vc class];
                NSString *identifier = [reuseClass description];
                NSMutableArray *reusableArray = [self.dict4ReusableArray objectForKey:identifier];
                [reusableArray addObject:vc];
            }
        }
    }
}
 ```

至此，`YKPageControllerScrollView` 内形成了VC实例的生成、回收、重用的闭环。

# 怎么通知VC实例生命状态的变更
VC实例在`YKPageControllerScrollView`里的生命状态的变更主要是：VC实例被 `YKPageControllerScrollView` 回收了。

所以，只需要在 `YKPageControllerScrollView` 执行回收操作的时候，通知被回收的VC实例即可。具体代码，可看`[YKPageControllerScrollView  recycleViewController]` 。

