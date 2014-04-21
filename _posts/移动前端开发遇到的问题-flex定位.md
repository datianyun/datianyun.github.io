---
layout: blog
title: flex定位与table-column table-cell
summary: 移动前端开发碰到的问题（1）
---

# {{ page.title }}

2014-04-18 北京 

## 简介

目前工作中碰到的问题最麻烦的就是几款手机的兼容问题，尤其是小米和note的自带或者android浏览器，最佳的兼容方案往往是最标准的CSS3标准，自己需要把这些知识点都切实搞清楚，而不是遇到问题再去查才能收获更多。

#FLEX弹性模式定位

W3Chool上的介绍比较简单，box-flex 属性规定框的子元素是否可伸缩其尺寸。可伸缩元素能够随着框的缩小或扩大而缩写或放大。只要框中有多余的空间，可伸缩元素就会扩展来填充这些空间。

`````

默认值：	0.0（指示该元素不可伸缩）
继承性：	no
版本：	CSS3
JavaScript 语法：	object.style.boxFlex=2.0


`````
#介绍--http://www.w3.org/TR/css3-flexbox/

在css2.1中定义了四种布局方式来决定和模型的大小和位置：

`````
.块级布局, designed for laying out documents
.行内布局, designed for laying out text
.table布局, designed for laying out 2D data in a tabular format
.position定位, designed for very explicit positioning without much regard for other elements in the document

`````
Flex布局跟块级布局很相似，在Flex中缺少复杂元素在BLock中可以应用的float、column，但是他却定义了很多有用的属性可以应用在webapp或者复杂的网页中：

`````
can be laid out in any flow direction (leftwards, rightwards, downwards, or even upwards!)can have their display order reversed or rearranged at the style layer (i.e., visual order can be independent of source and speech order)
can be laid out linearly along a single (main) axis or wrapped into multiple lines along a secondary (cross) axis
can “flex” their sizes to respond to the available space
can be aligned with respect to their container or each other
can be dynamically collapsed or uncollapsed along the main axis while preserving the container’s cross size
`````