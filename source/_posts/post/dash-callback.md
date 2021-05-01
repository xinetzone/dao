---
title: Dash 回调
lang: zh-CN
tags: Dash
categories: 教程
abbrlink: 8c8896d7
date: 2021-03-22 10:23:12
description:
updated:
---

## 高级回调

### 使用 PreventUpdate 捕获错误

在某些情况下，您不想更新回调输出。您可以通过在回调函数中引发 PreventUpdate 异常来实现此目的。

```python
import dash
import dash_html_components as html
from dash.dependencies import Input, Output
from dash.exceptions import PreventUpdate

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    html.Button('Click here to see the content', id='show-secret'),
    html.Div(id='body-div')
])

@app.callback(
    Output(component_id='body-div', component_property='children'),
    Input(component_id='show-secret', component_property='n_clicks')
)
def update_output(n_clicks):
    if n_clicks is None:
        raise PreventUpdate
    else:
        return "Elephants are the only animal that can't jump"

if __name__ == '__main__':
    app.run_server(debug=True)
```

### 使用`dash.no_update`显示错误

此示例说明如何使用`dash.no_update`来部分更新输出，从而在保留先前输入的同时显示错误。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    html.P('Enter a composite number to see its prime factors'),
    dcc.Input(id='num', type='number', debounce=True, min=1, step=1),
    html.P(id='err', style={'color': 'red'}),
    html.P(id='out')
])

@app.callback(
    Output('out', 'children'),
    Output('err', 'children'),
    Input('num', 'value')
)
def show_factors(num):
    if num is None:
        # PreventUpdate prevents ALL outputs updating
        raise dash.exceptions.PreventUpdate

    factors = prime_factors(num)
    if len(factors) == 1:
        # dash.no_update prevents any single output updating
        # (note: it's OK to use for a single-output callback too)
        return dash.no_update, '{} is prime!'.format(num)

    return '{} is {}'.format(num, ' * '.join(str(n) for n in factors)), ''

def prime_factors(num):
    n, i, out = num, 2, []
    while i * i <= n:
        if n % i == 0:
            n = int(n / i)
            out.append(i)
        else:
            i += 1 if i == 2 else 2
    out.append(n)
    return out

if __name__ == '__main__':
    app.run_server(debug=True)
```

### 确定使用`dash.callback_context`触发了哪个输入

除了`n_clicks`之类的事件属性会在事件发生（在这种情况下为单击）时发生更改之外，还有一个全局变量`dash.callback_context`，仅在回调内部可用。它具有以下特性：

- `triggered`：已更改属性的列表。在初始加载时，此属性将为空，除非 `Input prop` 从另一个初始回调获得其值。在用户操作之后，它是一个长度为1的列表，除非单个组件的两个属性同时更新，例如值和时间戳或事件计数器。
- `inputs`和`states`：允许您通过`id`和`prop`而不是通过 args 函数来访问回调参数。这些具有字典的形式`{ 'component_id.prop_name': value }`

这是如何完成此操作的示例：

```python
import json

import dash
import dash_html_components as html
from dash.dependencies import Input, Output

app = dash.Dash(__name__)

app.layout = html.Div([
    html.Button('Button 1', id='btn-1'),
    html.Button('Button 2', id='btn-2'),
    html.Button('Button 3', id='btn-3'),
    html.Div(id='container')
])


@app.callback(Output('container', 'children'),
              Input('btn-1', 'n_clicks'),
              Input('btn-2', 'n_clicks'),
              Input('btn-3', 'n_clicks'))
def display(btn1, btn2, btn3):
    ctx = dash.callback_context

    if not ctx.triggered:
        button_id = 'No clicks yet'
    else:
        button_id = ctx.triggered[0]['prop_id'].split('.')[0]

    ctx_msg = json.dumps({
        'states': ctx.states,
        'triggered': ctx.triggered,
        'inputs': ctx.inputs
    }, indent=2)

    return html.Div([
        html.Table([
            html.Tr([html.Th('Button 1'),
                     html.Th('Button 2'),
                     html.Th('Button 3'),
                     html.Th('Most Recent Click')]),
            html.Tr([html.Td(btn1 or 0),
                     html.Td(btn2 or 0),
                     html.Td(btn3 or 0),
                     html.Td(button_id)])
        ]),
        html.Pre(ctx_msg)
    ])


if __name__ == '__main__':
    app.run_server(debug=True)
```

### 传统行为：使用时间戳记

在 v0.38.0 之前的版本中，您需要比较诸如 `n_clicks_timestamp` 之类的时间戳属性以查找最近的点击。虽然`*_timestamp`的现有用法现在仍可以继续工作，但是不建议使用此方法，并且可以在以后的更新中将其删除。一个例外是`dcc.Store`的`Modifyed_timestamp`，它可以安全使用，并且不建议使用。

### 通过 memoization 提高性能

借助 Memoization，您可以通过存储函数调用的结果来绕开长时间的计算。

为了更好地理解记忆的工作原理，让我们从一个简单的示例开始。

```python
import time
import functools32

@functools32.lru_cache(maxsize=32)
def slow_function(input):
    time.sleep(10)
    return 'Input was {}'.format(input)
