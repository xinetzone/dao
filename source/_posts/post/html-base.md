---
title: HTML <base> 改变网站根目录
lang: zh-CN
tags: HTML
categories: 手册
abbrlink: 645e80b4
date: 2021-02-15 22:14:50
description:
updated:
---

`<base>` 元素为页面上的所有链接规定默认根地址（href）或默认目标（target）。通常情况下，浏览器会从当前文档的 URL 中提取相应的元素来填写相对 URL 中的空白。使用 `<base>` 元素可以改变这一点。浏览器随后将不再使用当前文档的 URL，而使用指定的基本 URL 来解析所有的相对 URL。这其中包括 `<a>`、`<img>`、`<link>`、`<form>` 元素中的 URL。

注意：`<base>` 元素必须位于 `<head>` 元素内部，且一个文档仅只能有一个 `<base>` 元素。

示例如下：

```html
<html>
    <head>
    <base href="http://www.w3school.com.cn/i/" />
    <base target="_blank" />
    </head>
    <body>
        <img src="eg_smile.gif" /><br />
        <p>请注意，我们已经为图像规定了一个相对地址。由于我们已经在 head 部分规定了一个基准 URL，浏览器将在如下地址寻找图片：</p>
        <p>"http://www.w3school.com.cn/i/eg_smile.gif"</p>
        <br /><br />
        <p><a href="http://www.w3school.com.cn">W3School</a></p>
        <p>请注意，链接会在新窗口中打开，即使链接中没有 target="_blank" 属性。这是因为 `<base>` 元素的 target 属性已经被设置为 "_blank" 了。</p>
    </body>
</html>
```

一个文档的 `<base>` URL, 可以通过使用 [`document.baseURI`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/baseURI) 的 JS 脚本查询。如果文档不包含 `<base>` 元素，`baseURI` 默认为 [`document.location.href`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/location/href)。

## 页内跳转

指向文档中某个片段的链接，例如 `<a href="#some-id">` 用 `<base>` 解析，触发对带有附加片段的基本 URL 的 HTTP 请求。

例如：给定 `<base href="https://example.com">` 以及此链接 `<a href="#anchor">Anker</a>`，链接j会指向 <https://example.com/#anchor>。