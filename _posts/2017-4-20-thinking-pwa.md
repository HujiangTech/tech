---

layout: post
title: Thinking in FE 谈谈 PWA 的那些事
categories: [iOS]
tags: [PWA，Service Worker，Offline Web]
description: Web 前端是一个发展迅速，更新频繁，充满野心的领域。一群开发者，以及浏览器甚至操作系统的产商都在思考着，如何让 Web 前端统一整个大前端，这一天似乎很遥远，却又感觉一天天的在接近。
---

>   作者：翁阳 （沪江iOS架构师）    
>   本文原创，转载请注明作者及出处。

Web 前端是一个发展迅速，更新频繁，充满野心的领域。一群开发者，以及浏览器甚至操作系统的产商都在思考着，如何让 Web 前端统一整个大前端，这一天似乎很遥远，却又感觉一天天的在接近。

近几年，随着 HTLM5 在移动端上的盛行，出现的各种技术、框架、解决方案，无一不是在向这样的目标前行。从 Hybrid 框架（_Cordova、ionic_），到一些大厂成熟的 Web 容器（_微信、支付宝_），再到最近的微信小程序，甚至是 react-native、weex，都在不断将 Web 前端推向更高的高度。

或许真有那么一天，iOS 开发没人要了，当然 Android 也只能被“呵呵”了，所以，趁着骨骼惊奇赶快撸一把 Web 前端吧，本篇我们来谈谈 PWA 的那些个~~破~~事。

<!-- more -->

## PWA 简介

