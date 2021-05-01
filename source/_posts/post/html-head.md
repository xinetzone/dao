---
title: HTML <head>
lang: zh-CN
abbrlink: 76468b6e
date: 2021-02-15 22:38:18
description:
updated:
tags: HTML
categories: 手册
---
HTML `<head>` 元素规定文档相关的配置信息（元数据），包括文档的标题，引用的文档样式和脚本等。HTML `<head>` 元素与 `<body>` 元素不同，它的内容不会在浏览器中显示，它的作用是保存页面的一些 **元数据**。

下面介绍一些常用的 `<head>` 元素的子元素。

## &lt;title> 元素

HTML `<title>` 元素定义文档的标题，显示在浏览器的标题栏或标签页上。它只可以包含文本，若是包含有标签，则它包含的任何标签都将被忽略。

页面 `<title>` 的内容对于搜索引擎优化（SEO）非常重要！搜索引擎算法使用页面标题来确定在搜索结果中列出页面时的顺序。用途如下：

- 在浏览器工具栏中定义标题
- 将页面添加到收藏夹时为其提供标题
- 在搜索引擎结果中显示页面标题

以下是创建优质 `<title>` 的一些技巧：

- 选择更长的描述性标题（避免使用一个或两个单词的标题）
- 搜索引擎将显示标题的大约50至60个字符，因此请确保标题的长度不超过该字符
- 不要仅将单词列表用作标题（这可能会降低页面在搜索结果中的位置）
- 尝试确保您的标题在您自己的网站中尽可能唯一。标题重复（或几乎重复）可能会导致搜索结果不准

因此，请尝试使 `<title>` 尽可能准确和有意义！

注意别和 `<h1>` 元素搞混了，`<h1>` 是为 `<body>` 添加标题的。有时候 `<h1>` 也叫作网页标题。但是二者并不相同。

- `<h1>` 元素在页面加载完毕时显示在页面中，通常只出现一次，用来标记页面内容的标题（故事名称、新闻摘要，等等）。
- `<title>` 元素是一项元数据，用于表示整个 HTML 文档的标题（而不是文档内容）。

## &lt;link> 元素

HTML外部资源链接元素 (`<link>`) 规定了当前文档与外部资源的关系。该元素最常用于链接样式表，此外也可以被用来创建站点图标(比如 PC 端的“favicon”图标和移动设备上用以显示在主屏幕的图标) 。

要链接一个外部的样式表，你需要在你的`<head>`中包含一个`<link>`元素：

```html
<link href="main.css" rel="stylesheet">
```

在这个简单的例子中，使用了 `href` 属性设置外部资源的路径，并设置 `rel` 属性的值为“stylesheet”(样式表)。`rel` 表示“关系 (relationship) ”，它可能是 `<link>` 元素其中一个关键的特性——属性值表示 `<link>` 项的链接方式与包含它的文档之间的关系。

### 在你的站点增加自定义图标

为了进一步丰富你的网站设计，你可以在元数据中添加对自定义图标的引用，这些将在特定的场合中显示。

这个不起眼的图标已经存在很多很多年了，16 x 16 像素是这种图标的第一种类型。你可以看见这些图标出现在浏览器每一个打开的页面中的标签页中中以及在书签面板中的书签页面中。

页面添加图标的方式有：

1. 将其保存在与网站的索引页面相同的目录中，以 `.ico` 格式保存（大多数浏览器将支持更通用的格式，如 `.gif` 或 `.png`，但使用 ICO 格式将确保它能在如 Internet Explorer 6 一样久远的浏览器显示）
2. 将以下行添加到 HTML `<head>` 中以引用它：

```html
<link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
```

如今还有很多其他的图标类型可以考虑。例如：

```html
<!-- third-generation iPad with high-resolution Retina display: -->
<link rel="apple-touch-icon-precomposed" sizes="144x144" href="https://developer.cdn.mozilla.net/static/img/favicon144.a6e4162070f4.png">
<!-- iPhone with high-resolution Retina display: -->
<link rel="apple-touch-icon-precomposed" sizes="114x114" href="https://developer.cdn.mozilla.net/static/img/favicon114.0e9fabd44f85.png">
<!-- first- and second-generation iPad: -->
<link rel="apple-touch-icon-precomposed" sizes="72x72" href="https://developer.cdn.mozilla.net/static/img/favicon72.8ff9d87c82a0.png">
<!-- non-Retina iPhone, iPod Touch, and Android 2.1+ devices: -->
<link rel="apple-touch-icon-precomposed" href="https://developer.cdn.mozilla.net/static/img/favicon57.a2490b9a2d76.png">
<!-- basic favicon -->
<link rel="shortcut icon" href="https://developer.cdn.mozilla.net/static/img/favicon32.e02854fdcf73.png">
```

这些注释解释了每个图标的用途：这些元素涵盖的东西提供一个高分辨率图标，这些高分辨率图标当网站保存到 iPad 的主屏幕时使用。

用于表示不同移动平台上特殊的图标类型，例如：

```html
<link rel="apple-touch-icon-precomposed" sizes="114x114"
      href="apple-icon-114.png" type="image/png">
```

`sizes` 属性表示图标大小，`type` 属性包含了链接资源的 MIME 类型。这些属性为浏览器选择最合适的图标提供了有用的提示。

你也可以提供一个媒体类型，或者在 `media` 属性内部进行查询；这种资源将只在满足媒体条件的情况下才被加载进来。例如：

```html
<link href="print.css" rel="stylesheet" media="print">
<link href="mobile.css" rel="stylesheet" media="screen and (max-width: 600px)">
```

`<link>` 也加入了一些新的有意思的性能和安全特性。举例如下：

```html
<link rel="preload" href="myFont.woff2" as="font"
      type="font/woff2" crossorigin="anonymous">
```

将 `rel` 设定为 `preload`，表示浏览器应该预加载该资源 (更多细节见[使用rel="preload"预加载内容](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)) 。`as` 属性表示获取特定的内容类。`crossorigin` 属性表示该资源是否应该使用一个 CORS 请求来获取。