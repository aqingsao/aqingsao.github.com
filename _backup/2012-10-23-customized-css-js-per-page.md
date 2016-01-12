---
layout: post
title: "为单个页面定制自己的CSS/JavaScript"
description: ""
category: Web 
tags: [Web, Html, CSS, JavaScript, Java, JSP]
---

Web开发的一个最佳实践是“把CSS文件放在HTML页面的Header中，JavaScript文件放在Body的后面”，所以页面的HTML看上去会是这样：

{%highlight html%}
<html>
<head>
	<link rel="stylesheet" href="/stylesheets/app.css">
</head>
<body>
	// content here...
	<script type="text/JavaScript" src="/js/app.js"> </script>
</body>
</html>
{%endhighlight%}


对典型的Web应用来说，一般会有1到多个布局，这样布局下的多个页面可以共享布局的模板和样式。Web开发的语言一般都支持模板，甚至模板嵌套，以减少页面的重复和工作量。这样上面的HTML框架可以放入到模板中，每个页面关注自己的内容，不需要重复引用CSS和JavaScript文件。

我们可以结合模板的使用来实现这一功能。  
看一个简单的网站布局：

