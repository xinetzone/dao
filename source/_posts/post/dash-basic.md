---
title: Dash 教程
lang: zh-CN
tags: Dash
categories: 教程
abbrlink: a7b795ee
date: 2021-03-20 18:35:10
description:
updated:
---

Dash 是用于构建 Web 分析应用程序的高效 Python 框架。Dash 建立在 Flask，Plotly.js 和 React.js 之上，是使用纯 Python 使用高度自定义用户界面构建数据可视化应用程序的理想选择。它特别适合使用 Python 处理数据的任何人。

通过几个简单的模式，Dash 提取了构建基于 Web 的交互式应用程序所需的所有技术和协议。Dash 应用程序在 Web 浏览器中呈现。您可以将应用程序部署到服务器，然后通过 URL 共享它们。由于 Dash 应用程序是在 Web 浏览器中查看的，因此 Dash 本质上是跨平台且可移动部署的。

## Dash Layout

Dash 应用程序由两部分组成。第一部分是应用程序的“`layout`”，它描述了应用程序的外观。第二部分描述了应用程序的交互性，并将在下一章中介绍。

Dash 为应用程序的所有可视组件提供了 Python 类。我们在 `dash_core_components` 和 `dash_html_components` 库中维护了一组组件，但是您也可以使用 JavaScript 和 React.js [构建自己的组件](https://github.com/plotly/dash-component-boilerplate)。

看一个例子：

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.
import dash
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
import pandas as pd

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

# assume you have a "long-form" data frame
# see https://plotly.com/python/px-arguments/ for more options
df = pd.DataFrame({
    "Fruit": ["Apples", "Oranges", "Bananas", "Apples", "Oranges", "Bananas"],
    "Amount": [4, 1, 2, 2, 4, 5],
    "City": ["SF", "SF", "SF", "Montreal", "Montreal", "Montreal"]
})

fig = px.bar(df, x="Fruit", y="Amount", color="City", barmode="group")

app.layout = html.Div(children=[
    html.H1(children='Hello Dash'),

    html.Div(children='''
        Dash: A web application framework for Python.
    '''),

    dcc.Graph(
        id='example-graph',
        figure=fig
    )
])

if __name__ == '__main__':
    app.run_server(debug=True)
```

`layout` 由诸如 `html.Div` 和 `dcc.Graph` 之类的 `"components"` 树组成。

对于每个 HTML 标签都有一个 `dash_html_components` 库的组件与之对应。`html.H1(children='Hello Dash')` 组件在您的应用程序中生成一个 `<h1> Hello Dash </h1>` HTML 元素。

并非所有组件都是纯 HTML。`dash_core_components` 描述了交互式的更高级组件，这些组件是通过 `React.js` 库使用 JavaScript，HTML 和 CSS 生成的。

每个组件都完全通过关键字属性来描述。Dash 是声明性的：您将主要通过这些属性来描述您的应用程序。`children` 属性是特殊的。按照惯例，它始终是第一个属性，这意味着您可以忽略它：`html.H1(children='Hello Dash')` 与 `html.H1('Hello Dash')` 相同。而且，它可以包含字符串，数字，单个组件或组件列表。

您的应用程序中的字体看起来与此处显示的字体略有不同。此应用程序使用自定义 CSS 样式表来修改元素的默认样式。您可以在 [CSS 教程](https://dash.plotly.com/external-resources)中了解更多信息，但现在您可以使用：

```python
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)
```

以获得与这些示例相同的外观。

### 进行第一次更改

dash 0.30.0 和 dash-renderer 0.15.0 中的新增功能

Dash 包含“热重载”（"hot-reloading"），默认情况下，当您使用 `app.run_server(debug=True)` 运行应用程序时，此功能已激活。这意味着当您更改代码时，Dash 将自动刷新浏览器。

试试看：在应用程序中更改标题“Hello Dash”或更改 x 或 y 数据。您的应用应随您的更改自动刷新。

> 不喜欢热重载吗？您可以使用 `app.run_server(dev_tools_hot_reload=False)` 将其关闭。在 [Dash Dev Tools](https://dash.plotly.com/devtools) 文档中了解更多信息有疑问吗？请参阅[社区论坛热重载](https://community.plotly.com/t/announcing-hot-reload/14177)讨论。

### 有关 HTML 的更多信息

`dash_html_components` 库包含每个 HTML 标记的组件类以及所有 HTML 参数的关键字参数。

<details>
    <summary>让我们通过修改组件的内联样式来自定义应用程序中的文本。使用以下代码创建一个名为“app.py”的文件：</summary>
    ```python
    # -*- coding: utf-8 -*-
    # Run this app with `python app.py` and
    # visit http://127.0.0.1:8050/ in your web browser.

    import dash
    import dash_core_components as dcc
    import dash_html_components as html
    import plotly.express as px
    import pandas as pd

    external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

    app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

    colors = {
        'background': '#111111',
        'text': '#7FDBFF'
    }

    # assume you have a "long-form" data frame
    # see https://plotly.com/python/px-arguments/ for more options
    df = pd.DataFrame({
        "Fruit": ["Apples", "Oranges", "Bananas", "Apples", "Oranges", "Bananas"],
        "Amount": [4, 1, 2, 2, 4, 5],
        "City": ["SF", "SF", "SF", "Montreal", "Montreal", "Montreal"]
    })

    fig = px.bar(df, x="Fruit", y="Amount", color="City", barmode="group")

    fig.update_layout(
        plot_bgcolor=colors['background'],
        paper_bgcolor=colors['background'],
        font_color=colors['text']
    )

    app.layout = html.Div(style={'backgroundColor': colors['background']}, children=[
        html.H1(
            children='Hello Dash',
            style={
                'textAlign': 'center',
                'color': colors['text']
            }
        ),

        html.Div(children='Dash: A web application framework for Python.', style={
            'textAlign': 'center',
            'color': colors['text']
        }),

        dcc.Graph(
            id='example-graph-2',
            figure=fig
        )
    ])

    if __name__ == '__main__':
        app.run_server(debug=True)
    ```
</details>

在此示例中，我们使用`style`属性修改了`html.Div`和`html.H1`组件的内联样式。

`html.H1('Hello Dash', style={'textAlign': 'center', 'color': '#7FDBFF'})` 在 Dash 应用程序中呈现为`<h1 style="text-align: center; color: #7FDBFF">Hello Dash</h1>`。

`dash_html_components` 和 HTML 属性之间有一些重要的区别：

1. HTML 中的 `style` 属性是用分号分隔的字符串。在 Dash 中，您可以仅提供字典。
2. style 字典中的键是驼峰式的。因此，它不是 `text-align`，而是 `textAlign`。
3. HTML `class` 属性是 Dash 中的 `className`。
4. HTML 标记的子代是通过 `children` 关键字参数指定的。按照惯例，这始终是第一个参数，因此经常被省略。

除此之外，您还可以在 Python 上下文中使用所有可用的 HTML 属性和标记。

### 可重复使用的组件

通过使用 Python 编写标记，我们可以创建复杂的可重用组件（例如表），而无需切换上下文或语言。

<details>
<summary>这是一个简单的示例，该示例根据 Pandas 数据框生成“表格”。使用以下代码创建一个名为“app.py”的文件：
</summary>
```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

import dash
import dash_html_components as html
import pandas as pd

df = pd.read_csv('https://gist.githubusercontent.com/chriddyp/c78bf172206ce24f77d6363a2d754b59/raw/c353e8ef842413cae56ae3920b8fd78468aa4cb2/usa-agricultural-exports-2011.csv')


def generate_table(dataframe, max_rows=10):
    return html.Table([
        html.Thead(
            html.Tr([html.Th(col) for col in dataframe.columns])
        ),
        html.Tbody([
            html.Tr([
                html.Td(dataframe.iloc[i][col]) for col in dataframe.columns
            ]) for i in range(min(len(dataframe), max_rows))
        ])
    ])


external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div(children=[
    html.H4(children='US Agriculture Exports (2011)'),
    generate_table(df)
])

if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

### 有关可视化的更多信息

