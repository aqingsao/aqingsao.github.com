---
layout: post
title: '如何做一个完美的HTML INPUT输入框（上）'
keywords: HTML Input, limit input, number, money, email
Description: HTML input box for number, email, money only
category: Html
tags: [Html, JavaScript, jQuery]
---

Web开发中，可能会遇到这样的需求，HTML的INPUT输入框中限制输入数字、或者字母、或者金额。这似乎不难，网上一搜，就会有大批的文章。但真正做一次，就会遇到不少问题。要做一个完美的INPUT输入框其实并不那么容易，要考虑的东西是相当多的。

其实面对实现这样的需求，我的第一建议是在而且只在服务器端做验证：最简单的做法是form提交之后服务器端验证，如果含有非法字符，显示错误信息。服务器端的验证简单有效，而且结合使用AJAX能够达到较好的交互操作效果。因此使用服务器端验证已经足够，应该作为首选方案。

但是，如果你或者客户已经决定了用客户端验证，那我们就尝试一下吧，看看要想做出一个完美的INPUT输入框，都有哪些陷阱。
这篇文章中，我们假设需求是这样的：输入框中的名字只能是数字或者字母，任何其他字符都是非法的。
页面中的HTML代码片段如下，JavaScript部分使用JQuery来实现：

{%highlight JavaScript%}
<form action="myAction">
    <label>Name:</label>
    <input type="text" name="name" maxlength="20"/>
    <input type="submit" value="Submit"/>
</form>
{%endhighlight%}

**方法一：输入完所有字符之后再做验证**

这通常指表单提交之前验证，或者鼠标离开INPUT输入框时验证，如果非法，显示错误信息，同时表单不能提交。这是最简单的一种方式，使用JavaScript和正则表达式就可以（[点击查看实际效果](/assets/html/NumericInput.html)）：

{%highlight JavaScript%}
$(document).ready(function(){
    $("input[name='name']").bind("click", function(){
        var reg = /^[0-9a-zA-Z]*$/;
        var valid = reg.test($(this).val());
        return valid;
    });
});
{%endhighlight%}

这种方式相当简单，只要正则表达式正确，不会有其他问题。
但是缺点就是用户可以任意输入非法字符，只有焦点离开或者提交之时才会有错误提示。

**方法二：输入单个字符后立刻检查输入框的内容**

还有另外一个简单的方法，就是用输入单个字符后立即检查输入框的所有内容，把非法字符去掉，代码如下（[点击查看实际效果](/assets/html/NumericInput.html)）：

{%highlight JavaScript%}
$(document).ready(function(){
    var reg = /[^0-9a-zA-Z]/g;
    $("input[name='name']").bind("keyup", function(){
        $(this).val($(this).val().replace(reg, ''));
    });
});
{%endhighlight%}

注意这里是检查输入框的所有内容，而不是输入的单个字符，因为它回避了后面方法的很多问题。但是其也有缺点

* 如果用户输入了非法字符，虽然很快就删掉了，但这个过程可以被用户看到，对特别看重用户体验的网站来说不可接受；  
* 如果用户输入非法字符后，很快按回车键，在这种极端情况下，非法字符会被传到服务器端。  

针对缺点2，我们可以稍微改进一下，除了keyup事件，表单提交之前再次检查输入框的内容：

{%highlight JavaScript%}
$(document).ready(function(){
    var reg = /[^0-9a-zA-Z]/g;
    $("input[name='name']").bind("keyup", function(){
        $(this).val((this).val().replace(reg, ''));
    });
    $("input[type='submit']").bind("click", function(){
        $("input[name='name']").val($("input[name='name']").val().replace(reg, ''));
    });
});
{%endhighlight%}

 改进之后，就保证不会把非法字符传入到服务器端，看起来也可以接受。

如果你无法忍受前两个方法的缺点，想做一个真正无法输入非法字符的输入框，只剩下最后一个方法了：输入时就立刻检查。这个方案复杂一些，需要检查每个字符是否合法，同时需要考虑不同浏览器的区别以及对特殊字符的支持。

**方法三：输入单个字符立即检查——基于JQuery插件**

Jquery有一个[alphanumeric](http://itgroup.com.ph/alphanumeric/)的插件，对特殊字符提供了一定的扩展功能（点击查看实际效果）：

{%highlight JavaScript%}
$("#input[name='name']").alphanumeric({nchars:"_"});
{%endhighlight%}

但这种方式有什么缺点呢？亲自尝试一下，你会发现：

*  在Firefox下不支持左右键。就其原因，该插件误认为Firefox下左键与%的char code都是37，因为左右键等无法使用。  
*  粘贴的支持不好。右键可以粘贴。Windows环境下无法使用Ctrl+V粘贴，但是Mac上可以使用。原因看看其源代码就知道了：if (e.ctrlKey&amp;&amp;k=='v') e.preventDefault(); 漏掉了Mac下的metaKey：
*  如果可以切换到中文输入，可以输入中文； 
*  提供的扩展功能有限，比如想让输入框支持粘贴，该插件无法扩展。  

另外该插件发布于2007年，作者早已经不维护了。因此这些BUG不好解决。

至此，只剩下最后一种方式了，就是自己写一个。在本文的[下篇](/2011/02/19/html-input-box-2/)中，我们将考虑几种不同的浏览器，来探讨如何做一个完美的INPUT输入框。
