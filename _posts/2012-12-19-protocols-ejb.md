---
layout: post
title: "Java常见分布式协议比较-EJB"
keywords: Java, J2EE, EJB, Session Bean, Message Driven Bean, Entity Bean, EJB 3.0, 分布式协议, 协议, 通信协议
description: "对Java中常见的分布式协议进行比较，包括语言、传输协议、性能等多方面，本文介绍了EJB和EJB3.0。"
category: Java
tags: [Java, 分布式协议, 协议, 通讯协议, EJB]
---

> Java的远程调用有多种分布式协议可供使用，但其种类繁多，容易让人困扰。本系列博客分别对它们做入门介绍：
> * [RMI](http://xiaoqing.me/2012/12/19/protocols-rmi/): 含JBoss-Remoting，Spring Remoting
> * [RPC](http://xiaoqing.me/2012/12/25/protocols-rpc/): 含XML-RPC, Binary-RPC
> * [CORBA](http://xiaoqing.me/2012/12/28/protocols-corba/): 
> * [SOAP](http://xiaoqing.me): (Web Service)
> * EJB 
> * [JMS](http://xiaoqing.me/2013/01/08/protocols-jms/)

EJB全称为Enterprise Java Beans，是Sun公司的服务器端组件模型，其设计目标是开发和部署可扩展的分布式Java企业级应用，并提供了持久化、事务处理、安全策略等解决方案。EJB本身是一个规范，需要部署在EJB容器上运行， 如WebLogic, JBoss等.

EJB的核心是编写企业级Bean，运行在业务逻辑层，开发时需要根据情况选择实现不同的企业Bean；

* 会话bean   
描述了与客户端的一个短暂的会话, 主要负责业务逻辑的处理，根据处理时的状态保持与否，又分为：有状态SessionBean和无状态SessionBean。  

* 实体bean   
描述了数据库表中的一条记录，负责数据库的访问, 通常由SessionBean调用。

* 消息驱动bean  
一种异步的消息处理机制。客户端向容器发送消息之后，立即返回。容器会在执行完成后调用相应的回调方法。

### 协议和语言
EJB采用了Java的RMI技术，不过底层采用了IIOP协议，故而称为RMI-IIOP。RMI-IIOP使得Java程序可以与其他语言的CORBA对象通信。但是由于EJB规范要求了远程对象必须实现某些接口，所以基本上EJB还只是局限于Java语言本身。

### 调用过程
一次EJB接口调用的典型过程如下：

<p class="image-container big">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/protocols-ejb-method-call.png"></a>
</p>

可以看出，一次普通的EJB调用，需要进行多次网路通信，再加上序列化，其成本非常高，这正是EJB广受诟病的原因之一。

### 性能
EJB2.0引入了本地接口的概念。如果客户端和EJBean部署在同一个虚拟机上，就不需要再通过RMI-IIOP协议进行远程通信了。  
除了本地接口，EJB也可以在多个方面进行性能调优：  
* 把多个远程实体Bean调用封装成一个会话Bean调用；  
* 提高方法粒度，减少远程方法调用频率；  
* 缓存EJBHome对象的引用，甚至远程对象，可以明显减少远程操作的次数.  

但是与POJO或者RMI方法调用相比，EJB的性能仍然是问题，单词调用达到几十毫秒以上：  
普通POJO方法调用：~0ms;  
EJB本地接口调用：~50ms;  
EJB远程方法调用：~100ms;  

### EJB 3.0
除了性能，EJB另一个典型的问题是开发、配置繁琐。它要求编必须按照制定的规范编写代码和部署描述符，比如必须实现Bean接口或Home接口。EJB3.0以后进行了显著的改善：  
1. 使用基于注解的编程方式。  
任何的企业级Bean只是一个简单的POJO对象，加上合适的注解，如@Stateful, @MessageDriven等。  
2. 提供了新的持久化API：JPA  
新的实体Bean仍然是简单的POJO，不过要加上@Entity注解。JPA提供了基于注解的O/R映射方式，使得Java中原本沉重的[DAO层变得瘦小起来](http://www.adam-bien.com/roller/abien/entry/jpa_ejb3_killed_the_dao)。

即便如此，其部署仍然依赖于EJB容器，搭建开发测试环境仍然有不小的成本。在大规模分布式应用中，目前大家更倾向于轻量级的方式，EJB正在逐步退出历史的舞台。不过EJB3.0中的某些技术可以独立使用，比如JPA，越来越受到大家的欢迎。


