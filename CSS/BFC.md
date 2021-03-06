# BFC

## 简介

**文档流**是由 CSS 定位语句定义的页面元素的排列以及 HTML 元素的顺序。

文档流分为：**常规流**、**浮动流**、**定位流**。

### 三种文档流的排版方式

**常规流：**最基础的排布方式，它是元素全部按照原来自身设定的排布方式排布。

- 在常规流中，盒一个接着一个排列。
- 块级格式化上下文里面，他们竖着排列。
- 行内格式化上下文里面，他们横着排列。
- 当position为static或relative，且float为none时会触发常规流。
- 对于`position:static`，盒的位置是常规流布局里的位置。
- 对于`position:relative`，盒的位置是由top、bottom、left、right属性定义。即使有便宜，仍然保持原有的位置，其他常规流不能占用这个位置。

**浮动流**：是指给元素CSS加属性`float: left/right`后元素的排布方式。

- 左浮动元素尽量靠左靠上，右浮动元素尽量靠右靠上。
- 这会导致普通流环绕在它周边，除非设置`clear`。
- 浮动元素不会直接影响块级元素的布局；
  但浮动元素会直接影响行内元素的布局，让其围绕在自己周围，撑大父级元素，从而间接影响块级元素的布局。
- 浮动最高点不会超过当前行、它前面的浮动元素的最高点。
- 不超过它的包含块，除非元素本身已经比包含块更宽。
- 行内元素出现在左浮动元素的右边和右浮动的左边，左浮动元素的左边和右浮动元素的右边是不会摆放浮动元素的。

**定位流：**是指元素CSS属性`position: absolute/fixed`后元素的排布方式。

- 被设置定位的盒从常规流中移除，不影响常规流的布局。
- 它的定位相对于它的包含块，由top、bottom、left、right属性定义。
- 对于`position:absolute`，元素定位将相对于上级元素中最近的一个`position:relative/absolute/fixed`的元素，如果没有则相对于body定位。





普通流 FC 直译过来是格式化上下文，它是页面中的一块渲染区域，有一套渲染规则，决定了其子元素如何布局，以及和其他元素之间的关系和作用。

常见的 FC 有：BFC、IFC、GFC 和 FFC。









## BFC 定义

> **BFC**（Block Formatting Context）直译为“块级格式化上下文”，它是一个独立的渲染区域，只有块级盒子参与，它规定了内部的块级盒子如何布局，并与这个区域外部毫不相干。

**Box：CSS 布局的基本单位**

Box 是 CSS 布局的对象和基本单位。元素的类型和 `display` 属性，决定了这个 Box 的类型。不同类型的 Box，会参与不同的格式化上下文（一个决定如何渲染文档的容器），因此 Box 内的元素会以不容的方式渲染。让我们看看有哪些盒子：

- `块级盒子`：display 属性为 block、list-item、table的元素，会生成 block-level box。并参与 Block formating context；
- `内联盒子`：display 属性为 inline、inline-block、inline-table 的元素，会生成 inline-box。并且会参与 inline formatting context；
- `Run-in box`：新出的，暂且不议。



**格式化上下文（FC）**

FC 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。最常见的FC有BFC、IFC（Inline Formatting Context）。

> BFC是一个独立的布局环境，其中的元素布局是不受外界影响，并在一个 BFC 中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直地沿着父元素的边框排列。

## BFC 的布局规则

- 内部的 Box 会在垂直方向，一个接一个地放置。
- Box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠。
-  每个盒子（块盒与行盒）的 margin box 的左边，与包含块 border box 的左边相接触（对于从左往右的格式化，否则相反）。即使存在浮动也是如此。
- BFC 的区域不会与 float box 重叠。
- BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之也如此。
- 计算 BFC 的高度时，浮动元素也参与计算。

## 触发 BFC

只要元素满足以下一种条件，即可触发 BFC 特性：

- 浮动元素：float 的值不是 none。
- position 的值不是 static 或者 relative。
- display 的值是 inline-block、table-cell、flex、table-caption、inline-flex。
- overflow 的值不是 visible。

## BFC 的作用

1. 利用 BFC 避免 margin 重叠 。

2. 阻止普通文本流元素被浮动元素覆盖。
   根据：

   - 每个盒子的 margin box 的左边，与包含块 border box 的左边相接触（对于从左往右的格式化，否则相反），即使存在浮动也是如此。

   又因为：

   - BFC 的区域不会与 float box 重叠。

3. 清除浮动。
   浮动的元素会脱离普通文本流，如果触发容器的 BFC，那么容器将会包裹着浮动元素。

## 总结

BFC 就是页面上的一个隔离的独立容器，容器里的子元素不会影响到外面的元素。反之也是如此。

因为 BFC 内部的元素和外部的元素绝对不会相互影响，因此，当 BFC 外部存在浮动时，它不应该影响 BFC 内部 Box 的布局， BFC 会通过变窄 而不与浮动有重叠。同样，当 BFC 内部有浮动时，为了不影响外部元素的布局，BFC 计算高度时会包括浮动的高度，避免 margin 重叠也是这样的一个道理。

