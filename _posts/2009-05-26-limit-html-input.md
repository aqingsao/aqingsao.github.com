---
layout: post
title: "HTML输入框中限制只输入金额（非负小数）"
description: "本文使用一个简单的方法限制HTML输入框只能输入金额"
category: HTML
tags: [HTML]
---
目前实现了：
1. 只能输入1个小数点
2. 只能输入数字
3. 可以输入以下特殊字符：回退；删除；home；end；箭头
4. 限制用户不能拷贝粘贴；
代码中使用char code进行验证，同时如果按的是数字键，则不能同时按Shift键。
 
下面是JavaScript方法：

{% highlight javascript linenos %}
function limit_money_input() {  
    $("input.money").bind("contextmenu", function(){  
        return false;  
    });  
  
    $("input.money").css('ime-mode', 'disabled');  
      
    $("input.money").bind("keydown", function(e) {  
        var key = window.event ? e.keyCode : e.which;  
        if (isFullStop(key)) {  
            return $(this).val().indexOf('.') < 0;  
        }  
        return (isSpecialKey(key)) || ((isNumber(key) && !e.shiftKey));  
    });  
}  
  
function isNumber(key) {  
    return key >= 48 && key <= 57  
}  
  
function isSpecialKey(key) {  
    //8:backspace; 46:delete; 37-40:arrows; 36:home; 35:end; 9:tab; 13:enter  
    return key == 8 || key == 46 || (key >= 37 && key <= 40) || key == 35 || key == 36 || key == 9 || key == 13  
}  
  
function isFullStop(key) {  
    return key == 190 || key == 110;  
}  
{% endhighlight %}

 
使用使只需要在页面加载时调用即可，比如用jquery：

{% highlight javascript linenos %}
$(limit_money_input);  
{% endhighlight %}
 
