---
layout: post
title: "Quartz Job之cron/Fixed Interval/Fixed Delay"
description: ""
category: Java
tags: [Java, Quartz]
---
对应用中的调度任务应该有3种：基于cron的;固定间隔的(Fixed Interval);固定延时的(Fixed Delay)。

### 1. 基于cron
这比较容易理解，只需要给出一个cron的表达式，就可以根据表达式去执行。比如
"0 0 12 ? * WED" 代表"每周三的12:00”
它的几个部分分别代表：秒、分、小时、基于月的天;月;基于周的天;年（可选）
如果你使用Quartz，可以参考Cron Trigger一章 。
 
### 2. 固定间隔的
这个也很常用，比如每天执行一次，每小时执行一次。在Quartz中，有对应的SimpleTrigger ，可以指定开始结束时间和重复间隔，比如：new SimpleTrigger("trigger name", "group name", startDate, endDate, repeatCount, repeatInterval)
 
### 3. 固定延时的
固定延时是指当前Job结束后，过固定的时间再执行下一次任务。
可惜的是，Quartz本身并不提供固定延时机制。所以只能根据情况hack。
 
最近项目遇到的一个问题是，本来想使用固定间隔，但是Job本身是长时的，有可能间隔时间到了，当前Job仍在执行。这时，如果启动新的Job，可能会引起状态不一致。

针对这种情况，因为同时执行会有状态不一致的问题，所以之需要保证Job不同时执行就好了。所以可以在上一个Job结束后再立即执行新Job。


解决方法就是让你的Job实现StatefulJob。因为是有状态的，Quartz会在标记该Job状态为“blocked”，直到Job结束，彩会改成“Awaiting”。所以只有上一个Job结束后才会执行下一个。