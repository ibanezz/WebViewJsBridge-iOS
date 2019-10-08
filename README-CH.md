# WebViewJsBridge-iOS

WebViewJsBridge-iOS 是 HTML5 和 UIWebView & WKWebView 之间用于通讯的工具库。

WebViewJsBridge-Android：[https://github.com/al-liu/WebViewJsBridge-Android](https://github.com/al-liu/WebViewJsBridge-Android)

它的特点是跨平台，支持 iOS，Android，JavaScript，接口统一，简单易用。工具库的实现对 WebView 无侵入性。使用以类的方式来管理通信的接口，每个接口的实现类对应唯一的命名空间，如 ui.alert，ui 对应一个实现类的命名空间，alert 是该实现类的一个实现方法。

下面这张图帮助理解它们之间的关系：

![WebViewJsBridge-namespace.png](https://i.loli.net/2019/10/08/a8hiDPQNAUOlByo.png)

## 系统版本要求
如果使用 UIWebView 则支持 iOS7 及以上的系统版本。
如果使用 WKWebView 则支持 iOS8 及以上的系统版本。

## 安装

### CocoaPods
[CocoaPods](https://cocoapods.org/) 是 Cocoa 项目的依赖管理器，使用方法和安装步骤请参考 CocoaPods 的官网。使用 CocoaPods 整合 WebViewJsBridge-iOS 到你的项目中，需要指定如下内容到你的 Podfile 文件：

```oc
platform :ios, '8.0'

target 'TargetName' do
  pod 'HCWebViewJsBridge', '~> 1.0.0'
end
```

然后，运行如下命令：

```oc
$ pod install
```

### 手动安装
下载 HCWebViewJsBridge 的源代码，并添加到自己的项目中即可使用。

### 引入 HCWebViewJsBridge 的 js 文件
在 html 文件中 `<script>引入 hcJsBridge.js</script>`。

## Example 的说明
`/Example/iOS Example` 文件夹下提供完整使用示例，包括基础的调用演示和进阶用法，如，使用 `UIImagePickerController` 调用相机拍摄一张图片，使用 `NSURLSession` 发起一个 GET 请求。

## 使用方法

### 初始化原生的 WebViewJsBridge 环境

#### UIWebView

```oc
_bridge = [HCWebViewJsBridge bridgeWithWebView:self.webView];
```

#### WKWebView

```oc
_bridge = [HCWKWebViewJsBridge bridgeWithWebView:self.wkWebView];
```

### 原生注册接口实现类供 HTML5 调用

```oc
UIJsApi *uiApi = [UIJsApi new];
[_bridge addJsBridgeApiObject:uiApi namespace:@"ui"];

RequestJsApi *requestJsApi = [RequestJsApi new];
[_bridge addJsBridgeApiObject:requestJsApi namespace:@"request"];
```

UIJsApi 实现类：

```oc
- (void)alert:(NSDictionary *)data callback:(HCJBResponseCallback)callback {
    callback(@"native api alert’callback.");
}
// 接口实现类支持四种方法签名：
// 1. 有参数，有回调
- (void)test1:(NSString *)data callback:(HCJBResponseCallback)callback {
    NSLog(@"Js call native api test1, data is:%@", data);
    callback(@"native api test1’callback.(备注：汉字测试)");
}
// 2. 有参数，无回调
- (void)test2:(NSDictionary *)data {
    NSLog(@"Js native api:test2, data is:%@", data);
}
// 3. 无参数，无回调
- (void)test3 {
    NSLog(@"Js native api:test3");
}
// 4. 无参数，有回调
- (void)test4:(HCJBResponseCallback)callback {
    NSLog(@"Js native api:test4");
    callback(@"native api test4'callback.(备注：汉字测试)");
}
```

### 原生调用 HTML5 接口

```oc
[_bridge callHandler:@"testCallJs" data:@{@"foo": @"bar"} responseCallback:^(id  _Nonnull responseData) {
    NSLog(@"testCallJs callback data is:%@", responseData);
}];
```

### 初始化 HTML5 的 WebViewJsBridge 环境

```js
<!DOCTYPE html>
<html>
    <head>
        ...
        <script src="./hcJsBridge.js"> </script>
    </head>
    ...
</html>
```

### HTML5 注册接口供原生调用

```js
hcJsBridge.registerHandler("testCallJs", function(data, callback) {
    log('Native call js ,handlename is testCallJs, data is:', data);
    callback('callback native, handlename is testCallJs');
})
```

### HTML5 调用原生接口

```js
var data = {foo: "bar"};
hcJsBridge.callHandler('ui.alert', data, function (responseData) {
    log('Js receives the response data returned by native, response data is', responseData);
})
```

### 开启 debug 日志

开启 debug 日志，将打印一些调用信息，辅助排查问题。debug 日志默认不开启，release 模式下屏蔽 debug 日志，但不屏蔽 error 日志。

```oc
[_bridge enableDebugLogging:YES];
```

## License
WebViewJsBridge-iOS 使用 MIT license 发布，查看 [LICENSE](./LICENSE) 详情。
