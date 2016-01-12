---
layout: post
title: 领域对象建模：保持数据的完整性
keywords: OO, Object Oriented, Domain Model
description: Keep integrity of data when building domain model
category: OO
tags: [OO, Domain Model]

---

上一节中，我们介绍了通过createdAt和updatedAt两个字段可以简化领域对象的设计，因为它们捕捉到了最常用的数据，可以简化模型设计。但是常用不代表够用，数据的完整性是任何一个系统必须考虑的。请看：

小明设计了PromotionItem表示促销商品，并知道商品的最新状态和更新时间。春节过后，框框网的促销专区还算可以，不及预想的那么火爆，但是销量比之前上升明显。业务人员想从中做一些数据分析，比如商品从上架到卖出花费多长时间，从而知道哪些类型的商品好卖，以及跟折扣的关系。

这种情况下，小明是没法数据分析的，因为他的建模缺少了历史数据：商品几月几号上架，几月几号下架了，几月几号卖出了。这些实实在在发生过的数据，在PromotionItem一个对象中是很难完整保留的。即使采用之前的onlineTime/offlineTime/expireTime等多个字段也不行，因为商品有可能下架之后重新上架。

所以一旦业务有了新的需求，就无法满足需要。更要命的是，如果系统已经上线，业务再提出这些需求，我们就永远失去了这些历史数据。

这种情况我们叫做丢失了数据的完整性。

此时，我们在建模时，可以给关键领域对象设置关联的历史纪录（或者事件），比如PromotionItemHistory（当然也可以是PromotionItemEvent，一样的思路），把促销商品的每一个状态变化记录下来：

![promotion item history model](/assets/images/promotionItemHistory.png "sequence diagram")

有了历史记录，就可以真正做到“手中有粮，心里不慌”了。这与徐昊在InfoQ上的文章《》其实是类似的思路：通过时标查找重要的领域对象，每个时标都会带来领域对象的变化，而且很多时候是状态的变化，并且把状态变化捕捉下来，以达到数据的完整性。

通过记录状态的变化，可以很容易和基于事件的，或者事件驱动的架构结合起来。比如，业务又来新需求了：每当有产品卖出的时候，业务的头想立刻收到邮件...

这个功能本身不难实现，但考虑到发送邮件不是业务的核心功能，而只是业务支撑，或者业务的增强，我们可以把发送邮件的代码与业务的核心代码分开。怎么分呢，发送邮件的Service可以监听商品的状态发生变化，并适时实现自己的功能，从时序图来说，就是从原来的：

![promotion service sequence](/assets/images/promotionServiceSequence.png "sequence diagram")

替换为下面的实现：

![promotion service sequence with listener](/assets/images/promotionServiceListener.png "sequence diagram")

通过修改，把短信和商品的紧耦合编程了松耦合。而且，随着业务量的增大，如果这里出现了性能瓶颈，可以考虑把同步的通知改成异步，甚至可以考虑引入消息中间件——当然，这取决于系统的规模和业务量。
