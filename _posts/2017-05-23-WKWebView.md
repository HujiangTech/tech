---   
layout: post
title: WKWebView
categories: [iOS]
tags: [WKWebView]
description: WKWebView是iOS8之后提供的一款浏览器组件，其载入速度和内存占用对比老的 `UIWebView` 来说简直是一次飞跃。下面对比 `UIWebView` 介绍该组件如何去使用，以及使用过程中会存在的问题。
---  


> WKWebView是iOS8之后提供的一款浏览器组件，其载入速度和内存占用对比老的 `UIWebView` 来说简直是一次飞跃。下面对比 `UIWebView` 介绍该组件如何去使用，以及使用过程中会存在的问题。

## 目录
1. 介绍：为什么需要 `WKWebView`

2. 使用：对比 `UIWebView` 介绍

3. 深入：`JS` 和 `Native`交互

4. 重点：已知问题和解决方案

5. 总结 & 结束

## 正文
### 一、介绍：为什么需要 `WKWebView`
用 `Web` 方式承载业务引入APP的混合开发已成为一个硬性需求，并且随着业务的扩展和审核的限制，该比例会愈加增大，这样对承载的平台就有严格要求，而老旧的 `UIWebView` 存在严重的性能和内存消耗问题，严重限制了业务方的自由度。而自iOS8之后，苹果提供了的WebKit库包含了WKWebView，WKWebView采用跨进程方案，Nitro JS解析器，高达60fps的刷新率，理论上性能和Safari比肩，我们没有理由拒绝这样的一个新事务。

### 二、使用：对比 `UIWebView` 介绍
#### 使用
`WKWebView` 和 `UIWebView` 二者在使用上差不多（如下代码所示），如果仅做页面呈现和简单交互，不需要考虑太多的投入成本。 对比代码，二者在创建和使用上基本一致，通过一个URL字符串构建Request对象，然后使用WebView内置接口载入Request对象即可。

```objc
UIWebView *webView = [[UIWebView alloc] init];
NSURL *url = [NSURL URLWithString:@"https://www.hujiang.com"];
[webView loadRequest:[NSURLRequest requestWithURL:url]];
```
```objc
WKWebView *webView = [[WKWebView alloc] init];
NSURL *url = [NSURL URLWithString:@"https://www.hujiang.com"];
[webView loadRequest:[NSURLRequest requestWithURL:url]];
```

流程上二者有一定的区别，如下图，左边是UIWebView，右边是WKWebView，在节点上，WKWebView比UIWebView多了一个询问过程，在服务器响应请求之后会询问是否载入内容到当前Frame，在控制上会比UIWebView粒度更细一些。

