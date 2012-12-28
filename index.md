---
layout: default
title: My Thought, My Works
keywords: Web, Java, J2EE, Spring, Hibernate, Ruby, Rails, JavaScript, NodeJS, html5, 自动化测试, Thoughtworks, 张晓庆, aqingsao, Algorithm, Technical, 技术
description: 张晓庆的技术博客(Technical blog for aqingsao): ThoughtWorker/Agile粉/技术控, 关注Web开发，企业级应用，J2EE，分布式开发，性能调优，Java, Ruby, NodeJS, 敏捷, Agile

---

<div id="posts">
  {% for post in site.posts limit: 10 %}
    <div class="post">
      <div class="title"><a href="{{ post.url }}">{{ post.title }}</a></div>
      <div class="content">{{ post.content }}</div>
      <div class="footer"> 
        <div class="tags">
          {% for tag in post.tags %}
            <a class="tag" href="/tags.html#{{ tag }}">#{{ tag }}</a>
          {% endfor %}
        </div>
        <span class="date">{{ post.date | date_to_string }}</span><span class="author"> posted by {{ site.author.name }} in</span>  
        <span><a class="category" href="/categories.html#{{ post.category }}">{{ post.category }}</a></span>
        <!-- <span><a class="comments" href="{{ post.url }}#disqus_thread"></a></span> -->
      </div>
    </div>
  {% endfor %}
</div>