`dash_core_components` 库包含一个名为 `Graph` 的组件。`Graph` 使用开源 [plotly.js](https://github.com/plotly/plotly.js) JavaScript 图形库呈现交互式数据可视化。Plotly.js 支持超过 35 种图表类型，并以矢量质量 SVG 和高性能 WebGL 呈现图表。

`dash_core_components.Graph` 组件中的 `Figure` 参数与 Plotly 的开源 Python 图形库 plotly.py 使用的图形参数相同。请查看 [plotly.py 文档和画廊](https://plotly.com/python) 以了解更多信息。

<details><summary>这是一个从 Pandas 数据框创建散点图的示例。使用以下代码创建一个名为“app.py”的文件：</summary>
```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

import dash
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
import pandas as pd


external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

df = pd.read_csv('https://gist.githubusercontent.com/chriddyp/5d1ea79569ed194d432e56108a04d188/raw/a9f9e8076b837d541398e999dcbac2b2826a81f8/gdp-life-exp-2007.csv')

fig = px.scatter(df, x="gdp per capita", y="life expectancy",
                 size="population", color="continent", hover_name="country",
                 log_x=True, size_max=60)

app.layout = html.Div([
    dcc.Graph(
        id='life-exp-vs-gdp',
        figure=fig
    )
])

if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

这些图是交互式的和响应式的。将鼠标悬停在点上以查看其值，单击图例项以切换轨迹，单击并拖动以缩放，按住 Shift 键，然后单击并拖动以平移。

### Markdown

虽然 Dash 通过 `dash_html_components` 库公开 HTML，但是用 HTML 编写副本可能很繁琐。要编写文本块，可以使用 `dash_core_components` 库中的 `Markdown` 组件。使用以下代码创建一个名为 `app.py` 的文件：

```python
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

import dash
import dash_core_components as dcc
import dash_html_components as html

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

markdown_text = '''
### Dash and Markdown

Dash apps can be written in Markdown.
Dash uses the [CommonMark](http://commonmark.org/)
specification of Markdown.
Check out their [60 Second Markdown Tutorial](http://commonmark.org/help/)
if this is your first introduction to Markdown!
'''

app.layout = html.Div([
    dcc.Markdown(children=markdown_text)
])

if __name__ == '__main__':
    app.run_server(debug=True)
```

### 核心组件

`dash_core_components` 包含一组更高级别的组件，例如下拉列表，图形，markdown 块等。

像所有 Dash 组件一样，对它们进行了完全声明式的描述。每个可配置的选项都可以用作组件的关键字参数。

在整个教程中，我们将看到许多这些组件。您可以在 [Dash Core 组件库](https://dash.plotly.com/dash-core-components) 中查看所有可用的组件。

<details><summary>以下是一些可用的组件。使用以下代码创建一个名为“app.py”的文件：</summary>

```python
# -*- coding: utf-8 -*-
# Run this app with `python app.py` and
# visit http://127.0.0.1:8050/ in your web browser.

import dash
import dash_core_components as dcc
import dash_html_components as html

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    html.Label('Dropdown'),
    dcc.Dropdown(
        options=[
            {'label': 'New York City', 'value': 'NYC'},
            {'label': u'Montréal', 'value': 'MTL'},
            {'label': 'San Francisco', 'value': 'SF'}
        ],
        value='MTL'
    ),

    html.Label('Multi-Select Dropdown'),
    dcc.Dropdown(
        options=[
            {'label': 'New York City', 'value': 'NYC'},
            {'label': u'Montréal', 'value': 'MTL'},
            {'label': 'San Francisco', 'value': 'SF'}
        ],
        value=['MTL', 'SF'],
        multi=True
    ),

    html.Label('Radio Items'),
    dcc.RadioItems(
        options=[
            {'label': 'New York City', 'value': 'NYC'},
            {'label': u'Montréal', 'value': 'MTL'},
            {'label': 'San Francisco', 'value': 'SF'}
        ],
        value='MTL'
    ),

    html.Label('Checkboxes'),
    dcc.Checklist(
        options=[
            {'label': 'New York City', 'value': 'NYC'},
            {'label': u'Montréal', 'value': 'MTL'},
            {'label': 'San Francisco', 'value': 'SF'}
        ],
        value=['MTL', 'SF']
    ),

    html.Label('Text Input'),
    dcc.Input(value='MTL', type='text'),

    html.Label('Slider'),
    dcc.Slider(
        min=0,
        max=9,
        marks={i: 'Label {}'.format(i) if i == 1 else str(i) for i in range(1, 6)},
        value=5,
    ),
], style={'columnCount': 2})

if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

### 回调 `help`

Dash 组件是声明性的：这些组件的每个可配置方面都在实例化期间设置为关键字参数。在任何组件上的 Python 控制台中回调 `help`，以了解有关组件及其可用参数的更多信息。

```python
>>> help(dcc.Dropdown)
class Dropdown(dash.development.base_component.Component)
|  A Dropdown component.
|  Dropdown is an interactive dropdown element for selecting one or more
|  items.
|  The values and labels of the dropdown items are specified in the `options`
|  property and the selected item(s) are specified with the `value` property.
|
|  Use a dropdown when you have many options (more than 5) or when you are
|  constrained for space. Otherwise, you can use RadioItems or a Checklist,
|  which have the benefit of showing the users all of the items at once.
|
|  Keyword arguments:
|  - id (string; optional)
|  - className (string; optional)
|  - disabled (boolean; optional): If true, the option is disabled
|  - multi (boolean; optional): If true, the user can select multiple values
|  - options (list; optional)
|  - placeholder (string; optional): The grey, default text shown when no option is selected
|  - value (string | list; optional): The value of the input. If `multi` is false (the default)
|  then value is just a string that corresponds to the values
|  provided in the `options` property. If `multi` is true, then
|  multiple values can be selected at once, and `value` is an
|  array of items with values corresponding to those in the
|  `options` prop.```
```

### 总结

Dash 应用程序的 `layout` 描述了该应用程序的外观。`layout` 是组件的分层树。`dash_html_components` 库提供了所有 HTML 标记的类，关键字参数描述了 HTML 属性，例如样式，`className` 和 `id`。`dash_core_components`库生成更高级别的组件，如控件和图形。

更多内容，请参阅：

- [`dash_core_components` 画廊](https://dash.plotly.com/dash-core-components)
- [`dash_html_components` 画廊](https://dash.plotly.com/dash-html-components)

Dash 教程的下一部分将介绍如何使这些应用程序具有交互性。

## Basic Dash Callbacks

在上一章中，我们了解到`app.layout`描述了应用程序的外观，并且是组件的分层树。`dash_html_components`库提供了所有 HTML 标记的类，关键字参数描述了 HTML 属性，例如样式，`className`和 `id`。`dash_core_components` 库生成更高级别的组件，如控件和图形。本章介绍如何使用回调函数制作 Dash 应用程序：每当输入组件的属性发生更改时 Dash 会自动调用的 Python 函数。

为了获得最佳的用户交互和图表加载性能，生产Dash应用程序应考虑 Dash Enterprise 的 [Job Queue](https://plotly.com/dash/job-queue), [HPC](https://plotly.com/dash/big-data-for-python), [Datashader](https://plotly.com/dash/big-data-for-python), 和 [horizontal scaling](https://plotly.com/dash/kubernetes) 功能。

让我们从一个交互式 Dash 应用程序的简单示例开始。

### 简单的交互式 Dash App

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    html.H6("Change the value in the text box to see callbacks in action!"),
    html.Div(["Input: ",
              dcc.Input(id='my-input', value='initial value', type='text')]),
    html.Br(),
    html.Div(id='my-output'),

])


@app.callback(
    Output(component_id='my-output', component_property='children'),
    Input(component_id='my-input', component_property='value')
)
def update_output_div(input_value):
    return 'Output: {}'.format(input_value)


if __name__ == '__main__':
    app.run_server(debug=True)
```

让我们分解这个例子：

1. 我们将应用程序接口的`"inputs"`和`"outputs"`声明性地描述为`@app.callback`装饰器的参数。

<details class="w3-pale-yellow"><summary>了解更多有关使用<code>@app.callback</code>装饰器的信息。</summary>
<ol type="a">
<li>通过编写此装饰器，我们告诉 Dash 每当 "input" 组件（文本框）的值更改时为我们调用此函数，以便更新页面上 "output" 组件的子级（HTML div）。</li>
<li>您可以为<code>@app.callback</code>装饰器包装的函数使用任何名称。约定是该名称描述了回调输出。</li>
<li>您可以为函数参数使用任何名称，但是必须像在常规 Python 函数中一样在回调函数中使用与定义时相同的名称。参数是位置性的：首先以与装饰器中相同的顺序给出 `Input` 项，然后给出任何 `State` 项。</li>
<li>当引用它作为<code>@app.callback</code>装饰器的输入或输出时，必须使用与给<code>app.layout</code>中的 Dash 组件相同的 ID。</li>
<li><code>@app.callback</code>装饰器需要直接位于回调函数声明的上方。如果装饰器和函数定义之间有空白行，则回调注册将不会成功。</li>
<li>如果您对装饰器语法的含义感到好奇，可以阅读此 <a href="https://stackoverflow.com/questions/739654/how-to-make-a-chain-of-function-decorators/1594484#1594484">StackOverflow 答案</a>，并通过阅读 <a href="https://www.python.org/dev/peps/pep-0318/#current-syntax">PEP 318-函数和方法的装饰器</a> 来了解有关装饰器的更多信息。</li>
</ol>
</details>