![WKWebView和UIWebView差异](https://hujiangtech.github.io/tech/assets/pic/523/img_01.png)

#### Delegate
以上流程的控制主要是用过Protocol去实现，`WKWebView` 的代理协议为 `WKNavigationDelegate`，对比 `UIWebDelegate` 首先跳转询问，就是载入URL之前的一次调用，询问开发者是否下载并载入当前URL，UIWebView只有一次询问，就是请求之前的询问，而WKWebView在URL下载完毕之后还会发一次询问，让开发者根据服务器返回的Web内容再次做一次确定。

```objc
#pragma mark - UIWebViewDelegate
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {return YES;}

#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler { }

- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler { }
```

同意载入之后，组件就开始下载指定URL的内容，在下载之前会调用一次 '开始下载' 回调，通知开发者Web已经开始下载。

```objc
#pragma mark - UIWebViewDelegate
- (void)webViewDidStartLoad:(UIWebView *)webView {}

#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation { }

- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation { }
```

页面下载完毕之后，UIWebView会直接载入视图并调用载入成功回调，而WKWebView会发询问，确定下载的内容被允许之后再载入视图。

```objc
#pragma mark - UIWebViewDelegate
- (void)webViewDidFinishLoad:(UIWebView *)webView {}

#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation { }
```

成功则调用成功回调，整个流程有错误发生都会发出错误回调。

```objc
#pragma mark - UIWebViewDelegate
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error {}

#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error { }

- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error {}
```

而除此之外，`WKWebView` 对比 `UIWebView` 有较大差异的地方有几点。

1、 WKWebView多了一个重定向通知，在收到服务器重定向消息并且跳转询问允许之后，会回调重定向方法，这点是UIWebView没有的，在UIWebView之上需要验证是否重定向，得在询问方法验证head信息。

```objc
#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation { }
```

2、 因为WKWebView是跨进程的方案，当WKWebView进程退出时，会对主进程做一次方法回调。注：该方法是在iOS9之后才出现，而我们最低支持版本是iOS8，所以还得考虑iOS8下`WKWebView`进程崩溃问题，另外该方法也不是很靠谱，不一定所有崩溃情况都会触发回调，具体的在后面的部分细说。

```objc
- (void)webViewWebContentProcessDidTerminate:(WKWebView *)webView API_AVAILABLE(macosx(10.11), ios(9.0)) {}
```

3、 https证书自定义处理

```objc
- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * _Nullable credential))completionHandler { }
```
注意：证书认证代理不一定百分百会回调，可能是iOS系统的Bug，目前无解，只能不要太过依赖该回调逻辑。

4、 新增 `WKUIDelegate` 协议，该协议包含一些UI相关的内容。 
	
  * 在 `UIWebView`中，Alert、Confirm、Prompt等视图是直接可执行的，但在`WKWebView`上，需要通过`WKUIDelegate`协议接收通知，然后通过IOS原生执行。
	
  * 其次，`WKWebView`关闭时的回调通知也在`WKUIDelegate`协议中。注：该方法也是iOS9才有。

  * 另外，对于类似 'A' 标签 'target=_blank' 这种情况，会要求创建一个新的WKWebView视图，这个消息的通知回调也在该协议中，不过针对iOS设备在当前一个视图中显示，该标签点击会没反应，所以在视图载入之后会清除掉所有的_blank标记

  * 还有在 'iOS 10' 之后，新增了链接预览的支持，相关方法也在该协议中
	
协议相关的就是这些，相对来说还是比较简单的，用来做呈现控制是足够了，更进一步，就需要涉及到和JS端的交互问题了，交互就是Native通知JS，JS通知Native，基于这个能力拟订具体的交互协议。

### 三、深入：`JS` 和 `Native`交互
#### Native通知JS
先来看Native通知JS，这个完全依靠WebView提供的接口实现，WKWebView提供的接口和UIWebView命名上较为类似，区别是WKWebView的这个接口是异步的，而UIWebView是同步接口

```objc
#pragma mark - UIWebView
NSString *title = [webView stringByEvaluatingJavaScriptFromString:@"document.title"];
    
#pragma mark - WKWebView
[wkWebView evaluateJavaScript:@"document.title"
            completionHandler:^(id _Nullable ret, NSError * _Nullable error) {
    NSString *title = ret;
}];
```

#### JS通知Native
对比Native通知JS，JS通知Native就要复杂许多

![JS调用Native](https://hujiangtech.github.io/tech/assets/pic/523/img_02.png)

在iOS6之前，UIWebView是不支持共享对象的，WEB端需要通知Native，需要通过修改location.url，利用跳转询问协议来间接实现，通过定义URL元素组成来规范协议
在iOS7之后新增了 `JavaScriptCore` 库，内部有一个 `JSContext` 对象，可实现共享
而 `WKWebView` 上，Web的window对象提供WebKit对象实现共享

而WKWebView绑定共享对象，是通过特定的构造方法实现，参考代码，通过指定 'UserContentController' 对象的 'ScriptMessageHandler' 经过 'Configuration' 参数构造时传入

```objc
WKUserContentController *userContent = [[WKUserContentController alloc] init];
[userContent addScriptMessageHandler:id<WKScriptMessageHandler> name:@"MyNative"];
    
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
config.userContentController = userContent;
    
WKWebView *webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:config];
```

而 'handler' 对象需要实现指定协议，实现指定的协议方法，当JS端通过 'window.webkit.messageHandlers' 发送Native消息时，handler对象的协议方法被调用，通过协议方法的相关参数传值

```objc
#pragma mark - WKScriptMessageHandler
	-	(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {}
```

```js
// Javascript
function callNative() {
	window.webkit.messageHandlers.MyNative.postMessage('body');
}
```

至此，WKWebView的呈现和交互就已经说完了，如果没有太高的要求，这些完全可以做简单的呈现交互操作，实现一个轻度页面。

但，我们的目标肯定不仅于此，于是，重点话题来了。

### 四、重点：已知问题和解决方案
WKWebView是一个新组件，并且是采用跨进程方案，实现了比较好的性能和体验，那同样的，肯定会带来一些问题

在说问题之前，我们先看一下接入 `WKWebView` 之后的内存结构

![接入内存结构](https://hujiangtech.github.io/tech/assets/pic/523/img_03.png)

为什么APP接入WKWebView之后，相对比UIWebView内存占用小那么多，是因为网页的载入和渲染这些耗内存和性能的过程都是由 `WKWebView` 进程去实现的，相对来说APP仅占很小一部分内存，甚至因为Web进程内存的膨胀，触发App的内存警告，导致App内存占用还会下跌。

但Web进程也是会崩溃的，能致 `UIWebView` 内存爆掉的页面在 `WKWebView` 上也可能导致Web进程崩溃，只是这个上限值相对比起来会高很多，而且崩溃体现到App也没那么严重，具体结果可能就是白屏、载入失败之类等。

根据问题的特性，我们将之大体分为三类。首先是最关键的 `Cookie` 问题；其次是使用上的一些功能性问题，需要针对适配的问题；还有就是前端适配的页面问题。针对这三类问题我们一一说。

#### Cookie问题
对于Coolie问题，如下图，这张图是正常的登录Cookie认证流程，用户发起登录请求，后台登录之后，对浏览器写入Cookie，该Cookie会写入磁盘，在下次请求发起时会带上该Cookie，后台通过验证该Cookie来认证身份，确定用户已登录

![Web流程](https://hujiangtech.github.io/tech/assets/pic/523/img_04.png)

而在 `WKWebView` 上，最大的问题就在于这里，看图，在整个访问的流程中，最重要的一环，`Cookie` 写入磁盘不支持，这就导致每次重新启动APP，`Cookie` 值都会丢失，做不到会话和Native同步。

针对这个问题，我们做了一些尝试。

既然WKWebView不支持写入磁盘，那我们手动干涉这个过程，手动读取Cookie写入磁盘，然后载入时再绑定Cookie是否可行？ 
整个操作的流程如图所示，在登录之后，获取服务器返回的Cookie值，持久化到本地，在下次WebView初始化时，读取本地的缓存值手动设置Cookie

![try](https://hujiangtech.github.io/tech/assets/pic/523/img_05.png)

##### 第一步，在请求发起时设置Cookie值，

Cookie的设置分两部分，一个是当前URL的Cookie信息，二个是当前URL页面的子元素请求Cookie

前者可以通过设置Request的head去绑定Cookie，这个值只在当前请求中有效
注：这里需要注意的一点，Request信息是通过IPC进程通信传递，因为HTTPBody和HTTPBodyStream信息可能比较大，在传递过程中就舍弃了，也就说发起请求的Request不能设置body值

```objc
NSURL *url = [NSURL URLWithString:@"https://www.hujiang.com"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
[request addValue:@"key0=value0" forHTTPHeaderField:@"Cookie"];
[wkWebView loadRequest:request];
```

![request](https://hujiangtech.github.io/tech/assets/pic/523/img_06.png)

而后者子资源请求的Cookie问题，可设置 `UserScript` , 通过 `UserContentController` 在页面载入前通过JS设置

```objc
NSString *js = @"document.cookie='key0=value0;path=/;domain=.hujiang.com'";
WKUserScript *script = [[WKUserScript alloc] initWithSource:js
                                              injectionTime:WKUserScriptInjectionTimeAtDocumentStart
                                           forMainFrameOnly:NO];

WKUserContentController *userContent = [[WKUserContentController alloc] init];
[userContent addUserScript:script];

WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
config.userContentController = userContent;
WKWebView *wkWebView = [[WKWebView alloc] initWithFrame:CGRectZero c
										   onfiguration:config];
```

![child request](https://hujiangtech.github.io/tech/assets/pic/523/img_07.png)

但这种方式也不是完全没问题，如果页面包含多 'Frame'，出现302重定向时，Cookie无法跨域设置，针对MainFrame还可以通过 Delegate 的请求询问方法中手动设置，然后重新载入， 但非MainFrame就没办法了，而且Ajax请求是不会走请求询问方法，只能通过JS重写XMLHttpRequest去拦截，成本较高。我们先不考虑成本先这么做

```objc
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler
{
	if(navigationAction.request 跨域 & mianFrame) {
		NSMutableURLRequest *cookieRequest = copy navigationAction.request & set Coolie;
		[webView loadRequest:cookieRequest]
		decisionHandler(WKNavigationActionPolicyCancel);
		return;
	}
	decisionHandler(WKNavigationActionPolicyAllow);
}
```

第一步设置就完成了

##### 第二步，Cookie的存储问题
针对Form方式，通过 Delegate 的响应询问方法中，读取response.header的SetCookie信息
而Ajax类型的请求，通过JS拦截去读取相关信息

```objc
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler
{
	NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)navigationResponse.response;
    id cookies = [httpResponse.allHeaderFields objectForKey:@“Set-Cookie”];
if(cookies) {
    [CookieStorage write:cookies];
}
	decisionHandler(WKNavigationResponsePolicyAllow);
}
```

##### 第三步，Cookie写入
通过之前描述的 `UserScript` 方式，通过JS将Key=Value值写入Cookie

##### 总结，失败
而整个流程的执行结果，是失败的
原因是最关键的 `Cookie` 值是 `HttpOnly`，标记为 `HttpOnly` 的值通过JS是无法获取和设置的，而后端又不可能帽子Cookie被冒用的风险将该标记去掉。

![OAuth2](https://hujiangtech.github.io/tech/assets/pic/523/img_08.png)

总结来说，在WKWebView Cookie无法完美持久化的前提下，登录会话无法通过依赖Cookie来实现

##### 新方案：OAuth2
抛弃对Cookie的依赖，可选的方案就比较有限了，比如OAuth2，OAuth2授权认证系统能很好的解决身份认证问题，并且支持多点，而且成熟

![OAuth2](https://hujiangtech.github.io/tech/assets/pic/523/img_09.png)

如图，Web第一次发起请求肯定是未授权状态，服务端验证失败，返回重定向登录，WKWebView通过监听指定URL（如这里的重定向地址），调用App内置的OAuth2认证服务，提示用户手动授权，APP得到授权后向第三方HJ认证服务器获取认证，得到认证token，序列化到本地，同时写入WKWebView Cookie，信息部署完毕之后，WKWebView带上认证token重新发起请求，服务器通过认证服务器验证通过，返回正常服务

针对现有的问题和背景，在不改变现有结构的情况下接入WKWebView是不可能的，而以OAuth2方案来执行的话，需要各客户端实现认证授权系统，服务端新增认证服务中心，web端服务得支持授权验证

针对Cookie的问题就到这里，Cookie问题确实是WKWebView最头疼的问题，对全局架构的影响也是大家选择它时重点考评的点

#### 功能性问题
除去Cookie问题之外，其它零零碎碎也比较多，但都还算好，遵循相关的规范都能处理掉
这里我列出了收集的一些其它比较容易遇到的问题

1、WKWebView进程崩溃引发的问题
	WKWebView进程崩溃，在APP内的效果就是白屏，我们要做的就是在得知白屏时重新载入Request，iOS9下有Delegate方法能收到崩溃的回调，但在打开相册或拍照比较耗内存的情况下，WKWebView崩溃Delegate方法却不会被调用，同时我们支持的最低版本是iOS8，而在iOS8下，可以通过校验webView.title属性是否为空来确定，title属性是WKWebView内置属性，自动读取document.title值，而在进程崩溃的情况下，该值为空

2、WKWebView视图尺寸变化对页面的影响
	WKWebView也是通过ScrollView实现，设置contentInset等相关偏移值会映射到Web页面，导致页面的长度增加
	其次WKWebView的页面渲染与JS执行同步进行的，可能你JS执行时布局渲染并未完成，所以不管是JS还是Native，在页面载入完成之后就获取innerHeight或者contentSize都是不准确的，要么通过延迟获取，要么监听属性值变化，实时修正获取的值

3、默认的跳转行为，打开ituns、tel、mail、open等
	在UIWebView上，如果超链接设置未tel://00-0000之类的值，点击会直接拨打电话，但在WKWebView上，该点击没有反应，类似的都被屏蔽了，通过打开浏览器跳转AppStore已然无法实现
	这类情况只能在跳转询问中处理，校验scheme值通过UIApplication外部打开

4、 下载链接无法处理
	下载链接在UIWebView上其实也是需要特殊处理，在服务器响应询问中校验流类型即可

5、跨域问题
	https对https、http对http跨域默认是能载入的，但如果是http想载入https跨域链接，因为安全考虑，WKWebView会被拦截，这问题在引入跨域https的页面也做https。我们HJ已经切换了https，所以不存在该问题

6、NSURLProtocol问题
	UIWebView是通过NSURLConneciton处理的http请求，而通过Conneciton发出的请求都会遵循NSURLProtocol协议，通过这个特性，我们可以代理Web资源的下载，做统一的缓存管理或资源管理
	但在WKWebView上这个不可行了，因为WKWebView的载入在单独进程中进行，数据载入APP无法干涉

7、缓存问题
	WKWebView内部默认使用一套缓存机制，开发能控制的权限很有限，特别是在iOS8下，根本没方式去操作，对于静态资源的更新，客户端经常出现读取缓存不更新的情况
	针对这个问题，如果仅仅是单个资源如此，并且其它缓存比较有用，那对该资源地址加时间戳避开缓存
	如果全局都是如此，这需要手动的去清理缓存，iOS9之后，系统提供了缓存管理接口 `WKWebsiteDataStore`
	
```objc
// RemoveCache
NSSet *websiteTypes = [NSSet setWithArray:@[
                                            WKWebsiteDataTypeDiskCache,
                                            WKWebsiteDataTypeMemoryCache]];
NSDate *date = [NSDate dateWithTimeIntervalSince1970:0];
[[WKWebsiteDataStore defaultDataStore] removeDataOfTypes:websiteTypes
                                           modifiedSince:date
                                       completionHandler:^{
}];
```

而iOS9之前，就只能通过删除文件来解决了，WKWebView的缓存数据会存储在 '~/Library/Caches/BundleID/WebKit/' 目录下，可通过删除该目录来实现清理缓存

![Cache](https://hujiangtech.github.io/tech/assets/pic/523/img_10.png)

8、其它问题
还有一些零碎的小问题，比如通过写入NSUserDefaults来统一修改UserAgent；第三方库可能修改Delegate引起问题等等就不一一例举了，通过上述的问题，主要想表明出现问题的解决思路，只要不断去尝试，这些都不是阻碍。

功能性的问题比较典型的大多在iOS更新中都会完善，相信随着最低支持版本的提高，问题会越来越少

#### 页面问题
最后，还有一些可能会对前端同学造成影响的因素，我这里例句了几条

1、页面退回上一页不会重新执行Script脚本，也不会触发reload事件
	这个是因为WKWebView的页面管理和缓存机制导致的
	
2、页面键盘弹出会触发resize事件

3、window.unload只有刷新页面才会触发，退出或跳转到其它页都无法触发

根据这几条问题来看，新组件的适配主要集中在流程周期上，前端同学适配新组件过程中，可以针对周期的依赖做特殊测试

## 总结 & 结束
WKWebView虽然很新，有许多问题需要去解决，但它有着新技术带来的种种诱惑，随着iOS系统版本的不断迭代升级，问题会越来越少，功能也会越来越多，而且有较多的一线团队已经做了切换，证明业务承载完全没问题。

我相信，它就像Https一样，以后完全有可能成为iOS平台下主流浏览器。 谢谢。



