---
title: YKWebViewJavascriptBridge设计总结
date: 2017-02-18 12:34:56
categories: ["技术"]
tags: ["2017", "大前端", "iOS"]
comments: true
---

# 前言

在 iOS 开发中，你应该会碰到以下这个问题：

>如何实现 APP 内的 web 页面和 Native 页面的交互功能?
>*该问题的本质是：实现 JS 和 OC 的相互调用。*

[YKWebViewJavascriptBridge](https://github.com/YK-Unit/YKWebViewJavascriptBridge) 是我针对上述问题实现的库。下面将会具体介绍该库的设计思路和具体实现。

<!-- more -->

# YKWebViewJavascriptBridge

`YKWebViewJavascriptBridge` 是基于 `WKWebView`+`messageHandler`+`自定义文本协议` 进行实现的，协议的传输采取的是请求响应模式。

`YKWebViewJavascriptBridge` 的设计很简单，其内部只有2个类：

- `YKNativeBridgeEngine`：提供 Native 层调用 JS 方法的接口；管理供 JS 调用的 OC 方法；与 Web 层进行协议通讯（对协议进行编码、解码以及传输）
- `YKWebBridgeEngine`：提供 Web 层调用 OC 方法的接口；管理供OC 调用的 JS 方法；与 Native 层进行协议通讯（对协议进行编码、解码以及传输）

## 关于『自定义文本协议』
`YKWebViewJavascriptBridge` 里设计的『协议』是一个 JSON 格式的文本协议，其具体格式如下：

- 请求方发出的协议数据：

``` json
{
    "reqId":100, // 请求ID
    "name":"GetLocation", // 请求响应方执行的方法
    "data":"some data from Requester" // 给响应方的数据（若没有数据给响应方，则不存在该key）
}
```

- 响应方返回的协议数据：

``` json
{
    "respId":100, // 响应ID，值源自reqId
    "data":"some data from Responser" // 给请求方的响应数据（若没有响应数据，则不存在该key）
}
```

## 关于『编码、解码』

`YKWebViewJavascriptBridge` 的编码和解码，其实就是把数据对象序列化为 JSON 和把 JSON 反序列化为数据对象。

## 关于『协议传输』

Native层的协议传输和Web层的协议传输分别由 `YKNativeBridgeEngine`  和 `YKWebBridgeEngine` 各自实现。其传输接口实现如下：

- `YKNativeBridgeEngine`的协议传输接口

``` objc
- (void)_bridgeEngineDidWriteData:(NSString *)jsonMessage
{
    if (!jsonMessage) {
        return;
    }
    
    if (![jsonMessage isKindOfClass:[NSString class]]) {
        return;
    }
    
    // 调用YKWebBridgeEngine的_bridgeEngineDidReadData方法传递协议数据给YKWebBridgeEngine
    NSString *jsCode = [NSString stringWithFormat:@"YKWebBridgeEngine._bridgeEngineDidReadData('%@')",jsonMessage];
    [self.webView evaluateJavaScript:jsCode completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"bridgeEngineDidWriteData:%@-%@",result,error);
    }];
}

- (void)_bridgeEngineDidReadData:(NSString *)jsonMessage
{
    if (!jsonMessage) {
        return;
    }
    
    if (![jsonMessage isKindOfClass:[NSString class]]) {
        return;
    }
    
    // 对接收到的json文本协议进行解码，转换层字典对象
    NSDictionary *messgaeDict = [self deserializeJSONMessage:jsonMessage];
    if (!messgaeDict) {
        return;
    }
    
    // 根据解码后的协议，执行具体操作
    ...
}
```

- `YKWebBridgeEngine`的协议传输接口

``` js
function _bridgeEngineDidWriteData(jsonMessage) {
	//通过WKWebView的messageHandler功能，传递数据给YKNativeBridgeEngine     
    window.webkit.messageHandlers.NativeBridgeEngineDidReadData.postMessage(jsonMessage);
}

function _bridgeEngineDidReadData(jsonMessage) {
	// 对接收到的json文本协议进行解码，转换层字典对象
    var messageDict = JSON.parse(jsonMessage);

    // 根据解码后协议内容，执行具体的操作
    ...
}
```

熟悉`WKWebView`+`messageHandler` 的童鞋看到这，应该知道，`YKWebViewJavascriptBridge` 的协议传输完全是基于 `WKWebView` 实现的：

- `[WKWebView evaluateJavaScript:completionHandler:]` 提供了 Native 层发数据给 Web 层的能力

- `window.webkit.messageHandlers. NativeMethod.postMessage(messageData);` 提供了 Web 层发送数据给 Native 层的能力

## YKNativeBridgeEngine 设计要点
### 在应用中的初始化流程

`YKNativeBridgeEngine` 在应用中的初始化代码如下：

``` objc
WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
configuration.userContentController = [WKUserContentController new];

WKPreferences *preferences = [WKPreferences new];
preferences.javaScriptCanOpenWindowsAutomatically = YES;
preferences.minimumFontSize = 40.0;
configuration.preferences = preferences;

// 1、根据WKWebViewConfiguration生成nativeBridgeEngine实例
self.nativeBridgeEngine = [[YKNativeBridgeEngine alloc] initWithWebViewConfiguration:configuration];;
self.webView = [[WKWebView alloc] initWithFrame:CGRectMake(0, 200, self.view.frame.size.width, self.view.frame.size.height - 100) configuration:self.nativeBridgeEngine.configuration];
// 2、绑定具体的WKWebView实例
[self.nativeBridgeEngine bridgeForWebView:self.webView];
```

如上述代码所示，`YKNativeBridgeEngine` 在应用中的初始化需要2步：

1. 根据 `WKWebViewConfiguration` 生成 `nativeBridgeEngine` 实例

  > 为什么需要 `WKWebViewConfiguration` 来生成`nativeBridgeEngine` 实例？
  > 看代码：
  > ``` objc
  > - (instancetype)initWithWebViewConfiguration:(WKWebViewConfiguration *)configuration
  > {
  >     self = [self init];
  >     if (self) {
  >         if (configuration) {
  >             self.configuration = configuration;
  >         }else{
  >             WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
  >             configuration.userContentController = [[WKUserContentController alloc] init];
  >             
  >             self.configuration = configuration;
  >         }
  >         
  >         if (!self.configuration.userContentController) {
  >             self.configuration.userContentController = [WKUserContentController new];
  >         }
  >         
  >         //通过JS注入方式，初始化WebBridgeEngine
  >         NSString *js = YKWebBridgeEngine_js();
  >         WKUserScript *script = [[WKUserScript alloc] initWithSource:js
  >                                                       injectionTime:WKUserScriptInjectionTimeAtDocumentStart
  >                                                    forMainFrameOnly:YES];
  >         [self.configuration.userContentController addUserScript:script];
  >         
  >         [self.configuration.userContentController addScriptMessageHandler:self name:@"NativeBridgeEngineDidReadData"];
  >     }
  >     return self;
  > }
  > 
  > ```
  > `WKWebViewConfiguration` 的作用主要是：
  > 
  > - 注入 JS 代码到 `WebView`，在 `Document` 元素开始生成时，执行 `YKWebBridgeEngine` 初始化代码
  > - 添加 ScriptMessageHandler，提供方法给 `YKWebBridgeEngine` 发送数据给 `YKNativeBridgeEngine` 
  > 

2. 绑定具体的 `WKWebView` 实例

  > 为什么需要绑定具体的 `WKWebView` 实例？
  > 看代码：
  > ``` objc
  > - (void)_bridgeEngineDidWriteData:(NSString *)jsonMessage
  > {
  >     if (!jsonMessage) {
  >         return;
  >     }
  >     
  >     if (![jsonMessage isKindOfClass:[NSString class]]) {
  >         return;
  >     }
  >     
  >     NSString *jsCode = [NSString stringWithFormat:@"YKWebBridgeEngine._bridgeEngineDidReadData('%@')",jsonMessage];
  >     [self.webView evaluateJavaScript:jsCode completionHandler:^(id _Nullable result, NSError * _Nullable error) {
  >         NSLog(@"bridgeEngineDidWriteData:%@-%@",result,error);
  >     }];
  > }
  > ```
  > `YKNativeBridgeEngine` 内部持有 `webView` 的用处就是在于借助 `webView` 的 `evaluateJavaScript:completionHandler:` 功能，达到发数据给 `YKWebBridgeEngine` 的目的。


### 关于『解码后的逻辑』
在 `YKNativeBridgeEngine` 收到 `YKWebBridgeEngine` 发来的 JSON文本数据后，`YKNativeBridgeEngine` 会对数据进行解码：反序列化JSON，得到一个 `messageDict` ，然后根据其内容进行具体的逻辑操作。核心代码如下：

``` objc
- (void)_bridgeEngineDidReadData:(NSString *)jsonMessage
{
    if (!jsonMessage) {
        return;
    }
    
    if (![jsonMessage isKindOfClass:[NSString class]]) {
        return;
    }
    
    NSDictionary *messgaeDict = [self deserializeJSONMessage:jsonMessage];
    if (!messgaeDict) {
        return;
    }
    
    NSArray *allKeys = messgaeDict.allKeys;
    
    //收到WebBridgeEngine发来的请求
    if ([allKeys containsObject:@"reqId"]) {
        NSInteger reqId = [[messgaeDict objectForKey:@"reqId"] integerValue];
        id data = [messgaeDict objectForKey:@"data"];
        NSString *name = [messgaeDict objectForKey:@"name"];
        
        if (!name) {
            return;
        }
        
        YKMessageHandler handler = [self.messageHandlerDict objectForKey:name];
        
        if (!handler) {
            return;
        }
        
        __weak typeof(self) weakSelf = self;
        YKResponseCallback responseCallback = ^(id responseData) {
            NSMutableDictionary *respMessageDict = [[NSMutableDictionary alloc] initWithCapacity:8];
            [respMessageDict setObject:@(reqId) forKey:@"respId"];
            
            if (responseData) {
                [respMessageDict setObject:responseData forKey:@"data"];
            }
            
            NSString *jsonMessage = [weakSelf serializeMessage:respMessageDict];
            [weakSelf _bridgeEngineDidWriteData:jsonMessage];
        };
        
        handler(data,responseCallback);

    }
    //收到WebBridgeEngine返回的响应数据
    else if ([allKeys containsObject:@"respId"]) {
        NSInteger respId = [[messgaeDict objectForKey:@"respId"] integerValue];
        id responseData = [messgaeDict objectForKey:@"data"];
        YKResponseCallback responseCallback = [self.responseCallbackDict objectForKey:@(respId)];
        if (responseCallback) {
            responseCallback(responseData);
            [self.responseCallbackDict removeObjectForKey:@(respId)];
        }
    }
}

```

在收到数据后，会根据解码后的 `messgaeDict` 的 key 判断这是一个『请求类型的数据』还是一个『响应类的数据』：若 key 中存在 `reqId ` ，则是『请求类型的数据』，然后执行对应的请求，并返回响应数据；若是存在 `respId ` ，则是一个『响应类型的数据』，然后把返回的响应数据通过回调告知应用层。

## 关于 YKWebBridgeEngine

`YKWebBridgeEngine` 的设计（接口和功能） 和 `YKNativeBridgeEngine` 是一样的，区别在于前者是 OC 语言实现的，后者是 JS 语言实现而已。另外需要注意的是，`YKWebBridgeEngine`初始化依赖与`YKNativeBridgeEngine`，通过 JS注入，让`webview`在开始加载 `Document` 元素时执行`YKWebBridgeEngine`的初始化代码。


# 其他
 想详细了解`WKWebView` 和 `messageHandler` 的童鞋，可以参考下面文章：
- [iOS下JS与OC互相调用（三）--MessageHandler](http://www.jianshu.com/p/433e59c5a9eb)


# 参考资料
- [iOS下JS与OC互相调用（三）--MessageHandler](http://www.jianshu.com/p/433e59c5a9eb)
- [marcuswestin](https://github.com/marcuswestin)/[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)

