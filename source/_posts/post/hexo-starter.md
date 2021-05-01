---
title: 从零开始制作 Hexo 博客
lang: zh-CN
tags: Hexo
categories: How To
abbrlink: 95d45a6b
date: 2021-04-14 23:32:17
---

本教程介绍如何动手从零开始制作 Hexo 主题的博客网站。

## Hexo 简介

1. 参考 [Hexo 官方文档](https://hexo.io/zh-cn/docs/) 安装 Hexo。
2. 在 GitHub 创建一个空白仓库，比如：[xinetzone/xin](https://github.com/xinetzone/xin)（本教程，就以此为例介绍）。接着，将其克隆到本地。
3. 在 [xinetzone/xin](https://github.com/xinetzone/xin) 下初始化 Hexo 环境：

```shell
hexo init book
```

4. 接着便可以预览网站（虽然会因为缺少主题而报一些警告，但不影响预览）：

```shell
hexo s
```

5. 在 `themes` 下创建自己的主题文件夹，比如：`themes/xin`，最后，还要修改网址配置文件 `_config.yml` 内的 `theme` 设定为 `xin`。

### 主题创建

创建 Hexo 主题非常容易，您只要在 `themes` 文件夹内，新增一个任意名称的文件夹（比如 `xin`），并修改网站配置文件 `_config.yml` 内的 `theme` 设定，即可切换主题。一个主题可能会有以下的结构：

```shell
.
├── _config.yml
├── languages
├── layout
├── scripts
└── source
```

1. `_config.yml`：主题的配置文件。和 Hexo 配置文件不同，主题配置文件修改时会自动更新，无需重启 Hexo Server。
2. `languages`：语言文件夹。请参见 [国际化 (i18n)](https://hexo.io/zh-cn/docs/internationalization)。
3. `layout`：布局文件夹。用于存放主题的模板文件，决定了网站内容的呈现方式，Hexo 内建 [Swig](https://github.com/node-swig/swig-templates) 模板引擎，您可以另外安装插件来获得 [EJS](https://github.com/hexojs/hexo-renderer-ejs)、[Haml](https://github.com/hexojs/hexo-renderer-haml)、[Jade](https://github.com/hexojs/hexo-renderer-jade) 或 [Pug](https://github.com/maxknee/hexo-render-pug) 支持，Hexo 根据模板文件的扩展名来决定所使用的模板引擎，例如：

```shell
layout.ejs   - 使用 EJS
layout.swig  - 使用 Swig
```

您可参考 [模板](https://hexo.io/zh-cn/docs/templates) 以获得更多信息。

4. `scripts`：脚本文件夹。在启动时，Hexo 会载入此文件夹内的 JavaScript 文件，请参见 [插件](https://hexo.io/docs/plugins) 以获得更多信息。
5. `source`：资源文件夹，除了模板以外的 assets，例如 CSS、JavaScript 文件等，都应该放在这个文件夹中。文件或文件夹开头名称为 `_`（下划线线）或隐藏的文件会被忽略。如果文件可以被渲染的话，会经过解析然后储存到 `public` 文件夹，否则会直接拷贝到 `public` 文件夹。
6. 可以直接参考一些优秀的主题进行一些主题配置，比如，复制主题 [`landscape`](https://hexojs.github.io/hexo-theme-landscape/) 的 `languages` 文件。

### 构建 xin 主题模板

模板决定了网站内容的呈现方式，每个主题至少都应包含一个 `index` 模板，以下是各页面相对应的模板名称：

模板|用途|回退
:-|:-|:-
index|首页|
post|文章|index
page|分页|index
archive|归档|index
category|分类归档|archive
tag|标签归档|archive

### 布局

如果页面结构类似，例如两个模板都有页首（Header）和页脚（Footer），您可考虑通过 `layout` 文件让两个模板共享相同的结构。每个布局文件都应包含一个 `body` 变量，以显示相关模板的内容，否则不能正常显示模板。例如：

<output>index.ejs</output>
```ejs
index
```

<output>layout.ejs</output>
```ejs
<!DOCTYPE html>
<html>
  <body><%- body %></body>
</html>
```

生成：

```html
<!DOCTYPE html>
<html>
  <body>index</body>
</html>
```

默认情况下，所有其他模板都使用`layout`模板。您可以在最前面指定其他布局，或将其设置为`false`以禁用它。通过在顶部布局中包含更多布局模板，甚至可以构建复杂的嵌套结构。

#### 局部模版

局部模板对于在模板之间共享组件很有用。典型示例包括页眉，页脚或侧边栏。您可能需要将局部文件放在单独的文件中，以使网站维护更加方便。例如：

<output>partial/header.ejs</output>
```ejs
<h1 id="logo"><%= config.title %></h1>
```

<output>index.ejs</output>
```ejs
<%- partial('partial/header') %>
<div id="content">Home page</div>
```

生成：

```html
<h1 id="logo">My Site</h1>
<div id="content">Home page</div>
```

#### 局部变量

您可以在局部模板中指定局部变量并使用。

<output>partial/header.ejs</output>
```ejs
<h1 id="logo"><%= title %></h1>
```

<output>index.ejs</output>
```ejs
<%- partial('partial/header', {title: 'Hello World'}) %>
<div id="content">Home page</div>
```

生成：

```html
<h1 id="logo">Hello World</h1>
<div id="content">Home page</div>
```

#### 优化

如果您的主题太过于复杂，或是需要生成的文件量太过于庞大，可能会大幅降低性能，除了简化主题外，您可以考虑 Hexo 2.7 新增的局部缓存（Fragment Caching） 功能。本功能借鉴于 [Ruby on Rails](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching)，它储存局部内容，下次便能直接使用缓存内容，可以减少文件夹查询并使生成速度更快。

它可用于页首、页脚、侧边栏等文件不常变动的位置，举例来说：

```ejs
<%- fragment_cache('header', function(){
  return '<header></header>';
});
```

如果您使用局部模板的话，可以更简单：

```ejs
<%- partial('header', {}, {cache: true});
```

> `fragment_cache()` 将会缓存第一次的渲染结果，并在之后直接输出缓存的结果。因此只有在不同页面的渲染结果都相同时才应使用局部缓存。比如，在配置中启用了 `relative_link` 后不应该使用局部缓存，因为相对链接在每个页面可能不同。

### Hexo 网站初步介绍

我们先了解 Hexo 网站的架构：

```shell
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

1. `_config.yml`：网站的 [配置](https://hexo.io/zh-cn/docs/configuration) 信息，可以在此配置大部分的参数。
2. `package.json`：应用程序的信息。[EJS](https://ejs.co/), [Stylus](http://learnboost.github.io/stylus/) 和 [Markdown renderer](http://daringfireball.net/projects/markdown/) 已默认安装，可以自由移除。
3. `scaffolds`：[模版](https://hexo.io/zh-cn/docs/writing) 文件夹。当新建文章时，Hexo 会根据 `scaffold` 来建立文件。Hexo 的模板是指在新建的文章文件中默认填充的内容。例如，如果修改 `scaffold/post.md` 中的 Front-matter 内容，那么每次新建一篇文章时都会包含这个修改。
4. `source`：资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件/文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。
5. `themes`：[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

可以在 `_config.yml` 或[备用配置文件](https://hexo.io/docs/configuration.html#Using-an-Alternate-Config) 中修改站点设置。

#### 网站

参数|描述
:-:|:-
title|网站标题
subtitle|网站副标题
description|网站描述
author|您的名字
language|网站使用的语言。对于简体中文用户来说，使用不同的主题可能需要设置成不同的值，请参考你的主题的文档自行设置，常见的有 zh-Hans 和 zh-CN。
timezone|网站时区。Hexo 默认使用您电脑的时区。请参考 [时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 进行设置，如 America/New_York, Japan, 和 UTC 。一般的，对于中国大陆地区可以使用 Asia/Shanghai。

其中，`description` 主要用于 SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author` 参数用于主题显示文章的作者。

#### 网址

参数|描述|默认值|
:-:|:-|:-
url|网址, must starts with http:// or https://|
root|网站根目录|url's pathname
permalink|文章的 永久链接 格式|:year/:month/:day/:title/
permalink_defaults|永久链接中各部分的默认值|
pretty_urls|改写 permalink 的值来美化 URL|
pretty_urls.trailing_index|是否在永久链接中保留尾部的 index.html，设置为 false 时去除|true
pretty_urls.trailing_html|是否在永久链接中保留尾部的 .html, 设置为 false 时去除 (对尾部的 index.html无效)

<div class="w3-pale-green">
<b>网站存放在子目录</b>：如果您的网站存放在子目录中，例如 <code>http://example.com/blog</code>，则请将您的 <code>url</code> 设为 <code>http://example.com/blog</code>。
</div>

例如：

```yml
# 比如，一个页面的永久链接是 http://example.com/foo/bar/index.html
pretty_urls:
  trailing_index: false
# 此时页面的永久链接会变为 http://example.com/foo/bar/
```

更多的配置信息请移步 [Hexo 配置](https://hexo.io/zh-cn/docs/configuration)。

## 重构 landscape 主题

为了探讨如何从零开始构建 Hexo 博客，对 landscape 主题进行重构，具体见 [xin/tags](https://github.com/xinetzone/xin/tags) v0.1 。接下来便以 v0.1 为基础重写 Hexo 主题。

为了 Hexo 可以正常使用，需要添加 Hexo 部署功能：

```shell
npm i hexo-deployer-git
```

接着修改网站的配置文件 `_config.yml`：

```yml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 道法自然
subtitle: 主宰自我
description: "利用 AI 开发一切有意思的东西"
author: xinetzone
language: zh-CN
timezone: "Asia/Shanghai"

# URL
## Set your site url here. For example, if you use GitHub Page, 
## set url as 'https://username.github.io/project'
url: https://xinetzone.github.io/xin
permalink: /:year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: false # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ""
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: false
  line_number: true
  auto_detect: false
  tab_replace: ""
  wrap: true
  hljs: false
prismjs:
  enable: true
  preprocess: true
  line_number: true
  tab_replace: ""

# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ""
  per_page: 12
  order_by: -date

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Metadata elements
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
## updated_option supports 'mtime', 'date', 'empty'
updated_option: "mtime"

# Pagination
## Set per_page to 0 to disable pagination
per_page: 12
pagination_dir: page

# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include:
exclude:
ignore:

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: xin

# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: 'git'
  repo: git@github.com:xinetzone/xin.git
  branch: gh-pages
```

为了支持 rss/rss2 需要安装：

```shell
npm install hexo-generator-feed
```

并在网站配置文件 `_config.yml` 下添加如下内容：

```yml
# 添加 RSS 订阅支持
## https://github.com/hexojs/hexo-generator-feed
## npm install hexo-generator-feed
feed:
  # Generate both atom and rss2 feeds
  type:
    - rss2
    - atom
  path:
    - rss2.xml
    - atom.xml
```

### 重构 `_partial/head.ejs`

对于一个网站，`<head>` 很重要，为此，我们首要重构 `_partial/head.ejs`。首先编写一些处理标题相关的代码：

```ejs
// _partial/head.ejs
<%
    let title = page.title
    // archives, category, tag pages title
    if (is_archive()) {
        title = __('archive_a')
        if (is_month()){
            title += ': ' + page.year + '/' + page.month
        } else if (is_year()){
            title += ': ' + page.year
        }
    } else if (is_category()){
        title = __('category') + ': ' + page.category
    } else if (is_tag()){
        title = __('tag') + ': ' + page.tag
    }
    // final page title.
    let pageTitle = title ? title + ' | ' + config.title : config.title
%>
```

这里将网页 `page.title` 进行一些预处理，可以令其自适应 `_config.yml` 与 `themes/xin/_config.yml` 的 `title`。

最后，便可以定制一个 `<head>` 元素模板：

```ejs
// _partial/head.ejs
...
<head>
    <meta charset="utf-8">
    <%- partial('google-analytics') %>
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title><%= pageTitle %></title>
    <%- open_graph({twitter_id: theme.twitter, fb_admins: theme.fb_admins, fb_app_id: theme.fb_app_id}) %>
    <% if (config.feed) { %>
        <%- feed_tag() %>
    <% } else if (theme.rss) { %>
        <%- feed_tag(theme.rss) %>
    <% } %>
    <%- css('css/style') %>
</head>
```

下面一一介绍 `<head>` 元素模板：

1. 第 5 行定义了 谷歌分析的模板（暂不展开）。
2. 第 6 行设置视口自适应不同平台和屏幕。
3. 第 7 行定义了页面标题，即 `<title>` 元素。
4. 第 8 行插入 Open Graph 资源。
5. 第 9~13 行，使用 `feed_tag` 辅助函数插入 RSS 链接。
6. 第 14 行定义了 CSS 链接。

可以参考如下示例理解 `feed_tag` 辅助函数：

```ejs
<%- feed_tag('atom.xml') %>
// <link rel="alternate" href="/atom.xml" title="Hexo" type="application/atom+xml">

<%- feed_tag('rss.xml', { title: 'RSS Feed', type: 'rss' }) %>
// <link rel="alternate" href="/atom.xml" title="RSS Feed" type="application/rss+xml">

/* Defaults to hexo-generator-feed's config if no argument */
<%- feed_tag() %>
// <link rel="alternate" href="/atom.xml" title="Hexo" type="application/atom+xml">
```

Hexo 提供辅助函数 `css` 用于载入 CSS 资源。语法：`<%- css(path, ...) %>`。`path` 可以是数组或字符串，如果 `path` 开头不是 `/` 或任何协议，则会自动加上根路径；如果后面没有加上 `.css` 扩展名的话，也会自动加上。使用对象类型可以自定义 CSS 属性。示例如下：

```ejs
<%- css('style.css') %>
// <link rel="stylesheet" href="/style.css">

<%- css(['style.css', 'screen.css']) %>
// <link rel="stylesheet" href="/style.css">
// <link rel="stylesheet" href="/screen.css">

<%- css({ href: 'style.css', integrity: 'foo' }) %>
// <link rel="stylesheet" href="/style.css" integrity="foo">

<%- css([{ href: 'style.css', integrity: 'foo' }, { href: 'screen.css', integrity: 'bar' }]) %>
// <link rel="stylesheet" href="/style.css" integrity="foo">
// <link rel="stylesheet" href="/screen.css" integrity="bar">
```

再回头看看那个有点奇怪的元数据 Open Graph。Open Graph Protocol(开放内容协议) 是一种新的 HTTP 头部标记，即这种协议可以让网页成为一个“富媒体对象”。用了 `meta property=og` 标签，就是你同意了网页内容可以被其他社会化网站引用等。

Hexo 提供了 Open Graph 的辅助函数用于插入 open graph 资源。语法：`<%- open_graph([options]) %>`。

参数|描述|默认值
:-|:-|:-
title|页面标题 (og:title)|page.title
type|页面类型 (og:type)|blog
url|页面网址 (og:url)|url
image|页面图片 (og:image)|内容中的图片
site_name|网站名称 (og:site_name)|config.title
description|页面描述 (og:description)|内容摘要或前 200 字
twitter_card|Twitter 卡片类型 (twitter:card)|summary
twitter_id|Twitter ID (twitter:creator)|
twitter_site|Twitter 网站 (twitter:site)|
google_plus|Google+ 个人资料链接|
fb_admins|Facebook 管理者 ID|
fb_app_id|Facebook 应用程序 ID|

### 关于资源文件夹

如果我们的文章里面有图片，我们可以在 source 文件夹下建立一个统一的 images 文件夹来存放图片，但是如果有的文章有很多的资源文件如图片，我们可以通过设置该配置为 `true`，这样在`source`文件夹下创建文件的同时也会创建一个同名文件夹来存放相应的资源。

```yml
post_asset_folder: true # 是否启动资源文件夹
marked: # 保证资源的链接正确
  prependRoot: true
  postAsset: true
  gfm: true
```

### 设定多语言支持

为了添加多语言支持，需要修改网站配置文件 `_config.yml`：

```yml
# URL
## Set your site url here. For example, if you use GitHub Page,
## set url as 'https://username.github.io/project'
url: https://xinetzone.github.io/xin
permalink: :lang/:year/:month/:day/:title/
permalink_defaults:
  lang: en-US # 设定默认
```

为了令网址具有唯一性，可以在配置文件 `_config.yml` 中加入如下内容并配置：

```yml
## npm install hexo-abbrlink
### https://github.com/rozbo/hexo-abbrlink
# abbrlink config
abbrlink:
  alg: crc32      #support crc16(default) and crc32
  rep: hex        #support dec(default) and hex
  drafts: false   #(true)Process draft,(false)Do not process draft. false(default) 
  # Generate categories from directory-tree
  # depth: the max_depth of directory-tree you want to generate, should > 0
  auto_category:
     enable: true  #true(default)
     depth:        #3(default)
     over_write: false 
  auto_title: false #enable auto title, it can auto fill the title by path
  auto_date: false #enable auto date, it can auto fill the date by time today
  force: false #enable force mode,in this mode, the plugin will ignore the cache, and calc the abbrlink for every post even it already had abbrlink.
permalink: :layout/:lang/:abbrlink.html # :year/:month/:day/:title/
permalink_defaults:
  lang: en-US # 设定默认
```

### 站内搜索

Hexo 使用 `hexo-generator-search` 添加 `local-search`，需要在网站配置文件 `_config.yml` 新增如下内容：

```yml
## Hexo使用hexo-generator-search添加local-search
search:
  path: search.xml
  field: post
  content: true
```

### 支持代码行显示数字

为了支持代码行支持显示数字序号，可以添加文件 `themes/xin/source/css/xin-prism.css/`，并在 `themes/xin/layout/_partial/head.ejs` 中新增 CSS 引用。

### 添加 W3.CSS 与 tab 切换 支持

在 `themes/xin/layout/_partial/head.ejs` 中新增：

```ejs
<%- css('https://xinetzone.github.io/w3css/4/w3.css') %>
<%- css('https://xinetzone.github.io/xinet-css/tabs.css') %>
```