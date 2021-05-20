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

## 与外部软件交互（待更）

从零开发一个产品难度很大，往往可以借助其他成熟的软件与自己的产品交互。下面以打开音乐播放器为例进行介绍。

首先，下载并安装一个音乐播放器，比如 [网易云音乐](https://music.163.com/)，然后，找到运行它的可执行性文件，即 "C:\Program Files (x86)\Netease\CloudMusic\cloudmusic.exe"。

![](wyy.png)

## Multi-Page Apps and URL Support

Dash 将 web 应用渲染为“单页应用”。这意味着当用户导航应用程序时，应用程序不会完全重新加载，从而使浏览速度非常快。

有两个组件可以帮助页面导航：`dash_core_components.Location` 和 `dash_core_components.Link`。

`dash_core_components.Location` 通过 `pathname` 属性表示 web 浏览器中的位置栏。这里有一个简单的例子：

```python
app = create_app()

layout = html.Div([
    # 表示 URL栏，不作任何渲染
    dcc.Location(id='url', refresh=False),

    dcc.Link('Navigate to "/"', href='/'),
    html.Br(),
    dcc.Link('Navigate to "/page-2"', href='/page-2'),
    # content will be rendered in this element
    html.Div(id='page-content')
])


@app.callback(Output('page-content', 'children'),
              [Input('url', 'pathname')])
def display_page(pathname):
    return html.Div([
        html.H3(f'You are on page {pathname}')
    ])
await run_server(app, layout, port=8050)
```


![](url-support.gif)

在这个例子中，回调`display_page`接收页面的当前路径名(URL 的最后一部分)。回调只是在页面上显示 `pathname`，但是它可以使用路径名来显示不同的内容。`Link` 元素更新浏览器的路径名，而不刷新页面。如果你使用 `html.A` 元素，那么路径名更新，页面也会刷新。具体细节见下图：请注意，尽管点击链接会更新 URL，但它不会刷新页面。

`Link` 允许你在一个多页面应用程序中创建一个可点击的链接。对于当前应用程序之外的目的地链接，`html.A` 是一个更好的组件选择。


`dcc.Location` 组件表示 web 浏览器中的位置或地址栏。通过它的 `href`, `pathname`, `search` 和 `hash` 属性，你可以访问应用程序加载的 url 的不同部分。

例如，给定 url `http://127.0.0.1:8050/page-2?a=test#quiz`，有：

- `href` = `"http://127.0.0.1:8050/page-2?a=test#quiz"`
- `pathname` = `"/page-2"`
- `search` = `"?a=test"`
- `hash` = `"#quiz"`

你可以修改上面的例子来根据 URL 显示不同的页面：

```python
# 因为我们给app.layout中不存在的元素添加了回调，
# Dash会提出一个异常来警告我们，我们可能做错了什么。
# 在本例中，我们通过回调添加元素，因此可以忽略异常。
app = create_app(suppress_callback_exceptions=True)

layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='page-content')
])


index_page = html.Div([
    dcc.Link('Go to Page 1', href='/page-1'),
    html.Br(),
    dcc.Link('Go to Page 2', href='/page-2'),
])

page_1_layout = html.Div([
    html.H1('Page 1'),
    dcc.Dropdown(
        id='page-1-dropdown',
        options=[{'label': i, 'value': i} for i in ['LA', 'NYC', 'MTL']],
        value='LA'
    ),
    html.Div(id='page-1-content'),
    html.Br(),
    dcc.Link('Go to Page 2', href='/page-2'),
    html.Br(),
    dcc.Link('Go back to home', href='/'),
])

@app.callback(Output('page-1-content', 'children'),
              [Input('page-1-dropdown', 'value')])
def page_1_dropdown(value):
    return f'You have selected "{value}"'


page_2_layout = html.Div([
    html.H1('Page 2'),
    dcc.RadioItems(
        id='page-2-radios',
        options=[{'label': i, 'value': i} for i in ['Orange', 'Blue', 'Red']],
        value='Orange'
    ),
    html.Div(id='page-2-content'),
    html.Br(),
    dcc.Link('Go to Page 1', href='/page-1'),
    html.Br(),
    dcc.Link('Go back to home', href='/')
])

@app.callback(Output('page-2-content', 'children'),
              [Input('page-2-radios', 'value')])
def page_2_radios(value):
    return f'You have selected "{value}"'


# Update the index
@app.callback(Output('page-content', 'children'),
              [Input('url', 'pathname')])
def display_page(pathname):
    if pathname == '/page-1':
        return page_1_layout
    elif pathname == '/page-2':
        return page_2_layout
    else:
        return index_page
    # You could also return a 404 "URL not found" page here

await run_server(app, layout, port=8050)
```

![](url-support-pages.gif)

在这个例子中，我们通过 `display_page` 函数显示不同的布局。

- 每个页面都可以有交互元素，即使这些元素可能不在初始视图中。Dash 优雅地处理这些“动态生成”的组件：当它们被渲染时，它们会用它们的初始值触发回调。
- 因为我们给app.layout中不存在的元素添加回调，Dash 会引发异常，警告我们可能做错了什么。在本例中，我们通过回调添加元素，因此可以通过设置`suppress_callback_exceptions=True`忽略异常。在不抑制回调异常的情况下也可以做到这一点。详细信息请参见下面的示例。
- 您可以修改此示例，以在不同的文件中导入不同页面的布局。
- 你看到的这个 Dash 用户指南本身就是一个多页的 Dash 应用程序，使用了相同的原则。

### 动态创建多页面应用验证的布局

Dash 将验证应用于回调，它将执行检查，例如验证回调参数的类型，检查指定的 `Input` 和 `Output` 组件是否具有指定的属性。

对于完全验证，回调中的所有组件都必须出现在应用程序的初始布局中，如果它们没有出现，你将看到一个错误。然而，在更复杂的 Dash 应用中，需要动态修改布局(如多页应用)，并非回调中出现的每个组件都包含在初始布局中。

 Dash 1.12 你可以设置`app.validation_layout`为一个`"complete"`布局，包含你将在任何页面 `/` 部分中使用的所有组件。`app.validation_layout` 必须是一个 Dash 组件，而不是一个函数。然后将 `app.layout` 设置为索引布局。在以前的 Dash 版本中，你可以使用一个技巧来实现相同的结果，检查布局函数中的`flask.has_request_context`—仍然可以工作，但不再推荐。

```python
app = create_app()
url_bar_and_content_div = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='page-content')
])

layout_index = html.Div([
    dcc.Link('Navigate to "/page-1"', href='/page-1'),
    html.Br(),
    dcc.Link('Navigate to "/page-2"', href='/page-2'),
])

layout_page_1 = html.Div([
    html.H2('Page 1'),
    dcc.Input(id='input-1-state', type='text', value='Montreal'),
    dcc.Input(id='input-2-state', type='text', value='Canada'),
    html.Button(id='submit-button', n_clicks=0, children='Submit'),
    html.Div(id='output-state'),
    html.Br(),
    dcc.Link('Navigate to "/"', href='/'),
    html.Br(),
    dcc.Link('Navigate to "/page-2"', href='/page-2'),
])

layout_page_2 = html.Div([
    html.H2('Page 2'),
    dcc.Dropdown(
        id='page-2-dropdown',
        options=[{'label': i, 'value': i} for i in ['LA', 'NYC', 'MTL']],
        value='LA'
    ),
    html.Div(id='page-2-display-value'),
    html.Br(),
    dcc.Link('Navigate to "/"', href='/'),
    html.Br(),
    dcc.Link('Navigate to "/page-1"', href='/page-1'),
])

# index layout
layout = url_bar_and_content_div

# "complete" layout
app.validation_layout = html.Div([
    url_bar_and_content_div,
    layout_index,
    layout_page_1,
    layout_page_2,
])


# Index callbacks
@app.callback(Output('page-content', 'children'),
              Input('url', 'pathname'))
def display_page(pathname):
    if pathname == "/page-1":
        return layout_page_1
    elif pathname == "/page-2":
        return layout_page_2
    else:
        return layout_index


# Page 1 callbacks
@app.callback(Output('output-state', 'children'),
              Input('submit-button', 'n_clicks'),
              State('input-1-state', 'value'),
              State('input-2-state', 'value'))
def update_output(n_clicks, input1, input2):
    return (f'The Button has been pressed {n_clicks} times,'
            f'Input 1 is "{input1}",'
            f'and Input 2 is "{input2}"')


# Page 2 callbacks
@app.callback(Output('page-2-display-value', 'children'),
              Input('page-2-dropdown', 'value'))
def display_value(value):
    print('display_value')
    return f'You have selected "{value}"'
```

### 结构化多页面应用程序

下面是如何构建一个多页面应用程序，每个应用程序都包含在一个单独的文件中。文件结构：

```python
- app.py
- index.py
- apps
   |-- __init__.py
   |-- app1.py
   |-- app2.py
```

`app.py`

```python
import dash

app = dash.Dash(__name__, suppress_callback_exceptions=True)
server = app.server
```

`apps/app1.py`

```python
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

from app import app

layout = html.Div([
    html.H3('App 1'),
    dcc.Dropdown(
        id='app-1-dropdown',
        options=[
            {'label': f'App 1 - {i}', 'value': i} 
            for i in ['NYC', 'MTL', 'LA']
        ]
    ),
    html.Div(id='app-1-display-value'),
    dcc.Link('Go to App 2', href='/apps/app2')
])


@app.callback(
    Output('app-1-display-value', 'children'),
    Input('app-1-dropdown', 'value'))
def display_value(value):
    return f'You have selected "{value}"'
```

`apps/app2.py`

```python
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State

from app import app

layout = html.Div([
    html.H2('App 2'),
    dcc.Input(id='input-1-state', type='text', value='Montreal'),
    dcc.Input(id='input-2-state', type='text', value='Canada'),
    html.Button(id='submit-button', n_clicks=0, children='Submit'),
    html.Div(id='output-state'),
    html.Br(),
    dcc.Link('Navigate to "/"', href='/'),
    html.Br(),
    dcc.Link('Navigate to "/apps/app1"', href='/apps/app1'),
])


@app.callback(Output('output-state', 'children'),
              Input('submit-button', 'n_clicks'),
              State('input-1-state', 'value'),
              State('input-2-state', 'value'))
def update_output(n_clicks, input1, input2):
    return (f'The Button has been pressed {n_clicks} times,'
            f'Input 1 is "{input1}",'
            f'and Input 2 is "{input2}"')
```

`index.py`

`index.py` 在不同的 url 上加载不同的应用，就像这样：

```python
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

from app import app
from apps import app1, app2


app.layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='page-content')
])


@app.callback(Output('page-content', 'children'),
              Input('url', 'pathname'))
def display_page(pathname):
    if pathname == '/apps/app1':
        return app1.layout
    elif pathname == '/apps/app2':
        return app2.layout
    else:
        return '404'


if __name__ == '__main__':
    app.run_server(debug=True)
```

或者，你可能更喜欢一个平面的项目布局，将回调和布局分开到不同的文件中：

```shell
- app.py
- index.py
- callbacks.py
- layouts.py
```

`app.py`

```python
import dash

app = dash.Dash(__name__, suppress_callback_exceptions=True)
server = app.server
```

`callbacks.py`

```python
from dash.dependencies import Input, Output

from app import app

@app.callback(
    Output('app-1-display-value', 'children'),
    Input('app-1-dropdown', 'value'))
def display_value(value):
    return 'You have selected "{}"'.format(value)

@app.callback(
    Output('app-2-display-value', 'children'),
    Input('app-2-dropdown', 'value'))
def display_value(value):
    return 'You have selected "{}"'.format(value)
```

`layouts.py`

```python
import dash_core_components as dcc
import dash_html_components as html

layout1 = html.Div([
    html.H3('App 1'),
    dcc.Dropdown(
        id='app-1-dropdown',
        options=[
            {'label': 'App 1 - {}'.format(i), 'value': i} for i in [
                'NYC', 'MTL', 'LA'
            ]
        ]
    ),
    html.Div(id='app-1-display-value'),
    dcc.Link('Go to App 2', href='/apps/app2')
])

layout2 = html.Div([
    html.H3('App 2'),
    dcc.Dropdown(
        id='app-2-dropdown',
        options=[
            {'label': 'App 2 - {}'.format(i), 'value': i} for i in [
                'NYC', 'MTL', 'LA'
            ]
        ]
    ),
    html.Div(id='app-2-display-value'),
    dcc.Link('Go to App 1', href='/apps/app1')
])
```

`index.py`

```python
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

from app import app
from layouts import layout1, layout2
import callbacks

app.layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='page-content')
])

@app.callback(Output('page-content', 'children'),
              Input('url', 'pathname'))
def display_page(pathname):
    if pathname == '/apps/app1':
         return layout1
    elif pathname == '/apps/app2':
         return layout2
    else:
        return '404'

if __name__ == '__main__':
    app.run_server(debug=True)
```

值得注意的是，在这两个项目结构中，Dash 实例是在单独的 `app.py` 中定义的，而运行应用的入口点是 `index.py`。这种分离是为了避免循环导入：包含回调定义的文件需要访问 Dash 应用程序实例，但是如果是从 `index.py` 导入的，`index.py` 的初始加载最终会要求自己已经导入，这是不能满足的。

## 