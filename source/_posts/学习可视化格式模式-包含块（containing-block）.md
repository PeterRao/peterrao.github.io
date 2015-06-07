title: "学习可视化格式模式-包含块（containing block）"
date: 2014-04-27 23:12:09
tags: CSS
---
[上节](/2014/04/25/VFM-intro.html)简单介绍了包含块的相关概念，这节详细讲解包含块。

## 包含块判定及其范围 ##

之前说过一个元素box的定位和尺寸计算，有时候和一个特定的矩形有关，这个矩形称作该元素的*containing block*。而元素会为它的子元素创建包含块，那么是不是说，元素的包含块就是它的父元素呢？答案是否定的！

一个元素包含块的确定，跟元素自身和它的祖先元素的样式等有关系。
包含块判定总流程图如下：
![包含块判定总流程图](/images/VFM/judge-containing-block.png)
<!-- more -->
### 根元素的包含块 ###

[根元素](http://www.w3.org/TR/CSS2/conform.html#root)，就是处于文档树最顶端的元素，它没有父节点。

根元素存在的矩形包含快叫做初始包含块(initial containing block)。对于连续媒介，它的尺寸就是视窗([viewport](http://www.w3.org/TR/CSS2/visuren.html#viewport))，固定在画布的原点上，也就是说在X(HTML)中，根元素是html元素（尽管有的浏览器会不正确地使用body元素）；对于分页媒介，它就是[页面区域](http://www.w3.org/TR/CSS2/page.html#page-area)。初始包含块的'direction'属性与根元素相同。

### 'relative'和'static'定位的元素###

对于其它元素：如果该元素的定位（position）为'relative'（相对定位）或者'static'（静态定位），它的包含块由最近的[块容器](http://www.w3.org/TR/CSS2/visuren.html#block-boxes)祖先元素的内容框边缘产生。

这里有两个概念：块容器，后面介绍块框会讲。内容框，可以去看CSS的盒模型。

元素如果未声明'position'属性，那么就会采用'position'的默认值'static'。所以，一般元素都是静态定位的。

比如：

```
<table id="table1">
    <tr>
        <td  id="td1">
            <div id="div1" style="padding:20px; border:1px solid red;">
                <span>
                    <strong id="greed" style="postition:relative;">greed is</strong>
                </span>
            </div>
        </td>
    </tr>
</table>
```

包含块关系表：

元素  | 包含块
----- | -----
table | body
td1   | table1
div1  | td1
greed | div1

span元素中包含的的文本在div1中的位置可以看出，div1创建的包含块的区域是它的内容边界，也就是内边界。

### 'position:fixed'定位的元素

如果元素是固定定位（'position:fixed'）元素，那么它的包含块是当前可视窗口（viewport）。

### 'position:absolute'定位的元素。


如果元素是绝对定位（'position:absolute'）元素，那么它的包含块由它最近的position属性为'absolute'、'relative'或者'fixed'的祖先元素创建。

1.如果其祖先元素是行内元素，则包含块取决于其祖先元素的'direction'特性。（我也是刚知道'direction'这个属性）。

- 如果'direction'是'ltr',包含块的顶、左边是祖先元素生成的第一个框的顶、左内边距边界（padding edges），右、下边是祖先元素生成的最后一个框的右、下内边距边界（padding edges）。

示例代码：

```
<p style="border:1px solid red; width:200px; padding:20px;">
    T
    <span style="background-color:#C0C0C0; position:relative;">
            这段文字从左向右排列，红 XX 和 蓝 XX 和黄 XX 都是绝对定位元素，它的包含块是相对定位的SPAN。 可以通过它们绝对定位的位置来判断它们包含块的边缘。
            <em style="position:absolute; color:red; top:0; left:0;">XX</em>
            <em style="position:absolute; color:yellow; top:20px; left:0;">XX</em>
            <em style="position:absolute; color:blue; bottom:0; right:0;">XX</em>
    </span>
</p>
```

以上代码中，文字采取默认从左到右的方式排列。红XX、蓝XX和黄XX都是绝对定位元素，它的包含块是相对定位的SPAN。它们定位需要参照包含块，按照标准来说，它们包含块的左顶边是SPAN形成的第一个框（即第一行的灰色部分）的顶、左内边距边，包含块的右、下边是SPAN生成的最后一个框（最后一行灰色的部分）的右、下内边距边界。

示意图：

Chrome34和IE11下的表现

![](/images/VFM/direction-ltr-chrome.png)

Firefox29的表现

![](/images/VFM/direction-ltr-firefox.png)

行内元素形成的包含块，在各个浏览器中各不相同，存在兼容性问题。上面两个例子可以看出，蓝色的\'xx\'在Chrome和FF中位置并不一样。

包含块的宽度可能是负的。（在Chrome和IE11下测试，这个包含块不是负值，应该是0）

示例代码：

```
<p style="border:1px solid red; width:200px; padding:20px;">
    TEXT TEXT TEXT
    <span style="background-color:#C0C0C0; position:relative;">
        这段文字从左向右排列，红 XX 和 蓝 XX 和黄 XX 都是绝对定位元素，它的包含块是相对定位的SPAN。 可以通过它们绝对定位的位置来判断它们包含块的边缘。
        <em style="position:absolute; color:red; top:0; left:0;">XX</em>
        <em style="position:absolute; color:yellow; top:20px; left:0;">XX</em>
        <em style="position:absolute; color:blue; bottom:0; right:0;">XX</em>
    </span>
</p>
```

示意图：

Chrome34和IE11下的表现

![](/images/VFM/direction-ltr-negative-chrome.jpg)

Firefox29的表现

![](/images/VFM/direction-ltr-negative-firefox.jpg)

- 如果'direction'是'rtl'，包含块的顶、右边是祖先元素生成的第一个框的顶、右内边距边界（padding edges），左、下边是祖先元素生成的最后一个框的左、下内边距边界（padding edges）

示例代码：

```
<p style="border:1px solid red; width:200px; padding:20px; direction:rtl;">
    T
    <span style="background-color:#C0C0C0; position:relative;">
         这段文字从右向左排列，红 XX 和 蓝 XX 和黄 XX 都是绝对定位元素，它的包含块是相对定位的SPAN。可以通过它们绝对定位的位置来判断它们……
        <em style="position:absolute; color:red; top:0; left:0;">XX</em>
        <em style="position:absolute; color:yellow; top:20px; left:0;">XX</em>
        <em style="position:absolute; color:blue; bottom:0; right:0;">XX</em>
    </span>
</p>
```

示意图：

Chrome34和IE11下的表现

![](/images/VFM/direction-rtl-chrome.jpg)

Firefox29的表现

![](/images/VFM/direction-rtl-firefox.jpg)


2.其他情况下，如果祖先元素不是行内元素，那么包含块的区域应该是祖先元素的**内边距边界**（padding edges）。

3.如果不存在这样的祖先元素，那么它的包含块就是初始包含块。

补充：关于祖先元素是行内元素的情况，看[W3C的文档](http://www.w3.org/TR/CSS2/visudet.html#containing-block-details)是这样的。

> In the case that the ancestor is an inline element, the containing block is the bounding box around the padding boxes of the first and the last inline boxes generated for that element. In CSS 2.1, if the inline element is split across multiple lines, the containing block is undefined.

也就是说，此时的包含块应该是第一个行内元素和最后一个行内元素决定的。在CSS2.1中，如果行内元素是多行的，包含块就是未定义的。各个浏览器并没区别多行和单行的概念。可以把下面的例子在浏览器中运行下试试。
这里Firefox还是只以第一行来确定包含块，显然是不符合CSS2.1标准的。

```
<p style="border:1px solid red; width:200px; padding:20px;">
    T
    <span style="background-color:#C0C0C0; position:relative;">
      <span>这段文字从左向右排列 </span><br>
      <span>XX都是绝对定位</span><br>
      <span>应该有包含块</span>
      <em style="position:absolute; color:red; top:0; left:0;">XX</em>
      <em style="position:absolute; color:yellow; top:20px; left:0;">XX</em>
      <em style="position:absolute; color:blue; bottom:0; right:0;">XX</em>
    </span>
</p>
```

## 资料
[W3Help-KB008](http://www.w3help.org/zh-cn/kb/008/)