2. 在 Dash 中，我们应用程序的输入和输出只是特定组件的属性。在此示例中，我们的输入是ID为`"my-input"`的组件的`"value"`属性。我们的输出是 ID 为 `"my-output"` 的组件的`"children"`属性。
3. 每当输入属性更改时，回调装饰器包装的函数将自动被调用。Dash 为函数提供输入属性的新值作为输入参数，Dash 使用函数返回的值更新输出组件的属性。
4. `component_id` 和 `component_property` 关键字是可选的（每个对象只有两个参数）。为了清楚起见，它们包含在此示例中，但是为了简洁和易读起见，在本文档的其余部分中将省略它们。
5. 不要混淆`dash.dependencies.Input`对象和`dash_core_components.Input`对象。前者仅用于这些回调中，而后者是实际组件。
6. 注意，我们如何不为`layout`中的`my-output`组件的`children`属性设置值。Dash 应用程序启动时，它将自动使用输入组件的初始值调用所有回调，以填充输出组件的初始状态。在此示例中，如果您指定了类似 `html.Div(id='my-output', children='Hello world')` 的名称，则在应用启动时它将被覆盖。

这有点像使用 Microsoft Excel 进行编程：每当输入单元格发生更改时，依赖于该单元格的所有单元格都会自动更新。这称为“反应式编程”（"Reactive Programming"）。

还记得每个组件是如何通过其一组关键字参数进行完整描述的吗？这些属性现在很重要。借助 Dash 交互性，我们可以通过回调函数动态更新这些属性中的任何一个。通常，我们将更新组件的`children`以显示新文本，或者更新`dcc.Graph`组件的`figure`以显示新数据，但是我们还可以更新组件的`style`，甚至更新`dcc.Dropdown`组件可用的`options`！

让我们看一下另一个示例，其中`dcc.Slider`更新了`dcc.Graph`。

### Dash App Layout With Figure and Slider

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
import plotly.express as px

import pandas as pd

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    dcc.Graph(id='graph-with-slider'),
    dcc.Slider(
        id='year-slider',
        min=df['year'].min(),
        max=df['year'].max(),
        value=df['year'].min(),
        marks={str(year): str(year) for year in df['year'].unique()},
        step=None
    )
])


@app.callback(
    Output('graph-with-slider', 'figure'),
    Input('year-slider', 'value'))
def update_figure(selected_year):
    filtered_df = df[df.year == selected_year]

    fig = px.scatter(filtered_df, x="gdpPercap", y="lifeExp",
                     size="pop", color="continent", hover_name="country",
                     log_x=True, size_max=55)

    fig.update_layout(transition_duration=500)

    return fig


if __name__ == '__main__':
    app.run_server(debug=True)
```

在此示例中，`Slider` 的 `"value"` 属性是应用程序的输入，而应用程序的输出则是`Graph`的`"figure"`属性。每当`Slider`的值更改时，Dash 就会使用新值调用回调函数 `update_figure`。该函数使用此新值过滤数据框，构造一个`figure`对象，并将其返回给 Dash 应用程序。

此示例中有一些不错的模式：

1. 我们正在使用 [Pandas](http://pandas.pydata.org/) 库来导入和过滤内存中的数据集。
2. 我们在应用程序的开头加载数据帧：`df = pd.read_csv('...')`。此数据框`df`处于应用程序的全局状态，可以在回调函数中读取。
3. 将数据加载到内存中可能会很昂贵。通过在应用程序的开始而不是在回调函数内部加载查询数据，我们确保仅在应用程序服务器启动时执行此操作。当用户访问该应用程序或与该应用程序进行交互时，该数据（`df`）已经在内存中。如果可能，应在应用程序的全局范围内而不是在回调函数内完成昂贵的初始化（如下载或查询数据）。
4. 回调不会修改原始数据，它只是通过 `pandas` 过滤器进行过滤来创建数据帧的副本。这很重要：您的回调函数绝不要在变量范围之外进行变量的更改。如果您的回调修改了全局状态，则一个用户的会话可能会影响下一个用户的会话，并且当应用程序部署在多个进程或线程上时，这些修改将不会在各个会话之间共享。
5. 我们正在使用`layout.transition`打开`transitions`，以了解数据集如何随时间演变：`transitions`允许图表从一个状态平滑地更新到下一个状态，就好像它是动态的一样。

### Dash App With Multiple Inputs

在 Dash 中，任何`"Output"`可以具有多个`"Input"`组件。这是一个简单的示例，它将五个 Inputs（2个`Dropdown`组件，2个`RadioItems`组件和1个`Slider`组件的`value`属性）绑定到1个 Output 组件（`Graph`组件的`figure`属性）。请注意`app.callback`如何在第二个参数的列表内列出所有五个`dash.dependencies.Input`。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
import plotly.express as px

import pandas as pd

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

df = pd.read_csv('https://plotly.github.io/datasets/country_indicators.csv')

available_indicators = df['Indicator Name'].unique()

app.layout = html.Div([
    html.Div([

        html.Div([
            dcc.Dropdown(
                id='xaxis-column',
                options=[{'label': i, 'value': i} for i in available_indicators],
                value='Fertility rate, total (births per woman)'
            ),
            dcc.RadioItems(
                id='xaxis-type',
                options=[{'label': i, 'value': i} for i in ['Linear', 'Log']],
                value='Linear',
                labelStyle={'display': 'inline-block'}
            )
        ],
        style={'width': '48%', 'display': 'inline-block'}),

        html.Div([
            dcc.Dropdown(
                id='yaxis-column',
                options=[{'label': i, 'value': i} for i in available_indicators],
                value='Life expectancy at birth, total (years)'
            ),
            dcc.RadioItems(
                id='yaxis-type',
                options=[{'label': i, 'value': i} for i in ['Linear', 'Log']],
                value='Linear',
                labelStyle={'display': 'inline-block'}
            )
        ],style={'width': '48%', 'float': 'right', 'display': 'inline-block'})
    ]),

    dcc.Graph(id='indicator-graphic'),

    dcc.Slider(
        id='year--slider',
        min=df['Year'].min(),
        max=df['Year'].max(),
        value=df['Year'].max(),
        marks={str(year): str(year) for year in df['Year'].unique()},
        step=None
    )
])

@app.callback(
    Output('indicator-graphic', 'figure'),
    Input('xaxis-column', 'value'),
    Input('yaxis-column', 'value'),
    Input('xaxis-type', 'value'),
    Input('yaxis-type', 'value'),
    Input('year--slider', 'value'))
def update_graph(xaxis_column_name, yaxis_column_name,
                 xaxis_type, yaxis_type,
                 year_value):
    dff = df[df['Year'] == year_value]

    fig = px.scatter(x=dff[dff['Indicator Name'] == xaxis_column_name]['Value'],
                     y=dff[dff['Indicator Name'] == yaxis_column_name]['Value'],
                     hover_name=dff[dff['Indicator Name'] == yaxis_column_name]['Country Name'])

    fig.update_layout(margin={'l': 40, 'b': 40, 't': 10, 'r': 0}, hovermode='closest')

    fig.update_xaxes(title=xaxis_column_name,
                     type='linear' if xaxis_type == 'Linear' else 'log')

    fig.update_yaxes(title=yaxis_column_name,
                     type='linear' if yaxis_type == 'Linear' else 'log')

    return fig


if __name__ == '__main__':
    app.run_server(debug=True)
```

在此示例中，只要`Dropdown`，`Slider`或`RadioItems`组件的`value`属性发生更改，就会调用`update_grap`h函数。

按指定顺序，`update_graph`函数的输入参数是每个`Input`属性的新值或当前值。

即使一次仅更改一个`Input`（用户只能在给定的时刻更改单个`Dropdown`的值），Dash 仍会收集所有指定`Input`属性的当前状态并将其传递给您的函数。您的回调函数始终保证传递给应用程序代表状态。

让我们扩展示例以包括多个输出。

### Dash App With Multiple Outputs

