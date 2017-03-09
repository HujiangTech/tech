
---
layout: post
title: Android开发之离线hybrid解决方案
categories: [移动端]
tags: [Android, 客户端, 移动]
description: 本文将会对在开发`hybrid`离线包中遇到的问题、对`WebView`请求拦截以及调整、`WebView`中所有请求`header`的添加、`Cookie`方面的使用、Debug技巧等这几方面做介绍。
---

> `Hybrid`本身的意思是混合的，其实用在这里，就是指的是原生`native`和`Web`开发混合起来，各展所长。

* 最近在做`Android` `hybrid`方面的开发和研究。有一些关于离线`hybrid`方面开发的心得体会。
* 本文将会对在开发`hybrid`离线包中遇到的问题、对`WebView`请求拦截以及调整、`WebView`中所有请求`header`的添加、`Cookie`方面的使用、Debug技巧等这几方面做介绍。

#### Hybrid离线包

因为`hybrid`方案上我们使用`WebView`加载，如果是在线访问，速度上会比较慢，体验不佳，所以我们采用在本地使用离线包的形式、这样来提升加载速度，从而不受网络的影响。

按以上做法就需要使用 `file:///`协议来加载本地离线`web`页面，这样使用起来发现会导致一个问题，服务端去拿存储进去的`cookie`值，在大部分`Android`手机和部分`iPhone`手机拿不到，

经过分析，发现应该是因为协议的问题，我们用的是`file:///`协议,而用`http://`协议就没有这个问题。

这样的话离线就遇到这样的问题，要么服务端做一个兼容，不用`cookie`,这样对于他们来说比较麻烦。或者是使用JS调用原生方法，每一个网络请求都起客户端进行封装，这样也是会导致工作量很大。

然后发现`webView`有这样一个方法`shouldInterceptRequest`,这个方法会在每一个请求执行前，进行拦截，然后开发者可以任意处理后，再返回一个处理后的网络请求`WebResourceResponse`，这个方法真是太酷了。

*  与是我们就可以将本地`file`协议先伪装成`http`协议，先随便请求一个网络地址，这个地址是什么不重要，只要这个地址的后缀和首页的后缀一样就行了

```
// 加载本地文件
mWebView.loadUrl("file:///android_asset/test.html");
```

```
private String mLocalUrlPrefix = "http://test.com/app/";

```
```
mWebView.loadUrl(mLocalUrlPrefix + currentVersion + "/test.html");
```

* 加载后，在此处进行拦截所有的请求，然后做处理，将所有的请求全部转换为本地文件、读取本地文件流，然后将文件流与文件格式拼接为`WebResourceResponse`.返回给`WebView`。

```

@Override
    public WebResourceResponse shouldInterceptRequest(final WebView webView, WebResourceRequest webResourceRequest) {
        final String url = webResourceRequest.getUrl().toString();
        if (url.contains(mLocalUrlPrefix)) {
            return handleRequest(url);
        } else {
            return getWebResourceResponse(url);
        }
    }
```

* 其中 `WebResourceResponse` 主要是由三个部分组成

```
 public WebResourceResponse(String mimeType, String encoding, InputStream data) {
        mMimeType = mimeType;
        mEncoding = encoding;
        mInputStream = data;
    }
```

其中 `mimeType`为请求文件的类型、`encoding`为文件的编码、`data`为文件的`inputStream`

`mimeType`这个可以根据文件后缀来映射，或者用第三方开源的工具，`encoding`我们一般就用`utf-8`，文件流就直接读取就行了。

例如读取本地文件流，这样读取：

```
if (TextUtils.equals(version, OfflineHtmlManager.DEFAULT_VERSION)) {
            inputStream = AssetUtils.getFromAssets(this, fileName);
        } else {
            try {
                String path = getApplicationContext().getCacheDir().getAbsolutePath() + "/" + fileName;
                File file = new File(path);
                inputStream = new FileInputStream(file);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
        }
```

* 这样就完美的将本地`web`页面`file`协议请求伪装成了`http`协议的请求，这样cookie的问题就解决了。


#### webView中的所有网络请求都要添加自定义header

* 肯定有很多产品会希望webView中的所有网络请求都要添加自定义header,但webView只提供了一种添加header的方法

```
  public void loadUrl(String url, Map<String, String> additionalHttpHeaders) {
        throw new RuntimeException("Stub!");
    }
```

但这种方法只能在`loadUrl()`中添加，其它页面中的请求就添加不上了，那怎么办呢？

此时我们一思考就会发现，用上面`shouldInterceptRequest`这个拦截所有请求的方法就能解决这个问题。
我们在所有网络请求到达时，拦截，然后用`http`请求的方法，先添加header，然后去请求这个文件流，然后返回组装成`webView`需要的`WebResourceResponse `。

