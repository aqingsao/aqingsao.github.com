---
layout: post
title: "JPA主键生成器和主键生成策略"
description: "介绍JPA中的主键生成器和主键生成策略"
category: Java
tags: [Java, JPA, J2EE]
keywords: Java, J2EE, JPA, 主键生成器, 主键生成策略, GenerationType, GeneratedValue, IDENTITY, TABLE, AUTO, SEQUENCE
---

JPA中创建实体时，需要声明实体的主键及其主键生成策略。我们有一个实体类叫做Email，其主键上声明如下：

{%highlight java linenos%}
@Id
@Column(name = "EMAIL_ID")
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emailSeq")
@SequenceGenerator(initialValue = 1, name = "emailSeq", sequenceName = "EMAIL_SEQUENCE")
private long id;
{%endhighlight%}

我们使用@GeneratedValue的strategry字段声明主键生成策略，generator声明主键生成器的名称，对应于同名的主键生成器@SequenceGenerator或者@TableGenerator。

与Hibernate不同，JPA只提供四种主键生成器策略，分别介绍如下：

### GenerationType.IDENTITY
多数数据库支持IDENTITY列，数据库会在新行插入时自动给ID赋值，这也叫做ID自增长列，比如MySQL中可以在创建表时声明“AUTO_INCREMENT”, 就是一个ID子增长列:

{%highlight sql linenos%}
CREATETABLE EMAIL{
    ID BIGINT NOT NULL AUTO_INCREMENT, MESSAGE VARCHAR(255) NOT NULL, PRIMARY KEY(ID)
}
{%endhighlight%}

JPA中IDENTITY类型的主键生成策略用法如下：

{%highlight java linenos%}
@GeneratedValue(strategy=GenerationType.IDENTITY)
@Column(name="id")
private long id;
{%endhighlight%}

由于主键由数据库自动插入，因此不需要额外的配置信息。

多数数据库支持IDENTITY策略：MySQL, SQL Server, DB2, Derby, Sybase, PostgreSQL。

### GenerationType.SEQUENCE
Oracle不支持ID子增长列而是使用序列的机制生成主键ID，对此，可以选用序列作为主键生成策略：

{%highlight java linenos%}
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "emailSeq")
@SequenceGenerator(initialValue = 1, name = "emailSeq", sequenceName = "EMAIL_SEQUENCE")
private long id;;
{%endhighlight%}

上述声明等同于在数据库上创建一个序列：

{%highlight sql linenos%}
create sequence EMAIL_SEQUENCE;
{%endhighlight%}

如果不指定序列生成器的名称，则使用厂商提供的默认序列生成器，比如Hibernate默认提供的序列名称为hibernate_sequence。

支持的数据库： Oracle、PostgreSQL、DB2

### GenerationType.TABLE
有时候为了不依赖于数据库的具体实现，在不同数据库之间更好的移植，可以在数据库中新建序列表来生成主键，序列表一般包含两个字段：第一个字段引用不	同的关系表，第二个字段是该关系表的最大序号。这样，只需要一张序列就可以用于多张表的主键生成。
用法：

{%highlight java linenos%}
@TableGenerator( name = "emailSeq", table = "MY_PROJECT_SEQUENCE_TABLE", pkColumnName = "SEQUENCE_NAME", valueColumnName = "SEQUENCE_COUNT", initialValue = 1, allocationSize = 1)
@GeneratedValue( strategy = GenerationType.TABLE, generator = "emailSeq")
{%endhighlight%}

如果不指定表生成器，JPA厂商会使用默认的表，比如Hibernate在Oracle数据库上会默认使用表hibernate_sequence。

这种方式虽然通用性最好，所有的关系型数据库都支持，但是由于不能充分利用具体数据库的特性，建议不要优先使用。

### GenerationType.Auto
把主键生成策略交给JPA厂商(Persistence Provider)，由它根据具体的数据库选择合适的策略，可以是Table/Sequence/Identity中的一种。假如数据库是Oracle，则选择Sequence。

{%highlight java linenos%}
@GeneratedValue(strategy = GenerationType.AUTO) 
{%endhighlight%}

如果不特别指定，这是默认的主键生成策略。