到目前为止，我们编写的所有回调仅更新单个`Output`属性。我们也可以一次更新几个：将要更新的所有属性作为列表放置在装饰器中，并从回调中返回那么多项。如果两个输出依赖于相同的计算密集型中间结果（例如慢速数据库查询），则特别好。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    dcc.Input(
        id='num-multi',
        type='number',
        value=5
    ),
    html.Table([
        html.Tr([html.Td(['x', html.Sup(2)]), html.Td(id='square')]),
        html.Tr([html.Td(['x', html.Sup(3)]), html.Td(id='cube')]),
        html.Tr([html.Td([2, html.Sup('x')]), html.Td(id='twos')]),
        html.Tr([html.Td([3, html.Sup('x')]), html.Td(id='threes')]),
        html.Tr([html.Td(['x', html.Sup('x')]), html.Td(id='x^x')]),
    ]),
])


@app.callback(
    Output('square', 'children'),
    Output('cube', 'children'),
    Output('twos', 'children'),
    Output('threes', 'children'),
    Output('x^x', 'children'),
    Input('num-multi', 'value'))
def callback_a(x):
    return x**2, x**3, 2**x, 3**x, x**x


if __name__ == '__main__':
    app.run_server(debug=True)
```

提醒您：即使您可以合并输出，也不总是一个好主意：

- 如果输出依赖于某些而非全部相同的输入，则将它们分开可以避免不必要的更新。
- 如果它们具有相同的输入，但使用这些输入进行独立的计算，则将回调分开设置可以使它们并行运行。

### Dash App With Chained Callbacks

您也可以将输出和输入链接在一起：一个回调函数的输出可以是另一个回调函数的输入。

此模式可用于创建动态 UI，其中一个输入组件将更新下一个输入组件的可用选项。这是一个简单的例子。

```python
# -*- coding: utf-8 -*-
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

all_options = {
    'America': ['New York City', 'San Francisco', 'Cincinnati'],
    'Canada': [u'Montréal', 'Toronto', 'Ottawa']
}
app.layout = html.Div([
    dcc.RadioItems(
        id='countries-radio',
        options=[{'label': k, 'value': k} for k in all_options.keys()],
        value='America'
    ),

    html.Hr(),

    dcc.RadioItems(id='cities-radio'),

    html.Hr(),

    html.Div(id='display-selected-values')
])


@app.callback(
    Output('cities-radio', 'options'),
    Input('countries-radio', 'value'))
def set_cities_options(selected_country):
    return [{'label': i, 'value': i} for i in all_options[selected_country]]


@app.callback(
    Output('cities-radio', 'value'),
    Input('cities-radio', 'options'))
def set_cities_value(available_options):
    return available_options[0]['value']


@app.callback(
    Output('display-selected-values', 'children'),
    Input('countries-radio', 'value'),
    Input('cities-radio', 'value'))
def set_display_children(selected_country, selected_city):
    return u'{} is a city in {}'.format(
        selected_city, selected_country,
    )


if __name__ == '__main__':
    app.run_server(debug=True)
```

第一个回调根据第一个`RadioItems`组件中的选定值更新第二个`RadioItems`组件中的可用选项。

当`options`属性更改时，第二个回调将设置一个初始值：它将其设置为该`options`数组中的第一个值。

最后的回调显示每个组件的选定`value`。如果更改国家`RadioItems`组件的`value`，则 Dash 将等待，直到更新了城市组件的值，然后才调用最后的回调。这样可以防止以`"America"`和`"Montréal"`之类的不一致状态调用您的回调。

### Dash App With State

在某些情况下，您的应用程序中可能会有“表单”类型的模式。在这种情况下，您可能希望读取输入组件的值，但是仅当用户完成了以表格形式输入其所有信息时才可以。

将回调直接附加到输入值可以看起来像这样：

```python
# -*- coding: utf-8 -*-
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

external_stylesheets = ["https://codepen.io/chriddyp/pen/bWLwgP.css"]

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div(
    [
        dcc.Input(id="input-1", type="text", value="Montréal"),
        dcc.Input(id="input-2", type="text", value="Canada"),
        html.Div(id="number-output"),
    ]
)


@app.callback(
    Output("number-output", "children"),
    Input("input-1", "value"),
    Input("input-2", "value"),
)
def update_output(input1, input2):
    return u'Input 1 is "{}" and Input 2 is "{}"'.format(input1, input2)


if __name__ == "__main__":
    app.run_server(debug=True)
```

在此示例中，只要`dash.dependencies.Input`描述的任何属性发生更改，就会触发回调函数。在上面的输入中输入数据，自己尝试一下。

`dash.dependencies.State`允许您传递额外的值而无需触发回调。这是与上述相同的示例，但`dcc.Input`为`dash.dependencies.State`，按钮为`dash.dependencies.Input`。

```python
# -*- coding: utf-8 -*-
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    dcc.Input(id='input-1-state', type='text', value='Montréal'),
    dcc.Input(id='input-2-state', type='text', value='Canada'),
    html.Button(id='submit-button-state', n_clicks=0, children='Submit'),
    html.Div(id='output-state')
])


@app.callback(Output('output-state', 'children'),
              Input('submit-button-state', 'n_clicks'),
              State('input-1-state', 'value'),
              State('input-2-state', 'value'))
def update_output(n_clicks, input1, input2):
    return u'''
        The Button has been pressed {} times,
        Input 1 is "{}",
        and Input 2 is "{}"
    '''.format(n_clicks, input1, input2)


if __name__ == '__main__':
    app.run_server(debug=True)
```

在此示例中，在`dcc.Input`框中更改文本不会触发回调，但单击按钮将起作用。`dcc.Input`值的当前值仍会传递到回调中，即使它们不会触发回调函数本身。

请注意，我们通过侦听`html.Button`组件的`n_clicks`属性来触发回调。`n_clicks`是一个属性，每次单击该组件时该属性都会增强。它在`dash_html_components`库中的每个组件中都可用。

### 小结

我们已经介绍了 Dash 中回调的基础。Dash 应用程序是基于一组简单但功能强大的原则构建的：声明性 UI，可通过反应性（reactive）和功能性（functional ）Python 回调进行自定义。声明性组件的每个元素属性都可以通过回调进行更新，并且该属性的子集（例如`dcc.Dropdown`的`value`属性）可以由用户在界面中进行编辑。

## Interactive Visualizations

`dash_core_components`库包含一个名为`Graph`的组件。

`Graph`使用开源 [plotly.js](https://github.com/plotly/plotly.js) JavaScript 图形库呈现交互式数据可视化。Plotly.js 支持超过 35 种图表类型，并以矢量质量 SVG 和高性能 WebGL 呈现图表。

`dash_core_components.Graph`组件中的 `figure` 参数与 Plotly 的开源 Python 图形库 `plotly.py` 使用的图形参数相同。请查看 [plotly.py 文档和画廊](https://plotly.com/python) 以了解更多信息。

Dash 组件通过一组属性声明性地描述。所有这些属性都可以通过回调函数进行更新，但是这些属性的子集只能通过用户交互来更新，例如，当您单击`dcc.Dropdown`组件中的某个选项时，该组件的`value`属性将发生更改。

`dcc.Graph`组件具有四个可以通过用户交互更改的属性：`hoverData`，`clickData`，`selectedData`，`relayoutData`。当您将鼠标悬停在点上，单击点或选择图形中的点区域时，这些属性会更新。

为了获得最佳的用户交互和图表加载性能，生产环境的 Dash 应用程序应考虑 Dash Enterprise 的 [Job Queue](https://plotly.com/dash/job-queue), [HPC](https://plotly.com/dash/big-data-for-python), [Datashader](https://plotly.com/dash/big-data-for-python), 和 [horizontal scaling](https://plotly.com/dash/kubernetes)。

<details><summary>这是一个在屏幕上打印这些属性的简单示例。</summary>

```python
import json

import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
import plotly.express as px
import pandas as pd

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

styles = {
    'pre': {
        'border': 'thin lightgrey solid',
        'overflowX': 'scroll'
    }
}

df = pd.DataFrame({
    "x": [1,2,1,2],
    "y": [1,2,3,4],
    "customdata": [1,2,3,4],
    "fruit": ["apple", "apple", "orange", "orange"]
})

fig = px.scatter(df, x="x", y="y", color="fruit", custom_data=["customdata"])

fig.update_layout(clickmode='event+select')

fig.update_traces(marker_size=20)

