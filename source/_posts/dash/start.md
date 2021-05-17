---
layout: dash
title: Dash 创建上位机界面
date: 2021-05-17 08:30:30
tags: 
    - dash
    - aispace
categories: notebook
---

`Dash` 是建立在 `Flask`、`Poltly.js` 以及 `React.js` 之上的 Python 框架，它降低了前端入门的门槛，帮助你快速搭建网站、数据可视化工具、上位机界面等应用。

为了更加方便使用 `Dash`，需要安装一些包：

- `numpy`，`pandas`，`matplotlib`
- `dash`，`jupyter_dash`

其中 [jupyter_dash](https://github.com/plotly/jupyter-dash) 用于支持在 Jupyter Notebook 中运行 Dash。为了提供可以同时在 CMD 和 Jupyter 运行的 Dash 环境，在 [aispace](https://github.com/xinetzone/aispace) 维护了一个代码库。

## 创建服务器主接口

关于 `aispace.server` 的细节可参考：[添加 CSS 和 JS，覆盖页面加载模板](https://xinetzone.github.io/dao/dash/zh-CN/ef498434c4c8.html)。

运行 Dash：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--1" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--1">Jupyter Notebook</label>
        <div class="tab-content w3-padding">
            ```python
            import dash_html_components as html
            from aispace.server import create_app, run_server

            app = create_app()
            layout = html.H1('第一个 Dash 应用！')
            await run_server(app, layout, port=10000)
            ```
        </div>
        <input id="tab-set--0-input--2" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--2">CMD</label>
        <div class="tab-content w3-padding">
            ```python
            import asyncio
            import dash_html_components as html
            from aispace.server import create_app, run_server

            app = create_app()
            layout = html.H1('第一个 Dash 应用！')


            if __name__ == '__main__':
                asyncio.run(run_server(app, layout, port=10000))
            ```
        </div>
    </div>
</article>

由于 Jupyter 十分便利，所以后面均以 Jupyter 作为运行环境。

本例便是一个完整的 Dash 应用，`app` 是 Dash 的应用主接口，`layout` 是 UI 的布局，`run_server` 启动服务器。

本例仅仅定义了 `<h1>` 元素，效果如下：

![](first.png)

<div class="w3-yellow">
<code>aispace.server</code> 提供了 <a href="https://www.w3schools.com/w3css/default.asp">W3.CSS</a> 与 <a href="https://xinetzone.github.io/Font-Awesome/css/all.css">Font Awesome</a> 支持。
</div>

为了便利，先载入一些必需包和模块（此处先不深入，后续会一一展开）：

```python
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
from dash.dependencies import Input, Output, State, MATCH, ALL

# 自定义
from aispace.server import create_app, run_server
from aispace.utils.nav import create_nav
```

本文将探讨如何使用 Dash 创建一个可交互的上位机界面。

## 与外部软件交互

从零开发一个产品难度很大，往往可以借助其他成熟的软件与自己的产品交互。下面以打开音乐播放器为例进行介绍。

首先，下载并安装一个音乐播放器，比如 [网易云音乐](https://music.163.com/)，然后，找到运行它的可执行性文件，即 "C:\Program Files (x86)\Netease\CloudMusic\cloudmusic.exe"。

![](wyy.png)

