---
layout: post
title: "Client templating"
description: "To reduce server side burden and get a better performance, template html and data, normally json, can be transported client side, and then merged into HTML pieces at client side."
category: HTML
tags: [HTML, client templating, JSON]
---
{% include JB/setup %}

<h2> {{page.title}}</h2>
<p> My first article</p>
<p>{{page.date | date_to_string}}</p>