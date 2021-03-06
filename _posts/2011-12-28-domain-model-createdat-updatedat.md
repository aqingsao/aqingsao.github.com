---
layout: post
title: 领域对象建模：正确使用createdAt和updatedAt
keywords: OO, Object Oriented, Domain Model
description: Proper use of createdAt and updatedAt to build domain model
category: OO
tags: [OO, Domain Model]

---

编程时，大家会经常用到两个字段，createdAt和updatedAt。不但很多公司有这样的数据库规范要求，一些框架对它们也有天然的支持。不过你知道如何正确使用它们吗？别忙，先让我们看一个小例子。

框框网是一家刚起步的B2C网站，为了招揽人气，准备在春节前搞一个促销专区，吸引大家抢购。做为框框网的开发人员，小明参与了这个任务的开发。他收到的需求是：管理员可以在指定时间把促销商品上架，用户可以看到并购买。小明有一定的工作经验，稍做分析，就建立了名为“促销商品”（PromotionItem）的对象，包含两个状态：NEW和ONLINE，分别表示新建和已上架。并且设计了一个额外的字段onlineTime，如图所示：

![onlineTime model](/assets/images/onlineTime.png "onlineTime model")

这样，不但可以知道促销商品是否已经上架，如果需要的话，管理员也可以知道上架的时间。
开发完成以后，一切工作正常。几天后，小明接到业务的反馈：如果缺货或者万一出错，管理员可以把促销商品撤下来，即下架。小明挠了挠脑袋，添加了一个新的状态：OFFLINE来表示下架。但是如果不添加offlineTime，万一业务想查看商品啥时候下架怎么办？考虑来考虑去，又在数据库表上添加了一个字段offlineTime。同时他想到应该把商品的卖出时间也记录下来，干脆一块加上：

![offlineTime model](/assets/images/offlineTime.png "offlineTime model")

添加一个状态还比较容易，但添加字段就麻烦了，要写数据库脚本，而且根据框框网的开发规范，所有新增的SQL脚本都要汇报给开发的李头，再找DBA审查。这一来一去耽误不少功夫。不过好歹改完了，审查也通过了。小明暗自埋怨业务怎么不一次说清楚，折腾来折腾去的多麻烦。

又过了一段时间，小明突然被告知，原来做的上下架部分需要有点小改动，这些促销商品只能在指定期限内内卖，过了促销期就不再卖了。用户看不到，但是管理员可以在“过期的促销商品”里面看到。略加思索，小明心中有了数，应该如何修改。首先要添加一个新的状态EXPIRED，但是要不要加一个字段expiredTime呢？

![expireTime model](/assets/images/expireTime.png "expireTime model")

他很纠结，时间字段越加越多，不但上面麻烦的流程要重新走一遍。万一业务再改怎么办？

其实他大可不必如此纠结，甚至这些字段全都不要。因为框框网的数据库规范上已经标明，任何数据库表都要有createdAt和updatedAt两个字段，只是他从未用过，曾经问过别的同事，但大家都说不出所以然来。我们使用这两个字段之后，发现促销商品简单多了：

![updatedAt model](/assets/images/updatedAt.png "updatedAt model")

在这个图中，createdAt表示促销商品的创建时间，updatedAt则根据状态而定：如果当前状态是已上架，则updatedAt表示上架时间；如果当前下架，updatedAt表示下架时间；如果当前过期，则表示过期时间。 
通过两个字段，不但可以知道商品的最新状态，也可以知道发生的时间。小明之前纠结的问题解决了。即使业务有类似新的需求，都不必改动太多。

结论：促销商品是一个领域对象，从新建到上架，再到下架或者过期，它状态的迁移明确表明了业务的流程变化。对这类领域对象，我们关注它的状态，可以借用createdAt和updatedAt来捕捉一些重要的数据，从而简化领域模型。
