---
layout: post
title: Thinking in FE 更好用的 UIWebView  
categories: [iOS]
tags: [UIWebView, Kaki]
description: 本篇我们来谈谈`UIWebView`，虽然在 iOS 8.0 之后更加推荐使用`WKWebView`，但在你没放弃 iOS 7.0 之前，不妨看看如何让这陈旧的`UIWebView`更加好用些，当然这里的一些思想同样可以迁移到`WKWebView`中。

---

继上一篇 Thinking 以来过去太久时间了，这些日子里一直在疲于奔命，无论如何繁忙，静下来写写博客还是一件非常值得的事情。

本篇我们来谈谈`UIWebView`，虽然在 iOS 8.0 之后更加推荐使用`WKWebView`，但在你没放弃 iOS 7.0 之前，不妨看看如何让这陈旧的`UIWebView`更加好用些，当然这里的一些思想同样可以迁移到`WKWebView`中。

<!--more-->

## 用 WebView 的动机

在深入这个话题之前，我们先静下心来想想，一般在什么情况下我们会想到使用`WebView`？作为大前端名副其实的粘合剂，`WebView`无疑是移动端和触屏端最直接的连接方式。那么这个答案就很简单了，我们要接入触屏页面（_HTML5_），那为什么我们要接入触屏页面呢？或许是因为下面这些点：

* 跨平台，减少开发资源
* 热更新，避免发版等待
* 多团队研发，集成成本低
* 作为平台扩展，开发者门槛低

作为一个能够将触屏页很好的嵌入的`WebView`而言，并非简单的加载一个`URL`就足够的，我们一直都在致力于将触屏的体验和原生拉进，所以，原生和触屏之间的交互必不可少。

## 浅谈 JS && Native 交互

如何在`UIWebView`实现`Javascript`和`Native`的相互调用呢？目前大致有两种方案：**JSBridge**、**JSCore**。

### JSBridge

`JSBridge`实现的原理其实很简单，核心依赖于`UIWebView`和`UIWebViewDelegate`的两个方法：

1. `stringByEvaluatingJavaScriptFromString:`
2. `webView:shouldStartLoadWithRequest:navigationType:`

第一个方法用于`Native`调用`Javascript`方法，实现了原生调用触屏这一条道路。那么触屏如果想调用原生，可以使用`iframe`构建一个`load`请求，原生端通过第二个回调方法进行拦截，根据`URL`中所传递的信息重定向到特定的原生方法，虽然有点绕，但这条道路依然是行通了。

对于此方法，已经有相关比较好的开源库，比如 [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)。

### JSCore

`JSCore`也就是`WebKit`中的`JavascriptCore`，这是一个非常直接的互通方式。最核心的类是`JSContext`，通过这个上下文，我们可以注入一个 OC 对象到 JS 的运行环境中，我们也可以通过这个上下文直接调用 JS 方法。具体就不展开，可以参考`JavascriptCore`的 API 文档。

## 如何组织好代码

在给`UIWebView`增加了很多业务相关功能后，常常会发现这个`Controller`会变得很庞大，又或是出现了很多类似的`Controller`，这通常会给后续的维护带来困扰。那么如何更好的来组织这些代码呢？

在面向对象的程序设计里，针对结构优化上通常有两个方向的思维：**提取间接层**和**拆解再组合**，当然这是我总结出来的思维方式。提取间接层是一种纵向的维度，通常包括：

* 提取公共基类
* 增加 Mediator，隔离依赖

而“拆解再组合”是一种横向的维度，将一个大的代码结构进行模块划分，最后再进行组合。可以看我之前的这篇文章：[《组合化繁为简的力量》](http://blog.makeex.com/2016/04/23/the-design-pattern-of-composite/)。

面对不同的结构问题，不同的人优化的方式也会有所不同，我更加倾向于“拆解再组合”。过多的层次很容易导致调用路劲过长，我觉得这并不易于理解，另外面向对象中常说**组合胜于继承**，这其中的缘由也就不展开了。

## 一个轻量、无侵入的方案

KakiWebView，一个简单易读的封装，核心思想便是采用了**拆解再组合**，以下是我设计的初衷：

* 对于现有的`UIWebView`无侵入性的使用
* 可扩展性强，可实现自定义扩展
* 简单易用，学习成本低

**项目地址**：https://github.com/prinsun/KakiWebView 

无论你现在的`UIWebView`是以何种方式使用的，都可以轻而易举的使用`KakiWebView`来获得扩展能力，该项目中已经内置了一些`Plugin`：

* KakiJavascriptCorePlugin：可注入 OC 对象到`Javascript`环境中，使用上`JSCore`的交互方式
* KakiProgressPlugin：基于`NJKWebViewProgress`的一个进度条插件，安装该插件后`UIWebView`加载页面时将会带有进度显示
* KakiPopGesturePlugin：屏幕边缘侧滑返回的插件，模仿 Safari 的回退效果
* KakiTitleObserverPlugin：监控网页 Title 变化的插件，可以实时获取到网页标题变化

这些内置的插件非常通用，也是大多项目中所需要的，使用的方式非常简单：

```objc
// 启用 Kaki
[self.webView setEnableKakiPlugins:YES];

// 安装 Kaki 插件
[self.webView installKakiPlugin:[KakiProgressPlugin.alloc init]];
[self.webView installKakiPlugin:[KakiPopGesturePlugin.alloc init]];
[self.webView installKakiPlugin:[KakiTitleObserverPlugin.alloc init]];

// 配置插件
__weak __typeof(self) wself = self;
[self.webView.titleObserverPlugin setOnTitleChanged:^(NSString *title) {
    wself.titleLabel.text = title;
}];
self.webView.progressPlugin.progressColor = [UIColor redColor];
```

内置的这些插件是你学习如何自定义插件的很好示例，详细请阅读源码。

## 构建适合你的容器

当然 KakiWebView 绝对不能满足你所有的业务需求，而这个项目的初衷也并不是要完成一个大而全的万能`WebView`，正确的定位是**一个提供了扩展`UIWebView`能力的基础组件**。通过这样一个基础组件，自定义符合你业务需求的`Plugin`，从而构建一个真正符合你所期望的容器。

如果你并不喜欢`JSCore`的交互方式，你完全可以自定义一个`Plugin`来实现`JSBridge`的交互方式。另外可以通过自定义一个`Plugin`配合`NSURLProtocol`，实现离线浏览这样的功能。总而言之，有了这样的一个基础组件，无论是从代码的组织还是后续的扩展，都有很大的帮助。

那么，放飞思想，重构又或是去构建一个更强大的 Web 容器吧！