app.layout = html.Div([
    dcc.Graph(
        id='basic-interactions',
        figure=fig
    ),

    html.Div(className='row', children=[
        html.Div([
            dcc.Markdown("""
                **Hover Data**

                Mouse over values in the graph.
            """),
            html.Pre(id='hover-data', style=styles['pre'])
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Click Data**

                Click on points in the graph.
            """),
            html.Pre(id='click-data', style=styles['pre']),
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Selection Data**

                Choose the lasso or rectangle tool in the graph's menu
                bar and then select points in the graph.

                Note that if `layout.clickmode = 'event+select'`, selection data also
                accumulates (or un-accumulates) selected data if you hold down the shift
                button while clicking.
            """),
            html.Pre(id='selected-data', style=styles['pre']),
        ], className='three columns'),

        html.Div([
            dcc.Markdown("""
                **Zoom and Relayout Data**

                Click and drag on the graph to zoom or click on the zoom
                buttons in the graph's menu bar.
                Clicking on legend items will also fire
                this event.
            """),
            html.Pre(id='relayout-data', style=styles['pre']),
        ], className='three columns')
    ])
])


@app.callback(
    Output('hover-data', 'children'),
    Input('basic-interactions', 'hoverData'))
def display_hover_data(hoverData):
    return json.dumps(hoverData, indent=2)


@app.callback(
    Output('click-data', 'children'),
    Input('basic-interactions', 'clickData'))
def display_click_data(clickData):
    return json.dumps(clickData, indent=2)


@app.callback(
    Output('selected-data', 'children'),
    Input('basic-interactions', 'selectedData'))
def display_selected_data(selectedData):
    return json.dumps(selectedData, indent=2)


@app.callback(
    Output('relayout-data', 'children'),
    Input('basic-interactions', 'relayoutData'))
def display_relayout_data(relayoutData):
    return json.dumps(relayoutData, indent=2)


if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

### Update Graphs on Hover

<details><summary>当我们将鼠标悬停在散点图中的点上时，让我们通过更新时间序列来更新上一章中的世界指标示例。</summary>

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
import pandas as pd
import plotly.express as px

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

df = pd.read_csv('https://plotly.github.io/datasets/country_indicators.csv')

available_indicators = df['Indicator Name'].unique()

app.layout = html.Div([
    html.Div([

        html.Div([
            dcc.Dropdown(
                id='crossfilter-xaxis-column',
                options=[{'label': i, 'value': i} for i in available_indicators],
                value='Fertility rate, total (births per woman)'
            ),
            dcc.RadioItems(
                id='crossfilter-xaxis-type',
                options=[{'label': i, 'value': i} for i in ['Linear', 'Log']],
                value='Linear',
                labelStyle={'display': 'inline-block'}
            )
        ],
        style={'width': '49%', 'display': 'inline-block'}),

        html.Div([
            dcc.Dropdown(
                id='crossfilter-yaxis-column',
                options=[{'label': i, 'value': i} for i in available_indicators],
                value='Life expectancy at birth, total (years)'
            ),
            dcc.RadioItems(
                id='crossfilter-yaxis-type',
                options=[{'label': i, 'value': i} for i in ['Linear', 'Log']],
                value='Linear',
                labelStyle={'display': 'inline-block'}
            )
        ], style={'width': '49%', 'float': 'right', 'display': 'inline-block'})
    ], style={
        'borderBottom': 'thin lightgrey solid',
        'backgroundColor': 'rgb(250, 250, 250)',
        'padding': '10px 5px'
    }),

    html.Div([
        dcc.Graph(
            id='crossfilter-indicator-scatter',
            hoverData={'points': [{'customdata': 'Japan'}]}
        )
    ], style={'width': '49%', 'display': 'inline-block', 'padding': '0 20'}),
    html.Div([
        dcc.Graph(id='x-time-series'),
        dcc.Graph(id='y-time-series'),
    ], style={'display': 'inline-block', 'width': '49%'}),

    html.Div(dcc.Slider(
        id='crossfilter-year--slider',
        min=df['Year'].min(),
        max=df['Year'].max(),
        value=df['Year'].max(),
        marks={str(year): str(year) for year in df['Year'].unique()},
        step=None
    ), style={'width': '49%', 'padding': '0px 20px 20px 20px'})
])


@app.callback(
    dash.dependencies.Output('crossfilter-indicator-scatter', 'figure'),
    [dash.dependencies.Input('crossfilter-xaxis-column', 'value'),
     dash.dependencies.Input('crossfilter-yaxis-column', 'value'),
     dash.dependencies.Input('crossfilter-xaxis-type', 'value'),
     dash.dependencies.Input('crossfilter-yaxis-type', 'value'),
     dash.dependencies.Input('crossfilter-year--slider', 'value')])
def update_graph(xaxis_column_name, yaxis_column_name,
                 xaxis_type, yaxis_type,
                 year_value):
    dff = df[df['Year'] == year_value]

    fig = px.scatter(x=dff[dff['Indicator Name'] == xaxis_column_name]['Value'],
            y=dff[dff['Indicator Name'] == yaxis_column_name]['Value'],
            hover_name=dff[dff['Indicator Name'] == yaxis_column_name]['Country Name']
            )

    fig.update_traces(customdata=dff[dff['Indicator Name'] == yaxis_column_name]['Country Name'])

    fig.update_xaxes(title=xaxis_column_name, type='linear' if xaxis_type == 'Linear' else 'log')

    fig.update_yaxes(title=yaxis_column_name, type='linear' if yaxis_type == 'Linear' else 'log')

    fig.update_layout(margin={'l': 40, 'b': 40, 't': 10, 'r': 0}, hovermode='closest')

    return fig


def create_time_series(dff, axis_type, title):

    fig = px.scatter(dff, x='Year', y='Value')

    fig.update_traces(mode='lines+markers')

    fig.update_xaxes(showgrid=False)

    fig.update_yaxes(type='linear' if axis_type == 'Linear' else 'log')

    fig.add_annotation(x=0, y=0.85, xanchor='left', yanchor='bottom',
                       xref='paper', yref='paper', showarrow=False, align='left',
                       bgcolor='rgba(255, 255, 255, 0.5)', text=title)

    fig.update_layout(height=225, margin={'l': 20, 'b': 30, 'r': 10, 't': 10})

    return fig


@app.callback(
    dash.dependencies.Output('x-time-series', 'figure'),
    [dash.dependencies.Input('crossfilter-indicator-scatter', 'hoverData'),
     dash.dependencies.Input('crossfilter-xaxis-column', 'value'),
     dash.dependencies.Input('crossfilter-xaxis-type', 'value')])
def update_y_timeseries(hoverData, xaxis_column_name, axis_type):
    country_name = hoverData['points'][0]['customdata']
    dff = df[df['Country Name'] == country_name]
    dff = dff[dff['Indicator Name'] == xaxis_column_name]
    title = '<b>{}</b><br>{}'.format(country_name, xaxis_column_name)
    return create_time_series(dff, axis_type, title)


@app.callback(
    dash.dependencies.Output('y-time-series', 'figure'),
    [dash.dependencies.Input('crossfilter-indicator-scatter', 'hoverData'),
     dash.dependencies.Input('crossfilter-yaxis-column', 'value'),
     dash.dependencies.Input('crossfilter-yaxis-type', 'value')])
def update_x_timeseries(hoverData, yaxis_column_name, axis_type):
    dff = df[df['Country Name'] == hoverData['points'][0]['customdata']]
    dff = dff[dff['Indicator Name'] == yaxis_column_name]
    return create_time_series(dff, axis_type, yaxis_column_name)


if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

尝试将鼠标悬停在左侧散点图中的点上。请注意，右侧的折线图是如何根据您悬停的点进行更新的。

### Generic Crossfilter Recipe

<details><summary>这是对六列数据集进行交叉过滤的更通用的示例。每个散点图的选择都会过滤基础数据集。</summary>

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
import numpy as np
import pandas as pd
from dash.dependencies import Input, Output
import plotly.express as px

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

# make a sample data frame with 6 columns
df = pd.DataFrame({"Col " + str(i+1): np.random.rand(30) for i in range(6)})

app.layout = html.Div([
    html.Div(
        dcc.Graph(id='g1', config={'displayModeBar': False}),
        className='four columns'
    ),
    html.Div(
        dcc.Graph(id='g2', config={'displayModeBar': False}),
        className='four columns'
        ),
    html.Div(
        dcc.Graph(id='g3', config={'displayModeBar': False}),
        className='four columns'
    )
], className='row')

