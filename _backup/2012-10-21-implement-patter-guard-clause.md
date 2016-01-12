---
layout: post
title: "《实现模式》读书笔记-Guard Clause"
description: ""
category: Clean Code
tags: [Implementation Pattern, Guard Clause, Clean Code, XP]
---

>《实现模式》是软件大师Kent Beck的又一部作品。与其他作品相比，本部稍显差一些：首先是很多地方陷于细节，对在XP领域浸淫多年的老兵来说，很多内容是老生常谈。不过我们还是应该从中汲取一些有用的营养，补充到自己的知识库。

我们经常遇到这样的代码：
{%highlight Java linenos%}
	public List<Book> findBooksIn(){
	}
}
{%endhighlight%}