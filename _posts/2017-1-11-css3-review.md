---
layout: post
title: CSS 3 流水账
categories: CSS
excerpt: 一个不深不浅的坑，趁有空多踩几脚。
---

![CSS3](http://upload-images.jianshu.io/upload_images/658453-71b77d2f1a68caf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> #### [菜鸟教程-CSS3教程](http://www.runoob.com/css/css-tutorial.html)

#### CSS 选择器
1. 元素选择器
  * `div {
		background-color: gray;
		font: italic normal bold 14pt normal 楷体_GB2312;
	}`
2. 属性选择器：
  * `div[id]`：所有含有`id`属性的`div`元素；
  * `div[id=xx]`：`id`属性值为`xx`的`div`元素；
  * `div[id ~= xx]`：`id`属性值为**以空格隔开的系列值，其中某个值为`xx`**；
  * `div[id |= xx]`：`id`属性值为**以`|`隔开的系列值，其中某个值为`xx`**；
  * `div[id*=xx]`：`id`属性**包含`xx`**的`div`元素；
  * `div[id^=xx]`：`id`属性以`xx`开始的`div`元素；
  * `div[id$=xx]`：`id`属性以`xx`结束的`div`元素；
  * `div[id^=xx]`：`id`属性以`xx`开始的`div`元素；
3. ID 选择器：`#idValue { ... }`
4. class 选择器：`.myclass`，对所有 `class` 为 `myclass` 的元素起作用；**`div.myclass`，对 `class` 为 `myclass` 的 `div` 起作用**；
5. **包含选择器**：`selector1 selector2 { ... }`，满足 `selector1`**内部的**，再满足`selector2`的元素：`div .a`，`div`**内部**，`class`为`a`的元素；
6. **子选择器**：`selector1 > selector2 { ... }`，满足`selector1`的元素的，**直接子元素**中，再满足`selector2`的元素：`div > .a`，`div`的子元素中，`class`为`a`的；
7. 选择器组合：有些时候，我们需要让一份 CSS 样式对多个选择器起作用，那就可以利用选择器组合来实现了。语法如下：
`selector1, selector2, selector3, ... { ... }`
`div, .a, #abc`：对`div`元素、`class`为`a`的元素、`id`为`abc`的元素都起作用。

#### CSS 3.0 新增的伪类选择器
* `first-child, last-child, nth-child, nth-last-child`
* `first-of-type, last-of-type, nth-of-type, nth-last-of-type`
* `link, visited, active, hover, focus, enabled, disabled, checked, default, read-only, read-write, selection`

#### 其他
* `text-align`：文本对其方式，`left`, `right`, `center`, `justify`, 左右中间两边；
* `word-break`：设置目标组件中文本内容的换行方式：
  * `normal`：靠浏览器的默认规则进行换行。通常，浏览器的处理规则是，对于英文单词，浏览器只会在半角空格、连字符等地方进行换行，不会在单词中间换行；对于中文来说，浏览器可以在任何一个中文字符后换行；
  * `keep-all`：只能在半角空格或连字符处换行；
  * `break-all`：允许在任何单词(中文&英文)中间换行；

# 组件的居中对齐方式呢？
`style="margin: auto; width=800px or 70%"`

#### CSS 3 盒子模型
![CSS 3 盒子模型](http://upload-images.jianshu.io/upload_images/658453-b524c053fedc1b26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`style.display: none`：隐藏元素且不占空间；

`style.visibility: visible/hidden`：显示/隐藏元素，但始终占空间；

#### CSS 3 分页
	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="utf-8"> 
	<title>菜鸟教程(runoob.com)</title> 
	<style>
	ul.pagination {
		display: inline-block;
		padding: 0;
		margin: 0;
	}

	ul.pagination li {display: inline;}

	ul.pagination li a {
		color: black;
		float: left;
		padding: 8px 16px;
		text-decoration: none;
		<!-- 过渡动画 -->
		transition: background-color .3s;
		border: 1px solid #ddd;
		<!-- 加上圆角 -->
		<!-- border-radius: 5px; -->
	}

	.pagination li:first-child a {
		border-top-left-radius: 5px;
		border-bottom-left-radius: 5px;
	}

	.pagination li:last-child a {
		border-top-right-radius: 5px;
		border-bottom-right-radius: 5px;
	}

	ul.pagination li a.active {
		background-color: #4CAF50;
		color: white;
		border: 1px solid #4CAF50;
	}

	ul.pagination li a:hover:not(.active) {background-color: #ddd;}
	</style>
	</head>
	<body>

	<h2>CSS3 分页实例</h2>
	<ul class="pagination">
	  <li><a href="#">«</a></li>
	  <li><a href="#">1</a></li>
	  <li><a class="active" href="#">2</a></li>
	  <li><a href="#">3</a></li>
	  <li><a href="#">4</a></li>
	  <li><a href="#">5</a></li>
	  <li><a href="#">6</a></li>
	  <li><a href="#">7</a></li>
	  <li><a href="#">»</a></li>
	</ul>

	</body>
	</html>

![CSS3 分页实例](http://upload-images.jianshu.io/upload_images/658453-459abbd973f808f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### CSS3 动画
CSS3 也是可以制作动画的，可以取代部分动态图片和 JavaScript。
[菜鸟教程连接](http://www.runoob.com/css3/css3-animations.html)

#### CSS 媒体查询 @media
> **1、什么是媒体查询**
>
> 媒体查询可以让我们根据设备显示器的特性（如视口宽度、屏幕比例、设备方向：横向或纵向）为其设定CSS样式，媒体查询由媒体类型和一个或多个检测媒体特性的条件表达式组成。媒体查询中可用于检测的媒体特性有 width 、 height 和 color （等）。使用媒体查询，可以在不改变页面内容的情况下，为特定的一些输出设备定制显示效果。

> **2、为什么响应式设计需要媒体查询**
>
> 如果没有CSS3的媒体查询模块，就不能针对设备特性（如视口宽度）设置特定的CSS样式

#### 媒体查询例子：

如果浏览器窗口小于 500px, 背景将变为浅蓝色：

```css
@media only screen and (max-width: 500px) {    
body { 
	background-color: lightblue; 
}}
```

# Try It！Amazing！

#### 图片的响应式设计

```css
img {
	<!-- 图片可能会超过其原始大小 -->
    <!-- width: 100%; -->
	<!-- 用 max-width 就行了 -->
	max-width: 100%;
    height: auto;
}
```

#### [菜鸟教程-响应式 Web 设计](http://www.runoob.com/css/css-rwd-intro.html)

#### viewport

> **什么是 Viewport?**
>
> viewport 是用户网页的可视区域。
viewport 翻译为中文可以叫做"视区"。
手机浏览器是把页面放在一个虚拟的"窗口"（viewport）中，通常这个虚拟的"窗口"（viewport）比屏幕宽，这样就不用把每个网页挤到很小的窗口中（这样会破坏没有针对手机浏览器优化的网页的布局），用户可以通过平移和缩放来看网页的不同部分。

一个常用的针对移动网页优化过的页面的 viewport meta 标签大致如下：

`<meta name="viewport" content="width=device-width, initial-scale=1.0">`

#### 响应式 Web 设计框架

# [Bootstrap](http://www.bootcss.com/)

#### 其他

`<meta http-equiv="X-UA-Compatible" content="IE=edge">`：

> 这是一个**文档兼容模式**的定义。Edge 模式告诉 IE 以最高级模式渲染文档，也就是任何 IE 版本都以当前版本所支持的最高级标准模式渲染，避免版本升级造成的影响。

来自 Bootstrap 的说明：

> Bootstrap 不支持 IE 古老的兼容模式。为了让 IE 浏览器运行最新的渲染模式下，建议将此 <meta> 标签加入到你的页面中：
`<meta http-equiv="X-UA-Compatible" content="IE=edge">`
按 F12 键打开 IE 的调试工具，就可以看到 IE 当前的渲染模式是什么。
此 meta 标签被包含在了所有 Bootstrap 文档和实例页面中，为的就是在每个被支持的 IE 版本中拥有最好的绘制效果。

---