```

第一次调用`slow_function('test')`将需要 10 秒钟。第二次使用相同的参数调用它几乎不需要花费时间，因为先前计算的结果已保存在内存中并可以重复使用。

Dash 文档的 [Performance](https://dash.plotly.com/performance) 部分深入研究了利用多个进程和线程以及备注来进一步提高性能的情况。

### 何时执行回调？

本节介绍了仪表板渲染器前端客户端可以向 Dash 后端服务器（或客户端回调代码）发出请求以执行回调函数的情况。

#### 首次加载 Dash 应用程序时

首次加载应用程序时，Dash 应用程序中的所有回调均以其输入的初始值执行。这称为回调的“初始调用”。若要了解如何抑制此行为，请参阅 Dash 回调的 [`prevent_initial_call`](https://dash.plotly.com/advanced-callbacks#prevent-callbacks-from-being-executed-on-initial-load)属性的文档。

重要的是要注意，当 Dash 应用程序最初由`dash-renderer`前端客户端加载到 Web 浏览器中时，将递归检查其整个回调链。

这使得`dash-renderer`可以预测执行回调的顺序，因为当回调的输入是尚未触发的其他回调的输出时，它们将被阻塞。为了取消阻止执行这些回调，必须先执行其输入立即可用的回调。此过程通过确保仅在所有回调的输入都达到其最终值时才请求执行回调，来帮助`dash-renderer`最大程度地减少其使用的时间和精力，并避免不必要地重画页面。

检查以下 Dash 应用程序：

```python
import dash
from dash.dependencies import Input, Output
import dash_html_components as html

app = dash.Dash()
app.layout = html.Div(
    [
        html.Button("execute callback", id="button_1"),
        html.Div(children="callback not executed", id="first_output_1"),
        html.Div(children="callback not executed", id="second_output_1"),
    ]
)


@app.callback(
    Output("first_output_1", "children"),
    Output("second_output_1", "children"),
    Input("button_1", "n_clicks")
)
def change_text(n_clicks):
    return ["n_clicks is " + str(n_clicks), "n_clicks is " + str(n_clicks)]

if __name__ == '__main__':
    app.run_server(debug=True)
```

请注意，当完成该应用程序的 Web 浏览器加载并准备好与用户进行交互时，`html.Div`组件不会像在应用程序的`layout`中声明的那样说“未执行回调”，而是`n_clicks`为`None`，这是由于 `change_text()` 回调正在执行。这是因为回调的“初始调用”是使用值为`None`的`n_clicks`发生的。

#### 用户交互的直接结果

通常，回调是用户交互的直接结果，例如单击按钮或在下拉菜单中选择一项。当发生这种交互时，Dash 组件将其新值传递给`dash-renderer`前端客户端，该客户端再请求 Dash 服务器执行将新更改的值作为输入的任何回调函数。

如果 Dash 应用程序具有多个回调，则`dash-renderer`会根据是否可以使用新更改的输入立即执行回调来请求执行回调。如果多个输入同时更改，则将请求全部执行它们。

这些请求是以同步还是异步方式执行取决于Dash后端服务器的特定设置。如果它在多线程环境中运行，那么所有回调都可以同时执行，并且它们将根据执行速度返回值。但是，在单线程环境中，回调将按照服务器接收到的顺序一次执行一次。

在上面的示例应用程序中，单击按钮将导致执行回调。

#### 用户交互的间接结果

当用户与组件进行交互时，生成的回调可能会有输出，这些输出本身就是其他回调的输入。`dash-renderer`将阻止此类回调的执行，直到执行了其输出为其输入的回调为止。

请使用以下 Dash 应用程序：

```python
import dash
from dash.dependencies import Input, Output
import dash_html_components as html
from datetime import datetime
import time

app = dash.Dash()
app.layout = html.Div(
    [
        html.Button("execute fast callback", id="button_3"),
        html.Button("execute slow callback", id="button_4"),
        html.Div(children="callback not executed", id="first_output_3"),
        html.Div(children="callback not executed", id="second_output_3"),
        html.Div(children="callback not executed", id="third_output_3"),
    ]
)


@app.callback(
    Output("first_output_3", "children"),
    Input("button_3", "n_clicks"))
def first_callback(n):
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return "in the fast callback it is " + current_time


@app.callback(
    Output("second_output_3", "children"), Input("button_4", "n_clicks"))
def second_callback(n):
    time.sleep(5)
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return "in the slow callback it is " + current_time


@app.callback(
    Output("third_output_3", "children"),
    Input("first_output_3", "children"),
    Input("second_output_3", "children"))
def third_callback(n, m):
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return "in the third callback it is " + current_time


if __name__ == '__main__':
    app.run_server(debug=True)
```

上面的 Dash 应用演示了回调如何链接在一起。请注意，如果您先单击"execute slow callback"，然后单击`"execute fast callback"`，则直到慢速回调完成执行后，才会执行第三个回调。这是因为第三个回调将第二个回调的输出作为其输入，这使`dash-renderer`知道应将其执行延迟到第二个回调完成之后。

#### 将 Dash 组件添加到`layout`时

回调可能会将新的 Dash 组件插入 Dash 应用程序的`layout`中。如果这些新组件本身是其他回调函数的输入，则它们在 Dash 应用程序`layout`中的出现将触发这些回调函数被执行。

在这种情况下，可能会发出多个请求以执行相同的回调函数。如果已经请求了有问题的回调，并且在将新组件（也就是其输入）添加到`layout`之前返回了其输出，则会发生这种情况。

### 防止在初始组件渲染时执行回调

您可以使用`prevent_initial_call`属性来防止在其输入最初出现在 Dash 应用程序的`layout`中时触发回调。

此属性在最初加载 Dash 应用程序的`layout`时适用，并且在触发回调时将新组件引入到`layout`中时也适用。

```python
import dash
from dash.dependencies import Input, Output
import dash_html_components as html
from datetime import datetime
import time

app = dash.Dash()

app.layout = html.Div(
    [
        html.Button("execute callbacks", id="button_2"),
        html.Div(children="callback not executed", id="first_output_2"),
        html.Div(children="callback not executed", id="second_output_2"),
        html.Div(children="callback not executed", id="third_output_2"),
        html.Div(children="callback not executed", id="fourth_output_2"),
    ]
)


@app.callback(
    Output("first_output_2", "children"),
    Output("second_output_2", "children"),
    Input("button_2", "n_clicks"), prevent_initial_call=True)
def first_callback(n):
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return ["in the first callback it is " + current_time, "in the first callback it is " + current_time]


@app.callback(
    Output("third_output_2", "children"), Input("second_output_2", "children"), prevent_initial_call=True)
