---
layout: post
title: "Java常见分布式协议比较-JMS"
keywords: Java, J2EE, 分布式协议, 协议, 通信协议, JMS, 点到点, 发布, 订阅, 消息确认, 持久化, 消息选择器, 选择器
description: "对Java中常见的分布式协议进行比较，包括语言、传输协议、性能等多方面，本文介绍了JMS。"
category: Java
tags: [Java, 协议, JMS]
---

> Java的远程调用有多种分布式协议可供使用，但其种类繁多，容易让人困扰。本系列博客分别对它们做入门介绍：
> * [RMI](http://xiaoqing.me/2012/12/21/protocols-rmi/): 含JBoss-Remoting，Spring Remoting
> * [RPC](http://xiaoqing.me/2012/12/25/protocols-rpc/): 含XML-RPC, Binary-RPC
> * [CORBA](http://xiaoqing.me/2012/12/28/protocols-corba/): 
> * [SOAP](http://xiaoqing.me): (Web Service)
> * [EJB](http://xiaoqing.me/2012/12/19/protocols-ejb/) 
> * JMS

JMS与任何其他的RPC或者RMI技术没有可比性，因为它不是一种远程方法调用，而是Java程序访问企业级消息系统的一种机制，即Java进程间的消息传递机制。它的出现统一了Java程序访问消息的接口，并提高了消息在不同厂商之间的可移植性。JMS对应的规范是[JSR914](http://www.jcp.org/en/jsr/detail?id=914)，产生于2001-2003年，规范主要定义了消息模型、消息传递的模式等等。

### 消息传递方式
参考其他的消息中间件，JMS规范定义了两种消息传递方式：  
1. 点到点(Point-To-Point)
这种方式就是消息队列。每个消息发送到队列中，消费者从队列中获取消息。一般一个消息队列只能有一个消费者。  
2. 发布/订阅(Publisher-Subscriber)
发布者发送消息到指定的主题，订阅者接受消息。这种方式允许一个消息发送到多个消费者。
一般订阅者只有订阅之后才能接受消息，如果订阅者不在线，无法接受消息，但如果使用持久化订阅，消息会被保留知道被接受或者消息过期。

JMS规范为这两种方式提供了统一的API，只是在不同方式下有不同的实现。  

<table class="large">
<thead>
<tr>
<th>JMS通用接口</th>
<th>发布/订阅方式</th>
<th>点到点方式</th>
</tr>
</thead>
<tbody>
<tr>
<td>ConnectionFactory</td>
<td>TopicConnectionFactory</td>
<td>QueueConnectionFactory</td>
</tr>
<tr>
<td>Connection</td>
<td>TopicConnection</td>
<td>QueueConnection</td>
</tr>
<tr>
<td>Destination</td>
<td>Topic</td>
<td>Queue</td>
</tr>
<tr>
<td>Session</td>
<td>TopicSession</td>
<td>QueueSession</td>
</tr>
<tr>
<td>MessageProducer</td>
<td>TopicPublisher</td>
<td>QueueSender</td>
</tr>
<tr>
<td>MessageConsumer</td>
<td>TopicSubscriber</td>
<td>QueueReceiver</td>
</tr>
</tbody>
</table>
<p></p>

### 消息模型
JMS的消息模型为所有消息应用提供了统一的接口，JMS消息包括三部分，除了消息体外，消息头和消息属性都是对消息的描述，而且是基于key/value的。  
1. 消息头
JMS的消息头是固定的，即所有的字段都由JMS规范定义，且以JMS开头，如JMSDestination、JMSMessageID、JMSTimestamp、JMSExpiration、JMSPriority等。  
2. 消息属性
由于消息头是固定的，JMS通过属性来扩展对消息的描述。属性可以有三种使用方式：JMS定义的属性，属性名以"JMSX"开头，如JMSXUserID、JMSXGroupID；JMS厂商特有的属性，以"JMS_&lt;vendor&gt;"开头；当然也可以有应用定义的属性。  
3. 消息体
JMS提供了5种类型的消息体，分别是：TextMessage支持文本；StreamMessage支持原始数据；MapMessage支持Map，但是key只能是String，value只能是原始数据；ObjectMessage支持可序列化的对象；BytesMessage支持二进制字节。
这5种消息体其实已经涵盖了Java多大多数的场景，毕竟JMS规范定义的是消息，只需要表示状态变化或者事件，而不需要传递太复杂的内容。

### 消息选择器
如果是发布/订阅模型，消息发出之后，如果JMS厂商可以根据指定的条件，有选择地发送给订阅者，则可以节省带宽，提高发送效率。这可以通过消息选择器完成。  
JMS的消息选择器是一个字符串，其语法是SQL92条件表达式的字符，开发人员可以据此定义自己的表达式，只接受感兴趣的消息。比如"age BETWEEN 15 AND 19"，可以过滤属性为"age"，大小在15和19之间的所有消息。如果表达式为空，代表没有任何过滤。  
需要注意的是消息选择器只能过滤消息头和属性，而不能基于消息体过滤。  
	
### 消息可靠性
JMS规范需要在可靠性和效率之间找到平衡，这通过几种方式实现：
1. 消息确认：如果是事务性会话，则会话提交时会自动确认消息；对非事务性会话，JMS定义了三种确认方式：Session自动确认；接收端显示确认；允许副本的确认。后者是一种懒惰的确认方式，可能导致消息重复发送。JMS厂商可能提供不同的确认方式，如WebLogic JMS允许不确认，当然这违反了JMS规范。
2. 事务：JMS会话可以声明为事务性的，这样支持一组消息的提交或回滚。如果是跨资源的分布式事务，可通过JTA规范来完成。
3. 持久化：非持久化的消息效率最高，JMS厂商最多发送一次该类消息，决不能发送两次，某些情况下可能丢失，适用于对可靠性要求不高的场景；持久化的消息要求JMS厂商发送一次且必须发送一次，因而可以提高可靠性。
4. 发送顺序：JMS对消息发送的顺序做了规定：同一个Session内的消息需要保证发送和接受的顺序一致。但如果消息有的需要持久化，有的不需要持久化，或者设置了不同的优先级就不必保证顺序。

### 协议和语言
由于只是消息传播机制而非远程接口调用，JMS规范只是提到了使用TCP/IP sockets，没有具体定义使用何种传输协议，因此由JMS厂商自行决定。JMS只要求如果消息体是对象，该对象必须可序列化。  

JMS一般用于Java程序之间的消息传递，当然JMS厂商也可以封装其他语言的消息中间件，从而实现跨语言和平台的消息传递。

### 编程模型
JMS的编程模型可以简单描述如下：

<p class="image-container middle">
<a href="#"><img alt="JMS programming model" src="/assets/images/protocols-jms-programming-model.png"></a>
</p>

其中ConnectionFactory和Destination又叫做被管理对象，一般由JMS厂商提供，由管理员实现注册到JNDI中供开发人员使用。

### 总结
JMS的性能一般不错，可以尽量使用异步消息来提高性能。

著名的开源实现有Apache ActiveMQ和Rabbit MQ, 二者都支持多种语言，都支持[AMQP](http://www.amqp.org/about/what)传输模式。不过Apache AQ可以使用更多的协议，如STOMPC、REST等，而后者只能通过插件来扩展。  
商业实现有Sonic MQ，是快钱公司在使用的产品。  
这里不推荐Oracle Advanced Queuing，因为有较多的Bug，易用性不好。
