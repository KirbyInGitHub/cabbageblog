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

`WKNavigationDelegate`

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

`WKUIDelegate`

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

`WKScriptMessageHandler`

这个协议包含一个必须实现的方法, 它可以直接将接收到的JS脚本转为Swift或者OC对象.

```swift
func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage)
```

这部分其实除了iOS10新加的几个代理方法, 其他的并没有什么特别的. 只不过把原本`UIWebView`里面相应的代理方法挪过来而已.

##### 三、WKWebView的Cookie

由于我们的APP内使用了大量的商品列表/活动等H5页面, H5需要知道是哪一个用户在访问这个页面, 那么用`Cookie`是最好也是最合适的解决方案了, 在UIWebView的时候, 我们并没有使用`Cookie`的困扰, 我们只需要写一个方法, 往`HTTPCookieStorage`里面注入一个我们用户的`HTTPCookie`就可以了.同一个应用，不同`UIWebView`之间的`Cookie`是自动同步的。并且可以被其他网络类访问比如`NSURLConnection`,`AFNetworking`。

它们都是保存在`HTTPCookieStorage`容器中。 当`UIWebView`加载一个URL的时候，在加载完成时候，Http Response，对`Cookie`进行写入,更新或者删除，结果更新`Cookie`到`HTTPCookieStorage`存储容器中。
代码类似于:

```swift
    public class func updateCurrentCookieIfNeeded() {
        
        let cookieForWeb: HTTPCookie?
        
        if let customer = CustomerUser.current {
        
            var cookie1Props: [HTTPCookiePropertyKey: Any] = [:]
        
            cookie1Props[HTTPCookiePropertyKey.domain] = customer.area?.webURLSource.webCookieHost
            cookie1Props[HTTPCookiePropertyKey.path] = "/"
            cookie1Props[HTTPCookiePropertyKey.name] = CustomerUser.CookieName
            cookie1Props[HTTPCookiePropertyKey.value] = customer.cookie
            
            cookieForWeb = HTTPCookie(properties: cookie1Props)
        } else {
            cookieForWeb = nil
        }
        
        let storage = HTTPCookieStorage.shared
        
        if let cookie = cookieForWeb, let cookie65 = cookieFor65daigou(customer: CustomerUser.current) {
            storage.setCookie(cookie)
            storage.setCookie(cookie65)
        } else {
            guard let cookies = storage.cookies else { return }
            
            let needDeleteCookies = cookies.filter { $0.name == CustomerUser.CookieName }
            needDeleteCookies.forEach({ (cookie) in
                storage.deleteCookie(cookie)
            })
        }
    }
```

但是在我迁移到`WKWebView`的时候, 我发现这一招不管用了, `WKWebView`实例不会把`Cookie`存入到App标准的的`Cookie`容器(`HTTPCookieStorage`)中, `WKWebView`拥有自己的私有存储. 

因为 `NSURLSession`/`NSURLConnection`等网络请求使用`HTTPCookieStorage`进行访问`Cookie`,所以不能访问`WKWebView`的`Cookie`,现象就是`WKWebView`存了`Cookie`,其他的网络类如`NSURLSession`/`NSURLConnection`却看不到. 同时`WKWebView`也不会读取存储在`HTTPCookieStorage`中的`Cookie`.

为了解决这一问题, 我查了大量的资料, 最后发现通过JS的方式注入`Cookie`是对于我们目前的代码来说是最合适也是最方便的. 因为我们已经有了注入到`HTTPCookieStorage`的代码, 那么只需要把这些`Cookie`转化成JS并且注入到`WKWebView`里面就可以了.

```swift
    fileprivate class func getJSCookiesString(_ cookies: [HTTPCookie]) -> String {
        var result = ""
        let dateFormatter = DateFormatter()
        dateFormatter.timeZone = TimeZone(abbreviation: "UTC")
        dateFormatter.dateFormat = "EEE, d MMM yyyy HH:mm:ss zzz"
        
        for cookie in cookies {
            result += "document.cookie='\(cookie.name)=\(cookie.value); domain=\(cookie.domain); path=\(cookie.path); "
            if let date = cookie.expiresDate {
                result += "expires=\(dateFormatter.string(from: date)); "
            }
            if (cookie.isSecure) {
                result += "secure; "
            }
            result += "'; "
        }
        return result
    }
```

注入的方法就是在每次init`WkWebView`的时候, 使用下面的`config`就可以了:

```swift
    public class func wkWebViewConfig() -> WKWebViewConfiguration {
        
        updateCurrentCookieIfNeeded()
        
        let userContentController = WKUserContentController()
        if let cookies = HTTPCookieStorage.shared.cookies {
            let script = getJSCookiesString(cookies)
            let cookieScript = WKUserScript(source: script, injectionTime: WKUserScriptInjectionTime.atDocumentStart, forMainFrameOnly: false)
            userContentController.addUserScript(cookieScript)
        }
        let webViewConfig = WKWebViewConfiguration()
        webViewConfig.userContentController = userContentController
        
        return webViewConfig
    }
    
    public class func getJSCookiesString() -> String? {
        
        guard let cookies = HTTPCookieStorage.shared.cookies else {
            return nil
        }
        return getJSCookiesString(cookies)
    }
```

##### 四、关于User-Agent

上面`Cookie`的问题解决了, 咱们的前端又提出了新的问题, 他们需要知道用户访问了网页是使用了客户端(iOS/Android)来的.

这个就好解决了, 其实和WKWebVIew的关系不大. 最合适添加的地方就是在`user-Agent`里面, 不过并没有使用WKWebView自己的User-Agent去定义, 因为这个字段只支持`iOS9`以上, 所有用下面的代码全局添加就可以.

```swift
    fileprivate func setUserAgent(_ webView: WKWebView) {
        
        let userAgentHasPrefix = "xxxxxx "
        
        webView.evaluateJavaScript("navigator.userAgent", completionHandler: { (result, error) in

            guard let agent = result as? String , agent.hasPrefix(userAgentHasPrefix) == false else { return }
            
            let newAgent = userAgentHasPrefix + agent
            UserDefaults.standard.register(defaults: ["UserAgent":newAgent])
        })
    }
```

##### 五、关于国际化

解决了上面的问题, 咱们产品经理又提出了国际化的需求, 因为我们的APP同时为至少5个国家的客户提供, `国际化的方案也是我做的, APP内部可以热切换语言, 也许在下一篇博文中会介绍我们项目中的国际化方案.` 那么请求H5页面的时候, 理所应当的就应该带上语言信息了. 

这部分的内容, 因为双十一临近, 目前还没有具体实施. 等功能上线以后, 再来补充.

其实我挺佩服我自己的, 9月6号开始写这个, 一直到11月5号才写的差不多能看...