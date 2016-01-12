---
layout: post
title: 'magic cube on html5'
keywords: Magic Cube, html5, canvas
description: a magic cube written in html5 canvas
category: Html
tags: [Html, Html5, Canvas, JavaScript, ThoughtWorks]

---

在去年秋天ThoughtWorks举行的Html5/CSS3/JavaScript比赛中，我的参赛作品是魔方。恰好当时对Canvas比较感兴趣，有个想法就是做一个类似JFreeChart/Google Chart的图形类库，但是已经有了类似的类库：RGraph，而且画出的图片相当漂亮，念头打消以后，决定做一个具有3D效果的魔方。
从创意到绘制草图，从学习API到算法，累计花了一天的时间，总算做出来了，下面是效果图([参见源代码](https://github.com/aqingsao/MagicCubeOnHtml5))：
[![](/assets/images/magicCube.png)]()

虽然很简单的一个小游戏，但有一些值得总结的地方：

* Canvas的原生API虽然简单，但是事件处理、效果等等都需要自己处理，因此代码量较大，而这正应是类库发挥作用的时候。对于复杂一些的应用，完全值得花时间去研究一下相关的类库，比如基于Canvas的RGraph图形类库、用于实现3D效果的WebGL、three.js等，甚至一些游戏引擎等等；
* 魔方的算法非常有意思，如果以魔方的中间为原点，则所有的面都可以用-1、0、1三个值表示的坐标系标示出来。因此总觉得会有一个合适的矩阵可以表示，这样所有的旋转操作都可以看作矩阵的变换，算法也应该当想简单。但穷尽了我的矩阵知识，以失败告终，于是回到了最土的方式计算。
* 随着Html5的日益流行，除了游戏，图片类库等等肯定大有用武之地。类似JFreeChart这样后台生成图片的方式已经很落伍；Google Chart也需要google服务器生成，而基于Canvas的图形类库会与基于SVG的图形类库一样进入大家的视线，很容易实现原来只有Flash能实现的缩放、旋转、事件处理等功能，所以不难理解除了RGraph，还已经有了不少这样的类库。
