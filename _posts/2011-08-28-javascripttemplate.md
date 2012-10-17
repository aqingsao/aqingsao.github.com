---
layout: post
title: '通过JavaScript Template简化前端JavaScript开发'
category: Web
tags: [Web, Html, JavaScript, Client Templating, trimpath]

---

在使用RESTFul架构的项目中，服务器端通过URI暴露可用的资源和状态表示，很多时候浏览器作为客户端来聚合不同的资源，这必然涉及到大量AJAX的使用，带来的问题就是前端JavaScript越来越重量级，不容易维护。下面通过一个例子来说明前端JavaScript膨胀的后果，以及如何通过JavaScript模板来进行简化。
某电子商务网站中，需要查看促销商品列表及其概要信息，假设对应的页面是这样的：

{%highlight html%}
<div class="resource-holder" data-resource-uri="items/recommended">
<table id="recommendedItems" style="display:none">
<thead>
<tr>
<th>名称</th>
<th>价格</th>
<th>折扣</th>
<th>描述</th>
</tr>
</thead>
</table>
<div id="noRecommendedItems" style="display: none">
<h2>对不起, 当前没有促销商品</h2>
</div>
</div>
{%endhighlight%}

浏览器通过AJAX获取促销商品列表，并对数据进行处理：如果没有任何促销商品，显示一条简单信息；有至少一个，则显示对应的table：

{%highlight javascript linenos%}
$(function() {
	$.getJSON($(".resource-holder").attr('data-resource-uri'), function(data) {
		if(data.length > 0){
			var recommendedItemsView = RecommendedItems.renderView(data);
			$("#promotionalItems").append();
			$("#promotionalItems").show();
		}
		else{
			$("#noPromotionalItems").show();
		}
	});
});
{%endhighlight%}

其中生成表格中tbody部分的函数实现如下：

{%highlight javascript linenos%}
var RecommendedItems = (function() {
	var renderView = function(items) {
		var body = $('<tbody></tbody>');
		var i = 0;
		for (i; i < items.length; i++) {
			var item = items[i];
			var tds = '<td>' + item.name + "</td><td>" + formatter.asRMB(item.price) + "</td><td>" + formatter.asPercentage(item.discount) + "</td><td>" + item.description + "</td>";
			body.append('<tr>' + tds + "</tr>");
		}
		return body;
	};
	return {
		renderView:renderView
	};
})();
{%endhighlight%}

在上面这段JavaScript中，renderView方法根据服务器返回的JSON数据，生成需要显示的tbody内容。由于资源中，价格是数字，折扣是小数，而前端页面显示时会进行一定的格式化，请看for循环里面的两个格式化方法：formatter.asRMB会把指定的金额转换成人民币的形式显示，如10000显示成¥10,000; formatter.asPercentage会把小数显示成分成，如0.40显示成40％。最终返回的tbody会被加入到table中。

由于在JavaScript中混入了HTML的标签，导致代码可读性很差；并且难以利用IDE的自动补全和检查功能，很容易出错。

这么费事，当然让人怀念服务器端的模板引擎。所幸，JavaScript也有模板引擎，比如JST（http://code.google.com/p/trimpath/wiki/JavaScriptTemplates）就很不错。在页面导入相应类库（如trimpath/template.js）后，我们的页面部分就可以改为：

{%highlight html%}
<div data-resource-uri="items/recommended">
</div>

<textarea id="recommendedItemsTemplate" style="display:none">
{if model.items.length > 0}
<table id="recommendedItems">
<thead>
<tr>
<th>名称</th>
<th>价格</th>
<th>折扣</th>
<th>描述</th>
</tr>
</thead>
<tbody>
{for item in model.items}
<tr>
<td>item.name</td>
<td>item.price</td>
<td>item.discount</td>
<td>item.description</td>
</tr>
{/for}
</tbody>
</table>
{else}
<div id="noRecommendedItems">
<h2>对不起, 当前没有促销商品</h2>
</div>
{/if}

</textarea>
{%endhighlight%}

非常类似服务器端的模板引擎，只不过是在浏览器端JavaScript对textarea内的模板内容进行解析，并添加到resource-holder中。
{%highlight javascript linenos%}
$(function() {
	$.getJSON($(".resource-holder").attr('data-resource-uri'), function(data) {
		var formattedData = RecommendedItems.formatData(data);
		var mergedTemplate = TrimPath.parseTemplate($("#recommendedItemsTemplate").html(), "#recommendedItemsTemplate").process({model:formattedData})
		$(".resource-holder").append(mergedTemplate);
		$("#recommendedItemsTemplate").remove();
	});
});
var RecommendedItems = (function() {
	var formatData = function(items) {
		var rows = [];
		var i = 0;
		for (i; i < items.length; i++) {
			rows.push({"name":items[i].planNumber,
			"price":formatter.asRMB(items[i].price),
			"discount":formatter.asPercentage(items[i].discount),
			"description":items[i].description});
		}
		return rows;
	};
	return {
		formatData:formatData
	};
})();
{%endhighlight%}

修改之后，把页面呈现逻辑（AJAX调用）和格式化逻辑完全分开。FormatData只负责数据的格式化，比如金额显示成人民币的央视；折扣显示成百分比，因此职责清晰，易于编写单元测试；AJAX方法先调用formatData方法，然后把模板内容和数据传递给JST的模板引擎，并最终在callback方法中把生成的内容添加到合适的位置，属于页面呈现的内容，无需单元测试。

至此，我们可以进行一些简单的重构：
1. 在引入单独的模板vm文件，比如把recommendedItemsTemplate的所有内容抽取到单独的recommended_items_template.vm文件

{%highlight html%}
<textarea id="recommendedItemsTemplate" style="display:none">
#include("recommended_items_template.vm")
</textarea>
{%endhighlight%}
抽取模板文件后，代码更清晰，而且模板vm可以被重用。

2. 封装JavaScript模板引擎方法，比如mergeAndShow方法，只需要提供两个参数：原始模板内容；格式化之后的数据。改方法调用模板引擎生成页面，并把页面内容以callback的方式传回：

{%highlight javascript linenos%}
jsTemplate.mergeAndShow($("#recommendedItemsTemplate").html(), formattedData, function(mergedTemplate) {
	$(".resource-holder").append(mergedTemplate);
	$("#recommendedItemsTemplate").remove();
});
{%endhighlight%}

与最初的方法相比，使用JavaScript模板并进行简单的重构之后，JavaScript代码和页面代码都比原来简洁的多，呈现和业务逻辑又了更好的分离，易于编写单元测试，代码质量也有了提高。