什么是 PWA？英文全称是`Progressive Web Apps`，也就是所谓的**渐进式增强的 Web App**，这个名词最早应该是在 2015 年被谷歌的 Alex Russell 在[《Progressive Web Apps: Escaping Tabs Without Losing Our Soul》](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/)这篇文章中提出来，GitHub 上的 awesome 系列也有一个 [awesome-pwa](https://github.com/hemanth/awesome-pwa)，其中便收录了这篇文章。要了解 PWA，最核心的便是`Progressive`，也就是**渐进式**，一个 Web 页面应该是可以按照用户的选择，渐进地增强为一个 App，而非一开始就运行与受限的环境中（_Hybrid_）。

所以，一个 PWA 应该是可以良好的运行在浏览器里，随着用户的选择，逐渐贴近于原生的体验。这听起来似乎很酷，也与现在多数 Web 前端大一统思想大相径庭，但这并不是一个新的概念，或许只是创造了一个新的名词。

## 创造了一个新名词

PWA 给用户最直观的表现，便是从一个浏览器上浏览的页面，变成一个可以从桌面上打开的 App，但这其实并不是啥新鲜事。早在 2009 年，微软发布的`Silverlight 3`中便增加了一条新功能：**允许在浏览器之外运行**。用户在浏览器打开 Silverlight 页面时，如果开发者有针对桌面端做支持，可以右键选择添加到桌面，这样 Silverlight 便脱离了浏览器，运行在 OOB 模式下（_Out Of Brower_），同时也获得了访问本地文件系统的能力。看！虽然不是移动环境中，但这不正是一个非常`Progressive`的表现么？

另外，如果说在移动环境中，那么 PWA 最直观的表现，便是添加一个网页的图标到桌面，然后能全屏运行，所以下面的图中（_摘自网络_）的某人笑了：

![](/images/2017/04/15/01.png)

也就是说，PWA 并不是一个新的概念，只是被人有意地总结了出来，从而创造了这样的名词。名词的好处在于方便传播，当然传播并不限于技术圈，而名词的制定者通常会有更大的话语权。

## 一个合格的 PWA

虽然说 PWA 并不是什么新鲜概念，但既然谷歌制定了这样一个名词，便会有更加详细、严谨的解释。那么一个合格的 PWA 应该具备哪些个特征呢？官方给出了这样 10 条描述：

> * **Progressive** - Works for every user, regardless of browser choice because it's built with progressive enhancement as a core tenet.
> * **Responsive** - Fits any form factor: desktop, mobile, tablet, or whatever is next.
> * **Connectivity independent** - Enhanced with service workers to work offline or on low-quality networks.
> * **App-like** - Feels like an app to the user with app-style interactions and navigation because it's built on the app shell model.
> * **Fresh** - Always up-to-date thanks to the service worker update process.
> * **Safe** - Served via HTTPS to prevent snooping and to ensure content hasn't been tampered with.
> * **Discoverable** - Is identifiable as an "application" thanks to W3C manifest and service worker registration scope, allowing search engines to find it.
> * **Re-engageable** - Makes re-engagement easy through features like push notifications.
> * **Installable** - Allows users to "keep" apps they find most useful on their home screen without the hassle of an app store.
> * **Linkable** - Easily share via URL, does not require complex installation.

总结下来，差不多也就是：渐进式、响应式、可离线、可全屏、可更新、安全、易发现、可推送、可安装、可分享。无论你用怎样的技术手段，只要满足了谷歌所说的这些条件，也就是一个合格的 PWA 了。似乎 PWA 与具体技术无关，只是一种表象，但配合它的表象，谷歌的确出现了一些新的技术，其中非常核心的便是`Service Worker`。

## 实践一下

既然有新的技术引入，那么实践是用来学习的最佳途径，在我们动手之前，不妨先来探讨下面几个小问题。

### Javascript 的线程

如果你面试过 Web 前端，可能会被面试官问到这样的问题：Javascript 是单线程还是多线程？这时候一些同学可能想也没想，就直接回答道：“单线程！”，另一些稍微狡诈点的同学可能会回答：“应该是多线程吧！”。其实我想说的是，这个题目本身就不够严谨，作为一门编程语言，和是否多线程并没有太多必然的联系，最简单的，我们可以通过`JavascriptCore`来实现一个 Javascript 的运行宿主，并且可以给它提供多线程相关的能力。所以这个题目应该需要一个前提，比如：在浏览器中，Javascript 是单线程还是多线程？

答案依旧不是“单线程”，所以单线程的同学，要注意了。在 HTML5 中，有个很有用的技术叫`Web Worker`，并且现在市面上大多数浏览器都已经支持，Web Worker 使得 Web 前端可以在另外一个线程中处理一些比较耗时的操作，自然也就是支持**多线程**了。

那么随着 PWA 的到来，一个名称非常类似的技术也随之诞生，也就是`Service Worker`。与 Web Worker 类似，Service Worker 也是在另一个线程中，但它的生命周期并非页面级的，而是会一直运行在浏览器中，作用于多个页面，并且，它拥有了更多更强大的能力。

### Web 前端缓存

为了能够提供更加良好的用户体验，Web 前端在缓存这块也是煞费苦心，目前用于缓存的手段大约有下面这些：

* Cookie
* Web Storage（Local Storage / Session Storage）
* IndexedDB

这些缓存手段都是用来存储从服务端获取到的数据，避免每次请求带来的延迟问题，但如果要想能够在完全脱离网络环境的情况下浏览页面，光有这些还是不够方便的。所以，HTML5 中有一个叫`Application Cache`的设计，允许通过配置一个`Manifest`来指定缓存哪些页面，以便在离线时提供降级服务，但`Application Cache`在使用起来还是有诸多的问题，比如：

* 动态页面无法很好缓存
* 缓存更新不够方便
* 不够安全
* ...

对于 Application Cache 的缺陷，[《Application Cache is a Douchebag》](https://alistapart.com/article/application-cache-is-a-douchebag)这篇文章做了一个很详细的吐槽。为此 Service Worker 出现的另一目的就是要废弃掉 Application Cache，Service Worker 拥有拦截页面请求的的能力，在不同的网络环境或者出错失败的情况下，可以决定是否返回已缓存的内容来进行降级。相比于 Application Cache，Service Worker 在很多方面都更加可控，将缓存逻辑交给了开发者而非为浏览器制定默认行为，这是一个更加明智的决定。

### 亲自试一下

完成一个 PWA 的核心基本都是围绕着 Service Worker 来展开的，所以掌握了 Service Worker，基本上完成一个 PWA 也就没有太多问题了。除了 Service Worker，还要顺带着去熟悉下 Notification、Push、Fetch，这些也都不过是些 API，实践下基本也就都能搞清楚了。

如果你已经准备好了，不妨按照官网 [Your First Progressive Web App](https://codelabs.developers.google.com/codelabs/your-first-pwapp) 来一步步的去实现吧！

## 未来还是未来

PWA 给 Web 前端的大一统计划又多了一条选择，但这个选择从当下看来还是非常遥远，毕竟 Safari 目前还尚未支持 Service Worker，也就是说，iOS 还没有买 PWA 的单。大一统这条路注定是艰辛的，但随着硬件设施的越来越强悍，以及 Web 前端技术不断翻新的情况下，这条路似乎注定是最终唯一的选择。现在的 Javascript 都开始支持`byte code`了，也就是说浏览器已经承担了编译器的相关职责，你说在这群执念深刻的人推动下，这样的未来还有多远呢？

或许 Web 前端与你目前而言无法学以致用，但作为基层开发者的我们，学学忘忘，这其实很正常，有过痛苦才知道众生真正的痛苦，有过执着，才能放下执着，有过牵挂，了无牵挂。把一件事情放到不同的高度去看，心境和心态也会完全不同，回望你这一路走来，难道那些被你遗忘的，就真的对你没有任何潜移默化的影响么？