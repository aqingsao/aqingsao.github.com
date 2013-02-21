---
layout: post
title: "扩展自己的Junit Test Runner"
description: "本文介绍了如何扩展自己的Junit Test Runner，让单元测试更有效率。"
category: Java
tags: [Java, JUnit]
keywords: Java, JUnit, BlockJUnit4ClassRunner, Runner, @Before, @After, @BeforeClass, @AfterClass, 测试用例, 测试类, 所有测试方法前执行, 所有测试类前执行
---

典型的XUnit测试框架，在测试用例、测试类执行前后会提供回调机制，让开发人员执行自定义任务，如加载资源释放资源。在JUnit框架中，是通过注解@Before, @After, @BeforeClass, @AfterClass完成的。然而，有些情况下这种机制无法满足我们的要求：比如开发RESTful应用，使用JUnit框架测试RESTful API，每个测试用例执行前，都要启动服务，并在测试完成后关闭。

这样的测试场景并不少见，不但要支持IDE，即开发人员可以选择任何一个类中的一个或者多个测试用例运行，也可以同时多个测试类运行；还要支持命令行，比如在开发环境中或者持续集成环境中使用Maven/Gradle/Ant等构建工具执行所有测试。

常见的做法是使用@Before或者@BeforeClass，可以达到我们的目的，但是并不高效：如果运行了几十个类几百个测试用例，RESTful服务会被频繁启动很多次。此时，更有效的机制是运行所有测试前后回调，分别用于启动服务和关闭服务。JUnit默认不提供这一功能，但是基于其API，我们可以自己实现这一功能：即通过@RunWith注解使用自己定义的Test RUnner。

编写一个新的类名为WithServerTestRunner，继承自BlockJUnit4ClassRunner：

{%highlight java linenos%}
public class WithServerTestRunner extends BlockJUnit4ClassRunner {
    public WithServerTestRunner(Class<?> klass) throws InitializationError {
        super(klass);
    }
}
{%endhighlight%}

覆盖runChild方法，在该方法前后加入我们的代码：

{%highlight java linenos%}
@Override
protected void runChild(FrameworkMethod method, RunNotifier notifier) {
    startRestfulServer();
    super.runChild(method, notifier);
    stopRestfulServer();
}
{%endhighlight%}

在startRestfulServer()方法中启动RESTful服务，可以使用嵌入式Web容器，如Grizzly或Jetty，这里不细述。并在stopRestfulServer()方法中关闭服务。运行发现，每个测试用例前后都会启动停止Web容器！原来runChild()方法是运行每个具体的测试用例，加在这里当然不对。正确的位置是run()方法：

{%highlight java linenos%}
@Override
protected void run(RunNotifier notifier) {
    startRestfulServer();
    super.run(notifier);
    stopRestfulServer();
}
{%endhighlight%}

再次运行，发现结果如我们所期，把上述方法改进一下，确保测试结束后停止Web容器：

{%highlight java linenos%}
@Override
protected void run(RunNotifier notifier) {
	try{
    	startRestfulServer();
    	super.run(notifier);
    }finally{
    	stopRestfulServer();
    }
}
{%endhighlight%}

最后一步就是在测试类上添加@RunWith注解，指定我们自己的Test Runner：

{%highlight java linenos%}
@RunWith(WithServerTestRunner.class)
public class EmailResourceTest {
}
{%endhighlight%}

好了，至此，我们扩展了Junit的Test Runner，实现了所有测试用例执行之前运行特定的代码，而且只运行一次。事实上，这是很多框架如SpringJUnit4ClassRunner扩展JUnit Test Runner的方式。