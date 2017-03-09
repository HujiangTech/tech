---
layout: post
title: 由一次 JS Code Review 说开来
categories: [Web前端]
tags: [JavaScript , Code Review]
description: 没有人写的代码是100%无bug的，所以不必感觉不好意思去向他人寻求帮助，在前端界最有经验的开发者，不论是框架的作者或者是浏览器开发人员都会周期性地提交代码审核的请求，看看代码上是否有什么是可以被优化的。

---

开始之前，先抛出一个问题：你最近一次要求别人review代码是什么时候？
毫无疑问，代码review是提升代码质量的最佳实践，如果你擅用此实践，你将从他人那里听到更多建议，从来让你的代码变得更好。

没有人写的代码是100%无bug的，所以不必感觉不好意思去向他人寻求帮助，在前端界最有经验的开发者，不论是框架的作者或者是浏览器开发人员都会周期性地提交代码审核的请求，看看代码上是否有什么是可以被优化的。

今天我们就来看看codereview的一些事情：

1. 从哪里开始codeReview
2. 怎么组织codeReview请求
3. reviewer都review些什么

我最近进行了一个javascript的codeReview, 我想分享一下我的感受，它覆盖了JS一些必须深入大脑的基础。

## 介绍
review 代码和维护很强的代码规范是相辅相承的。 但同时要注意，即便代码规范很重要，但它并不能避免逻辑上的错误，或者程序语言上对怪异模式的误解。即使是最有经验的开发者也有可能范这些错误，代码review可以有效帮助避免这些错误。

大多数人出于保护自我的本能（保护代码）会抵触甚至于否定代码review， 当然被人评论甚至指出代码有问题可能会有一点难堪，但是换一个角度说，这做为一个学习方式可以鞭策我们做的更好，成为更好的自己。事实上当我们冷静下来认真平心静气地对待，确实是这样的。

另外，请明白一件事。没有人的工作中有义务对你的代码进行反馈，如果确实能得到有建设性的反馈，请感激那个对你代码进行review的同事在此花费的时间。

Reveiw能让我们通过另外一双眼睛和另一种编程的经验来构建你的代码。通过代码review可以让我们不断改善编写代码的质量，当然是否利用这个方法完全取决于你的选择。

##从哪里开始 codeReview ?

通常来说，最有挑战的部分实际上是找到一个有经验并且你最信任的人来做这件事。下面是一个你可以请求他们进行review的地方（本文以js为例，其它语言同理）：

