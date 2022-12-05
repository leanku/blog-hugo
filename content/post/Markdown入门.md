---
title: "Markdown入门"
date: 2022-11-19
draft: false
categories: ["Markdown"]
tags: ["Markdown"]
keywords: ["Markdown"]
---

# Markdown

1.Markdown 简介

2.常用语法入门


[标题](#table1) 
[段落](#table2) 
[分割线](#table3) 
[列表](#table4) 
[引用](#table5) 
[强调](#table6) 
[辅助线](#table7) 
[字体字号](#table8) 
[背景色](#table9) 
[行内代码](#table10) 
[代码块](#table11) 
[超链接](#table12) 
[图片](#table13) 

<br>



## Markdown 是什么？

Markdown 是一种轻量级标记语言，创始人是约翰・格鲁伯（John Gruber）。它允许人们 “使用易读易写的纯文本格式编写文档，然后转换成有效的 HTML 文档”。

—— 维基百科。

## 常用语法总结
<br>

### <a id="table1">标题</a> 
1.在标题文字下方添加标记，连续的三个 “等号”（===） 代表一级标题，连续的三个 “减号”（—）代表二级标题。

2.在标题文字行首增加连续的 “井号” (#) 及空格。1 个 “井号” 代表一级标题，2 个连续 “井号” 代表二级标题，以此类推，最多支持到 6 级标题。  
<br>

### <a id="table2">段落</a> 
不同于分段换行，不分段换行不是用 p 标签描述段落，而是用 br 标签折断文字。
如果要让文字另起一行而不分段，需在行尾增加两个空格。
<br>

### <a id="table3">分割线</a> 
连续的三个「星号 *」，或者连续的三个「减号 -」，或者连续的三个「下划线 _」会被渲染成分割线。
<br>

###  <a id="table4">列表</a> 
支持有序和无序两种列表，无序列表使用 「星号 "*"」、「加号 "+"」、「减号 "-"」表示，有序列表使用数字定义，如: 1. xxx 2.xxx 3.xxx 等。
<br>

### <a id="table5">引用</a> 
使用邮件风格「大括号 >」的引用声明。如果你已了解如何在邮件中引用文章，那代表你也掌握了在 Markdown 文本中引用文字的方法了。其实现方式就是在被引用的文字行开头添加「大于号 >」。
如果需要在块引用内的换行，可以在行尾增加两个连续的空格。
<br>

### <a id="table6">强调</a> 
用 「星号 *」或者 「下划线 _」 包裹住，一个符号的时候代表斜体，如 *斜体*，两个符号的时候代表粗体，如 \*\*粗体\*\*。

### <a id="table7">辅助线</a> 
辅助线包含「中划线（删除线）」一种形式，其他形式的辅助线要通过 HTML 标签进行扩展。中划线使用 「波浪线 ~」来描述。

### <a id="table8">字体字号</a> 
原生语法并不支持修改字体、字号。为了实现丰富文字样式的需求，我们需要通过增加 HTML 标签实现此类效果。
使用 \<font> 标签的 face 属性修改文字字体
>字体中文名称	字体写法

>>黑体	\<font face='SimHei'>

>>宋体	\<font face='SimSun'>

>>新宋体	\<font face='NSimSun'>

>>仿宋	\<font face='FangSong'>

>>楷体	\<font face='KaiTi'>

>>仿宋_GB2312	\<font face='FangSong_GB2312'>

>>楷体_GB2312	\<font face='KaiTi_GB2312'>

>>微软雅黑	\<font face='Microsoft YaHei'>

字号定义有三种主要方式

第一种是使用 \<font> 标签 size 属性修改局部字号；如 \<font size="1">1号字</font>

第二种通过\ <big> 或者 \<small> 标签；如 \<big>放大的文字\</big> \<small>缩小的文字\</small>

第三种是通过修改 \style 样式实现全局字体字号的设置。如

 ```css
 <style>
    h1 { font: 26pt song !important; }
    h2 { font: 22pt song !important; }
    h3 { font: 16pt song !important; }
    p { font: 14pt kai !important; }
</style>
```
<br>

### <a id="table9">背景色</a> 

使用 <font> 标签的 color 属性修改文字颜色。

    <font style="color: red">红色</font>
    <font style="color: green">绿色</font>
    <font style="color: blue">蓝色</font>
    <font style="color: rgb(200,100,100)">使用 rgb 颜色值</font>
    <font style="color: #FF00BB">使用十六进制颜色值</font>

定义文字背景色需要通过修改 style 样式实现。

使用 `style` 属性修改文字的背景色

    <font style="background: red">红色</font>
    <font style="background: green">绿色</font>
    <font style="background: blue">蓝色</font>
    <font style="background: rgb(200,100,100)">使用 rgb 颜色值</font>
    <font style="background: #FF00BB">使用十六进制颜色值</font>

### <a id="table10">行内代码</a> 
行内代码用一对 「反引号 `」符号将需要转换的文字内容包括起来，它让我们方便地在行内编辑带有特殊字符的文字内容

    行内的 html 代码：`<head><title>网页标题</title></head>`
    行内的 json 代码：`var json = {key: value};`

### <a id="table11">代码块</a> 
相比普通的文本段落，代码块可以保留文字内容的多行换行、缩进等格式。

在 Markdown 文档中生成代码块，需要在每行的开头输入不少于 4 个空格符号或者 1 个 tab 符号。
在行首添加 4 个连续的空格，可将行内容定义为代码块
代码块的另一种定义方式是以三个连续的 「反引号 “`”」 作为开始行和结束行。
使用反引号定义代码块，并定义高亮
```php 
echo 'hello';
```
#### 一张字符画

​```
.__           .__  .__                               .__       .___
|  |__   ____ |  | |  |   ____   __  _  _____________|  |    __| _/
|  |  \_/ __ \|  | |  |  /  _ \  \ \/ \/ /  _ \_  __ \  |   / __ |
|   Y  \  ___/|  |_|  |_(  <_> )  \     (  <_> )  | \/  |__/ /_/ |
|___|  /\___  >____/____/\____/    \/\_/ \____/|__|  |____/\____ |
     \/     \/                                                  \/
​```
<br>

### <a id="table12">超链接</a> 
由 「中括号 []」来声明。

如果需要创建行内链接的创建方式，用一对紧跟「中括号 []」的「小括号 ()」描述目标链接，小括号内不仅可以包含链接的地址，也可以用「引号 ""」设定链接的标题。
#### 声明超链接的细节

点击下面的连接将跳转到\[慕课网](https://www.imooc.com/ '慕课')首页

### <a id="table13">图片</a> 
使用 \!\[替换文字](图片路径 "标题(可选)") 的形式定义图片
#### 插入一张图片

    图片前的文字。

    ![](https://tva3.sinaimg.cn/crop.0.0.180.180.180/6d04a77djw1e8qgp5bmzyj2050050aa8.jpg?KID=imgbed,tva&Expires=1588529780&ssig=vNCcwnltm2)

    图片后的文字

#### 拼图九宫格
```
![][img6]
![][img5]
![][img4]

![][img3]
![][img2]
![][img1]

![][img9]
![][img8]
![][img7]

[img1]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101433_eTTNZ.thumb.300_300_c.jpeg
[img2]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101434_iWadw.thumb.300_300_c.jpeg
[img3]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101434_Z3JVy.thumb.300_300_c.jpeg
[img4]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101435_NiLkv.thumb.300_300_c.jpeg
[img5]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101437_CxzYm.thumb.300_300_c.jpeg
[img6]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101437_wdizF.thumb.300_300_c.jpeg
[img7]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101438_J8vff.thumb.300_300_c.jpeg
[img8]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101439_cVcLx.thumb.300_300_c.jpeg
[img9]: https://c-ssl.duitang.com/uploads/item/202004/10/20200410101439_yhUv3.thumb.300_300_c.jpeg

<style>
img {
	width: 150px !important;
	height: 150px !important;
	border: 1px solid #EEE;
}
</style>

```