```
  try {
    String method = webResourceRequest.getMethod();
  			  if (TextUtils.equals(method, HttpGet.METHOD_NAME)) {
                httpRequest = new HttpGet(url);
            } else if (TextUtils.equals(method, HttpPost.METHOD_NAME)) {
                httpRequest = new HttpPost(url);
            } else if (TextUtils.equals(method, HttpHead.METHOD_NAME)) {
                httpRequest = new HttpHead(url);
            } else if (TextUtils.equals(method, HttpPut.METHOD_NAME)) {
                httpRequest = new HttpPut(url);
            } else if (TextUtils.equals(method, HttpDelete.METHOD_NAME)) {
                httpRequest = new HttpDelete(url);
            } else if (TextUtils.equals(method, HttpTrace.METHOD_NAME)) {
                httpRequest = new HttpTrace(url);
            } else if (TextUtils.equals(method, HttpOptions.METHOD_NAME)) {
                httpRequest = new HttpOptions(url);
            }
            DefaultHttpClient client = new DefaultHttpClient();
            HttpPost httpPost = new HttpPost(url);
            httpPost.setHeader(MY_USER_AGENT, RunTimeManager.instance().getUserAgent());
            HttpResponse httpResponse = client.execute(httpPost);
            InputStream responseInputStream = httpResponse.getEntity().getContent();

            String[] temp = url.split("\\.");
            String suffix = temp[temp.length - 1];
            String encoding = UTF_8;

			  // get file suffix
            suffix = suffix.replace("?","-");
            String[] suffixStrings = suffix.split("-");

            suffix = suffixStrings[0];
            // According to the suffix to get content type.
            String type = Html5MimeUtil.getMimeContentType(this, Html5MimeUtil.MIME_TXT_NAME).get(suffix);
            return new WebResourceResponse(type, encoding, responseInputStream);
        } catch (ClientProtocolException e) {
            //return null to tell WebView we failed to fetch it WebView should try again.
            return null;
        } catch (IOException e) {
            //return null to tell WebView we failed to fetch it WebView should try again.
            return null;
        }
```


#### Cookie问题

* 在使用第三方微博登录时，发现当用户没有安装微博时，微博web端会在登陆成功后清除整个应用`webView`的`cookie`,这个就导致此时我们的`cookie`丢失，失效的问题，怎么解决呢？
* 其实仔细研究发现`webView`也为我们提供了非常有用的cookie设置和cookie读取问题

* 我们可以首先要读取`cookie`，放在内存中：

```

if (cookieManager.hasCookies()) {
            mCookie = cookieManager.getCookie(LoginActivity.DAMAIN);
        }    
```
* 然后在微博将`cookie`清除后，将`cookie`再保存进去
```
private void saveCookies() {
        if (!TextUtils.isEmpty(mCookie)) {
            CookieSyncManager.createInstance(mContext);
            CookieManager.getInstance().setAcceptCookie(true);
            CookieManager.getInstance().setCookie(LoginActivity.DAMAIN, mCookie);
            CookieSyncManager.getInstance().sync();
        }
    }
```

这样问题就方便地解决了。

#### Hybrid Debug 技巧

1. Android版本必需是4.4及以上
2. 打开Android设备调试状态
3. 在使用WebView的代码中加入以下代码,从而开启调试状态。  可参看如下  
4. Chrome 30及以上版本才支持Android设备调试
5. 在Chrome地址栏，输入“about:inspect”或通过“菜单”->“工具”->“检查设备”打开设备检查页面,DevTools工具会自动检测已连接设备运行的可调试页面列表，点击对应页面的“inspect”链接打开调试页面。

6. 在页面中，可以做如下：

	* 在Elements下查看DOM结构
	* 选中DOM元素后，在设备上会高亮显示，右侧Styles下修改CSS属性可即时生效：
	* 在Sources下断点调试JavaScript
	* 在Console中可运行调试（例如在Android中注入一个java对象别名为："app"）,里面有方法test(). 此时可调用app.test()来直接调试。非常方便


* 开启调试状态

```
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
     WebView.setWebContentsDebuggingEnabled(true);
   }
```


* 比较坑的地方

1. 点击"inspect"时，如果遇到启动了一个白屏界面，说明要翻墙才能使用。一般情况下，只在第一次使用"inspect"时需要翻墙，以后会缓存在本地。
2. 一定要打开调试状态且版本要>=4.4 还要检查chrome版本

* 其它

* 前端也可以用HBuilder来配合调试应用
* 应用启动后则可通过Chrome的DevTools工具连接进行调试。


#### 总结 

* `Hybrid`是一种很好的方案，`webView`提供的功能很强大，后续有更多有好的想法会持续分享给大家。从目前来看，体验上和原生已经很接近，开发效率上和灵活上比原生要高。
* 这是一个种很好的方案，两个平台解决方案都是想通的，iOS里的这个方法叫`cachedResponseForRequest()`.