def get_figure(df, x_col, y_col, selectedpoints, selectedpoints_local):

    if selectedpoints_local and selectedpoints_local['range']:
        ranges = selectedpoints_local['range']
        selection_bounds = {'x0': ranges['x'][0], 'x1': ranges['x'][1],
                            'y0': ranges['y'][0], 'y1': ranges['y'][1]}
    else:
        selection_bounds = {'x0': np.min(df[x_col]), 'x1': np.max(df[x_col]),
                            'y0': np.min(df[y_col]), 'y1': np.max(df[y_col])}

    # set which points are selected with the `selectedpoints` property
    # and style those points with the `selected` and `unselected`
    # attribute. see
    # https://medium.com/@plotlygraphs/notes-from-the-latest-plotly-js-release-b035a5b43e21
    # for an explanation
    fig = px.scatter(df, x=df[x_col], y=df[y_col], text=df.index)

    fig.update_traces(selectedpoints=selectedpoints,
                      customdata=df.index,
                      mode='markers+text', marker={ 'color': 'rgba(0, 116, 217, 0.7)', 'size': 20 }, unselected={'marker': { 'opacity': 0.3 }, 'textfont': { 'color': 'rgba(0, 0, 0, 0)' }})

    fig.update_layout(margin={'l': 20, 'r': 0, 'b': 15, 't': 5}, dragmode='select', hovermode=False)

    fig.add_shape(dict({'type': 'rect',
                        'line': { 'width': 1, 'dash': 'dot', 'color': 'darkgrey' }},
                       **selection_bounds))
    return fig

# this callback defines 3 figures
# as a function of the intersection of their 3 selections
@app.callback(
    Output('g1', 'figure'),
    Output('g2', 'figure'),
    Output('g3', 'figure'),
    Input('g1', 'selectedData'),
    Input('g2', 'selectedData'),
    Input('g3', 'selectedData')
)
def callback(selection1, selection2, selection3):
    selectedpoints = df.index
    for selected_data in [selection1, selection2, selection3]:
        if selected_data and selected_data['points']:
            selectedpoints = np.intersect1d(selectedpoints,
                [p['customdata'] for p in selected_data['points']])

    return [get_figure(df, "Col 1", "Col 2", selectedpoints, selection1),
            get_figure(df, "Col 3", "Col 4", selectedpoints, selection2),
            get_figure(df, "Col 5", "Col 6", selectedpoints, selection3)]


if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

尝试单击并拖动任何图以过滤不同区域。在每次选择时，将使用每个图的最新选定区域触发三个图形回调。根据所选点过滤熊猫数据帧，并以突出显示所选点的方式重新绘制图形，并将所选区域绘制为虚线矩形。

>顺便说一句，如果您发现自己过滤和可视化高维数据集，则应考虑检查平行坐标图表类型。

### Current Limitations

目前，图形交互存在一些限制。

- 当前无法自定义悬停交互或选择框的样式。这个问题正在 <https://github.com/plotly/plotly.js/issues/1847> 中处理。

