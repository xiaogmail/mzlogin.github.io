---
layout: post
title: HTML 5 流水账
categories: HTML
description: 第二个不深不浅的坑，踩一踩。
---

![HTML5](http://upload-images.jianshu.io/upload_images/658453-18021a7df5734359.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 引言
> HTML5 致力于解决跨浏览器问题，可以取代部分原来要靠JavaScript实现的功能；
> 
> HTML 的发展历史“比较复杂”，因为它实在太“随意”了，而负责解析 HTML 的浏览器又太“宽容”了，以至于到了“写一份对的 HTML 文档很容易，写一份错的 HTML 文档很难”的程度。而且不同浏览器之间又存在一些差异，因此导致 HTML 文档给人的感觉比较混乱。
> 
> 2008年，一项关于Alexa全球500强网站的调查表明，仅有 6.57%的网站能通过 HTML 规范验证。如果把那些名不见经传的小网站考虑在内，整个互联网上就几乎都是不符合规范的 HTML 页面。
> 
> 一方面，W3C 组织“声嘶力竭”的呼吁大家应该制作遵守规范的 HTML 页面；另一方面，各种浏览器却可以正常解析、显示那些不符规范的页面，HTML 页面制作者根本不太理会这种呼吁。

现有的 HTML 页面中大量存在以下 4 种不符合规范的内容：

1. 元素标签名大小写混杂：`<p>HTML</P> `

2. 元素没有结束符

3. 元素中的属性，只有属性名，没有属性值：`<input type="text" disabled>`

4. 属性值没有用引号：`<input type=text>`

可能是出于“存在即合理”的考虑，WHATWG组织开始制定一种“妥协式”的规范：HTML 5。**既然互联网上大量存在上面 4 种不符合规范的内容，而且制作者从来也不打算修改这些页面，因此 HTML 5 干脆承认它们是符合规范的。换句话说，HTML 5 是规范制定者对现实的妥协。**

# HTML 5 的优势
1. 解决跨浏览器问题；
2. 部分代替了原来的JavaScript：
  * `document.getElementById("price").focus();`
  * `< input type="text" autofocus name="price" >`
3. 更明确的**语义**支持：
	* 在 HTML 5 以前，如果要表达一个文档结构，可能只能通过 div 元素来实现：	 
		- `<div id="header" > ... < /div >`
		 - `<div id="nav" > ... < /div >`
		 - `<div id="article" > ... < /div >`
		 - `<div id="section" > ... < /div >`
		 - `<div id="aside" > ... < /div >`
		 - `<div id="footer" > ... < /div >`
	* HTML 5 为上面的页面布局提供了更明确的语义元素：
		 - `< header > ... < header >`
		 - `< nav > ... < nav >`
		 - `< article > ... < article >`
		 - `< section > ... < section >`
		 - `< aside > ... < aside >`
		 - `< footer > ... < footer >`
4. 增强了 Web 应用的功能：例如以前上传文件时想同时选择多个文件都不行，必须通过 JavaScript 来实现；为了弥补这些不足，HTML 5 规范增加了不少新的 API ，而各种浏览器正努力实现这些 API 功能，在未来的日子里，使用 HTML 5 开发 Web 应用将变得更加轻松。

# HTML 5 基本结构和语法变化

#### DTD 定义：

`< !DOCTYPE html >`

#### 基本结构：

```html
<! DOCTYPE html>
<html>
	<head>
		<title>页面标题，显示在浏览器标签栏</title>
		<meta charset="utf-8" />
		<!-- 此处可以插入其他meta、样式单等信息 -->
		<link href="css/bootstrap.min.css" rel="stylesheet" />
	</head>
	
	<body>
		<h1> Hello, World! </h1>
		页面内容
		
		<script src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
	</body>
</html>
```


#### 基本元素：

	<style> 用于引入CSS
	<h1> 到 <h6> 用于引入标题一到标题六
	<p> 定义段落
	<br> 换行
	<hr> 定义水平线
	<div> 定义文档中的节
	<span> 与 <div> 基本类似，区别是该定义的节默认不会换行
`<span>`、`<p>`、`<div>`三个元素的效果有点类似，它们都可以作为其他内容的容器。在默认情况下，`<span>`元素不会导致换行，而`<div>`元素会导致换行，而`<p>`元素会产生一个段落，段落和段落之间默认有更大的间距。

除此之外，`<p>` 和`<span>`只能包含**文本类**内容，如文本、图像、超链接、文本格式化元素(`<em>`等)和表单控件等内容；
**`<p>`可以包含`<span>`元素，但`<span>`不能包含`<p>`元素**

而`<div>`几乎可以包含所有内容：`<h1>到<h6>、<form>、<table>、<ol>、<ul>`和`<div>、<p>、<span>`。**正是因为`<div>`元素可以包含各种各样的内容，因此在 HTML 5 以前，经常会大量使用`<div>`元素来完成页面布局。**

##### 文本格式化元素
* `<b>` 粗体文本
* `<i>` 斜体文本
* `<em>` 强调文本，实际效果和`<i>`差不多
* `<strong>` 粗体文本，与`<b>`基本相同
* `<big>` 大号字体文本
* `<small>` 小号字体文本
* `<sup>` 上标文本
* `<sub>` 下标文本

上面这些文本格式化元素能继续包含**文本类**内容，如文本、图像、超链接、文本格式化元素和表单控件元素；**还可以包含**`<span>`**！**

##### 语义相关元素
* `<abbr>` 表示一个缩写
* `<address>` 表示一个地址，浏览器通常会用斜体字显示地址内容
* `<blockquote>` 引用，长的带换行的大段引用
* `<q>` 引用，短的不带换行的引用
* `<code>` 一段计算机代码
* `<dfn>` 一个专业术语，浏览器通常会用粗体或斜体显示
* `<del>` 被删除的文本，浏览器以中划线显示
* `<ins>` 插入的文本，通常以下划线显示
* `<pre>` 表示该元素包含的文本已经进行了**预格式化**，也就是说`<pre>` 所包含的空格、回车、Tab和`< > / \ ' " ` 等**格式字符**都会被保留下来
* `<var>` 表示一个变量。浏览器通常以斜体显示`<var>`所包含的文本

##### 超链接和锚点

`< a... />`

* `href`：链接地址

* `target`：指定使用框架集中的哪个框架来装载。属性值可以为_self、_blank、_top、_parent 四个，分别代表使用自身、新窗口、顶层框架、父框架来加载新资源

* `media`：指定目标 url 所引用的媒体类型。默认值为 all。是 HTML 5 新增的属性。

**命名锚点就是页面内的定位点，用来实现页面内的定位：**

`<a name="anchor1">`

`<a href="#anchor1">`

##### 列表元素
* `<ol>` 有序列表
  * 可以通过`start`和`type`属性指定起始数字和编号类型（数字、字母、罗马数字）
* `<ul>` 无序列表
* `<li>` 列表项
* `<dl>` 自定义列表

```html
<dl>
	<!-- 定义标题列表项 -->
	<dt>要掌握的编程语言</dt>
	<dd>Java</dd>
	<dd>C++</dd>
	<dd>Python</dd>
	<dd>JavaScript</dd>
</dl>
```

![自定义列表](http://upload-images.jianshu.io/upload_images/658453-be440d36bde5ca78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 图片相关元素

`<img>`


此外，还可以指定以下属性：

* `height`：高度，可以为**百分比**或像素值

* `width`：宽度，可以为**百分比**或像素值

**图片映射：将一幅图片划分成不同的区域，每一个区域可以设置一个链接地址**
http://blog.csdn.net/qiantujava/article/details/18305709
![图片映射 图中每一个头像都连接到不同的地址](http://upload-images.jianshu.io/upload_images/658453-a9365b7172e2a1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 表格

**< table >**

* `<caption>` 定义表格标题
* `<thead>` 表格头
* `<tfoot>` 表格脚
* `<tr>` 表格行
* `<td>` 单元格
* `<th>` 表格页眉
* `<tbody>` 表格体

> 使用`<tbody>`标签，可以将一个表格分成几个独立的部分。`<tbody>`元素可以将表格中的一行或几行合并成一组，当我们使用 Ajax 编程时常常需要动态修改表格中的某几行，这就需要使用`<tbody>`元素了。

```html
    <table border="1" style="width:400px">
    <caption><b>疯狂Java体系图书</b></caption>
    <thead>
        <tr>
            <th>书名</th>
            <th>作者</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>疯狂Java讲义</td>
            <td>李刚</td>
        </tr>
        <tr>
            <td>轻量级Java EE企业应用实战</td>
            <td>李刚</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="2" style="text-align:right">现总计：2本图书</td>
        </tr>
    </tfoot>
    </table>
```
![tbody thead tfoot](http://upload-images.jianshu.io/upload_images/658453-6eea66a5cfd5c28f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 框架
> HTML 5 不再推荐在页面中使用**框架集**，因此 HTML 5 删除了`<frameset>`、`<frame>`、和`<noframes>`三个标签。

HTML 5 依然保留了一个与框架相关的元素：`<iframe>`，**该元素用于在普通 HTML 页面中生成一个内联框架，可以直接放在 HTML 页面的任意位置，用于加载一个 url 指向的页面。**

`<iframe src="https://www.baidu.com/" >`

![iframe](http://upload-images.jianshu.io/upload_images/658453-369bfc926842b078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### HTML 5 新增的通用属性
> HTML 5 保留了大部分原有的 HTML 元素，但为这些元素增加了一些通用属性，这些属性极大地增强了 HTML 元素的功能。

* contentEditable 属性：允许直接编辑元素里的内容。此处的 HTML 元素并不是指那些原来就允许用户输入的表单元素，如文本框、文本域子类的；而是可以把`<table>`、`<div>`等元素变成可编辑状态。
* designMode 属性：相当于一个全局的 contentEditable 属性，如果把整个页面的 designMode属性设为 on 时，该页面上所有可支持 contentEditable 的元素都变为可编辑状态。
* hidden 属性：通知浏览器不显示元素，且**不占用空间**；相当于`style="display: none"`。
* spellcheck：为`<input>`、`<textarea>`等元素增加了 spellcheck 属性，浏览器会对输入进行单词拼写检查（错误的单词划红色的下划线）。

##### HTML 5 新增的常用元素

###### 文档结构元素

* `<article>`：代表页面上独立、完整的一篇“文章”，可以使一个帖子、一篇 Blog 文章、一篇短文、一条完整的回复等。
  * `<article>`内部可以用`<header>`定义文章标题部分，用`<footer>`定义脚注，用多个`<section>`将文章内容分成几个段落，同时还可以嵌套多个`<article>`作为它的附属文章，比如一篇 Blog 文章后面可以有多篇回复文章。
* `<section>`：对页面内容进行分块；
* `<nav>`：定义导航条。HTML 5 推荐将导航链接放在`<nav>`元素中进行管理。
* `<aside>`：定义侧边栏；
* `<header>`：`<article>`文章的头部，该元素内部可以包含多个`<h1>`~`<h6>`这样的标题元素，也可以包含`<hgroup>`元素；
* `<hgroup>`：用于组织多个`<h1>`~`<h6>`这样的标题元素；
* `<footer>`：`<article>`的脚注部分；
* `<figure>`：一个独立的图片区域，内部可包含多个`<img>`元素所代表的的图片。

###### 语义元素
* `<mark>`：荧光笔效果，浏览器默认会用黄色显示；
* `<time>`：表示时间；

###### 两个特殊功能的元素
* `<meter>`：用于表示一个已知最大值和最小值的计数仪表，比如电池的剩余电量；
* `<progress>`：进度条；

```html
当前行车速度是：
<meter value="120" min="0" max="220" low="0" high="160">120</meter>千米/小时。<br>
任务完成比：
<progress value="30" max="100">30/100</progress>
```
![`<meter> & <progress>`](http://upload-images.jianshu.io/upload_images/658453-031cb33e10ecf2fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### HTML 5 的头部和元信息

# <head>
* `<script>`：用于包含 JavaScript 脚本；
* `<style>`：用于**定义** CSS 样式；
* `<link>`：用于链接外部 CSS 样式等资源；
* `<title>`：显示在浏览器标签栏的标题；
* `<base>`：用于指定页面中所有链接的基准链接；
  * `<href>`：指定所有链接的基准链接；
  * `<target>`：指定页面中所有链接的默认打开方式：`<_blank>`, `<_self>`, `<_parent>`, `<_top>`；
* `<meta>`：用于定义 HTML 页面的元数据，也就是一些 name-value 对，它可以指定以下 3 个属性：
  * `<http-equiv>`：指定元信息的名称，**该属性指定的名称具有特殊意义，它可以向浏览器传回一些有用的信息，帮助浏览器正确的处理网页内容。**
  * `<name>`：指定元信息的名称，该名称值可以随意指定；
  * `<content>`：元信息的值；
> `<meta>`元素里`<http-equiv>`和`<name>`的作用基本相同，只是`<http-equiv>`属性值通常规定为应该是浏览器可以识别的、具有特殊意义的名称。

`<http-equiv>`属性支持以下几个值：

* `<Expires>`：指定网页的过期时间，一旦网页过期，必须重新从服务器上下载。例如：
  * `<meta http-equiv="Expires" content="Sat Sep 27 16:12:36 CST 2008"/>`
* `<Pragma>`：`<meta http-equiv="Pragma" content="no-cache">`禁止缓存页面，等价于 HTTP 头中的`Cache-Control: no-cache`
* `<Refresh>`：***指定浏览器多长时间后自动刷新页面：***
  * `<meta http-equiv="Refresh" content="2"/>`两秒后自动刷新；
  * `<meta http-equiv="Refresh" content="2; URL=http://www.baidu.com"/>`两秒后自动刷新至百度；
* `<Set-Cookie>`：设置 Cookie：
  * `<meta http-equiv="Set-Cookie" content="name=value expires= Sat Sep 27 16:12:36 CST 2008, path=/">`
* `<Content-Type>`：设置页面内容类型和字符集：
  * `<meta http-equiv="Content-Type" content="text/html; charset=utf-8">`

##### HTML 5 新增的拖放 API
HTML 5 新增了关于拖放的 API ，通过拖放 API 可以让 HTML 页面的任意元素都变成可拖动的（比如图片和链接就默认是可拖动的）。
结合 JavaScript 就可以设置拖放时**携带的数据。**

##### HTML 5 表单
一个问题，不要被 HTML 5 的“强大”迷惑，上传文件这种事情还是要靠 JavaScript 来完成；只不过 HTML 5 提供了更方便的接口。

`<form>`三个重要的属性：

* `<action>`：目标地址；
* `<method>`：`post` or `get`；
* `<enctype>`：**对表单内容进行编码所使用的字符集**
  * `application/x-www-form-urlencoded`：默认的编码方式；
  * `multipart/form-data`：含有文件时；
  * `text/plain`：适用于发送邮件；

HTML 5 对**客户端校验**进行了增强（以前完全要靠 JavaScript 来实现），甚至支持基于**正则表达式**的校验。

##### HTML 5 绘图
> 在 HTML 5 以前的时代，如果需要在页面上动态的生成图片，要么在服务器端生成位图后输出到 HTML 页面上显示；要么要么使用 Flash 等第三方工具。HTML 5 的出现改变了这种局面， HTML 5 新增了一个 `<canvas>`元素，结合 JavaScript 的绘图 API ，能够实现强大的绘图功能。

##### HTML 5 的多媒体支持
> 在 HTML 5 的规范以前，如果希望在网页上播放视频、音频，通常需要借助第三方插件，比如 Flash；HTML 5 新增了`<audio>`、`<video>`两个元素，**无需使用任何插件**，即可播放视频、音频。

##### 其他
HTML 5 内联支持 SVG —— Scalable Vector Graphics——可伸缩矢量图形

	<svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="190> 
		<polygon points="100,10 40,180 190,60 10,60 160,180"
		style="fill:lime;stroke:purple;stroke-width:5;fill-rule:evenodd;">
	</svg>

![SVG - Scalable Vector Graphics](http://upload-images.jianshu.io/upload_images/658453-cf5ebceaf2feb4a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)