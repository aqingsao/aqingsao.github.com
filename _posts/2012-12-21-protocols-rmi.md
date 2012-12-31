---
layout: post
title: "Java常见分布式协议比较-RMI"
keywords: Java, J2EE, RMI, 分布式协议, 协议, 通信协议, JBoss Remoting, Spring Remoting, HttpInvoker
description: "对Java中常见的分布式协议进行比较，包括语言、传输协议、性能等多方面，本文介绍了RMI, JBoss-Remoting, Spring-Remoting"
category: Java
tags: [Java, 分布式协议, 协议，通讯协议, RMI, RPC, Spring Remoting, JBoss Remoting, HttpInvoker]
---

> Java的远程调用有多种分布式协议可供使用，但其种类繁多，容易让人困扰。本系列博客分别对它们做入门介绍：
> * RMI: 含JBoss-Remoting，Spring Remoting
> * [RPC](http://xiaoqing.me/2012/12/25/protocols-rpc/): 含XML-RPC, Binary-RPC
> * [CORBA](http://xiaoqing.me/2012/12/28/protocols-corba/): 
> * [SOAP](): (Web Service)
> * [EJB](http://xiaoqing.me/2012/12/19/protocols-ejb/) 

Java中很容易调用一个实例的方法，但是想调用其他虚拟机上的实例怎么办？RMI技术应运而生，可以让Java程序调用远程虚拟机上的对象。

### 协议和语言
RMI主要采用Java对象序列化技术和TCP协议进行远程通信. 由于不是HTTP协议，不能穿越防火墙。

JRMP: JRMP是RMI最早的实现，也是Java中默认的实现，只能在Java语言中使用。  
CORBA：CORBA也是基于RMI技术发展起来的，采用了基于HTTP的IIOP协议，以及接口定义语言IDL，不但可以跨越防火墙，还可以支持跨语言的调用。
EJB: EJB也是一种RMI技术，只不过底层使用了基于HTTP的IIOP协议。

### 调用过程
其典型的通信过程如下

<p class="image-container big">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/protocols-rmi-method-call.png"></a>
</p>

在此过程中，参数和返回值需要分别进行Java的序列化和反序列化。

### 样例代码
RMI需要实现特定的接口，一般开发过程分为三步：  
1. 编写服务器端代码：
服务器端需要继承Remote接口：

{%highlight javascript linenos%}
public interface RemoteObjextends Remote {
    public String getMessage() throws RemoteException;
}
public class RmiServerextends UnicastRemoteObject implements RemoteObj { 
    public String getMessage() {
        return “Hello World”;
    }
    public static void main(String args[]) {
        // Create and install a security manager
        System.setSecurityManager(new RMISecurityManager());
        LocateRegistry.createRegistry(1099); 
        RmiServer obj = new RmiServer();
        Naming.rebind("//localhost/RmiServer", obj);
    }
}
{%endhighlight%}

2. 生成代理文件：
Java提供了命令行工具可以生成代理类，给客户端使用：
`rmic RmiServer`

3. 编写客户端代码调用：

{%highlight javascript linenos%}
public class RmiClient { 
    RmiServerIntf obj = null; 
    public String getMessage() { 
            obj = (RmiServerIntf)Naming.lookup("//localhost/RmiServer");
            return obj.getMessage(); 
    }
    publicstaticvoid main(String args[]) {
        // Create and install a security manager
        System.setSecurityManager(new RMISecurityManager()); 
        RmiClient cli = new RmiClient();
    }
}
{%endhighlight%}

当然，如果使用Spring Remoting，已经集成了RMI，不需要继承指定的接口了：

### JBoss Remoting
JBoss Remoting是JBoss开发的Java领域的远程通信框架，提供了简单的API，可以基于多种协议进行远程方法调用。是JBoss内部很多产品的通信框架。  
与Java RMI的区别在于，JBoss Remoting是一个“框架”，即支持多种通信协议（Socket/Http等）;支持异步通信;可以把请求封装到已有的请求对象InvocationRequest，因而不需要再编写客户端代理。

由于做了性能优化，在某些测试中JBoss Remoting比Java RMI快[7-8倍](http://176.34.122.30/blog/2009/02/19/jboss-remoting-jboss-serialization-kills-javarmi-and-spring-remoting/)；然而也有测试发现二者的性能差不多。

### Spring Remoting
Spring Remoting是Spring对已有远程通信技术做的集成，支持以下技术：
1. RMI：通过RmiProxyFactoryBean和RmiServiceExporter支持传统的RMI（即实现了Remote借口），并且支持透明的RMI调用(任何POJO)
2. Spring Http Invoker: Spring自己的远程调用技术，基于HTTP和Java的序列化。支持任何Java接口，对应的类是HttpInvokerProxyFactory/HttpInvokerServiceExporter。要求通信双方不但是Java，而且必须使用Spring框架。
3. Hessian、Burlap：通过HessianProxyFactoryBean、BurlapProxyFactoryBean等支持著名的Hessian/Burlap协议。
4. 当然，Spring Remoting也支持非RMI的远程调用，如JMS、WebService。
Spring Remoting采用了一些最佳实践，如采用代理和缓存机制，因而在性能上略有提升。

### 性能
一般而言，一个远程方法调用花费[~10ms](https://community.jboss.org/message/111631)。与比XML的RPC或者Web Service相比，性能高很多。与Hessian相比，则性能接近或稍差一些。