def second_callback(n):
    time.sleep(2)
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return "in the second callback it is " + current_time


@app.callback(
    Output("fourth_output_2", "children"),
    Input("first_output_2", "children"),
    Input("third_output_2", "children"), prevent_initial_call=True)
def third_output(n, m):
    time.sleep(2)
    now = datetime.now()
    current_time = now.strftime("%H:%M:%S")
    return "in the third callback it is " + current_time


if __name__ == '__main__':
    app.run_server(debug=True)
```

但是，仅当应用程序初始加载时在应用程序`layout`中同时存在回调输出和输入的情况下，以上行为才适用。

重要的是要注意，在应用程序最初加载之后，如果回调的输入由于另一个回调的结果而被插入到`layout`中，除非输出与该输入一起插入，否则`prevent_initial_call`不会阻止回调的触发！

换句话说，如果在将回调的输入插入到`layout`之前，该回调的输出已经存在于应用程序`layout`中，那么当将输入首次插入到`layout`中时，`prevent_initial_call`将不会阻止其执行。

考虑以下示例：

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State
import urllib
app = dash.Dash(__name__, suppress_callback_exceptions=True)
server = app.server
app.layout = html.Div([
    dcc.Location(id='url'),
    html.Div(id='layout-div'),
    html.Div(id='content')
])


@app.callback(Output('content', 'children'), Input('url', 'pathname'))
def display_page(pathname):
    return html.Div([
        dcc.Input(id='input', value='hello world'),
        html.Div(id='output')
    ])


@app.callback(Output('output', 'children'), Input('input', 'value'), prevent_initial_call=True)
def update_output(value):
    print('>>> update_output')
    return value


@app.callback(Output('layout-div', 'children'), Input('input', 'value'), prevent_initial_call=True)
def update_layout_div(value):
    print('>>> update_layout_div')
    return value
```

在这种情况下，`prevent_initial_call`将防止由于`display_page()`回调而将其输入首次插入应用程序`layout`时触发`update_output()`回调。这是因为执行回调时，回调的输入和输出都已包含在应用`layout`中。

但是，由于应用程序`layout`仅包含回调的输出，而不包含其输入，因此`prevent_initial_call`不会阻止`update_layout_div()`回调触发。由于此处指定了`prevent_callback_exceptions = True`，因此 Dash 必须假定在初始化应用程序时输入出现在应用程序`layout`中。从本示例中的输出元素的角度来看，新输入组件的处理方式就像已为现有输入提供了新值一样，而不是将其视为初始呈现。

### 循环回调

从 dash v1.19.0 开始，您可以在同一回调中创建循环更新。

不支持涉及多个回调的循环回调链。

循环回调可用于保持多个输入彼此同步。

#### 将 Slider 与 Text Input 同步的示例

```python
import dash
from dash.dependencies import Input, Output
import dash_core_components as dcc
import dash_html_components as html

external_stylesheets = ["https://codepen.io/chriddyp/pen/bWLwgP.css"]

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div(
    [
        dcc.Slider(
            id="slider-circular", min=0, max=20, 
            marks={i: str(i) for i in range(21)}, 
            value=3
        ),
        dcc.Input(
            id="input-circular", type="number", min=0, max=20, value=3
        ),
    ]
)
@app.callback(
    Output("input-circular", "value"),
    Output("slider-circular", "value"),
    Input("input-circular", "value"),
    Input("slider-circular", "value"),
)
def callback(input_value, slider_value):
    ctx = dash.callback_context
    trigger_id = ctx.triggered[0]["prop_id"].split(".")[0]
    value = input_value if trigger_id == "input-circular" else slider_value
    return value, value

if __name__ == '__main__':
    app.run_server(debug=True)
```

#### 显示两个具有不同单位的输入示例

```python
import dash
from dash.dependencies import Input, Output, State
import dash_core_components as dcc
import dash_html_components as html
external_stylesheets = ["https://codepen.io/chriddyp/pen/bWLwgP.css"]

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

app.layout = html.Div([
    html.Div('Convert Temperature'),
    'Celsius',
    dcc.Input(
        id="celsius",
        value=0.0,
        type="number"
    ),
    ' = Fahrenheit',
    dcc.Input(
        id="fahrenheit",
        value=32.0,
        type="number",
    ),
])

@app.callback(
    Output("celsius", "value"),
    Output("fahrenheit", "value"),
    Input("celsius", "value"),
    Input("fahrenheit", "value"),
)
def sync_input(celsius, fahrenheit):
    ctx = dash.callback_context
    input_id = ctx.triggered[0]["prop_id"].split(".")[0]
    if input_id == "celsius":
        fahrenheit= None if celsius is None else (float(celsius) * 9/5) + 32
    else:
        celsius = None if fahrenheit is None else (float(fahrenheit) - 32) * 5/9
    return celsius, fahrenheit

if __name__ == "__main__":
    app.run_server(debug=True)
```

#### 同步两个清单

```python
import dash
from dash.dependencies import Input, Output, State
import dash_core_components as dcc
import dash_html_components as html

external_stylesheets = ["https://codepen.io/chriddyp/pen/bWLwgP.css"]

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

options = [
    {"label": "New York City", "value": "NYC"},
    {"label": "Montréal", "value": "MTL"},
    {"label": "San Francisco", "value": "SF"},
]
all_cities = [option["value"] for option in options]

app.layout = html.Div(
    [
        dcc.Checklist(
            id="all-checklist",
            options=[{"label": "All", "value": "All"}],
            value=[],
            labelStyle={"display": "inline-block"},
        ),
        dcc.Checklist(
            id="city-checklist",
            options=options,
            value=[],
            labelStyle={"display": "inline-block"},
        ),
    ]
)
@app.callback(
    Output("city-checklist", "value"),
    Output("all-checklist", "value"),
    Input("city-checklist", "value"),
    Input("all-checklist", "value"),
)
def sync_checklists(cities_selected, all_selected):
    ctx = dash.callback_context
    input_id = ctx.triggered[0]["prop_id"].split(".")[0]
    if input_id == "city-checklist":
        all_selected = ["All"] if set(cities_selected) == set(all_cities) else []
    else:
        cities_selected = all_cities if all_selected else []
    return cities_selected, all_selected

if __name__ == "__main__":
    app.run_server(debug=True)
```

