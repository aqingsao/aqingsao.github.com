---
layout: post
title: "JPA事务"
description: "介绍JPA下的事务，以及没有JEE容器时如何使用声明式事务"
category: Java
keywords: Java, J2EE, JPA, Transaction, 事务, Delcarative Transaction, 声明式事务, CMT, 容器管理的事务, 没有容器的容器管理事务, CMT without JEE container
tags: [Java, J2EE, JPA]
---

讨论JPA事务时，经常会遇到几个容易混淆的概念：容器管理的(Container Managed)，还是程序管理的(Application Managed)；分布式事务(JTA)，还是本地资源事务(RESOURCE_LOCAL)；单事务的范围(Transaction-Scoped)，还是扩展的范围(Extended-Scoped)，本文我们将逐步介绍并理解它们的区别。

应用程序中的事务分为不同的层次，比如数据的增删改查，可以直接使用JDBC驱动或者DataSource作为封装层，这与数据库的事务直接对应，在事务处理处于比较低的层次，所以叫做本地事务(Resource-Local)。这是比较简单的事务。分布式事务(JTA)需要JavaEE服务器的支持，是企业级应用中典型也是推荐的事务处理方式，可以进行多个不同种类资源而不局限于数据库资源的事务处理。

本地事务总是由应用程序显示管理事务，这叫做程序管理的事务；而JTA有两种方式：把事务代理给容器，由容器负责事务的传播和开始提交事宜，这叫做容器事务；除此之外，JTA也支持程序管理的事务。

### 容器管理的事务
默认情况下，EntityManager通过@PersistenceContext注解注入的，或者通过JNDI查找的，使用的是容器管理的持久化。

JPA中我们有不同的说法，如容器管理的持久化，容器管理的事务，容器管理的EntityManager，容器管理的持久化上下文。



本地事务都是由应用程序显示划分事务，而

### 容器管理的持久化(CMP) VS. 程序管理的持久化(AMP)

应用程序想要控制如何持久化，需要获取EntityManagerFactory(可以通过容器注入或者JNDI查找)，显示创建EntityManager实例。

### Extended VS.