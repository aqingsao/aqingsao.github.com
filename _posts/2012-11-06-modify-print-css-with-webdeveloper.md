---
layout: post
title: "通过WebDeveloper修改浏览器打印媒介的CSS"
description: ""
category: Tools
tags: [Tools, WebDeveloper, Firebugs, CSS]
---

网上找到一篇好文章，想打印出来怎么办？作为一个码农，为了节省纸张，我会打开Firebugs或其他开发工具，把不需要的头部、尾部、左右边框全部删掉，这通常没有问题。但有时为了更好的打印效果，或者节省更多的纸张，我还会通过CSS调整段落的上下边距、字体大小等等，问题出现了：打印时，它们并没有生效。

这是因为我们在浏览器中看到的显示效果是基于屏幕媒介（@media screen)，而打印时使用的是打印媒介(@media print)。通过Firebugs工具修改的只是前者，所以打印时无效。看来，我们需要一个工具，帮助我们修改打印媒介——这个工具就是[WebDeveloper](addons.mozilla.org/en-US/firefox/addon/web-developer),下载以后，就可以选择想用的打印媒介：
<p class="image-container middle">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/select-print-media.png"></a>
</p>

作为码农，相信你已经知道该怎么办了：打开Firebugs，随心所欲地修改CSS吧，配合Preview的功能，打印出来就是你想要的效果。

除了Firefox，WebDeveloper在Chrome下也有插件以供使用。
