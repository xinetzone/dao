---
title: 常用的 HTML 元素使用参考
lang: zh-CN
abbrlink: 91553c88
date: 2021-03-15 13:48:27
description:
updated:
tags: HTML
categories: 手册
---

## 1 &lt;ul> & &lt;ol>

属性|使用建议
:-:|:-:
`compact`👎|`<ul>` 元素应当使用 CSS 来更改样式去渲染为其更紧凑的样式。（CSS）可以提供与 `compact` 属性相同的效果，将  CSS 属性 [line-height](https://developer.mozilla.org/zh-CN/docs/CSS/line-height) 的值设为 80% 即可。
`type`👎|用于设置列表的着重号样式。不要使用这个属性，它已经被废弃了：使用 CSS [list-style-type](https://developer.mozilla.org/zh-CN/docs/Web/CSS/list-style-type) 属性作为代替

### 1.1 &lt;ul> 无序列表

特别作用于 `<ul>` 元素的 CSS 属性:

* [list-style](https://developer.mozilla.org/en-US/CSS/list-style "en/CSS/list-style") 属性, 作用于选择哪种序数的样式来显示,
* [CSS counters](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Lists_and_Counters/Using_CSS_counters "en/CSS_Counters"), 作用于操作复杂的嵌套列表,
* [line-height](https://developer.mozilla.org/en-US/CSS/line-height "en/CSS/line-height") 属性, 作用于模拟过时的 [`compact`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/ul#attr-compact) 属性,
* [margin](https://developer.mozilla.org/en-US/CSS/margin "en/CSS/margin") 属性, 作用于控制列表的缩进.

### 1.2 &lt;ol> 有序列表

属性|使用建议
:-:|:-:
`reversed`|此布尔值属性指定列表中的条目是否是倒序排列的，即编号是否应从高到低反向标注。
`start`|一个整数值属性，指定了列表编号的起始值。此属性的值应为阿拉伯数字，尽管列表条目的编号类型 `type` 属性可能指定为了罗马数字编号等其他类型的编号。比如说，想要让元素的编号从英文字母 "d" 或者罗马数字 "iv" 开始，都应当使用 `start="4"`。
`type`|设置编号的类型：`a` 表示小写英文字母编号；`A` 表示大写英文字母编号；`i` 表示小写罗马数字编号；`I` 表示大写罗马数字编号；`1` 表示数字编号（默认）（除非列表中序号很重要（比如，在法律或者技术文件中条目通常被需要所引用），否则请使用 CSS [list-style-type](https://developer.mozilla.org/zh-CN/docs/Web/CSS/list-style-type) 属性替代。）

对 `<ol>` 元素常用的 CSS 属性:

* the [`list-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/list-style) 属性, 有用的选择序数的显示方式,
* [CSS计数器](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Lists_and_Counters/Using_CSS_counters), 用于处理复杂的嵌套列表,
* [`line-height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/line-height) 属性，可以模拟过时的 [`compact`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/ol#attr-compact) 属性；
* [`margin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin) 属性，用来控制列表的缩进。

	