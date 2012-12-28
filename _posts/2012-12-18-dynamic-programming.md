---
layout: post
title: "通过动态规划求绳子最大长度"
keywords: Dynamic Programming, 动态规划, Algorithm
description: "Use dynamic programming to solve a problem: Calculate the max length of a rope，动态规划"
category: Algorithm
tags: [Algorithm, 动态规划, Dynamic Programming]
---

动态规划，由20世纪50年代初美国数学家R.E.Bellman等人创建，把多阶段过程转化为一系列单阶段问题，利用各阶段之间的关系，逐个求解，创立了解决这类过程优化问题的新方法——动态规划。本文以一个具体的例子，说明如何在实际问题用应用动态规划算法。

###问题描述
假如有n根棍子，每个棍子的高度可以调节，最低为1，但最高不超过数组MH\[n\]中定义的最大高度，相邻两个棍子的距离为1。现在把每根棍子的顶部用绳子连接起来，请问如何调节棍子使得绳子达到最长？

比如，共有四根棍子，最大长度分别为：MH=\[6, 8, 2, 7\].如果调整棍子高度，才能使得连接使用的绳子最长?
<p class="image-container middle">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/dynamic-programming-all.png"></a>
</p>


###问题分析
通过简单分析，可以知道，对N根棍子来说，一共有MH\[1\]\*MH\[2\]\*...\*MH\[n\]种可能的长度。假如棍子的平均长度为m，把每一种可能的长度计算出来再取最大值，则算法的复杂度很高，达到m^n，即指数级别。这里我们可以通过动态规划算法，降低算法复杂度。

我们首先看倒数第二根棍子（即N-1），看其距离最后一根棍子的最大长度是多少。只有两根棍子，问题就简化了很多。

假如绳子需要从该棍子的高点通过，则从其高点到最后一根棍子，共有MH\[n\]中可能的选择，但是通过计算，很容易算出距离最长的那个。在本图所示案例中，共有7种选择，不难算出最后一根棍子需取高点，使得距离最远，距离为Math.sqrt(1^2 + (MH(n) - MH(n-1))^2)，即Math.sqrt(1+(7-2)^2).
由此得出结论，如果绳子必须从n-1高点通过，那么必然选择第n跟棍子的高点。

但是绳子未必会选择(N-1)根棍子的高点，如果选择其低点呢？类似可以得到，最后一根棍子的MH\[n\]种可能中，选择其高点也会使绳子的长度最大: Math.sqrt(1^2 + (MH(n) - 1)^2)，即Math.sqrt(1+(7-1)^2)，如图所示：

<p class="image-container middle">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/dynamic-programming-last2.png"></a>
</p>

同理我们可以算出，如果绳子需要经过第(N-1)根棍子的任何一点i，无论高点、低点、还是中间某点，都可以在最后一根棍子上选择一点，使得绳子最长，我们我们称这个最大长度为ML\[n-1\]\[i\]，\[n-1\]表示倒数第二根棍子，\[i\]表示其上的第i点，而ML\[n-1\]\[i\]表示改点最后一根棍子的最长距离。

如果只有两根棍子，很容易比较所有的ML\[n-1\]\[i\]，找到最大的那个。然而实际情况是，前面还有棍子，无法得知倒数第二根选择哪一点才能使得总体的长度最大。不过没关系，我们已经完成了很重要的一步，即把所有的MH\[n-1\]\*MH\[n\]种可能，简化为了MH\[n-1\]种可能。我们向前走一步，看倒数第三根棍子(N-2)。

同样假设绳子通过其高点，其与倒数第二根棍子有MH(n-1)种可能，不过这次计算时，不仅需要考虑两者的长度，还需要考虑到最后一根棍子绳子的总长度。如图所示，本例中，不难算出，如果倒数第二根棍子选择低点，可以使得绳子的总长度最大：Math.sqrt(1^2+(MH\[n-2\] - 1)^2) + Math.sqrt(1^2 + (MH(n) - 1)^2); 即Math.sqrt(1+(8-1)^2) + Math.sqrt(1+(7-1)^2).

同理，如果绳子必须通过N-2根棍子的低点，或者任何其他一点，总可以在第N-1根棍子上找到合适一点，使得绳子的总长度达到最大。

其实分析到这里，我们已经成功的把MH\[N-2\]\*MH\[N-1\]\*MH\[N\]种可能，降低为只有MH\[n-2\]种可能。
<p class="image-container middle">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/dynamic-programming-last3.png"></a>
</p>

以此向前推，直到第一根棍子，我们不难算出，无论选择其上的任何一点，其实总可以在第二根棍子上找到合适的一点，使得第一根棍子到最后一根棍子连接起来，绳子的长度达到最大，亦即只有MH\[1\]种可能。把这MH\[1\]中可能进行比较，自然可以找到绳子的最大长度，如图所示：
<p class="image-container middle">
<a href="#"><img alt="Select css media from webDeveloper" src="/assets/images/dynamic-programming-result.png"></a>
</p>

###算法优化
上面的算法中，从后往前算，对每根棍子来说，都把可能的选择降到了最低。其实，通过分析我们可以得出一个结论：每一根棍子要么取最小值1，要么取最大值MH\[n\]，才能使得最后的绳子长度最大。依赖这个假设，上面的计算过程可以进一步简化。

###算法实现
一旦算法出来，实现非常简单。下面是Java实现的代码片段:

{%highlight javascript linenos%}
double totalLengthOnHigh = 0;
double totalLengthOnLow = 0;

for (int i = maxHeight.length - 1; i > 0; i--) {
	double lengthHigh2High = Math.sqrt(Math.pow(maxHeight\[i - 1\] - maxHeight\[i\], 2) + 1);
    double lengthHigh2Low = Math.sqrt(Math.pow(maxHeight\[i - 1\] - 1, 2) + 1);

    double lengthLow2High = Math.sqrt(Math.pow(maxHeight\[i\] - 1, 2) + 1);
    double lengthLow2Low = 1;

    double lengthOnHigh = Math.max(lengthHigh2High + totalLengthOnHigh, lengthHigh2Low + totalLengthOnLow);
    double lengthOnLow = Math.max(lengthLow2High + totalLengthOnHigh, lengthLow2Low + totalLengthOnLow);
    totalLengthOnHigh = lengthOnHigh;
    totalLengthOnLow = lengthOnLow;
}
double maxLength = Math.max(totalLengthOnHigh, totalLengthOnLow);
{%endhighlight%}

全部源码和测试，可以参考我在[GitHub上的实现](https://github.com/aqingsao/algorithm-sticks).
