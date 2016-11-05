---
layout: post
title:  "WKWebView实践分享"
date:   2016-09-06 17:46:23 +0800
description: ezbuy内部iOS小组分享
categories: Home
---

ezbuy内部iOS小组分享

---

## WKWebView实践分享

自从公司的`ezbuy` App最低支持版本提升到`iOS8`以后, 使用更多的`iOS8`以后才特有的新特性就被提上了议程, 比如`WebKit`.
	作为公司最没有节操, 最没有底线的程序员之一, 这项任务不可避免的就落到了我的身上.

既然要使用`Webkit`, 那么首先我们就得明白为什么要使用它, 它相对于`UIWebView`来说, 有什么优势, 同时, 还得知道它的劣势,以及这些劣势是否会对公司现有业务造成影响.

首先我们来说说它的优势:

* 性能更高 高达60fps的滚动刷新以及内置手势
* 内存占用更低 内存占用只有UIWebView的1/4左右
* 允许JavaScript的Nitro库的加载并使用
* 支持更多的HtML5特性
* 原生支持加载进度
* 支持自定义UserAgent(`iOS9`以上)

再来说说它的劣势:

* 不支持缓存
* 不能拦截修改Request

说完了优势劣势, 那下面就来说说它的基本用法.

##### 一、加载网页

加载网页的方法和`UIWebView`相同, 代码如下:

```swift
let webView = WKWebView(frame: self.view.bounds,configuration: config)
webView.autoresizingMask = [.flexibleHeight, .flexibleWidth]
view.addSubview(webView)
```

##### 二、WKWebView的代理方法

*WKNavigationDelegate*

用来追踪加载过程（页面开始加载、加载完成、加载失败）的方法：

```swift
// 页面开始加载时调用
func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!)
// 当内容开始返回时调用
func webView(_ webView: WKWebView, didCommit navigation: WKNavigation!)
// 页面加载完成之后调用
func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!)
// 页面加载失败时调用
func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error)
```

用来跳转页面的方法:

```swift
// 接收到服务器跳转请求之后调用
func webView(_ webView: WKWebView, didReceiveServerRedirectForProvisionalNavigation navigation: WKNavigation!)
// 在收到响应后，决定是否跳转
func webView(_ webView: WKWebView, decidePolicyFor navigationResponse: WKNavigationResponse, decisionHandler: @escaping (WKNavigationResponsePolicy) -> Void)
// 在发送请求之前，决定是否跳转
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void)
```

*WKUIDelegate*

```swift
// 创建一个新的webView
func webView(_ webView: WKWebView, createWebViewWith configuration: WKWebViewConfiguration, for navigationAction: WKNavigationAction, windowFeatures: WKWindowFeatures) -> WKWebView?
// webView中的确认弹窗
func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void)
// webView中的输入框
func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void)
// webView中的警告弹窗
func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void)
//TODO: iOS10中新添加的几个代理方法待补充
```

*WKScriptMessageHandler*

这个协议包含一个必须实现的方法, 它可以直接将接收到的JS脚本转为Swift或者OC对象.

```swift
func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage)
```

##### 三、WKWebView的Cookie

