---
layout: post
title: "JPA简介"
description: "介绍JPA的基本概念：EntityManager, EntityManagerFactory, Persistence Unit, Persistence Context"
keywords: Java, J2EE, JPA, EntityManager, EntityManagerFactory, Persistence Unit, Persistence Context, 实体, 持久化单元
category: Java
tags: [Java, J2EE, JPA]
---

JPA全称是Java Persistence API，定义了对象关系映射的标准，对应于[JSR317](http://jcp.org/en/jsr/detail?id=317)。它本身是EJB3.0规范的一部分，但可以独立使用，并且同时支持JEE和JSE环境，使得ORM可以在不同厂商之间移植，是目前公认的ORM的行业标准。本文介绍JPA中常用的几个基本概念。

### 实体(Entity)
实体类是数据库表与程序中对象的映射，通过在类上添加@Entity注解得到，当然也可以使用XML文件映射。实体对象就是实体类的具体实例，简单情况下，对应于数据库表的某一行，如果存在数据的引用，实体对象中也可以有同样的关联。后面我们把实体对象简称为实体。  
实体的创建、修改、查询和删除会引起对应数据库的变化，那么JPA如何知道实体与数据库的映射关系呢，这通过实体元数据(Entity Metadata)完成。

### 实体元数据(Entity Metadata)
实体与数据库的映射既可以通过在注解完成，也可以使用XML文件。比如使用类Email用来保存发送的邮件内容，
在类上添加注解@Entity(name = "EMAIL")就是一个元数据，标明该类是一个实体，对应的数据库表名称为"EMAIL"。其他常用的注解如@Id标示该字段对应数据库的主键，@Column可以描述对应列的信息，@OneToMany等描述实体之间一对多的关联...  
实体元数据是JPA很重要的部分，因为它直接描述了二者的映射关系和增删改查策略，也是性能优化的重点。

### 实体管理器(EntityManager)
实体的生命周期由EntityManager负责，可以理解为大家熟悉的Session。EntityManager是一个接口，实现由具体的厂商（persistence provider）负责。那么，应用程序中如何获取EntityManager呢?  
所有的EntityManager都由EntityManagerFactory创建，在JEE环境中，可以直接注入或者通过JNDI查找EntityManager，此时的EntityManager是容器托管的，应用程序直接调用API操作实体即可，其他包括事务都托管给容器。  
目前大家使用比较多的是Spring或者Guice这样的依赖注入框架，其使用方式非常灵活，比如Guice提供了JpaPersistModule，既可以注入EntityManag
er，也可以注入EntityManagerFactory:

{%highlight java linenos%}
    @Inject
    public EmailService(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
{%endhighlight%}

如果是JSE环境又没有使用上述框架，应用程序只能编程获取，如：    

{%highlight java linenos%}
EntityManagerFactory emf = Persistence.createEntityManagerFactory("EmployeeService");
EntityManager em = emf.createEntityManager();
{%endhighlight%}

此时的EntityManager完全由应用程序管理，包括使用完毕后的关闭：

{%highlight java linenos%}
em.close();
{%endhighlight%}

### 实体管理器工厂(EntityManagerFactory)
EntityManager的实例由EntityManagerFactory创建，前者的配置信息，比如数据库URL、用户名、密码一经创建也确定了下来，因为继承自EntityManagerFactory，而后者的信息依赖于持久化单元中的定义。
如果是应用程序自己管理，程序结束后应该关闭：

{%highlight java linenos%}
emf.close();
{%endhighlight%}

### 持久化上下文(Persistence Context)
一个EntityManager实例可能同时持久化多个实体，这多个实体统称为持久化上下文，其生命周期都有该EntityManager实例负责。
在一个持久化上下文中，相同实体识别符只能对应一个实体。

### 持久化单元(Persistence Unit)
持久化单元定义了所用数据库的基本信息，因而使用同一个持久化单元创建的EntityManagerFactory配置也相同。要想使用不同的配置，比如说应用程序连接2个以上不同的数据库，或者所用的用户名密码等不同，可以配置不同的持久化单元分别创建EntityManagerFactory。

### 持久化类(Persistence)
持久化类是JPA的入口，用于创建EntityManagerFactory，创建工作则依赖于不同厂商的实现，可以在persistece.xml文件中指定具体的Persistence Provider。

本文介绍了JPA中最常用的一些概念，为了更形象地描述它们之间的关系，可以参考下图：
<p class="image-container middle">
<a href="#"><img alt="Relations between JPA concepts" src="/assets/images/jpa-introduction-relations.png"></a>
</p>























