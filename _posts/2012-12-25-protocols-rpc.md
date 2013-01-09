---
layout: post
title: "Java常见分布式协议比较-RPC"
keywords: Java, J2EE, RMI, 分布式协议, 协议, 通信协议, RPC, XML-RPC, Binary-RPC, Hessian, Burlap
description: "对Java中常见的分布式协议进行比较，包括语言、传输协议、性能等多方面，本文介绍了RPC, Binary-RPC, XML-RPC。"
category: Java
tags: [Java, 分布式协议, 协议，通讯协议, RPC, XML-RPC, Binary-RPC, Hessian, Burlap]
---

> Java的远程调用有多种分布式协议可供使用，但其种类繁多，容易让人困扰。本系列博客分别对它们做入门介绍：
> * [RMI](http://xiaoqing.me/2012/12/21/protocols-rmi/): 含JBoss-Remoting，Spring Remoting
> * RPC: 含XML-RPC, Binary-RPC
> * [CORBA](http://xiaoqing.me/2012/12/28/protocols-corba/): 
> * [SOAP](http://xiaoqing.me): (Web Service)
> * [EJB](http://xiaoqing.me/2012/12/19/protocols-ejb/) 
> * [JMS](http://xiaoqing.me/2013/01/08/protocols-jms/)

RPC, 远程过程调用， 也叫远程函数调用, 最早出现在Sun公司和HP公司的运行Unix操作系统的计算机中，用于系统间通信的一种机制. 

### 协议和语言
RPC是一种大而泛的远程调用机制，不局限于语言，更不局限于协议。我们可以认为RMI就是一种RPC，其区别在于RMI是基于对象的，本地虚拟机中通过Stub可以调用远程虚拟机的对象。  
不过从传输的数据格式来说，RPC可以分为基于二进制的RPC、基于XML的RPC、以及其他诸如基于Json的RPC。

### Binary-RPC
RMI可以认为是二进制RPC的一个实现。另外一个著名的实现是Hessian。Hessian官网宣称自己是一种Binary-RPC。不过分析细微差别，从使用方式上，我更倾向于其是一种RMI的实现。

Hessian由caucho公司开发，是一种高效、跨平台、基于HTTP的协议。Hessian的设计目标是：自描述的，即不需要额外的Schema或者接口定义；尽可能的紧走；简单；尽量快。 
为保持高效，Hessian自定义了自己的数据序列化反序列化协议，并有多种语言的实现，因此很容易跨平台；在转换为对象流时可以进行压缩。尤其是Hessian2.

### XML-RPC
XML-RPC有两种涵义，第一种泛指以XML格式进行远程方法调用的过程，比如演进出来的SOAP、Caucho公司的Burlap都可归为此类；第二种特指XML-RPC协议本身，定义了标准的数据传输格式，并规定了使用HTTP作为传输协议，Apache XML-RPC是其Java的一个实现。

基于XML的RPC通常都是跨平台的，但其主要缺点是性能：
1. 描述接口的冗余信息太多，与二进制协议相比，文件能大几倍甚至10倍；
2. 基于XML的序列化反序列化效率不高。
所以其性能与RMI、Hessian等协议相比，至少要差一个数量级。

这里值得一提的是caucho公司的Burlap，与Hessian协议相对应，是目前性能较高的的XML-RPC实现。