这些交互式绘图功能可以做很多事情。如果需要帮助来探索用例，请在 [Dash 社区论坛](https://community.plotly.com/c/dash)中打开一个线程。

## Sharing Data Between Callbacks

[Dash Callbacks入门指南](https://dash.plotly.com/basic-callbacks) 中解释的 Dash 核心原则之一是，**Dash Callback 绝不能修改超出其作用域的变量**。修改任何`global`变量是不安全的。本章说明了原因，并提供了一些用于在回调之间共享状态的替代模式。

### 为什么要共享状态？

在某些应用中，您可能会有多个回调，这些回调依赖于昂贵的数据处理任务，例如进行 SQL 查询，运行模拟或下载数据。

您可以让一个回调运行该任务，然后将结果共享给其余的回调，而不是让每个回调都运行相同的昂贵任务。

现在，您可以为一个回调提供[多个输出](https://dash.plotly.com/basic-callbacks)，这种需求已得到一定程度的缓解。这样，该昂贵的任务可以一次完成，并立即在所有输出中使用。但是在某些情况下，这还是不理想的，例如，如果有简单的后续任务可以修改结果，例如单位转换。我们只需要将结果从华氏温度更改为摄氏温度，就不必重复大型数据库查询！

### 为什么`global`变量会破坏您的应用程序

Dash 旨在在多用户环境中工作，在该环境中，多个人可以同时查看该应用程序，并且将进行**独立的会话**。

如果您的应用程序使用修改后的`global`变量，则一个用户的会话可以将变量设置为一个值，这将影响下一个用户的会话。

Dash 还设计为能够与**多个 python worker** 一起运行，以便可以并行执行回调。这通常是使用类似的语法在 `gunicorn` 上完成的：

```shell
$ gunicorn --workers 4 app:server
```

（`app`指的是名为`app.py`的文件，而`server`指的是该文件中名为`server`的变量：`server = app.server`）。

当 Dash 应用程序跨多个工作程序运行时，它们的内存不会共享。这意味着，如果您在一个回调中修改全局变量，则该修改将不会应用于其余的工作程序。

<details><summary>这是一个带有回调的应用程序的草图，该回调会在其作用域之外修改数据。由于上述原因，这种类型的模式<em>无法可靠地运行</em>。</summary>

```python
df = pd.DataFrame({
    'a': [1, 2, 3],
    'b': [4, 1, 4],
    'c': ['x', 'y', 'z'],
})

app.layout = html.Div([
    dcc.Dropdown(
        id='dropdown',
        options=[{'label': i, 'value': i} for i in df['c'].unique()],
        value='a'
    ),
    html.Div(id='output'),
])

@app.callback(Output('output', 'children'),
              Input('dropdown', 'value'))
def update_output_1(value):
    # Here, `df` is an example of a variable that is
    # "outside the scope of this function".
    # *It is not safe to modify or reassign this variable
    #  inside this callback.*
    global df = df[df['c'] == value]  # do not do this, this is not safe!
    return len(df)
```
</details>

<details><summary>要解决此示例，只需将过滤器重新分配给回调中的新变量，或遵循本指南下一部分概述的策略之一。</summary>

```python
df = pd.DataFrame({
    'a': [1, 2, 3],
    'b': [4, 1, 4],
    'c': ['x', 'y', 'z'],
})

app.layout = html.Div([
    dcc.Dropdown(
        id='dropdown',
        options=[{'label': i, 'value': i} for i in df['c'].unique()],
        value='a'
    ),
    html.Div(id='output'),
])

@app.callback(Output('output', 'children'),
              Input('dropdown', 'value'))
def update_output_1(value):
    # Safely reassign the filter to a new variable
    filtered_df = df[df['c'] == value]
    return len(filtered_df)
```
</details>

### 回调之间共享数据

为了在多个 Python 进程之间安全地共享数据，我们需要将数据存储在每个进程可访问的位置。

有三个主要位置可存储此数据：

1. 在用户的浏览器会话中

2. 在磁盘上（例如在文件或新数据库上）

3. 在与 Redis 一样的共享内存空间中

下面的三个示例说明了这些方法。

#### 示例1-在具有隐藏 Div 的浏览器中存储数据

要在用户浏览器的会话中保存数据，请执行以下操作：

- 通过使用 <https://community.plotly.com/t/sharing-a-dataframe-between-plots/6173> 中解释的方法将数据保存为 Dash 前端存储的一部分来实现
- 数据必须转换为 JSON 之类的字符串才能存储和传输
- 以这种方式缓存的数据将仅在用户的当前会话中可用。
    - 如果您打开新的浏览器，则应用程序的回调将始终计算数据。数据仅在会话内的回调之间进行缓存和传输。
    - 因此，与缓存不同，此方法不会增加应用程序的内存占用。
    - 网络传输可能会产生成本。如果您在回调之间共享 10MB 数据，则该数据将在每个回调之间通过网络传输。
    - 如果网络成本太高，请预先计算聚合并进行传输。您的应用可能不会显示 10MB 的数据，而只会显示其子集或聚合。

<details><summary>此示例概述了如何在一个回调中执行昂贵的数据处理步骤，将输出序列化为 JSON 并将其提供为其他回调的输入。本示例使用标准的 Dash 回调并将 JSON 格式的数据存储在应用程序中的隐藏 div 中。</summary>

```python
global_df = pd.read_csv('...')
app.layout = html.Div([
    dcc.Graph(id='graph'),
    html.Table(id='table'),
    dcc.Dropdown(id='dropdown'),

    # Hidden div inside the app that stores the intermediate value
    html.Div(id='intermediate-value', style={'display': 'none'})
])

@app.callback(Output('intermediate-value', 'children'), Input('dropdown', 'value'))
def clean_data(value):
     # some expensive clean data step
     cleaned_df = your_expensive_clean_or_compute_step(value)

     # more generally, this line would be
     # json.dumps(cleaned_df)
     return cleaned_df.to_json(date_format='iso', orient='split')

@app.callback(Output('graph', 'figure'), Input('intermediate-value', 'children'))
def update_graph(jsonified_cleaned_data):

    # more generally, this line would be
    # json.loads(jsonified_cleaned_data)
    dff = pd.read_json(jsonified_cleaned_data, orient='split')

    figure = create_figure(dff)
    return figure

@app.callback(Output('table', 'children'), Input('intermediate-value', 'children'))
def update_table(jsonified_cleaned_data):
    dff = pd.read_json(jsonified_cleaned_data, orient='split')
    table = create_table(dff)
    return table
```
</details>

#### 示例2- Computing Aggregations Upfront

如果数据很大，则通过网络发送计算的数据可能会很昂贵。在某些情况下，序列化此数据和 JSON 可能也很昂贵。

在许多情况下，您的应用只会显示计算或过滤后的数据的子集或汇总。在这些情况下，您可以在数据处理回调中预先计算聚合，然后将这些聚合传输到其余的回调中。

<details><summary>这是一个简单的示例，说明如何将经过筛选或聚合的数据传输到多个回调。</summary>

```python
@app.callback(
    Output('intermediate-value', 'children'),
    Input('dropdown', 'value'))
def clean_data(value):
     # an expensive query step
     cleaned_df = your_expensive_clean_or_compute_step(value)

     # a few filter steps that compute the data
     # as it's needed in the future callbacks
     df_1 = cleaned_df[cleaned_df['fruit'] == 'apples']
     df_2 = cleaned_df[cleaned_df['fruit'] == 'oranges']
     df_3 = cleaned_df[cleaned_df['fruit'] == 'figs']

     datasets = {
         'df_1': df_1.to_json(orient='split', date_format='iso'),
         'df_2': df_2.to_json(orient='split', date_format='iso'),
         'df_3': df_3.to_json(orient='split', date_format='iso'),
     }

     return json.dumps(datasets)

@app.callback(
    Output('graph', 'figure'),
    Input('intermediate-value', 'children'))
def update_graph_1(jsonified_cleaned_data):
    datasets = json.loads(jsonified_cleaned_data)
    dff = pd.read_json(datasets['df_1'], orient='split')
    figure = create_figure_1(dff)
    return figure

@app.callback(
    Output('graph', 'figure'),
    Input('intermediate-value', 'children'))
def update_graph_2(jsonified_cleaned_data):
    datasets = json.loads(jsonified_cleaned_data)
    dff = pd.read_json(datasets['df_2'], orient='split')
    figure = create_figure_2(dff)
    return figure

@app.callback(
    Output('graph', 'figure'),
    Input('intermediate-value', 'children'))
def update_graph_3(jsonified_cleaned_data):
    datasets = json.loads(jsonified_cleaned_data)
    dff = pd.read_json(datasets['df_3'], orient='split')
    figure = create_figure_3(dff)
    return figure
```
</details>

#### 示例3- Caching and Signaling

这个例子：

- 通过 Flask-Cache 使用 Redis 来存储“全局变量”。该数据通过一个函数访问，该函数的输出由其输入参数缓存并键入键。
- 完成昂贵的计算后，使用隐藏的 div 解决方案将信号发送到其他回调。
- 请注意，除了 Redis 之外，您还可以将其保存到文件系统中。有关更多详细信息，请参见 <https://flask-caching.readthedocs.io/en/latest/>。
- 这种“signaling”很酷，因为它允许昂贵的计算仅占用一个进程。没有这种类型的信令（signaling），每个回调可能最终并行计算昂贵的计算，从而锁定四个进程而不是一个。

该方法的优点还在于，将来的会话可以使用预先计算的值。这对于输入量较少的应用程序将非常有效。

这是此示例的样子。注意事项：

- 我使用`time.sleep(5)`模拟了一个昂贵的过程。
- 应用加载后，需要五秒钟的时间来渲染所有四个图形。
- 初始计算仅阻止一个过程。
- 计算完成后，将发送信号并并行执行四个回调以渲染图形。这些回调中的每一个都从“全局存储”：Redis 或文件系统缓存中检索数据。
- 我在`app.run_server`中设置了`processs = 6`，以便可以并行执行多个回调。在生产中，这可以通过`$ gunicorn --workers 6 --threads 2 app:server`来完成。
- 如果过去已经选择过，则在下拉列表中选择一个值将花费不到五秒钟的时间。这是因为该值是从缓存中提取的。
- 同样，重新加载页面或在新窗口中打开应用程序也很快，因为已经计算了初始状态和初始昂贵的计算量。

<details><summary>这是此示例在代码中的样子：</summary>

```python
import os
import copy
import time
import datetime

import dash
import dash_core_components as dcc
import dash_html_components as html
import numpy as np
import pandas as pd
from dash.dependencies import Input, Output
from flask_caching import Cache


external_stylesheets = [
    # Dash CSS
    'https://codepen.io/chriddyp/pen/bWLwgP.css',
    # Loading screen CSS
    'https://codepen.io/chriddyp/pen/brPBPO.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)
CACHE_CONFIG = {
    # try 'filesystem' if you don't want to setup redis
    'CACHE_TYPE': 'redis',
    'CACHE_REDIS_URL': os.environ.get('REDIS_URL', 'redis://localhost:6379')
}
cache = Cache()
cache.init_app(app.server, config=CACHE_CONFIG)

N = 100

df = pd.DataFrame({
    'category': (
        (['apples'] * 5 * N) +
        (['oranges'] * 10 * N) +
        (['figs'] * 20 * N) +
        (['pineapples'] * 15 * N)
    )
})
df['x'] = np.random.randn(len(df['category']))
df['y'] = np.random.randn(len(df['category']))

app.layout = html.Div([
    dcc.Dropdown(
        id='dropdown',
        options=[{'label': i, 'value': i} for i in df['category'].unique()],
        value='apples'
    ),
    html.Div([
        html.Div(dcc.Graph(id='graph-1'), className="six columns"),
        html.Div(dcc.Graph(id='graph-2'), className="six columns"),
    ], className="row"),
    html.Div([
        html.Div(dcc.Graph(id='graph-3'), className="six columns"),
        html.Div(dcc.Graph(id='graph-4'), className="six columns"),
    ], className="row"),

    # hidden signal value
    html.Div(id='signal', style={'display': 'none'})
])


# perform expensive computations in this "global store"
# these computations are cached in a globally available
# redis memory store which is available across processes
# and for all time.
@cache.memoize()
def global_store(value):
    # simulate expensive query
    print('Computing value with {}'.format(value))
    time.sleep(5)
    return df[df['category'] == value]


def generate_figure(value, figure):
    fig = copy.deepcopy(figure)
    filtered_dataframe = global_store(value)
    fig['data'][0]['x'] = filtered_dataframe['x']
    fig['data'][0]['y'] = filtered_dataframe['y']
    fig['layout'] = {'margin': {'l': 20, 'r': 10, 'b': 20, 't': 10}}
    return fig


@app.callback(Output('signal', 'children'), Input('dropdown', 'value'))
def compute_value(value):
    # compute value and send a signal when done
    global_store(value)
    return value


@app.callback(Output('graph-1', 'figure'), Input('signal', 'children'))
def update_graph_1(value):
    # generate_figure gets data from `global_store`.
    # the data in `global_store` has already been computed
    # by the `compute_value` callback and the result is stored
    # in the global redis cached
    return generate_figure(value, {
        'data': [{
            'type': 'scatter',
            'mode': 'markers',
            'marker': {
                'opacity': 0.5,
                'size': 14,
                'line': {'border': 'thin darkgrey solid'}
            }
        }]
    })


@app.callback(Output('graph-2', 'figure'), Input('signal', 'children'))
def update_graph_2(value):
    return generate_figure(value, {
        'data': [{
            'type': 'scatter',
            'mode': 'lines',
            'line': {'shape': 'spline', 'width': 0.5},
        }]
    })


@app.callback(Output('graph-3', 'figure'), Input('signal', 'children'))
def update_graph_3(value):
    return generate_figure(value, {
        'data': [{
            'type': 'histogram2d',
        }]
    })


@app.callback(Output('graph-4', 'figure'), Input('signal', 'children'))
def update_graph_4(value):
    return generate_figure(value, {
        'data': [{
            'type': 'histogram2dcontour',
        }]
    })


if __name__ == '__main__':
    app.run_server(debug=True, processes=6)
```
</details>

#### 示例4-服务器上基于用户的会话数据

前面的示例在文件系统上缓存了计算，并且所有用户都可以访问这些计算。

在某些情况下，您想使数据与用户会话隔离：一个用户的派生数据不应更新下一个用户的派生数据。一种实现方法是将数据保存在隐藏的 Div中，如第一个示例所示。

执行此操作的另一种方法是使用会话ID将数据保存在文件系统缓存中，然后使用该会话 ID 引用数据。因为数据保存在服务器上而不是通过网络传输，所以此方法通常比“hidden div”方法更快。

该示例最初是在 [Dash 社区论坛主题](https://community.plotly.com/t/capture-window-tab-closing-event/7375/2?u=chriddyp)中讨论的。

这个例子：

- 使用`flask_caching`文件系统缓存来缓存数据。您还可以保存到内存数据库，例如 Redis。
- 将数据序列化为 JSON。
    - 如果您使用的是 Pandas，请考虑使用 Apache Arrow 进行序列化。[社区话题](https://community.plotly.com/t/fast-way-to-share-data-between-callbacks/8024/2)
- 将会话数据最多保存为预期的并发用户数。这样可以防止缓存中的数据过多。
- 通过将隐藏的随机字符串嵌入到应用程序的布局中并在每次页面加载时提供唯一的布局来创建唯一的会话ID。

>注意：与将数据发送到客户端的所有示例一样，请注意，这些会话不一定是安全的或加密的。这些会话ID可能容易受到会话固定样式攻击。

<details><summary>这是此示例在代码中的样子：</summary>

```python
import dash
from dash.dependencies import Input, Output
import dash_core_components as dcc
import dash_html_components as html
import datetime
from flask_caching import Cache
import os
import pandas as pd
import time
import uuid

external_stylesheets = [
    # Dash CSS
    'https://codepen.io/chriddyp/pen/bWLwgP.css',
    # Loading screen CSS
    'https://codepen.io/chriddyp/pen/brPBPO.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)
cache = Cache(app.server, config={
    'CACHE_TYPE': 'redis',
    # Note that filesystem cache doesn't work on systems with ephemeral
    # filesystems like Heroku.
    'CACHE_TYPE': 'filesystem',
    'CACHE_DIR': 'cache-directory',

    # should be equal to maximum number of users on the app at a single time
    # higher numbers will store more data in the filesystem / redis cache
    'CACHE_THRESHOLD': 200
})


def get_dataframe(session_id):
    @cache.memoize()
    def query_and_serialize_data(session_id):
        # expensive or user/session-unique data processing step goes here

        # simulate a user/session-unique data processing step by generating
        # data that is dependent on time
        now = datetime.datetime.now()

        # simulate an expensive data processing task by sleeping
        time.sleep(5)

        df = pd.DataFrame({
            'time': [
                str(now - datetime.timedelta(seconds=15)),
                str(now - datetime.timedelta(seconds=10)),
                str(now - datetime.timedelta(seconds=5)),
                str(now)
            ],
            'values': ['a', 'b', 'a', 'c']
        })
        return df.to_json()

    return pd.read_json(query_and_serialize_data(session_id))


def serve_layout():
    session_id = str(uuid.uuid4())

    return html.Div([
        html.Div(session_id, id='session-id', style={'display': 'none'}),
        html.Button('Get data', id='get-data-button'),
        html.Div(id='output-1'),
        html.Div(id='output-2')
    ])


app.layout = serve_layout


@app.callback(Output('output-1', 'children'),
              Input('get-data-button', 'n_clicks'),
              Input('session-id', 'children'))
def display_value_1(value, session_id):
    df = get_dataframe(session_id)
    return html.Div([
        'Output 1 - Button has been clicked {} times'.format(value),
        html.Pre(df.to_csv())
    ])


@app.callback(Output('output-2', 'children'),
              Input('get-data-button', 'n_clicks'),
              Input('session-id', 'children'))
def display_value_2(value, session_id):
    df = get_dataframe(session_id)
    return html.Div([
        'Output 2 - Button has been clicked {} times'.format(value),
        html.Pre(df.to_csv())
    ])


if __name__ == '__main__':
    app.run_server(debug=True)
```
</details>

在此示例中，需要注意三件事：

- 检索数据时，数据帧的时间戳不会更新。此数据被缓存为用户会话的一部分。
- 最初检索数据需要五秒钟，但是由于已缓存数据，因此连续的查询是即时的。
- 第二个会话显示的数据与第一个会话不同：在回调之间共享的数据被隔离到各个用户会话。

问题？在 [Dash 社区论坛](https://community.plotly.com/c/dash)上讨论这些示例。

## FAQs

问：如何自定义 Dash 应用程序的外观？
答：Dash 应用程序在浏览器中呈现为符合现代标准的 Web 应用程序。这意味着您可以像使用标准 HTML 一样使用 CSS 来设置 Dash 应用的样式。

所有`dash-html-components`都通过`style`属性支持内联 CSS 样式。通过定位组件的 ID 或类名称，外部 CSS 样式表还可用于设置`dash-html-components`和`dash-core-components`的样式。`dash-html-components`和`dash-core-components`都接受属性`className`，该属性对应于 HTML 元素属性类。

[Dash HTML Components](https://dash.plotly.com/dash-html-components) 中的“Dash HTML组件”部分说明了如何为`dash-html-components`提供内联样式和 CSS 类名，您可以使用 CSS 样式表作为目标。Dash指南中的[Adding CSS & JS and Overriding the Page-Load Template](https://dash.plotly.com/external-resources)部分说明了如何将自己的样式表链接到 Dash 应用。

----

问：如何将 JavaScript 添加到 Dash 应用程序？
答：您可以将自己的脚本添加到 Dash 应用程序中，就像将 JavaScript 文件添加到 HTML 文档中一样。请参阅《Dash 指南》中的[Adding CSS & JS and Overriding the Page-Load Template](https://dash.plotly.com/external-resources)部分。

----

问：我可以制作包含多个页面的 Dash 应用程序吗？
答：是的！Dash 支持多页应用程序。请参阅《Dash用户指南》中的[Multi-Page Apps and URL Support](https://dash.plotly.com/urls)部分。

----

问：如何将 Dash 应用程序组织成多个文件？
答：可以在《Dash用户指南》的[Multi-Page Apps and URL Support](https://dash.plotly.com/urls)部分中找到执行此操作的策略。

---

问：如何确定哪个输入已更改？
答：请参阅[Advanced Callbacks](https://dash.plotly.com/advanced-callbacks)部分中的`dash.callback_context`。

---

问：我可以将 Jinja2 模板与 Dash 一起使用吗？

答：Jinja2 模板在作为 HTML 页面发送到客户端之前，先在服务器上呈现（通常在 Flask 应用程序中）。另一方面，Dash 应用程序是使用 React 在客户端上呈现的。这使这些在浏览器中显示 HTML 的方法截然不同，这意味着这两种方法无法直接组合。但是，您可以将 Dash 应用程序与现有的 Flask 应用程序集成在一起，以便 Flask 应用程序可以处理某些 URL 端点，而 Dash 应用程序位于特定的 URL 端点。

---


问：我可以将 jQuery 与 Dash 一起使用吗？

答：在大多数情况下，您不能这样做。Dash 使用 React 在客户端浏览器上呈现您的应用程序。React 与 jQuery的根本不同之处在于，它利用虚拟 DOM（文档对象模型）来管理页面呈现。由于 jQuery 不会讲 React 的虚拟DOM，因此您无法使用 jQuery 的任何 DOM 操作工具来更改页面布局，这经常就是为什么要使用 jQuery 的原因。但是，您可以使用 jQuery 功能中不接触 DOM 的部分，例如注册事件侦听器以使击键导致页面重定向。

通常，如果您希望在应用程序中添加自定义客户端行为，我们建议将该行为封装在[自定义 Dash 组件](https://dash.plotly.com/plugins)中。

----

问：那些很棒的撤消和重做按钮去哪了？

答：好的，主要是我们遇到了相反的问题：[How do I get rid of the undo/redo buttons](https://community.plotly.com/t/is-it-possible-to-hide-the-floating-toolbar/4911/10)。尽管从技术角度来看此功能很简洁，但大多数人在实践中并不认为它有价值。从 Dash 1.0 开始，默认情况下会删除按钮，不需要任何怪异的 CSS 技巧。如果您想让他们回来，请使用 `show_undo_redo`：

```python
app = Dash(show_undo_redo=True)
```

---

问：我还有其他问题！我在哪里可以问他们？
答：[Dash 社区论坛](https://community.plotly.com/c/dash)上挤满了讨论 Dash 主题，互相帮助的人以及共享Dash创作的人。跳过并加入讨论。