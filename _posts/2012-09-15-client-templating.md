---
layout: post
title: "客户端模板优化性能介绍"
description: "To reduce server side burden and get a better performance, template html and data, normally json, can be transported client side, and then merged into HTML pieces at client side."
category: WEB
tags: [HTML, client templating, JSON, web，客户端模板]
---
{% include JB/setup %}

<h2> {{page.title}}</h2>
<p> My first article</p>
<p>{{page.date | date_to_string}}</p>