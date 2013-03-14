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

是覆盖runChild方法吗？不是，其对应一个具体的测试用例。那我们尝试一下覆盖run()方法：

{%highlight java linenos%}
@Override
protected void run(RunNotifier notifier) {
    startRestfulServer();
    super.run(notifier);
    stopRestfulServer();
}
{%endhighlight%}

在startRestfulServer()方法中使用嵌入式Web容器，如Grizzly或Jetty启动RESTful服务，并在stopRestfulServer()方法中关闭服务，这里不细述。
在IDEA中运行多个测试类时，发现每个测试类前后都会执行启动停职服务的方法。原因是在IDEA这样的集成开发环境运行测试时，会启动自己的TestRunner，我们的TestRunner被包装在了IDEA的TestRunner中，因而无法控制所有测试前只运行一次。

事实上JUnit有一个概念叫做RunNotifier，是测试的监听器实现，会在测试开发、测试结束、测试失败等各个时机发送消息给注册的监听器，通过在上面注册自己的监听器，可以在测试启动、结束时得到通知：

{%highlight java linenos%}
void fireTestRunStarted(Description description);

void fireTestRunFinished(Result result);

void fireTestStarted(Description description) throws StoppedByUserException;

void fireTestFinished(Description description);
{%endhighlight%}

我们修改自己的Test Runner:
{%highlight java linenos%}
@Override
public void run(final RunNotifier notifier) {
    notifier.addListener(new RunListener() {
        @Override
        public void testRunStarted(Description description) throws Exception {
            beforeAllTestsRun();
        }
        public void testRunFinished(Result result) throws Exception {
            afterAllTestsRun();
        }
    });
    try {
        beforeRun();
        super.run(notifier);
    } finally {
        afterRun();
    }
}
{%endhighlight%}

再次选择多个测试类同时运行，发现出了问题：beforeAllTestsRun()方法根本没有调用，因此服务没有启动；而afterAllTestsRun()方法会被调用多次。原因是在我们通过覆盖run()方法添加的监听器，而由于每个测试类都会触发该方法，因而监听器添加了多次；另外我们的监听器添加时机台湾，无法监听到所有测试启动的时间。
其实，想在RunNotifier初始化时添加监听器比较困难，因为它是在JUnitCore中初始化，而我们也无法拿到后者的实例。接下来我们尝试使用状态标志：

{%highlight java linenos%}
@Override
public void run(final RunNotifier notifier) {
    if (!listenerAdded) {
        beforeAllTestsRun();
        notifier.addListener(new RunListener() {
            public void testRunFinished(Result result) throws Exception {
                afterAllTestsRun();
            }
        });
        listenerAdded = true;
    }
    try {
        beforeRun();
        super.run(notifier);
    } finally {
        afterRun();
    }
}
{%endhighlight%}

这样，即使run()方法在多个类中会被调到，我们通过一个静态变量listenerAdded来标志是否添加监听器，并在第一次添加监听器时启动beforeAllTestsRun()方法。最后一步就是在测试类上添加@RunWith注解，指定我们自己的Test Runner：

{%highlight java linenos%}
@RunWith(WithServerTestRunner.class)
public class EmailResourceTest {
}
{%endhighlight%}

好了，至此，我们扩展了Junit的Test Runner，实现了所有测试用例执行之前运行特定的代码，而且只运行一次。