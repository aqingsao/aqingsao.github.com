---
layout: post
title: 'ThoghtWorks technology radar'
keywords: ThoughtWorks, Technical Radar
description: a html version written in HTML5 canvas for ThoughtWorks technical radar
category: Html
tags: [Html, Html5, Canvas, JavaScript, ThoughtWorks, Technology Radar]

---

[技术雷达](http://www.thoughtworks.com/radar)是[ThoughtWorks](http://www.thoughtworks.com/cn/)定期发布的技术白皮书，它对当下流行的技术趋势采用多种维度评估，并以客观、中立的角度将其分为采用、试用、评估、保留四个级别，供IT企业的决策者进行参考，现在每半年发布一次。
目前ThoughtWorks技术雷达的发布格式为PDF，在此基础上，做了个小项目，提供Web版的技术雷达，如下图所示：
[![ThoughtWorks Technology Radar](/assets/images/tech-radar.png "ThoughtWorks Technology Radar")](#)

该项目采用HTML Canvas和JavaScript的方式实现。首先尝试了Canvas原生API，但是手写需要很大的工作量，转而采用类库[RGraph](http://www.rgraph.net/)，享受别人已经发明的轮子。在RGraph支持的众多图表类型中，雷达图就是其一。  
可惜的是，其雷达图扩展接口太少，只好修改了部分代码。
您可以[点击查看源代码](https://github.com/aqingsao/TechRadar)。如果您有问题或者有更好的创意，欢迎联系我。
