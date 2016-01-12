---
layout: post
title: "如何做一个完美的HTML INPUT输入框（下）"
keywords: HTML Input, limit input, number, money, email, keyPress, keyDown, 
Description: HTML input box for number, email, money only
category: Html
tags: [Html, JavaScript, jQuery]
---

在[上篇](/2011/02/15/html-input-box-1/)中，我们列出了几种常见的实现方式，但由于其各有优缺点，要想做一个完美的INPUT输入框，只能自己写了。

我们的思路是，当用户按下按键时，截获keyDown/keyUp或者keyPress事件，阻止其默认行为，这样就可以禁止用户输入非法字符。
为了后面理解清晰，我们把按键进行分类：第一类是特殊键，比如Shift、Control、Alt、Home、End等等。特殊键没有对应的Ascii代码，因此点击不会产生字符。第二类是普通键，有对应的Ascii码，又分为可以显示的字符键和不可以显示的控制键。字符键包括字母、数字以及特殊字符，控制键包括Backspace、回车键、Tab键。简单分类如下：

<table style="font-size: 80%;">
<tbody>
<tr>
<td style="padding: 0;" width="10%"></td>
<td style="padding: 0;"></td>
<td style="padding: 0;"></td>
<td style="padding: 0;"></td>
</tr>
</tbody>
<tbody>
<tr>
<td style="padding: 0;">-特殊键</td>
<td style="padding: 0;" colspan="3">Special Key，比如Home、End、Insert、Delete以及Function Keys</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">+修饰键</td>
<td style="padding: 0;" colspan="2">（Shift、Control、Alt）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">+导航键</td>
<td style="padding: 0;" colspan="2">Home、End、Page Up等）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">+功能键</td>
<td style="padding: 0;" colspan="2">（F1、F2等）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">+其它特殊键</td>
<td style="padding: 0;" colspan="2">（大写键、Insert、Delete等）</td>
</tr>
<tr>
<td style="padding: 0;">-普通键</td>
<td style="padding: 0;" colspan="3">Normal Key</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">+控制键</td>
<td style="padding: 0;" colspan="2">（非字符，不可显示，比如Backspace、回车、Tab等键）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;">-字符键</td>
<td style="padding: 0;" colspan="2">（字符，可显示）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;"></td>
<td style="padding: 0;">+允许输入的字符</td>
<td style="padding: 0;">（本例中比如数字键、字母键）</td>
</tr>
<tr>
<td style="padding: 0;"></td>
<td style="padding: 0;"></td>
<td style="padding: 0;">+不允许输入的字符</td>
<td style="padding: 0;">（本例中的非数字字母键]）</td>
</tr>
</tbody>
</table>
为了提高输入框的易用性，需要支持特殊键、普通键中的控制键，并允许用户输入指定的字符键，而不允许输入非法字符。因此需要识别不同的按键，JavaScript中event对象提供了有用的几个属性：event.keyCode、event.which和event.charCode。同时还有一些标记属性可以参考：event.shiftKey、event.ctrlKey、event.altKey、event.metaKey。有了它们，就可以判断按下的是特殊键还是普通键，是控制键还是字符键，从而可以决定是否禁止事件的默认行为。

现在一个主要的问题是，当按键按下时，不同浏览器的行为相差甚远，详情可以参考文章[JavaScript Madness: Keyboard Events](http://unixpapa.com/js/key.html)。不过闲话少说，在使用JQuery类库的基础上，还是亲自试一试吧，看看不同浏览器到底有什么区别:
（注：下表中的×代表不会产生keyPress事件；37|0|0分别表示keyCode、which和charCode；U代表undefined）
<table style="font-size: 80%;">
<thead>
<tr>
<td style="padding: 0;">Example key</td>
<td style="padding: 0;">Firefox</td>
<td style="padding: 0;">WebKit(Safari&amp;Chrome)</td>
<td style="padding: 0;">Internet Explorer</td>
<td style="padding: 0;">Opera</td>
<td style="padding: 0; color: red;"><strong>阻止显示？</strong></td>
</tr>
</thead>
<tbody>
<tr>
<td style="padding: 0;">特殊键(Home)</td>
<td style="padding: 0;">×</td>
<td style="padding: 0;">×</td>
<td style="padding: 0;">×</td>
<td style="padding: 0;">36|0|U</td>
<td style="padding: 0; color: red;"><strong>No</strong></td>
</tr>
<tr>
<td style="padding: 0;">控制键（左键）</td>
<td style="padding: 0;">37|0|0</td>
<td style="padding: 0;">×</td>
<td style="padding: 0;">×</td>
<td style="padding: 0;">37|0|U</td>
<td style="padding: 0; color: red;"><strong>No</strong></td>
</tr>
<tr>
<td style="padding: 0;">Legal Char(a)</td>
<td style="padding: 0;">0|97|97</td>
<td style="padding: 0;">97|97|97</td>
<td style="padding: 0;">97|97|U</td>
<td style="padding: 0;">97|97|U</td>
<td style="padding: 0; color: red;"><strong>No</strong></td>
</tr>
<tr>
<td style="padding: 0;">Illegal Char(%)</td>
<td style="padding: 0;">0|37|37</td>
<td style="padding: 0;">37|37|37</td>
<td style="padding: 0;">37|37|U</td>
<td style="padding: 0;">37|37|U</td>
<td style="padding: 0; color: red;"><strong>Yes</strong></td>
</tr>
</tbody>
</table>
从上表中不难发现，我们只需要检测到输入的非法字符，并禁止显示即可。而对所有其他的键需要支持。
测试中也发现了新的问题，某些控制键的keyCode和字符键的charCode值相同，比如左键和“%”的都是37，因此处理时需要留心，不要为禁止%，而禁掉了左键。这也是alphaNumeric插件的Firefox下的一个问题。

对上表的进一步分析表明，虽然不同浏览器的处理迥异，对经过JQuery的封装，event事件的which属性有一定的规律：对所有的特殊键和控制键，要么不产生keyPress事件，要么其值为0。对所有的可显示字符，which属性就是字符对应的ascii码。所以which属性可以很好的标示按键的种类。
有了这样的结论，代码也就不难写了：

{%highlight javascript%}
$("input[name='name']").bind('keypress', function(event) {
    if (!isSpecialKey(event.which) &amp;&amp; !isNumberOrLetter(event.which)) {
         event.preventDefault();
    }
});
function isNumberOrLetter(key) {
    return (key &gt;= 48 &amp;&amp; key &lt;= 57) || (key &gt;= 65 &amp;&amp; key &lt;= 90) || (key &gt;= 97 &amp;&amp; key &lt;= 122);
}
function isSpecialKey(key) {
    return key == 0 || key == 8 || key == 13 || key == 9;
}
{%endhighlight%}

要想在输入框中禁用输入法，加上这么一句：

{%highlight javascript%}
$("input[name='name']").css('ime-mode', 'disabled');
{%endhighlight%}

只剩下对粘贴的处理了，因为用户有可能粘贴一些非法字符，而多数浏览器并不会为此出发keyPress事件。如果想禁用粘贴键，可以加入者些代码：

{%highlight javascript%}
$("input[name='name']").bind("contextmenu", function(){
    return false;
});
{%endhighlight%}

我们的输入框基本完成了，说基本，是因为用户仍然可以点击菜单上的Edit-&gt;Paste粘贴非法字符。如果你对此很介意，可以考虑参考前面的方法，在keyUp时检测并删除非法字符。

好的，HTML　INPUT输入框就做到这里。如果你觉得麻烦，可以参考我编写的[jQuery plugin](/2011-03-15-jquerynumberlettersplugin.html).
