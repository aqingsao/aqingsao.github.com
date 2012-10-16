---
layout: post
title: "Web自动化测试之WebDriver"
description: ""
category: Functional Test
tags: [WebDriver, Functional Test]
---

对Web进行自动化测试，我们首先想象一个简单的场景，来看看需要测试哪些东西：

* 元素定位：无论使用XPath, Dom还是CSS，需要简单方便的API定位元素，可以延时等待元素出现；  
* 交互操作：包括文本框、单选框、多选框、按钮、表格单元的输入或者点击；  
* 页面操作：页面切换和关闭、对话框切换和关闭；  
* 其他要求：对主流浏览器测试的支持、对JavaScript的支持等。

说起Web自动化测试，首先想到的就是Selenium。其实WebDriver就是基于Selenium的一个自动化测试类库，但它不再是运行在浏览器内的JS程序，而是自己可以控制浏览器。旨在改进Selenium1.0中出现的诸多问题，并且提供了非常易用、可读性很强的API。

### 1. 简单例子
我们通过一个例子来初步认识一下WebDriver。简单起见，我们通过WebDriver连接到google网站上，通过关键词进行搜索，并且验证搜索结果。
代码如下：

{% highlight java linenos%}
package selenium;  
  
import org.junit.Before;  
import org.junit.Test;  
import org.openqa.selenium.By;  
import org.openqa.selenium.WebDriver;  
import org.openqa.selenium.WebElement;  
import org.openqa.selenium.htmlunit.HtmlUnitDriver;  
  
import static junit.framework.Assert.assertNotNull;  
  
public class WebDriverTest {  
    private WebDriver page;  
  
    @Before  
    public void before() {  
        page = new HtmlUnitDriver();  
    }  
  
    @Test  
    public void testHasAnImageSearchPage() throws Exception {  
        page.get("http://www.google.cn");  
  
        WebElement searchBox = page.findElement(By.name("q"));  
        searchBox.sendKeys("JavaEye");  
  
        WebElement subBtn = page.findElement(By.name("btnG"));  
        subBtn.submit();  
  
        WebElement result = page.findElement(By.linkText("http://www.iteye.com"));  
        assertNotNull(result);  
    }  
}  
{% endhighlight%}

### 2. HamcrestWebDriverTestCase
除了上面的写法，WebDriver还根据Hamcrest类库，利用它优秀的Matcher API，另外提供了的基类：HamcrestWebDriverTestCase，从该基类集成的测试用例，更易读。
代码如下：

{% highlight java linenos%}
package selenium;  
  
import org.junit.Test;  
import org.openqa.selenium.WebDriver;  
import org.openqa.selenium.htmlunit.HtmlUnitDriver;  
import org.openqa.selenium.lift.HamcrestWebDriverTestCase;  
  
import static org.hamcrest.Matchers.equalTo;  
import static org.openqa.selenium.lift.Finders.div;  
import static org.openqa.selenium.lift.Finders.textbox;  
import static org.openqa.selenium.lift.find.InputFinder.submitButton;  
import static org.openqa.selenium.lift.find.LinkFinder.link;  
import static org.openqa.selenium.lift.match.AttributeMatcher.attribute;  
  
public class HamcrestTest extends HamcrestWebDriverTestCase {  
  
    @Override  
    protected WebDriver createDriver() {  
        return new HtmlUnitDriver();  
    }  
  
    @Test  
    public void testHasAnImageSearchPage() throws Exception {  
        goTo("http://www.google.cn");  
  
        type("JavaEye", into(textbox().with(attribute("name", equalTo("q")))));  
        clickOn(submitButton().with(attribute("name", equalTo("btnG"))));  
  
        assertPresenceOf(link("http://www.iteye.com"));  
    }  
}  
{% endhighlight%}

### 3. WebDriver, Web Driver
WebDriver有以下几种浏览器驱动器：
HtmlUnit Driver：
速度最快；平台独立；支持JavaS次日平台；
不是图形化的，即你无法在浏览器中看到页面元素被点击的过程；
其JavaScript引擎是Rhino，与主流浏览器的均不同（Chrome V8, Safari Nitro, FF TraceMonkey...），因此JavaScript执行结果可能稍微不同；
而另外三种FireFox Driver、Internet Explorer Driver和Chrome Driver都可在真正的浏览器中运行，因此是可视化的；并且支持JavaScript；只是运行速度较慢；
 
4. 更多的API
继续尝试下去，会遇到一些常用的API，主要包括：
Finder：在某些时候可以作为XPath查找方式的替代，使用比较方便；
Matcher：就是Hamcrest类库提供的Matcher API；
页面、frame等的切换、Cookie等等。
 
5. 局限？
a. WebDriver支持XPath的方式进行元素定位，可惜现在还不支持常用的CSS Selector定位。
b. WebDriver对javascript弹出的对话框暂时还不支持，不过应该很快了吧。
 
6. 总结：
由于提供了良好的API，解决了很多Selenium1.0的问题，它被即将发布的Selenium 2.0集成了进去，因此还是很值得一试。
 
参考资料：
1. Selenium 2.0 and WebDriver
2. Issue 24 : Allow use of CSS selectors
3. Selenium FAQ