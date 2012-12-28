---
layout: post
title: 'jQueryNumberLettersPlugin 0.1.1 Released'
keywords: jQuery plugin, HTML Input, limit input, number, money, email, keyPress, keyDown, 
Description: A jQuery plugin for users to input number, email, money only
category: Html
tags: [Html, jQuery, jquerynumberlettersplugin]
---

休了几天假，让我有时间写了一个jQuery的插件：NumberLetters。这个想法来自于前两篇博客：如何做一个完美的HTML INPUT输入框（[上](/2011/02/15/html-input-box-1/)）和（[下](/2011/02/19/html-input-box-2/)），希望做成jQuery的插件后，能够有人使用。

在前面的例子中，通过在keyPress()事件判断该按键是否允许，从而拦截非法字符。但是该例子非常特殊，因为需求非常明确，可以预先判断哪些字符是允许的。想写一个通用的插件，需要考虑用户自定义的情况，比如用户想在INPUT输入框中限制输入IP地址，只靠字符本身不够，必须通过正则表达式。于是，插件的实现使用正则表达式做的，这样可以带来最大的灵活性。

既然决定用正则表达式，面临的一个直接问题是：需要知道该输入框的值是多少，因为用户可能在后面添加字符，有可能在中间插入字符，甚至可能选中后替换。而这只有keyDown()之后，用户输入的字符显示出来，才能得到，但此时已经无法阻止用户输入，体验会比较差。

经过考虑，采用的方法是keyPress()时，通过JavaScript Event来判断，该键按下之后，输入框的值可能变成什么。这用到了jQuery对event事件的封装，通过selectionStart和selectionEnd查看鼠标位置，是否选中字符，并把其变成按键对应的字符。经过实验，该方法还是可以的，于是jQueryNumberLettersPlugin 0.1.1 发布啦。

其用法如下：

{%highlight JavaScript%}
$("input").letters();
$("input").ip();
{%endhighlight%}

更多信息，您可以在<a href="http://plugins.jquery.com/project/jQueryNumberLettersPlugin" target="_blank">jQuery Plugin页面</a>上查看，也可以在<a href="http://xiaoqing.me/projects/jquerynumberlettersplugin/">jQueryNumberLettersPlugin主页查看</a>。
