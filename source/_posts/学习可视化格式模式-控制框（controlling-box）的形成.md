title: "学习可视化格式模式-控制框（controlling box）的形成"
date: 2014-04-30 20:19:10
tags: CSS
---
接下来，介绍一下CSS2.1中可能产生的框的类型。在某种程度上，框的类型影响了它在可视化格式模型中的表现。[`diaplay`](http://www.w3.org/TR/CSS2/visuren.html#propdef-display)属性用来指定框的类型。
<!-- more -->
## 块级元素（block-level elements）和块框（block boxes）
块级元素是源文档中那些在视觉上被格式化为块（如：段落）的元素。以下`display`的取值会让一个元素成为块级元素：`block`、`list-item`、`table`。

块级框是参与块格式化上下文（[block formatting context]((http://www.w3.org/TR/CSS2/visuren.html#block-formatting))）的框。每一个块级元素生成一个主要的块级框(principal block-level box)来包含其子框和生成的内容，同时任何定位方案都会与这个主要的框有关。某些块级元素还会在主要框之外产生额外的框：例如`list-item`元素。这些额外的框会相对于主要框来放置。（个人感觉那些额外的框是用来放置标示的，比如LI元素前面的点）

除了tab框（后面会讲到）和可置换元素（replaced elements），一个块级框（block-level box）同时也是一个块容器框（block container box），一个块容器框要么只包含块级框，要么创建一个行格式化上下文（inline formatting context ）而只包含行内级框（inline-level boxes）。并非所有的块容器框都是块级框：不可置换行内块（non-replaced inline blocks ）和不可置换table cell是块容器但不是块级框。是块级框的块容器称作块框（block boxes）。

三个术语：块级框（block-level box）、块容器框（block container box）和块框（block box）在意义明确的时候简称为块（block）

这三个概念感觉和乱麻一样，撸个图更直观：

![](/images/VFM/block-box.png)

### 匿名块框（Anonymous block boxes）
有这样一个文档：

```
<div>
    some text
    <p>More text</p>
</div>
```

（并假定div和p都设置了`display:block`），div似乎同时包含了行内类型的内容和块类型的内容。为了使格式化简单一些，我们假定有一个**匿名块框**围绕在"Some text"周围。

![](/images/VFM/anon-block.png)

换句话说：如果一个块容器框（如上列中为div生成的框）有一个块级框（如上例中的p），那么我们强制它只包含块级框在其中。

当一个行内框（inline box）包含一个属于正常排版的块级框，这个行内框（及与其位于同一行框（line box）的行内祖先（inline ancestors））会被从周围的块级框（及其任何连续（中间被插入可折叠的空白或在正常排版以外的元素也算是连续的）的块级兄弟）中分离出来，把行内框分离成两个框（即使其中一边是空的），它在块级框两边。在截断之前和截断之后的行框都附入匿名块框，并且该块级框与匿名块框成为兄弟。当这样的行内框受到相对定位的影响，任何产生的移动同样影响到包含在行内框内的块级框。

假设有以下的例子：

```html
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Anonymous text interrupted by a block</title>
    <style>
        p {
            margin: 0;
        }
        span {
            border: 1px solid red;
        }
    </style>
</head>
<body>
    <div>
       <span>This is anonymous text before the P.
           <P>This is the content of P.</P>
             This is anonymous text after the P.
       </span>
    </div>
</body>
</html>
```

这个例子中p元素包含一段匿名文本（C1），随后是一个款及元素，再随后是另一端匿名文本（C2）。对此生成了一个代表body的块框，它包含了一个围绕C!的匿名块框、代表span的块框和围绕C2的另一匿名块框。（这段代码中P元素的盒模型在各个浏览器中差距很大）。

匿名框的继承属性从包含它的非匿名框那里继承。非继承的属性将取其初始值。比如匿名框的字体属性继承自div，但是margin将会是0。

当一个元素上设置的属性导致了匿名块框的生成，则该属性依然能应用于该元素生成的框和其内容之上。例如，上面的例子中，如果在p元素上设置了边框，则这个边框画在C1（在最后一行下方开放）和C2（在第一行上方开发）周围。

一些用户代理用其他方式实现了行内包含块上的边框。例如，将其内嵌的块放入“匿名行框”中，并在这些匿名行框周围绘出行内边框。由于CSS1和CSS2没有定义这些效果，仅支持CSS1和CSS2的用户代理会以其他形式来实现，并可仍旧声称与CSS2.1的特征保持部分一致。对于CSS2.1规范发布之后才发布的用户代理，就不能这么做了。

当处理百分比值时，应该忽略匿名块框，而以最近的非匿名祖先框来替代。例如，如果一个匿名块框的孩子在div里面需要知道包含块的高度来解析一个百分比高度。那么它将使用div形成的包含块的高度，而不是匿名块框的高度。


## 行内级元素（inline-level element）和行内框（inline box）

行内级元素（inline-level element）是源文档中那些不为其内容形成新的块，而让其内容分布在多行中的元素（如，段落内着重的文本，行内图片等等）。以下的`display`属性值产生一个行内级元素：`inline`、`inline-table`、`inline-block`。行内级元素生成行内级框（inline-level box），而这些框会参与某个行格式化上下文（inline formatting context）

一个行内框（inline box）是行内级框（inline-level box），且其内容参与了包含它的行格式化上下文（inline formatting context）。一个`display`值是`inline`的非置换元素会生成一个行内框。那些不是行内框（inline box）的行内级框（inline-level box），例如，可置换的行内级元素（replaced inline-level element），行内块元素（inline-block element），行内表格元素（inline-table）被称为原子行内级框（atomic inline-level box），因为它们是以单一不透明框的形式来参与格式化上下文。

换句话说，inline-level box分成inline box和atomic inline-level box两种类型，inline box由`display`值时`inline`的no-replaced元素产生，atomic inline-level box由replaced inline-level元素、inline-block元素和inline-table元素生成。（这个地方弄的我快疯掉了。。。）

### 匿名行内框（Anonymous inline box）

任何直接被包含在一个块容器（block container）元素（而不是行内元素）中的文本必须视为匿名行内元素。

在如下的文档中

```
<p>Some <em>emphasized</em> text</p>
```

P元素生成一个块框，其中还有几个行内框。"emphasized"的框是由行内元素（em）产生的一个行内框。而其他的框（"Some"和"text"）是块级元素（p）产生的行内框。后者就称为匿名行内框，因为它们没有与之相关的行内级元素。

这样的行内框从其父块框那里继承可以继承的属性。非继承属性取其初始值。例子中，匿名行内框的颜色继承自p，而背景是透明的。

空格内容，如果可被折叠（根据`white-space`属性），则其不会生成任何匿名行内框。

在本规范中，如果一个匿名框的类型可根据上下文来清晰界定，则匿名行内框和匿名块框都可被简称为匿名框。

自格式化[表格](http://www.w3.org/html/ig/zh/wiki/CSS2/tables#anonymous-boxes)时，还会出现更多类型的匿名框。

### 插入框（Run-in box）

`display: run-in`是CSS3（Experimental values）的定义，因为我也没有过这个属性，后续用到在学习（可以参考[CSS基本框模型](http://www.w3.org/TR/css3-box)）。

### 'display'属性

**'display'**

值： block | list-item | inline-block | table | inline-table | table-row-group | table-header-group | table-footer-group | table-row | table-column-group | table-column | table-cell | table-caption | none | inherit
初始值： inline
适用于： 所有元素
可否继承： 否
百分比： N/A
媒介： 所有

这里注意的是其实所有元素默认的`display`都为`inline`，用户端（对我们来说是浏览器）的缺省样式表可以覆盖它。比如，P 和 DIV 的`display`特性值是`block`，TABLE的`display`特性值是`table`。

本属性各取值的意义如下：

**block**：这个值让元素产生一个块框。
**inline-block**：这个值让元素产生一个行内级块容器（inline-level block container）。行内块（inline-block）的内部会被当做块框来格式化，而此元素本身会被当作原子行内级框来格式化。
**inline**：这个值让元素产生一个或多个的行内框。
**list-item**：这个值让元素（例：HTML里的LI）产生一个主块框与标记框。详情请看[列表](http://www.w3.org/html/ig/zh/wiki/index.php?title=CSS2/generate&action=edit&redlink=1)章节以获得列表相关的资讯与列表排版的范例
**none**：这个值让给元素不出现在[排版结构](http://www.w3.org/TR/CSS2/intro.html#formatting-structure)中（也就是在视觉媒体中，元素既不产生框也不影响布局）。其子孙元素也不产生任何框；该元素及其内容会被从排版结构中完全移除。对子孙元素设定`display`属性不能覆盖这个结果。
请注意`display`属性取值为`none`并不会产生隐形的框：`none`完全不产生任何框。CSS具有可以让元素在排版结构中出现并影响排版，但是框本身却不见的属性。详情请看[可见性](http://www.w3.org/html/ig/zh/wiki/CSS2/visufx#visibility)的小节

table、inline-table、table-row-group、table-column、table-column-group、table-header-group、table-footer-group、table-row、table-cell 与 table-caption：这些值让元素产生表现的像表格元素的行为（详情见[表格](http://www.w3.org/TR/CSS2/tables.html)章节的描述）

除了定位元素、浮动元素（请参考['display'、'position' 与 'float' 的关系](http://www.w3.org/html/ig/zh/wiki/CSS2/visuren#dis-pos-flo)）与根元素，本属性的计算值与指定值相同。根元素上的本属性计算值的变动会依照['display'、'position' 与 'float' 的关系](http://www.w3.org/html/ig/zh/wiki/CSS2/visuren#dis-pos-flo)中描述的规则。