[Code Review](http://codereview.stackexchange.com/)
 
 Code Review 是一个看起来和StackOverflow很像的网站，但它非常有用。你可以理解为： 在StackOverflow上问“为什么我的代码跑不起来”， 而在CodeReview上问的是 “为什么我的代码看起来这么丑？”， 如果你还有任何疑问，你可以访问 [FAQs](http://codereview.stackexchange.com/faq)
 
 [GitHub](http://codereview.stackexchange.com/) 
 
 利用 github 和 [reviewth.is](https://github.com/supermatter/reviewth.is) 来做review. [详情见这里](http://stackoverflow.com/questions/3730527/workflow-for-github-based-code-review) 
 
 
## 怎么组织 codeReview 请求

下面是一些基于经验的指导原则，以便提高请求被通过的机率。 如果review的人是你的同事，你可以更随意一些。但如果是外部人员这些原则可能帮你节省不小时间：

* 把你想被review在代码进行隔离； 确保它能很容易的跑起来，forked,和添加评论；
* 尽可能的使reviewer能够很容易的看到并且修改
* 不要提交你整个网站或者工程的zip包，没有人有耐心把整个工程过一遍。
* 把你想要被review的代码放到jsfiddle, jsbin或者github上，方便reviewer很容易的Fork ，commit 。 如果你还需要diff功能，可以试试[PasterBin](http://pastebin.com/)
* 不要只留一个链接，让对方通过“View source”的方式来进行codeReview
* 明确的指出你个人希望在哪个方面得到改善, 这样能帮助Reviewer将注意力集中你希望的点上，以节省时间。
* 指出你在希望得到改善的方向上已经做了哪些研究和改进，这样有助于帮助reviewer提供除那之外的帮助（而不是你已知的方面）
* 耐心。 有些reviewer会花费数日才能完成review。 他们也有自己的事情要处理，所以不要着急。多一些耐心。

## 代码 Review 应该提供些什么？

Jonathan Betz 前 Google 开发者，曾说过： code review 理想中应该六件事情 ：

1. 正确性 代码是否像声明的那样正确完成了功能？
2. 复杂性 是否很直接的完成了目标？
3. 连续性 代码是否始终如一的完成目标？
4. 维护性 代码是否能被其它人员不用费太多努力就可以完成扩展？
5. 扩展性 代码在用户数100的情况下工作，那么在10000人的情况下是否能正确工作？ 是否有优化的可能？
6. 风格 代码是否遵守了一个特别的风格规范

我同意以上的列表内容，同时认为reviewer应该尤其注意以下一些事项，扩展之后可以成为codeReview的指导实践：

1. 提供清晰的评论，讲述知识，良好沟通。
2. 指出代码实现上的不足之处（避免过分批评）
3. 陈述为什么某一方法不被推荐，如果可能，提供博文，文档，规范， MDN页面， jsPerf测试结果以支持陈述
4. 提出替换方案，以独立的可运行的形式或者通过fork的方式集成。这样开发者就能清晰的看到他们哪里做的不好
5. 优先聚焦于解决方案，然后才是编码风格。编码风格上的建议可以在review之外给出，在此之前尽量忙于解决基本问题。
6. Review请求之外的部分。当然这完成由reviewer自己决定。 我经常会在要求的部分之外给出自己的建议，到目前为止还没有听到抱怨，所在我假定这不是一件坏事。

## 协作的 Code Review

尽管一个人单独进行Code Review 也是可以的。但是另一个方案是让更多的人参与进来。这样做有很多明显的好处： 包括减少reviewer的负担；增加更多人的建议；如果某一reviewer不少心犯错，其它reviewer可以辅助指出。

为了方便以小组的方式进行Code Review, 你可能需要一些工来让reviewer 同步查看 以及评论代码。 幸好已经有不少很棒的工具值得我们考虑：

* [Review Board](http://www.reviewboard.org/)

  这是一个遵守MIT协议基于web的工具，他集成了git, cvs, mercurial, subversion,和一些其它源代码控制系统。 Review Board 可以被安装在任何可以跑Apache的机器上，并且对个人和商业使用免费
 	
* [Crucible](http://www.atlassian.com/software/crucible)

  这是一个收费的软件，就不翻译了，有兴趣的朋友可以自行官网购买服务

* Others
  
  有一些工具不是为了Code Review诞生，但是很好用。比如免费基于Web的 [Collabedit](http://collabedit.com/) 和 我的最爱一样免费基于Web的 [EtherPad](http://piratepad.net/front-page/)
  
## 关于一次 Javascript Code Review 的经验总结

说回 review.

最近一个开发者让我对他的代码进行review 并给出一些有用的建议，以便改善现有代码。 我其实不是一个code review的专家，（别让上面的文字把你骗了）， 下面是我发现的一些问题和改善建议：

### 问题 1：

**问题:** 当函数和对象被当做参数传递的时候，未加任何类型验证。

**回复:** 为了确保输入的参数是你想要的数据类型，请必须对参数做类型验证。否则如果参数类型不符，程序会有很高的风险运行出意外的结果。对于函数，我们至少应该做到以下几件事：

1. 做必要的判断以保证传入的参数确实存在
2. 做类型检查，防止执行一个不是函数类型的参数

````
if (callback && typeof callback === "function"){
    /* rest of your logic */
}else{
    /* not a valid function */
}
````

不幸的是typeof检查还不够，正如 Angus Croll 指出的 [Fixing the typeof operator](http://javascriptweblog.wordpress.com/2011/08/08/fixing-the-javascript-typeof-operator/), 如果你使用typeof检查除function之外的类型，你需要注意不少问题。

举个栗子， `typeof null` 返回 `object`. 实际上，当 `typeof`被应用在任何一个不是函数的对象上，不管是 `Array`, `Date`, `RegEx`，返回的都是 `objct`

解决办法是使用 `Object.prototype.toString`来检测。如：

`Object.prototype.toString.call([1,2,3]); //"[object Array]"`

你可以把它抽象成一个方法，以便日后调用

````
function betterTypeOf( input ){
    return Object.prototype.toString.call(input).match(/\[object(.*)\]/)[1];
}
````

### 问题2
**问题:**  在代码中重复性的判断环境（native or 浏览器环境）

**建议:** 请使用加载时配置模式：

````
var tools = {
    addMethod: null,
    removeMethod: null
};

if(/* condition for native support */){
    tools.addMethod = function(/* params */){
        /* method logic */
    }
}else{
    /* fallback - eg. for IE */
    tools.addMethod = function(/* */){
        /* method logic */
    }
}
````

### 问题3
**问题:** 修改Object.prototype

**建议:** 这是一个很坏的方式，因为有可能造成意想不到的问题，如冲突，覆盖等。退一万步，如果你不得不修改，请一定在之前判断是否已经存在同名方法

````
if(typeof Object.prototype.myMethod != ’function’){
    Object.prototype.myMethod = function(){
        //implem
    };
}
````
### 问题4
**问题:** 一些代码严重的阻塞了页面的加载，当在处理加载数据的时候。

**建议:** 页面的阻塞会带来很糟糕的用户体验，一种解决方法是使用 'deferred' 执行（通过[Promise](http://es6.ruanyifeng.com/#docs/promise)）

### 问题5
**问题:** 使用双等号判断数字
**建议:** 应使用全等 ===

````
3 == "3" // true
3 == "03" // true
3 == "0003" // true
3 == "+3" //true
3 == [3] //true
3 == (true+2) //true
"  " == 0 // true
````

### 问题6
**问题:** 用length遍历数组时不缓存lenth
**建议:** 应缓存length,以提高性能

````
/* cached outside loop */
var len = myArray.length;
for ( var i = 0; i < len; i++ ) {
}

/* cached inside loop */
for ( var i = 0, len = myArray.length; i < len; i++ ) {
}

/* cached outside loop using while */
var len = myArray.length;
while (len--) {
}
````

这个 [jsPerf测试](http://jsperf.com/)很好的说明了该问题。


## 结论

Code Review是一个很棒的方法增强可维护性，提高代码质量，修正浅在问题，把代码提升到更高层次。  不管是对于 Reviewer 还是 Reviewee 都是很好的学习途径。

最近各大电台都在热播《女医明妃传》，你看连我们的谭允贤编写一本药膳集锦，也邀请了太医院的刘院判和程村霞一起去做CodeReview。

所以我强烈建议所有的开发者从现在的项目中开始进行codeReview吧！
