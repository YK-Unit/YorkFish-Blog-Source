---
title: iOS WebView生成长截图的第三种解决方案
date: 2018-09-15 12:34:56
categories: ["技术"]
tags: ["2018", "iOS"]
comments: true
---

## 前言

由于项目需要，新近实现了一个长截图库 [SnapshotKit](https://github.com/YK-Unit/SnapshotKit)。其中，需要支持 `UIWebView`、`WKWebView` 组件生成长截图。为了实现这个特性，查阅了很多资料，同时也做了不同的新奇思路尝试，最终实现了一个新的、取巧的技术方案。

以下主要总结了在“WebView生成长截图”需求方面，“网上已有方案”和“我的全新方案”的各自实现要点和优缺点。

## WebView生成长截图的已有方案

根据 Google 所搜索到的资料，目前iOS WebView生成长截图的方案主要有2种：

- 方案一：修改Frame，截图组件
- 方案二：分页截图组件内容，合成长图

下面将会简述方案一和方案二的具体实现。

#### 方案一：修改Frame，截图组件

方案一的实现要点在于：修改 `webView.scrollView` 的 `frameSize`  为 `contentSize`，然后对整个 `webView.scrollView` 进行截图。

不过，这个方案只适用 `UIWebView` 组件，因为其是一次性加载网页所有的内容。而 `WKWebView` 组件，为了节省内存，加载网页内容时，只加载可视部分——这一点类似 `UITableView` 组件。在修改`webView.scrollView` 的 `frameSize` 后，立即执行了截图操作， 这时候，`WKWebView`由于还没把网页的内容加载出来，导致生成的长截图是空白的。

方案一核心代码如下：
```swift
extension UIScrollView {
   public func takeSnapshotOfFullContent() -> UIImage? {
        let originalFrame = self.frame
        let originalOffset = self.contentOffset

        self.frame = CGRect.init(origin: originalFrame.origin, size: self.contentSize)
        self.contentOffset = .zero

        let backgroundColor = self.backgroundColor ?? UIColor.white

        UIGraphicsBeginImageContextWithOptions(self.bounds.size, true, 0)

        guard let context = UIGraphicsGetCurrentContext() else {
            return nil
        }
        context.setFillColor(backgroundColor.cgColor)
        context.setStrokeColor(backgroundColor.cgColor)

        self.drawHierarchy(in: self.bounds, afterScreenUpdates: true)
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()

        self.frame = originalFrame
        self.contentOffset = originalOffset

        return image
    }
}
```

测试代码：
```swift
// example code
 private func takeSnapshotOfUIWebView() {
    let image = self.webView.scrollView.takeSnapshotOfFullContent()
   // 处理image
}    
```

#### 方案二：分页截图组件内容，合成长图

方案二的实现要点在于：分页滚动WebView组件的内容，然后生成分页截图，最后把所有分页截图合成一张长图。

这个方案适用于 `UIWebView` 组件和 `WKWebView` 组件。

方案二核心代码如下：
```swift
extension UIScrollView {
    public func takeScreenshotOfFullContent(_ completion: @escaping ((UIImage?) -> Void)) {
        // 分页绘制内容到ImageContext
        let originalOffset = self.contentOffset

        // 当contentSize.height<bounds.height时，保证至少有1页的内容绘制
        var pageNum = 1
        if self.contentSize.height > self.bounds.height {
            pageNum = Int(floorf(Float(self.contentSize.height / self.bounds.height)))
        }

        let backgroundColor = self.backgroundColor ?? UIColor.white

        UIGraphicsBeginImageContextWithOptions(self.contentSize, true, 0)

        guard let context = UIGraphicsGetCurrentContext() else {
            completion(nil)
            return
        }
        context.setFillColor(backgroundColor.cgColor)
        context.setStrokeColor(backgroundColor.cgColor)

        self.drawScreenshotOfPageContent(0, maxIndex: pageNum) {
            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()
            self.contentOffset = originalOffset
            completion(image)
        }
    }

    fileprivate func drawScreenshotOfPageContent(_ index: Int, maxIndex: Int, completion: @escaping () -> Void) {

        self.setContentOffset(CGPoint(x: 0, y: CGFloat(index) * self.frame.size.height), animated: false)
        let pageFrame = CGRect(x: 0, y: CGFloat(index) * self.frame.size.height, width: self.bounds.size.width, height: self.bounds.size.height)

        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.3) {
            self.drawHierarchy(in: pageFrame, afterScreenUpdates: true)

            if index < maxIndex {
                self.drawScreenshotOfPageContent(index + 1, maxIndex: maxIndex, completion: completion)
            }else{
                completion()
            }
        }
    }
}
```

测试代码：
```swift
// example code
private func takeSnapshotOfUIWebView() {
    self.uiWebView.scrollView.takeScreenshotOfFullContent { (image) in
        // 处理image
    }
}

private func takeSnapshotOfWKWebView() {
    self.wkWebView.scrollView.takeScreenshotOfFullContent { (image) in
        // 处理image
    }
}
```

## WebView生成长截图的新方案

除了方案一和方案二，还有新方案吗？

答案是肯定加确定以及一定的。

这个新方案的要点在于：iOS系统的WebView打印功能。

iOS系统支持把WebView的内容打印到PDF文件上，借助这个特性，新方案的设计如下：

1. 把 WebView组件的内容全部打印到一页PDF上

2. 把PDF转换成图片

新方案的核心代码如下：
```swift
import UIKit
import WebKit

/// WebViewPrintPageRenderer: use to print the full content of webview into one image
internal final class WebViewPrintPageRenderer: UIPrintPageRenderer {

    private var formatter: UIPrintFormatter

    private var contentSize: CGSize

    /// 生成PrintPageRenderer实例
    ///
    /// - Parameters:
    ///   - formatter: WebView的viewPrintFormatter
    ///   - contentSize: WebView的ContentSize
    required init(formatter: UIPrintFormatter, contentSize: CGSize) {
        self.formatter = formatter
        self.contentSize = contentSize
        super.init()
        self.addPrintFormatter(formatter, startingAtPageAt: 0)
    }

    override var paperRect: CGRect {
        return CGRect.init(origin: .zero, size: contentSize)
    }

    override var printableRect: CGRect {
        return CGRect.init(origin: .zero, size: contentSize)
    }

    private func printContentToPDFPage() -> CGPDFPage? {
        let data = NSMutableData()
        UIGraphicsBeginPDFContextToData(data, self.paperRect, nil)
        self.prepare(forDrawingPages: NSMakeRange(0, 1))
        let bounds = UIGraphicsGetPDFContextBounds()
        UIGraphicsBeginPDFPage()
        self.drawPage(at: 0, in: bounds)
        UIGraphicsEndPDFContext()

        let cfData = data as CFData
        guard let provider = CGDataProvider.init(data: cfData) else {
            return nil
        }
        let pdfDocument = CGPDFDocument.init(provider)
        let pdfPage = pdfDocument?.page(at: 1)

        return pdfPage
    }

    private func covertPDFPageToImage(_ pdfPage: CGPDFPage) -> UIImage? {
        let pageRect = pdfPage.getBoxRect(.trimBox)
        let contentSize = CGSize.init(width: floor(pageRect.size.width), height: floor(pageRect.size.height))

        // usually you want UIGraphicsBeginImageContextWithOptions last parameter to be 0.0 as this will us the device's scale
        UIGraphicsBeginImageContextWithOptions(contentSize, true, 2.0)
        guard let context = UIGraphicsGetCurrentContext() else {
            return nil
        }

        context.setFillColor(UIColor.white.cgColor)
        context.setStrokeColor(UIColor.white.cgColor)
        context.fill(pageRect)

        context.saveGState()
        context.translateBy(x: 0, y: contentSize.height)
        context.scaleBy(x: 1.0, y: -1.0)

        context.interpolationQuality = .low
        context.setRenderingIntent(.defaultIntent)
        context.drawPDFPage(pdfPage)
        context.restoreGState()

        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()

        return image
    }

    /// print the full content of webview into one image
    ///
    /// - Important: if the size of content is very large, then the size of image will be also very large
    /// - Returns: UIImage?
    internal func printContentToImage() -> UIImage? {
        guard let pdfPage = self.printContentToPDFPage() else {
            return nil
        }

        let image = self.covertPDFPageToImage(pdfPage)
        return image
    }
}

extension UIWebView {
    public func takeScreenshotOfFullContent(_ completion: @escaping ((UIImage?) -> Void)) {
        self.scrollView.setContentOffset(CGPoint(x: 0, y: 0), animated: false)
        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.3) {
            let renderer = WebViewPrintPageRenderer.init(formatter: self.viewPrintFormatter(), contentSize: self.scrollView.contentSize)
            let image = renderer.printContentToImage()
            completion(image)
        }
    }
}

extension WKWebView {
    public func takeScreenshotOfFullContent(_ completion: @escaping ((UIImage?) -> Void)) {
        self.scrollView.setContentOffset(CGPoint(x: 0, y: 0), animated: false)
        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.3) {
            let renderer = WebViewPrintPageRenderer.init(formatter: self.viewPrintFormatter(), contentSize: self.scrollView.contentSize)
            let image = renderer.printContentToImage()
            completion(image)
        }
    }
}
```

`WebViewPrintPageRenderer` 是该方案的核心类，负责把 `WebView组件`内容打印到PDF，然后把PDF转换为图片。

`UIWebView` 和 `WKWebView` 则实现对应的扩展。

测试代码：
```swift
// example code
private func takeSnapshotOfUIWebView() {
    self.uiWebView.scrollView.takeScreenshotOfFullContent { (image) in
        // 处理image
    }
}

private func takeSnapshotOfWKWebView() {
    self.wkWebView.scrollView.takeScreenshotOfFullContent { (image) in
        // 处理image
    }
}
```

## 三种技术方案优劣对比

那么，这三种技术方案各自存在什么优缺点呢，适用什么场景呢？

- 方案一：只适用 `UIWebView`；若网页内容很多，生成长截图时，会占用过多内存。 所以，该方案只适合不需要支持 `WKWebView`， 且网页内容不会太多的场景。
- 方案二：适用 `UIWebView` 和 `WKWebView`，且特别适合 `WKWebView`。由于采用分页生成截图机制，有效减少内存占用。不过，这个方案存在一个问题：若网页存在 `position: fixed` 的元素（如网页头部固定的导航栏），该元素会重复出现在生成的长图上。
- 方案三：适用 `UIWebView` 和 `WKWebView`。其中最重要的一步——“把WebView内容打印到PDF” 是由iOS系统实现，所以该方案的性能在理论上是可以得到保障的。不过，这个方案存在一个问题：在把网页内容打印到PDF时，iOS系统获取的 `contentSize` 比WebView的实际`contentSize` 要大，从而导致生成的图片在靠近底部的内容部分和实际存在一点差异。具体可以下载运行我的长截图库 [SnapshotKit](https://github.com/YK-Unit/SnapshotKit) 的 Demo，通过其中的 `UIWebView` 和 `WKWebView` 截图示例查看具体截图效果。

以上三个方案，总的来说，解决了部分场景的需求，但都不够完美，仍需做进一步的优化。

