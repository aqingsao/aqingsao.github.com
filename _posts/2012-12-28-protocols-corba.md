---
layout: post
title: "Java常见分布式协议比较-CORBA"
keywords: Java, J2EE, 分布式协议, 协议, 通信协议, CORBA, IIOP, IDL
description: "对Java中常见的分布式协议进行比较，包括语言、传输协议、性能等多方面，本文介绍了CORBA, IIOP。"
category: Java
tags: [Java, 协议, 通信协议, 分布式协议, RMI, XML-RPC, Binary-RPC, Hessian, Burlap, JBoss-Remoting, Http Invoker, Deiban, SOAP, EJB, JMS]
---

> Java的远程调用有多种分布式协议可供使用，但其种类繁多，容易让人困扰。本系列博客分别对它们做入门介绍：
> * [RMI](http://xiaoqing.me/2012/12/21/protocols-rmi/): 含JBoss-Remoting，Spring Remoting
> * [RPC](http://xiaoqing.me/2012/12/25/protocols-rpc/): 含XML-RPC, Binary-RPC
> * CORBA: 
> * [SOAP]("/"): (Web Service)
> * [EJB](http://xiaoqing.me/2012/12/19/protocols-ejb/) 

CORBA是OMG定义的分布式对象系统的标准，可以抽象系统平台、网络通讯以及编程语言的差异，提供了分布式对象跨平台调用的能力。换句话说，CORBA对象可以用任何语言编写，运行在任何平台上，并位于网络的任何位置。

CORBA的分布式计算和跨平台的能力，主要依赖于两项技术：IDL和ORB。

### IDL
IDL全称接口定义语言，是用来描述软件组件接口的一种规范语言。它抽象了不同语言的差异，定义了自己的语法和数据类型，把服务以语言中立的方式描述并且暴露出来。  
下面是IDL一个具体的例子，可以看出，用户可以定义模块、接口、属性、方法、输入输出参数，甚至异常等等。

{%highlight java linenos%}
module finance {
  interface account {
    readonly attribute string owner;
    readonly attribute float balance;
    void makeLodgement(in float amount, out float newBalance);
    void makeWithdrawal(in float amount, out float newBalance);
  };
};
{%endhighlight%}

IDL在不同的语言下都有相应的实现，可以把IDL描述的接口编译为目标语言，包括客户端代理和服务器端框架，以及相应的帮助类等等。比如Java中提供过了idlj命令用来编译。

### ORB

CORBA的核心是对象请求代理ORB（Object Request Broker），ORB提供了一种机制，对象可以透明的发出请求和接收响应, 客户端程序使用一个对象时，不需要知道对象的位置、编程语言，或者平台类型，因为ORB抽象了这些细节。其调用过程可以描述为：

<p class="image-container big">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/protocols-corba-orb.png"></a>
</p>

从图中可以看出，CORBA的分布式对象调用能力依赖于ORB，而ORB之间进行通信是通过GIOP(General Inter-ORB Protocol)协议完成的。GIOP定义了ORB之间互操作的传输语法和标准消息格式，比如请求头、请求体所包含的字段和长度。
GIOP是一个抽象的协议，而IIOP是其一个具体的实现，定义了，如何通过TCP/IP协议交换GIOP消息。	所以通常我们说CORBA是基于IIOP协议的。
	
### 协议和语言
CORBA抽象了系统平台、编程语言的差异，因而是跨平台和语言的，只需要语言具有IDL的映射。

CORBA使用的协议是IIOP，前面已有描述，可以理解为一种基于TCP/IP的协议。

### 开发过程
我们开发一个简单的Hello World程序：

1. 编写IDL接口定义文件

{%highlight java linenos%}
module HelloApp
{
    interface Hello
    {
        string sayHello(in string message);
    };
};
{%endhighlight%}

2. 编译生成服务器框架和客户端代理
比如使用JDK中的idlj命令：

{%highlight bash linenos%}
idlj -oldImplBase -fall Hello.idl
{%endhighlight%}

3. 编写服务器端实现

{%highlight java linenos%}
public class HelloImpl extends _HelloImplBase {
    public String sayHello(String message) {
        return “hello ” + message;
    }
}
{%endhighlight%}

4. 启动服务器端

{%highlight java linenos%}
public class HelloServer {
    public static void main(String args[]) {
            ORB orb = ORB.init(args, null);
            HelloImpl helloImpl = new HelloImpl();
            orb.connect(helloImpl);
            org.omg.CORBA.Object objRef =orb.resolve_initial_references("NameService");
            NamingContext ncRef = NamingContextHelper.narrow(objRef);
            NameComponent nc = new NameComponent("Hello", "");
            NameComponent path[] = { nc };
            ncRef.rebind(path, helloImpl);
            orb.run();
    }
}
{%endhighlight%}

启动之前，要确保先运行命名服务器处于运行状态。

5. 编写客户端调用程序

{%highlight java linenos%}
public class HelloClient { 
    public static void main(Stringargs[]) { 
        ORB orb = ORB.init(args,null); 
        org.omg.CORBA.ObjectobjRef = orb.resolve_initial_references("NameService"); 
        NamingContext ncRef = NamingContextHelper.narrow(objRef); 
        NameComponent nc =newNameComponent("Hello",""); 
        NameComponent path[] = { nc }; 
        Hello h = HelloHelper.narrow(ncRef.resolve(path)); 
        h.sayHello("world"); 
    } 
}
{%endhighlight%}

### 总结
与RMI相比，CORBA更为强大，并定义了企业级应用通用的服务：名字服务；事务服务；对象生命周期服务；并发控制服务；时间服务等。

CORBA曾经是分布式计算的主流技术，在电信等领域使用广泛。不过其开发过程繁琐，强烈依赖于CORBA中间件的实现。虽然跨平台，但不同中间件的互操作有兼容性的问题。因此开发和部署成本较高，目前属于已经基本被遗弃的技术，被轻量级的Web服务、RESTful服务等代替了。