## 客户端回调

有时，回调可能会导致相当大的开销，尤其是在以下情况下：

- 接收和/或返回大量数据（传输时间）
- 经常被调用（网络延迟，排队，握手）
- 是回调链的一部分，该回调链需要浏览器和 Dash 之间进行多次往返

当回调的开销成本变得太大并且无法进行其他优化时，可以将回调修改为直接在浏览器中运行，而不是向 Dash 发出请求。

回调的语法几乎完全相同。您可以像在声明回调时一样正常使用`Input`和`Output`，但是还可以将 JavaScript 函数定义为`@app.callback`装饰器的第一个参数。

例如，以下回调：

```python
@app.callback(
    Output('out-component', 'value'),
    Input('in-component1', 'value'),
    Input('in-component2', 'value')
)
def large_params_function(largeValue1, largeValue2):
    largeValueOutput = someTransform(largeValue1, largeValue2)
    return largeValueOutput
```

可以重写为使用 JavaScript，如下所示：

```python
from dash.dependencies import Input, Output

app.clientside_callback(
    """
    function(largeValue1, largeValue2) {
        return someTransform(largeValue1, largeValue2);
    }
    """,
    Output('out-component', 'value'),
    Input('in-component1', 'value'),
    Input('in-component2', 'value')
)
```

您还可以选择在`assets/`文件夹中的`.js`文件中定义函数。为了获得与上面的代码相同的结果，`.js`文件的内容如下所示：

```js
window.dash_clientside = Object.assign({}, window.dash_clientside, {
    clientside: {
        large_params_function: function(largeValue1, largeValue2) {
            return someTransform(largeValue1, largeValue2);
        }
    }
});
```

在 Dash 中，回调现在将写为：

```python
from dash.dependencies import ClientsideFunction, Input, Output

app.clientside_callback(
    ClientsideFunction(
        namespace='clientside',
        function_name='large_params_function'
    ),
    Output('out-component', 'value'),
    Input('in-component1', 'value'),
    Input('in-component2', 'value')
)
```

### 一个简单的例子

下面是两个使用客户端回调与`dcc.Store`组件一起更新图形的示例。在这些示例中，我们在后端更新了`dcc.Store`组件。为了创建和显示图形，我们在前端有一个客户端回调，该回调添加了一些有关我们使用`"Graph scale"`下的单选按钮指定的`layout`的其他信息。

```python
import dash
from dash.dependencies import Input, Output
import dash_core_components as dcc
import dash_html_components as html
import pandas as pd

import json

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')

available_countries = df['country'].unique()

app.layout = html.Div([
    dcc.Graph(
        id='clientside-graph'
    ),
    dcc.Store(
        id='clientside-figure-store',
        data=[{
            'x': df[df['country'] == 'Canada']['year'],
            'y': df[df['country'] == 'Canada']['pop']
        }]
    ),
    'Indicator',
    dcc.Dropdown(
        id='clientside-graph-indicator',
        options=[
            {'label': 'Population', 'value': 'pop'},
            {'label': 'Life Expectancy', 'value': 'lifeExp'},
            {'label': 'GDP per Capita', 'value': 'gdpPercap'}
        ],
        value='pop'
    ),
    'Country',
    dcc.Dropdown(
        id='clientside-graph-country',
        options=[
            {'label': country, 'value': country}
            for country in available_countries
        ],
        value='Canada'
    ),
    'Graph scale',
    dcc.RadioItems(
        id='clientside-graph-scale',
        options=[
            {'label': x, 'value': x} for x in ['linear', 'log']
        ],
        value='linear'
    ),
    html.Hr(),
    html.Details([
        html.Summary('Contents of figure storage'),
        dcc.Markdown(
            id='clientside-figure-json'
        )
    ])
])


@app.callback(
    Output('clientside-figure-store', 'data'),
    Input('clientside-graph-indicator', 'value'),
    Input('clientside-graph-country', 'value')
)
def update_store_data(indicator, country):
    dff = df[df['country'] == country]
    return [{
        'x': dff['year'],
        'y': dff[indicator],
        'mode': 'markers'
    }]


app.clientside_callback(
    """
    function(data, scale) {
        return {
            'data': data,
            'layout': {
                 'yaxis': {'type': scale}
             }
        }
    }
    """,
    Output('clientside-graph', 'figure'),
    Input('clientside-figure-store', 'data'),
    Input('clientside-graph-scale', 'value')
)


@app.callback(
    Output('clientside-figure-json', 'children'),
    Input('clientside-figure-store', 'data')
)
def generated_figure_json(data):
    return '```\n'+json.dumps(data, indent=2)+'\n```'


if __name__ == '__main__':
    app.run_server(debug=True)
```

请注意，在此示例中，我们通过从数据框中提取相关数据来手动创建`figure`字典。这就是存储在我们的`dcc.Store`组件中的内容； 展开上面的"Contents of figure storage"，以准确查看用于构建图形的内容。

### 使用 Plotly Express 生成 figure

