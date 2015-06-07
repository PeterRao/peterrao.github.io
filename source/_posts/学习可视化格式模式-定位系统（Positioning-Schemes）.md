title: "学习可视化格式模式-定位系统（Positioning Schemes）"
date: 2014-04-30 23:28:24
tags: CSS
---
之前提到过，在可视化格式模型中，每一个元素都会根据盒子模型产生0个或多个box，而这些box的布局会受四大因素的影响（box尺寸和类型、定位系统、文档树中元素之间的关系、外部信息），下面就学习下定位系统。
<!-- more -->
在CSS2.1中，会根据三种定位系统来摆放框：

1. 常规流（[Normal flow](http://www.w3.org/TR/CSS2/visuren.html#normal-flow)）。在CSS2.1中，常规流包括了块级框（block-level box）的块格式化（[block formating](http://www.w3.org/TR/CSS2/visuren.html#block-formatting)）、行内级框（inline-level box）的行内格式化（[inline formatting](http://www.w3.org/TR/CSS2/visuren.html#inline-formatting)）以及块级框和行内级框的相对定位（[relative positioning](http://www.w3.org/TR/CSS2/visuren.html#relative-positioning)）
2. 浮动（[Floats](http://www.w3.org/TR/CSS2/visuren.html#floats)）。在浮动模型中，一个框首先根据常规流摆放，然后，再将它从（常规）流中取出并尽可能地向左或向右偏移。其他内容可以排在一个浮动的周围。
3. 绝对定位（[Absolute positioning](http://www.w3.org/TR/CSS2/visuren.html#absolute-positioning)）。在绝对定位模型中，一个框会从常规流中完全脱离出来（它对后续的兄弟没有影响），并根据它的包含块来分配其位置。

如果一个元素是浮动、绝对定位或根元素，那么它被称为脱离了常规流（out of flow）的元素。没有脱离流的元素被称为常规流内的元素（in-flow）。元素A的流包括元素A以及这么一组元素：它们是流内的元素，且它们在流之外最近的祖先是元素A。

> 注意：CSS2.1中，定位系统可以实现网页作者期望的布局效果，而无需使用特殊的标记手段（例如：不可见的图片），这让文档具有更好的可读性。

### 选择一个定位方案：'position'属性

'position'属性和'float'属性决定了要用CSS2.1的哪种定位算法来计算一个框的位置。

'position'
　　取值：relative | absolute | fixed | inherit
　　初始: static
　　适用于： 所有元素
　　百分比：（不适用）
　　媒介：[视觉](http://www.w3.org/TR/CSS2/media.html#visual-media-group)
　　计算值：同指定值
本属性各取值的意义如下：
static
　　元素生成的框是按常规流方式来摆放的普通框。其'top'、'right'、'bottom'和'left'属性不起作用。
relative
　　元素生成的框先按常规流方式来计算其位置（这称作在常规流下的位置），然后框相对于其正常位置进行偏移。若框B使用了相对定位，则会以假设框B还没有发生偏移之前的状态来计算算其后面的框的位置。本规范未定义'position: relative'在table-row-group、table-header-group、table-footer-group、table-column-group、table-cell和table-caption元素上的效果。
absolute
　　框的位置（可能包括大小）由'top'、'right'、'bottom'与'left'属性来指定。这些属性指定了框相对于其包含块（[containing block](http://www.w3.org/TR/CSS2/visuren.html#containing-block)）的偏移量。绝对定位框会被排除在常规流之外，这意味着它们对之后的兄弟框的布局没有影响。另外，虽然绝对定位框有外边距，这些外边距不与其他外边距[折叠](http://www.w3.org/html/ig/zh/wiki/CSS2/box#collapsing-margins)
fixed
　　框的位置按照'absolute'模型来计算，但是框相对于某个参考位置固定。如同'absolute'模型，框的外边距不与别的外边距折叠。若媒介型态是手持、投影、荧幕、打字机或是电视，则框相对于视窗（[viewport]）固定，而并不随着滚动而移动。若媒介类型是打印机，则框要在每一页被渲染，并相对于页面框固定渲染，就算页面透过视窗（[viewport]）阅读（举例来说，预览打印的时候）也是这样。本规范未定义这个值在其他媒介类型上的呈现方式。网页作者可以依媒介指定'fixed'值，举例来说，网页作者可能希望一个框维持在荧幕视窗（[viewport]）的上方，但又不出现在印出来的每个页面的上方。这两种定位方案的设定可用[@media](http://www.w3.org/html/ig/zh/wiki/CSS2/media#at-media-rule)规则分开，如同：

```
@media screen {
    h1#first { position: fixed }
}
@media print {
    h1#first { position: static }
}
```

　　　用户代理绝对不能将fixed的内容分页显示。请注意用户代理可用别的方式来打印隐形的内容，详情请参考13章[页面框外面的内容](http://www.w3.org/html/ig/zh/wiki/index.php?title=CSS2/page&action=edit&redlink=1)

用户代理可将根元素上的'position'视为'static'。

### 框偏移 - 'top'、'right'、'bottom'、'left'

'position'属性值不为'static'的元素称为定位元素。定位元素生成定位框，并根据以下四个属性来确定其摆放的位置：

'top'、'right'、'bottom'、'left'
　　取值： <length> | <percentage> | auto | inherit
　　初始： auto
　　适用于： 定位元素
　　继承： 否
　　百分比： 相对于包含块的高度
　　媒介： 视觉
　　计算值：若指定值为长度，计算值为对应的绝对长度。若指定值为百分比值，计算值同指定值。否则，值为'auto'。
该属性可以用来指定绝对定位（[absolutely positioned]）框的外边距边（margin边界）和其包含块（[containing block]）的边界偏移的距离。若用在相对定位框上，偏移量相对于自己的外边距边。
这四个属性的取值都可以为负值。
当取值为'auto'时，对于非置换（non-replaced）元素，这个值的效果根还有哪些相关属性的值也是'auto'x相关，详情请参考说明绝对定位非置换元素[宽度](http://www.w3.org/html/ig/zh/wiki/CSS2/visudet#abs-non-replaced-width)和[高度](http://www.w3.org/html/ig/zh/wiki/CSS2/visudet#abs-non-replaced-height)小节。对于替换元素，这个值的效果只与置换内容的固定尺寸相关，详情请参考说明就绝对定位置换元素[宽度](http://www.w3.org/html/ig/zh/wiki/CSS2/visudet#abs-replaced-width)和[高度](http://www.w3.org/html/ig/zh/wiki/CSS2/visudet#abs-replaced-height)的小节。

[viewport]: http://www.w3.org/TR/CSS2/visuren.html#viewport
[absolutely positioned]: http://www.w3.org/TR/CSS2/visuren.html#absolutely-positioned
[containing block]: http://www.w3.org/TR/CSS2/visuren.html#containing-block
