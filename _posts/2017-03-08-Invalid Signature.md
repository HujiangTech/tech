---
layout: post
title: Invalid Signature 问题排查
categories: [其他]
tags: [app , 测试]
description: 注意到上述的 application-identifier 的 App ID Prefix 和 team-identifier 不一致，会不会是这个引起的？
正好有两个同事在旧金山参加WWDC，就这个问题去咨询过苹果工程师（一位女工程师，今年貌似苹果提倡女性，很多WWDC上的session都是女性工程师和manager），她说：“this should be the same”，看来就是这个引起的。

---

苹果邮件描述如下：  
"Dear developer, We have discovered one or more issues with your recent delivery for "App". To process your delivery, the following issues must be corrected: **Invalid Signature** - Code object is not signed at all. Make sure you have signed your application with a distribution certificate, not an ad hoc certificate or a development certificate. Verify that the code signing settings in Xcode are correct at the target level (which override any values at the project level). Additionally, make sure the bundle you are uploading was built using a Release target in Xcode, not a Simulator target. If you are certain your code signing settings are correct, choose "Clean All" in Xcode, delete the "build" directory in the Finder, and rebuild your release target. For more information, please consulthttps://developer.apple.com/library/ios/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html Once these issues have been corrected, you can then redeliver the corrected binary."

为了这个问题，纠结了三天，当然最终还是解决掉了，给大家分享下这个过程

Application Loader 仅提醒无效签名，看上去就是证书的问题，反复确认过是Distribution证书，如下图

![](/content/images/2015/08/invalid-signature.png)

注意到上述的 application-identifier 的 App ID Prefix 和 team-identifier 不一致，会不会是这个引起的？
正好有两个同事在旧金山参加WWDC，就这个问题去咨询过苹果工程师（一位女工程师，今年貌似苹果提倡女性，很多WWDC上的session都是女性工程师和manager），她说：“this should be the same”，看来就是这个引起的。

好吧，下一步就是想办法去让这两个保持一致，但最后还是被坑了~，经过尝试，发现以下几点：
1. Team-identifier是每年续费时都会变更的，之前的是“3N4DUBQL4S”，而现在变成了“J52Z9F8XVP”
2. AppID是三年前申请的，并且这个App ID Prefix是无法修改的
3. AppID由于已经是AppStore正在使用的，无法删除重建
如上，根本就没有办法保持一致的。

找了一下之前上线的App，沪江网校和沪江CCTalk都是不一致，提交应用完全无压力，因为最近几天他们还被提交过的。为了进一步验证上述结论，我新建了一个空工程，将bundle ID和Icon都设置为这个问题App一样，证书也是一样，当然这里的application-identifier 的 App ID Prefix 和 team-identifier 也是不一致的，经测试，确实可以正常提交。看来问题真不出在这里～

这下真的傻眼了，问题似乎无解了～

看到一篇博客，也讲到上述问题，如下：
http://blog.sina.com.cn/s/blog_5e6fd4290102ux8d.html

里面有提到过签名的一些基本流程：
>Invalid Signature属于比较简单的状况，可以很清楚问题出在签名部分。CodeResources存储了所有资源文件在包中的路径和Hash值，整个包还有一个整体的Hash值，这个文件可以起到校验苹果收到的包（或者用户下载到的包）是完好还是损坏的作用，出了问题的包当然不会做进一步处理；codesign对包做签名时会把.*隐藏文件包含进去，但Xcode打包时并不把这类文件拷到目标包内。也许Xcode工具链认为签名是程序自动生成的用户不会手工干预，一定是可靠的就在提交前没有做CodeResources的验证，结果就出了篓子，工具链的乌龙事件让我花去了两天时间来解决，特在此记录，也许能帮助到搜到这里的同学。此事已向苹果反馈，最简单的至少他们可以让报错邮件再详细一点，不要Invalid Signature这么笼统。

是否工程里有一些隐藏文件被打包进去了呢？ 好吧，接下来就是长达三天的查找过程，但最终还是没有发现特别的地方，期间有把所有的.md，.js等全部清除，重新打包，上传，如此反复尝试，但最终还是没有解决掉。

stackoverflow上也是有不少这个问题的答案，但都不得法，其中有一个提到将工程重建后解决了上述问题，但都没有提具体的原因是什么，估计也搞不清原因是什么，但我们这个项目是三年前的，要重建下工程真是费劲呀～

时间一点点的流逝，但问题仿佛更加的扑朔迷离，难道真的无法解决呢？

等等，这次提交审核的版本最大的一个改动是支持arm64，难道是这个引起的？
接下来我开始审查距上次发布版本后的每条Git提交记录，一条一条的过。。。

最终我发现了一个非常奇怪的事情：
**这个项目有一些库是用CocoaPods管理的，在转换64位时发现只有一条修改Pods.project的记录，仅仅所有的Pods库的Architectures修改为${ARCHS_STANDARD}，其余文件没有提交记录。看来是某人直接手动修改的，并没有使用“pod update”方法自动转换。**

**如此，使用 “pod update”重新构建了一把，发现pods库中有不少文件修改，很有可能之前发现某些库有些问题，但没有通过Pods库更新来解决，直接在本项目中修改Pods库的源码。这个可以解释为啥某人在处理64位问题上，没有采用pod update方法。**

好吧，通过上述的方法，终于提交成功了。

这个问题排查过程真是浪费了不少时间，当然写这篇总结还是有必要的。
>1. 一切要以事实说话，即便是一些有权威的建议，带着质疑态度去处理问题还是有帮助的。比如此次苹果的建议，确实没有定位到问题根本。
2. 有些历史坑，迟早是要填的。当下我们应当做好手头的每件事情，不要留下隐患。
