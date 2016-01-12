---
layout: post
title: "Hibernate中get()与load()方法的区别"
keywords: Java, Hibernate
description: "Hibernate中get()方法与load()方法的区别"
category: Java
tags: [Java, Hibernate]
---
使用Hibernate加载对象可以使用get方法，也可以使用load方法，它们的区别如下： 

**Load方法**: 如果类的映射使用了代理(proxy)，load()方法会返回一个未初始化的代理，直到你调用该代理的某方法时才会去访问数据库。而如果没有匹配的数据库记录，load()方法可能抛出无法恢复的异常(unrecoverable exception) 。  

**get方法**: 如果你不确定是否有匹配的行存在，应该使用get()方法，它会立刻访问数据库，如果没有对应的记录，会返回null。 

那为什么还要使用load()呢？ 

若你希望在某对象中创建一个指向另一个对象的关联，又不想在从数据库中装载该对象时同时装载相关联的那个对象，那么这种操作方式就用得上的了。 如果为相应类映射关系设置了batch-size， 那么使用这种操作方式允许多个对象被一批装载（因为返回的是代理，无需从数据库中抓取所有对象的数据）。