通过 Plotly Express，您可以创建`figures`的单行声明。当使用诸如`plotly_express.Scatter`创建 graph 时，您将获得一个字典作为返回值。该字典的形状与`dcc.Graph`组件的`figure`参数相同。（有关`figure`形状的更多信息，请参见[此处](https://plotly.com/python/creating-and-updating-figures/)。）

我们可以重做上面的示例以使用 Plotly Express。

```python
import dash
from dash.dependencies import Input, Output
import dash_core_components as dcc
import dash_html_components as html
import pandas as pd
import json

import plotly.express as px

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')

available_countries = df['country'].unique()

app.layout = html.Div([
    dcc.Graph(
        id='clientside-graph-px'
    ),
    dcc.Store(
        id='clientside-figure-store-px'
    ),
    'Indicator',
    dcc.Dropdown(
        id='clientside-graph-indicator-px',
        options=[
            {'label': 'Population', 'value': 'pop'},
            {'label': 'Life Expectancy', 'value': 'lifeExp'},
            {'label': 'GDP per Capita', 'value': 'gdpPercap'}
        ],
        value='pop'
    ),
    'Country',
    dcc.Dropdown(
        id='clientside-graph-country-px',
        options=[
            {'label': country, 'value': country}
            for country in available_countries
        ],
        value='Canada'
    ),
    'Graph scale',
    dcc.RadioItems(
        id='clientside-graph-scale-px',
        options=[
            {'label': x, 'value': x} for x in ['linear', 'log']
        ],
        value='linear'
    ),
    html.Hr(),
    html.Details([
        html.Summary('Contents of figure storage'),
        dcc.Markdown(
            id='clientside-figure-json-px'
        )
    ])
])


@app.callback(
    Output('clientside-figure-store-px', 'data'),
    Input('clientside-graph-indicator-px', 'value'),
    Input('clientside-graph-country-px', 'value')
)
def update_store_data(indicator, country):
    dff = df[df['country'] == country]
    return px.scatter(dff, x='year', y=str(indicator))


app.clientside_callback(
    """
    function(figure, scale) {
        if(figure === undefined) {
            return {'data': [], 'layout': {}};
        }
        const fig = Object.assign({}, figure, {
            'layout': {
                ...figure.layout,
                'yaxis': {
                    ...figure.layout.yaxis, type: scale
                }
             }
        });
        return fig;
    }
    """,
    Output('clientside-graph-px', 'figure'),
    Input('clientside-figure-store-px', 'data'),
    Input('clientside-graph-scale-px', 'value')
)


@app.callback(
    Output('clientside-figure-json-px', 'children'),
    Input('clientside-figure-store-px', 'data')
)
def generated_px_figure_json(data):
    return '```\n'+json.dumps(data, indent=2)+'\n```'


if __name__ == '__main__':
    app.run_server(debug=True)
```

同样，您可以展开上方的 "Contents of figure storage" 部分，以查看生成的内容。您可能会注意到，这比前面的示例要广泛得多。特别是已经定义了`layout`。因此，我们不必像以前那样创建`layout`，而是必须对 JavaScript 代码中的现有`layout`进行更改。

注意：有一些限制要牢记：

- 客户端回调在浏览器的主线程上执行，并在执行时阻止渲染和事件处理。
- Dash 当前不支持异步客户端回调，如果返回 `Promise`，它将失败。
- 如果您需要引用服务器上的全局变量，或者需要数据库调用，则无法进行客户端回调。

## 模式匹配回调

Dash 1.11.0 的新功能！（需要`dash-renderer`1.4.0或更高版本）

模式匹配的回调选择器 `MATCH`，`ALL` 和 `ALLSMALLER`允许您编写响应或更新任意数量或动态数量的组件的回调。

### ALL 的简单例子

此示例呈现任意数量的`dcc.Dropdown`元素，并且只要任何`dcc.Dropdown`元素发生更改，就会触发回调。尝试添加一些下拉菜单并选择其值，以查看应用程序如何更新。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State, MATCH, ALL

app = dash.Dash(__name__, suppress_callback_exceptions=True)

app.layout = html.Div([
    html.Button("Add Filter", id="add-filter", n_clicks=0),
    html.Div(id='dropdown-container', children=[]),
    html.Div(id='dropdown-container-output')
])

@app.callback(
    Output('dropdown-container', 'children'),
    Input('add-filter', 'n_clicks'),
    State('dropdown-container', 'children'))
def display_dropdowns(n_clicks, children):
    new_dropdown = dcc.Dropdown(
        id={
            'type': 'filter-dropdown',
            'index': n_clicks
        },
        options=[{'label': i, 'value': i} for i in ['NYC', 'MTL', 'LA', 'TOKYO']]
    )
    children.append(new_dropdown)
    return children

@app.callback(
    Output('dropdown-container-output', 'children'),
    Input({'type': 'filter-dropdown', 'index': ALL}, 'value')
)
def display_output(values):
    return html.Div([
        html.Div('Dropdown {} = {}'.format(i + 1, value))
        for (i, value) in enumerate(values)
    ])


if __name__ == '__main__':
    app.run_server(debug=True)
```

有关此示例的一些注意事项：

- 注意`dcc.Dropdown`中的`id`是字典而不是字符串。这是我们为模式匹配回调启用的一项新功能（以前，ID 必须为字符串）。
- 在第二个回调中，我们有`Input({'type': 'filter-dropdown', 'index': ALL}, 'value')`。这意味着“匹配具有ID字典的任何输入，其中`'type'`是`'filter-dropdown'`并且`'index'`是任何东西。只要任何下拉列表的`value`属性发生变化，就将其所有值发送到回调中。”
- ID 字典的键和值（`type`, `index`, `filter-dropdown`）是任意的。可以将其命名为`{'foo': 'bar', 'baz': n_clicks}`。
- 但是，出于可读性考虑，我们建议使用 `type`, `index` 或 `id` 之类的键。`type`可以用来引用类或集合的动态组件，而`index`或`id`可以用来引用在该集合内匹配的组件。在此示例中，我们只有一组动态组件，但是在更复杂的应用程序中或者在使用`MATCH`时，您可能具有多组动态组件（请参见下文）。
- 实际上，在此示例中，我们实际上并不需要`'type': 'filter-dropdown'`。相同的回调将与`Input({'index': ALL}, 'value')`一起使用。如果您创建了多组动态组件，我们将`'type': 'filter-dropdown'`作为额外的说明符。
- 组件属性本身（例如，`value`）无法通过模式进行匹配，只有 ID 是动态的。
- 此示例使用带有`State`的通用模式-单击按钮时，`dropdown-container`组件中当前显示的下拉列表集将传递到回调中。在回调中，新的下拉列表将添加到列表中，然后返回。
- 您还可以使用`dash.callback_context`来访问输入和状态，并知道哪个输入已更改。这是在页面上呈现两个下拉菜单时数据可能看起来的样子。
    - `dash.callback_context.triggered`。请注意，`prop_id`是没有空格的字符串化字典。
```json
[
  {
    'prop_id': '{"index":0,"type":"filter-dropdown"}.value',
    'value': 'NYC'
  }
]
```
    - `dash.callback_context.inputs`。请注意，键是没有空格的字符串化字典。
    ```json
    {
  '{"index":0,"type":"filter-dropdown"}.value': 'NYC',
  '{"index":1,"type":"filter-dropdown"}.value': 'LA'
}
```

    - `dash.callback_context.inputs_list`。列表中的每个元素都对应于一个输入声明。如果输入声明之一与模式匹配，则它将包含值列表。
    ```json
    [
  [
    {
      'id': {
        'index': 0,
        'type': 'filter-dropdown'
      },
      'property': 'value',
      'value': 'NYC'
    },
    {
      'id': {
        'index': 1,
        'type': 'filter-dropdown'
      },
      'property': 'value',
      'value': 'LA'
    }
  ]
]
```

    - `dash.callback_context.outputs_list`

    ```json
        {
    'id': 'dropdown-container-output',
    'property': 'children'
    }
    ```

### MATCH 的简单示例

像`ALL`一样，当组件的任何属性更改时，`MATCH`都会触发回调。但是，`MATCH`不会将所有值都传递给回调函数，而只会将单个值传递给回调函数。与其更新单个输出，不如更新与之`"matched"`的动态输出。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State, MATCH, ALL

app = dash.Dash(__name__, suppress_callback_exceptions=True)

app.layout = html.Div([
    html.Button("Add Filter", id="dynamic-add-filter", n_clicks=0),
    html.Div(id='dynamic-dropdown-container', children=[]),
])

@app.callback(
    Output('dynamic-dropdown-container', 'children'),
    Input('dynamic-add-filter', 'n_clicks'),
    State('dynamic-dropdown-container', 'children'))
def display_dropdowns(n_clicks, children):
    new_element = html.Div([
        dcc.Dropdown(
            id={
                'type': 'dynamic-dropdown',
                'index': n_clicks
            },
            options=[{'label': i, 'value': i} for i in ['NYC', 'MTL', 'LA', 'TOKYO']]
        ),
        html.Div(
            id={
                'type': 'dynamic-output',
                'index': n_clicks
            }
        )
    ])
    children.append(new_element)
    return children


@app.callback(
    Output({'type': 'dynamic-output', 'index': MATCH}, 'children'),
    Input({'type': 'dynamic-dropdown', 'index': MATCH}, 'value'),
    State({'type': 'dynamic-dropdown', 'index': MATCH}, 'id'),
)
def display_output(value, id):
    return html.Div('Dropdown {} = {}'.format(id['index'], value))


if __name__ == '__main__':
    app.run_server(debug=True)
```

关于此示例的注释：

- `display_dropdowns`回调返回具有相同`index`的两个元素：dropdown 和 div。
- 第二个回调使用`MATCH`选择器。使用此选择器，我们要求 Dash 执行以下操作：
    1. 每当 id 为`'type': 'dynamic-dropdown'`的任何组件的`value`属性更改时，都会触发回调：`Input({'type': 'dynamic-dropdown', 'index': MATCH}, 'value')`
    2. 使用id `'type': 'dynamic-output'` 和与输入的相同索引匹配的`index`更新组件：`Output({'type': 'dynamic-output', 'index': MATCH}, 'children')`
    3. 将下拉列表的 `id` 传递到回调中：`State({'type': 'dynamic-dropdown', 'index': MATCH}, 'id')`
- 使用`MATCH`选择器，对于每个`Input`或`State`，仅将一个值传递到回调中。这与之前的`ALL`选择器示例不同，在该示例中，Dash 将所有值都传递到了回调中。
- 请注意，设计将输入与输出`"line up"`的 ID 字典非常重要。`MATCH`约定是 Dash 将更新具有与`id`相同的动态ID的任何输出。在这种情况下，“动态ID”是索引的值，我们设计了布局以返回具有相同索引值的下拉列表和div。
- 在某些情况下，了解哪个动态组件已更改可能很重要。如上所述，您可以通过在回调中将`id`设置为`State`来访问它。
- 您还可以使用`dash.callback_context`来访问输入和状态，并知道哪个输入已更改。`output_list`在`MATCH`中特别有用，因为它可以告诉您该特定的回调调用负责更新哪个动态组件。这是我们更改第一个下拉菜单后在页面上呈现两个下拉菜单时数据的外观。
    - `dash.callback_context.triggered`。请注意，`prop_id`是没有空格的字符串化字典。
    ```json
    [
    {
        'prop_id': '{"index":0,"type":"dynamic-dropdown"}.value',
        'value': 'NYC'
    }
    ]
    ```
    - `dash.callback_context.inputs`。请注意，键是没有空格的字符串化字典。
    ```json
        {
    '{"index":0,"type":"dynamic-dropdown"}.value': 'NYC'
    }
    ```
    - `dash.callback_context.inputs_list`。列表中的每个元素都对应于一个输入声明。如果输入声明之一与模式匹配，则它将包含值列表。
    ```json
    [
        [
            {
            'id': {
                'index': 0,
                'type': 'dynamic-dropdown'
            },
            'property': 'value',
            'value': 'NYC'
            }
        ]
    ]
    ```
    - `dash.callback_context.outputs_list`
    ```json
    {
        'id': {
            'index': 0,
            'type': dynamic-output'
        },
        'property': 'children'
    }
    ```

### ALLSMALLER 的简单示例

在下面的示例中，`ALLSMALLER`用于传递页面上所有索引小于与 div 对应的索引的下拉列表的值。

下例中的用户界面显示了过滤器结果，随着我们应用每个其他下拉菜单，过滤器结果在每个过滤器中都越来越具体。

`ALLSMALLER`仅可用于输入和状态项，并且必须用于在输出项中具有`MATCH`的键上。

`ALLSMALLER`并非总是必要的（您通常可以使用ALL并在回调中过滤掉索引），但是它将使您的逻辑更简单。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State, MATCH, ALL, ALLSMALLER
import pandas as pd

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

app = dash.Dash(__name__, suppress_callback_exceptions=True)

app.layout = html.Div([
    html.Button('Add Filter', id='add-filter-ex3', n_clicks=0),
    html.Div(id='container-ex3', children=[]),
])

@app.callback(
    Output('container-ex3', 'children'),
    Input('add-filter-ex3', 'n_clicks'),
    State('container-ex3', 'children'))
def display_dropdowns(n_clicks, existing_children):
    existing_children.append(html.Div([
        dcc.Dropdown(
            id={
                'type': 'filter-dropdown-ex3',
                'index': n_clicks
            },
            options=[{'label': i, 'value': i} for i in df['country'].unique()],
            value=df['country'].unique()[n_clicks]
        ),
        html.Div(id={
            'type': 'output-ex3',
            'index': n_clicks
        })
    ]))
    return existing_children


@app.callback(
    Output({'type': 'output-ex3', 'index': MATCH}, 'children'),
    Input({'type': 'filter-dropdown-ex3', 'index': MATCH}, 'value'),
    Input({'type': 'filter-dropdown-ex3', 'index': ALLSMALLER}, 'value'),
)
def display_output(matching_value, previous_values):
    previous_values_in_reversed_order = previous_values[::-1]
    all_values = [matching_value] + previous_values_in_reversed_order

    dff = df[df['country'].str.contains('|'.join(all_values))]
    avgLifeExp = dff['lifeExp'].mean()

    # Return a slightly different string depending on number of values
    if len(all_values) == 1:
        return html.Div('{:.2f} is the life expectancy of {}'.format(
            avgLifeExp, matching_value
        ))
    elif len(all_values) == 2:
        return html.Div('{:.2f} is the average life expectancy of {}'.format(
            avgLifeExp, ' and '.join(all_values)
        ))
    else:
        return html.Div('{:.2f} is the average life expectancy of {}, and {}'.format(
            avgLifeExp, ', '.join(all_values[:-1]), all_values[-1]
        ))

if __name__ == '__main__':
    app.run_server(debug=True)
```

- 在上面的示例中，尝试添加一些过滤器，然后更改第一个下拉列表。请注意，更改此下拉菜单将如何更新具有依赖于该下拉菜单的索引的每个`html.Div`的文本。
- 也就是说，只要索引小于它的任何下拉列表发生变化，每个`html.Div`都会被更新。
- 因此，如果添加了10个过滤器，并且第一个下拉列表已更改，则Dash将触发您的回调10次，一次更新每个`html.Div`，这取决于已更改的`dcc.Dropdown`。
- 如上所述，您还可以使用`dash.callback_context`来访问输入和状态，并知道哪个输入已更改。这是在更改第一个下拉列表后，使用页面上呈现的两个下拉列表更新第二个div时数据的外观。
    - `dash.callback_context.triggered`。请注意，prop_id是没有空格的字符串化字典。
    ```json
    [
        {
            'prop_id': '{"index":0,"type":"filter-dropdown-ex3"}.value',
            'value': 'Canada'
        }
    ]
    ```
    - `dash.callback_context.inputs`。请注意，键是没有空格的字符串化字典。
    ```json
    {
        '{"index":1,"type":"filter-dropdown-ex3"}.value': 'Albania',
        '{"index":0,"type":"filter-dropdown-ex3"}.value': 'Canada'
    }
    ```
    - `dash.callback_context.inputs_list`。列表中的每个元素都对应于一个输入声明。如果输入声明之一与模式匹配，则它将包含值列表。

    ```json
    [
    {
        'id': {
        'index': 1,
        'type': 'filter-dropdown-ex3'
        },
        'property': 'value',
        'value': 'Albania'
    },
    [
        {
        'id': {
            'index': 0,
            'type': 'filter-dropdown-ex3'
        },
        'property': 'value',
        'value': 'Canada'
        }
    ]
    ]
    ```
    - `dash.callback_context.outputs_list`

    ```json
    {
    'id': {
        'index': 1,
        'type': output-ex3'
    },
    'property': 'children'
    }
    ```

### Todo App

创建 Todo 应用程序是一个经典的 UI 练习，它演示了常见的“创建，读取，更新和删除”（CRUD）应用程序中的许多功能。

```python
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State, MATCH, ALL

app = dash.Dash(__name__)

app.layout = html.Div([
    html.Div('Dash To-Do list'),
    dcc.Input(id="new-item"),
    html.Button("Add", id="add"),
    html.Button("Clear Done", id="clear-done"),
    html.Div(id="list-container"),
    html.Div(id="totals")
])

style_todo = {"display": "inline", "margin": "10px"}
style_done = {"textDecoration": "line-through", "color": "#888"}
style_done.update(style_todo)


@app.callback(
    [
        Output("list-container", "children"),
        Output("new-item", "value")
    ],
    [
        Input("add", "n_clicks"),
        Input("new-item", "n_submit"),
        Input("clear-done", "n_clicks")
    ],
    [
        State("new-item", "value"),
        State({"index": ALL}, "children"),
        State({"index": ALL, "type": "done"}, "value")
    ]
)
def edit_list(add, add2, clear, new_item, items, items_done):
    triggered = [t["prop_id"] for t in dash.callback_context.triggered]
    adding = len([1 for i in triggered if i in ("add.n_clicks", "new-item.n_submit")])
    clearing = len([1 for i in triggered if i == "clear-done.n_clicks"])
    new_spec = [
        (text, done) for text, done in zip(items, items_done)
        if not (clearing and done)
    ]
    if adding:
        new_spec.append((new_item, []))
    new_list = [
        html.Div([
            dcc.Checklist(
                id={"index": i, "type": "done"},
                options=[{"label": "", "value": "done"}],
                value=done,
                style={"display": "inline"},
                labelStyle={"display": "inline"}
            ),
            html.Div(text, id={"index": i}, style=style_done if done else style_todo)
        ], style={"clear": "both"})
        for i, (text, done) in enumerate(new_spec)
    ]
    return [new_list, "" if adding else new_item]


@app.callback(
    Output({"index": MATCH}, "style"),
    Input({"index": MATCH, "type": "done"}, "value")
)
def mark_done(done):
    return style_done if done else style_todo


@app.callback(
    Output("totals", "children"),
    Input({"index": ALL, "type": "done"}, "value")
)
def show_totals(done):
    count_all = len(done)
    count_done = len([d for d in done if d])
    result = "{} of {} items completed".format(count_done, count_all)
    if count_all:
        result += " - {}%".format(int(100 * count_done / count_all))
    return result


if __name__ == "__main__":
    app.run_server(debug=True)
```

## Callback Gotchas

Dash的工作方式在某些方面可能违反直觉。对于回调系统的工作方式尤其如此。本节概述了在开始构建更复杂的Dash应用程序时可能遇到的一些常见 Dash 陷阱。如果您已经阅读了[Dash教程](https://dash.plotly.com/)的其余部分，并且遇到了意外的行为，那么这是通读的好部分。如果您还有其他问题，可以在[Dash社区论坛](https://community.plotly.com/c/dash)中提问。

### 回调要求其 `Inputs`，`States` 和 `Output`出现在布局中

默认情况下，Dash将验证应用于您的回调，这将执行检查，例如验证回调参数的类型以及检查指定的`Input`和`Output`组件是否实际上具有指定的属性。为了进行全面验证，在您的应用启动时，回调中的所有组件都必须存在于布局中，否则，您将看到错误。

但是，在涉及动态修改布局的更复杂的Dash应用程序（例如多页应用程序）的情况下，并非出现在回调中的每个组件都将包含在初始布局中。您可以通过禁用回调验证来消除此限制，如下所示：

```python
app.config.suppress_callback_exceptions = True
```

### 回调要求在页面上呈现所有 `Inputs` 和 `States`

如果已禁用回调验证以支持动态布局，则不会自动提醒您在布局内未找到回调内组件的情况。在这种情况下，布局中缺少向回调注册的组件，则将无法触发该回调。例如，如果您定义仅在当前页面布局中存在指定`Inputs`的子集的回调，则根本不会触发该回调。

## 组件/属性对只能是一个回调的`Output`

对于给定的组件/属性对（例如`'my-graph'`，`'figure'`），只能将其注册为一个回调的`Output`。如果要将两个逻辑上分开的`Inputs`集与一个输出组件/属性对关联，则必须将它们捆绑成一个更大的回调，并检测哪个相关的`Inputs`触发了函数内的回调。对于`html.Button`元素，可以使用`n_clicks_timestamp`属性来检测是哪个触发了回调。有关此示例，请参阅FAQ中的问题，如何确定哪个输入已更改？

### 必须在服务器启动之前定义所有回调

必须在Dash应用程序的服务器开始运行之前（即在调用`app.run_server(debug=True)`之前）定义所有回调。这意味着，尽管您可以在处理回调过程中动态组装更改后的布局片段，但无法在处理回调过程中定义动态回调来响应用户的输入。如果您具有动态接口，其中回调将布局更改为包括一组不同的输入控件，那么您必须已经预先定义了为这些新控件提供服务所需的回调。

例如，一个常见的场景是一个`Dropdown`组件，该组件会更新当前布局以用另一个逻辑上不同的仪表板替换一个仪表板，该仪表板具有一组不同的控件（控件的数量和类型可能取决于其他用户输入）和不同的逻辑用于生成基础数据。明智的组织应是每个仪表板都具有单独的回调。在这种情况下，每个回调都需要在应用开始运行之前进行定义。

一般而言，如果Dash应用程序的功能是`Inputs`或`States`的数量由用户的输入确定，那么您必须预先预先定义用户可能触发的每个回调排列。有关如何使用`callback`装饰器以编程方式完成此操作的示例，请参见[Dash社区论坛上的帖子](https://community.plotly.com/t/callback-for-dynamically-created-graph/5511)。

### 布局中的所有 Dash Core 组件都应使用回调注册

注意：本部分仅出于遗留目的而存在。在 v0.40.0 之前，仅当组件连接到回调时才定义`setProps`。这就需要像这样在组件内进行复杂的状态管理。现在，始终定义`setProps`，这将简化组件的状态管理。在此[社区论坛主题](https://community.plotly.com/t/callbacks-clearing-all-unconnected-core-components-values/7631)中了解更多信息。

如果布局中存在 Dash Core 组件但未向回调注册（作为`Input`, `State` 或`Output`），则当任何回调更新页面时，用户对其值所做的任何更改都将重置为原始值。

这是一个已知问题，您可以在此[GitHub Issue](https://github.com/plotly/dash-renderer/issues/40)中跟踪其状态。

### 回调定义不需要在列表中

从 Dash 1.15.0 开始，回调定义中的 `Input`，`Output` 和 `State` 不必在列表中。您仍然需要先提供`Output`项，然后提供`Input`项，然后提供`State`，并且仍然支持列表形式。特别是，如果要返回包装在长度为1的列表中的单个`Output`项目，则仍应将输出包装在列表中。这对于过程生成的回调很有用。
