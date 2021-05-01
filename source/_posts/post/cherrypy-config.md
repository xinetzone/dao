---
title: CherryPy 配置
lang: zh-CN
tags: CherryPy
categories: 教程
abbrlink: 5148c19a
date: 2021-03-16 08:40:06
description:
updated:
---

CherryPy 中的配置是通过字典实现的。键是命名映射值的字符串。值可以是任何类型。在 CherryPy3 中，您可以使用配置（文件或字典）直接在引擎，服务器，请求，响应和日志对象上设置属性。因此，要了解配置文件中所有可用内容的全部范围，最好的方法是简单地导入这些对象，然后查看 `help(obj)` 告诉您什么。

## Architecture

关于 CherryPy 3 的配置，您需要了解的第一件事是它将全局配置与应用程序配置分开。如果您要在同一站点上部署多个应用程序（越来越多的人（由于 Python Web 应用程序趋向于分散管理）），则还需要注意分开配置。只有一个“全局配置”，但是您部署的每个应用都有一个单独的“应用配置”。

CherryPy Requests 是应用程序的一部分，该应用程序在全局上下文中运行，并且配置数据可能适用于这三个范围中的任何一个。让我们依次查看每个范围。

### 全局配置

全局配置条目适用于任何地方，并存储在 `cherrypy.config` 中。这个简单的字典只保存全局配置数据。也就是说，“站点范围”配置条目会影响所有已安装的应用程序。

全局配置存储在 `cherrypy.config` 字典中，因此您可以通过调用 `cherrypy.config.update(conf)` 对其进行更新。`conf` 参数可以是文件名，打开的文件或配置条目的字典。这是一个传递 `dict` 参数的示例：

```python
cherrypy.config.update({'server.socket_host': '64.72.221.48',
                        'server.socket_port': 80,
                       })
```

本示例中的 `server.socket_host` 选项确定 CherryPy 将在哪个网络接口上侦听。`server.socket_port` 选项声明要侦听的 TCP 端口。

### 应用配置

应用程序条目适用于单个已安装的应用程序，并作为 [`app.config`](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cptree.html#cherrypy._cptree.Application.config) 存储在每个 Application 对象本身上。这是一个两级字典，其中每个顶级关键字是一个路径或“相对URL”（例如，`"/"` 或 `"/my/page"`），每个值是一个配置条目的字典。该 URL 相对于该应用程序的脚本名称（挂载点）。通常，所有这些数据都在对 `tree.mount(root(), script_name='/path/to', config=conf)` 的调用中提供，尽管您也可以使用 `app.merge(conf)`。`conf` 参数可以是文件名，打开的文件或配置条目的字典。

配置文件的例子：

```conf
[/]
tools.trailing_slash.on = False
request.dispatch: cherrypy.dispatch.MethodDispatcher()
```

或者 Python 代码：

```python
config = {'/':
    {
        'request.dispatch': cherrypy.dispatch.MethodDispatcher(),
        'tools.trailing_slash.on': False,
    }
}
cherrypy.tree.mount(Root(), config=config)
```

CherryPy 仅使用以 `"/"` 开头的部分（`[global]`除外，请参见下文）。这意味着您可以通过为自己的配置条目提供一个不以 `"/"` 开头的部分名称来将它们放置在 CherryPy 配置文件中。例如，您可能包括以下数据库条目：

```conf
[global]
server.socket_host: "0.0.0.0"

[Databases]
driver: "postgres"
host: "localhost"
port: 5432

[/path]
response.timeout: 6000
```

然后，在您的应用程序代码中，您可以在请求期间通过 `cherrypy.request.app.config['Databases']` 读取这些值。对于超出请求流程的代码，您必须将引用传递给您的应用程序。

### Request 配置

每个 Request 对象都具有一个 [request.config](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cprequest.html#cherrypy._cprequest.Request.config) 字典。在请求过程的早期，通过合并全局配置，应用程序配置以及在查找页面处理程序时获取的任何配置来填充此 dict（请参见下文）。该字典仅包含适用于给定请求的那些配置条目。

<p class="w3-pale-red"> 当您执行 <a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy._cperror.html#cherrypy._cperror.InternalRedirect">InternalRedirect</a> 时，将为新路径重新计算此 config 属性。<p>

## Declaration

配置数据可以作为 Python 字典，文件名或打开文件对象提供。

### 配置文件

当提供文件名或文件时，CherryPy 使用 Python 的内置 ConfigParser。您可以通过将每个路径写为 section header，并将每个条目写为 `"key: value"`（或 `"key = value"`）对来声明 Application config：

```conf
[/path/to/my/page]
response.stream: True
tools.trailing_slash.extra = False
```

#### 组合配置文件

如果仅部署单个应用程序，则可以制作一个包含全局条目和应用程序条目的配置文件。只需将全局条目粘贴到名为 `[global]` 的配置节中，然后将同一文件传递到 `config.update` 和 `tree.mount <cherrypy._cptree.Tree.mount()`。如果您要调用 `cherrypy.quickstart(app root, script name, config)`，它将为您将配置传递到两个地方。但是，一旦您决定将另一个应用程序添加到同一站点，就需要将两个配置文件/字典分开。

#### 单独的配置文件

如果您要在同一过程中部署多个应用程序，则需要（1）文件进行全局配置，另外还需要（1）文件用于每个应用程序。 全局配置是通过调用 `cherrypy.config.update` 来应用的，而应用程序配置通常是在对 `cherrypy.tree.mount` 的调用中传递的。

通常，您应该先设置全局配置，然后再使用自己的配置挂载每个应用程序。除其他好处外，它还使您可以设置全局日志记录，这样，如果在尝试装入应用程序时出现问题，您将看到回溯。换句话说，使用以下顺序：

```python
# global config
cherrypy.config.update({'environment': 'production',
                        'log.error_file': 'site.log',
                        # ...
                        })

# Mount each app and pass it its own config
cherrypy.tree.mount(root1, "", appconf1)
cherrypy.tree.mount(root2, "/forum", appconf2)
cherrypy.tree.mount(root3, "/blog", appconf3)

if hasattr(cherrypy.engine, 'block'):
    # 3.1 syntax
    cherrypy.engine.start()
    cherrypy.engine.block()
else:
    # 3.0 syntax
    cherrypy.server.quickstart()
    cherrypy.engine.start()
```

#### 配置文件中的值使用 Python 语法

Config 条目始终是键/值对，例如 `server.socket_port = 8080`。键始终是名称，值始终是 Python 对象。也就是说，如果您要设置的值是一个 int（或其他数字），则它必须看起来像 Python int；例如 8080。如果该值为字符串，则需要将其引号，就像 Python 字符串一样。也可以创建任意对象，就像在 Python 代码中一样（假定可以找到/导入）。这是一个扩展的示例，向您展示了一些不同的类型：

```conf
[global]
log.error_file: "/home/fumanchu/myapp.log"
environment = 'production'
server.max_request_body_size: 1200

[/myapp]
tools.trailing_slash.on = False
request.dispatch: cherrypy.dispatch.MethodDispatcher()
```

#### `_cp_config`：将配置附加到处理程序

配置文件有一个严格的限制：值始终由URL键入。例如：

```conf
[/path/to/page]
methods_with_bodies = ("POST", "PUT", "PROPPATCH")
```

显然，额外的方法是该路径的规范；实际上，如果没有该代码，则可以认为该代码已损坏。在 CherryPy 中，您可以直接在页面处理程序上附加该配置位：

```python
@cherrypy.expose
def page(self):
    return "Hello, world!"
page._cp_config = {"request.methods_with_bodies": ("POST", "PUT", "PROPPATCH")}
```

`_cp_config` 是保留的属性，调度程序在对象树中的每个节点上查找该属性。`_cp_config` 属性必须是 CherryPy 配置字典。如果调度程序找到了 `_cp_config` 属性，它将将该字典合并到配置的其余部分中。整个合并的配置字典放置在 `cherrypy.request.config` 中。

可以在对象树的任何位置完成此操作。例如，我们可以将该配置附加到包含 page 方法的类中：

```python
class SetOPages:

    _cp_config = {"request.methods_with_bodies": ("POST", "PUT", "PROPPATCH")}

    @cherrypy.expose
    def page(self):
        return "Hullo, Werld!"
```

<p class="w3-pale-red">
仅默认调度程序可以保证此行为。其他调度程序可能对在何处附加 <code>_cp_config</code> 属性有不同的限制。另外，由于分派器负责处理 <code>_cp_config</code>，因此无法更改分派器（即，在此构造中不执行 <code>request.dispatch</code>）。
</p>

此技术使您可以：

- 将配置放在用于提高可读性和可维护性的位置。
- 将配置附加到对象而不是 URL。这样一来，多个 URL 都可以指向同一个对象，但是您只需定义一次配置即可。
- 提供仍可在配置文件中覆盖的默认值。

## Namespaces

因为配置条目通常只是在对象上设置属性，所以它们几乎都是以下形式：`object.attribute`。其中一些格式为：`object.subobject.attribute`。它们看起来像普通的 Python 属性链，因为它们像它们一样工作。我们将链中的名字称为“配置名称空间”。当您提供配置条目时，它会尽早绑定到名称空间引用的实际对象。例如，条目 `response.stream` 实际上设置了 `cherrypy.response` 的 `stream` 属性！这样，您可以通过启动 Python 解释器并键入以下内容来轻松确定默认值：

```python
>>> import cherrypy
>>> cherrypy.response.stream
False
```

每个配置名称空间都有其自己的处理程序。例如，“request” 名称空间具有一个处理程序，该处理程序获取您的配置条目并在适当的“请求”属性上设置该值。有一些名称空间无法像幕后的普通属性那样工作； 但是，它们仍然使用点键，并被认为是“具有名称空间”。

### 内置名称空间

可以在全局，应用程序根目录（`"/"`）或每个路径配置或以下组合中允许来自每个名称空间的条目。

#### engine

该命名空间中的条目控制着“应用程序引擎”。这些只能在全局配置中声明。`cherrypy.engine` 的任何属性都可以在 `config` 中设置；但是，在 `config` 中还有一些额外的条目可用：

- <span class="w3-pale-yellow">Plugin 属性</span>：许多引擎插件本身就是 `cherrypy.engine` 的属性。 您可以通过简单地命名附件插件来设置任何属性。例如，在 `engine.autoreload` 处有一个 [Autoreloader](https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.plugins.html#cherrypy.process.plugins.Autoreloader) 类的实例。您可以通过配置条目 `engine.autoreload.frequency = 60` 设置其“frequency”属性。此外，您可以通过将 `engine.autoreload.on = True` 或 `False` 设置为打开或关闭此类插件。
- <span class="w3-pale-yellow">engine.SIGHUP/SIGTERM</span>：这些条目可用于设置给定通道的侦听器列表。通常，这用于关闭通过 `cherrypy.quickstart()` 自动获取的信号处理。

#### hooks

声明其他请求处理功能。使用它可以将自己的 Hook 函数附加到请求。例如，要将 `my_hook_func` 添加到 `before_handler` 挂钩点：

```conf
[/]
hooks.before_handler = myapp.my_hook_func
```

#### log

配置日志记录。这些只能在全局配置（用于全局日志记录）或 `[/]` 配置（对于每个应用程序）中声明。有关可配置属性的列表，请参见 [LogManager](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cplogging.html#cherrypy._cplogging.LogManager)。通常，“access_file”，“error_file” 和 “screen” 属性是最常用的配置。

#### request

在每个 Request 上设置属性。有关完整列表，请参见 [Request](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cprequest.html#cherrypy._cprequest.Request) 类。

#### response

在每个 Response 上设置属性。有关完整列表，请参见 [Response](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cprequest.html#cherrypy._cprequest.Response) 类。

#### server

通过 [cherrypy.server](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpserver.html#cherrypy._cpserver.Server) 控制默认的 HTTP 服务器（有关可配置属性的完整列表，请参见该类）。这些只能在全局配置中声明。

#### tools

启用和配置其他请求处理程序包。有关更多信息，请参见 /tutorial/tools 概述。

#### wsgi

将 WSGI 中间件添加到应用程序的“pipeline”中。这些只能在应用的根配置（“/”）中声明。

- `wsgi.pipeline`：附加到 WSGI 管道。该值必须是（name, app factory）对的列表。每个应用程序工厂必须是 WSGI 可调用类（或返回 WSGI 可调用的可调用类）；它必须带有一个初始的 “nextapp” 参数，以及任何可选的关键字参数。可选参数可以通过 `wsgi.<name>.<arg>` 进行配置。
- `wsgi.response_class`：覆盖默认的 Response 类。

#### checker

控制“checker”，该检查器在引擎启动时在应用程序状态（包括配置）中查找常见错误。您可以通过在配置中将单个检查设置为 `False` 来关闭单个检查。请参阅 [cherrypy._cpchecker.Checker](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpchecker.html#cherrypy._cpchecker.Checker) 以获取完整列表。仅全局配置。

### 自定义配置名称空间

如果愿意，您可以定义自己的名称空间，它们可以做的事情远不止简单地设置属性。例如，`test/test_config` 模块显示了一个自定义名称空间的示例，该名称空间强制输入的参数和输出的正文内容。[cherrypy._cpwsgi](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpwsgi.html#module-cherrypy._cpwsgi) 模块包括一个附加的内置名称空间，用于调用 WSGI 中间件。

本质上，配置名称空间处理程序只是一个函数，该函数将传递其名称空间中的所有配置条目。您将其添加到名称空间注册表（字典），其中键是名称空间名称，值是处理函数。当遇到您的名称空间的配置条目时，将调用相应的处理程序函数，并传递 `config` 键和值；即 `namespaces[namespace](k, v)`。例如，如果您编写：

```python
def db_namespace(k, v):
    if k == 'connstring':
        orm.connect(v)
cherrypy.config.namespaces['db'] = db_namespace
```

然后 `cherrypy.config.update({"db.connstring": "Oracle:host=1.10.100.200;sid=TEST"})` 将调用`db_namespace('connstring', 'Oracle:host=1.10.100.200;sid=TEST')`。

调用名称空间处理程序的时间点取决于添加它的位置：

Scope|Namespace dict|Handler is called in
:-:|:-:|:-:|
Global|cherrypy.config.namespaces|cherrypy.config.update
Application|[app.namespaces](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cptree.html#cherrypy._cptree.Application.namespaces)|Application.merge (which is called by cherrypy.tree.mount)
Request|[app.request_class.namespaces](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cprequest.html#cherrypy._cprequest.Request.namespaces)|Request.configure (called for each request, after the handler is looked up)

名称可以是任何字符串，并且处理程序必须是可调用的或（Python 2.5 样式）上下文管理器。

如果在收集所有名称空间键时需要其他代码来运行，则可以提供可调用的上下文管理器来代替处理程序的常规功能。在 [PEP 343](https://www.python.org/dev/peps/pep-0343) 中定义了上下文管理器。

### Environments

命名空间中唯一不存在的键是“environment”条目。它仅适用于全局配置，并且仅在使用 `cherrypy.config.update` 时适用。此特殊条目从存储在 `cherrypy._cpconfig.environments[environment]` 中的以下模板中导入其他配置条目。

```python
Config.environments = environments = {
    'staging': {
        'engine.autoreload.on': False,
        'checker.on': False,
        'tools.log_headers.on': False,
        'request.show_tracebacks': False,
        'request.show_mismatched_params': False,
    },
    'production': {
        'engine.autoreload.on': False,
        'checker.on': False,
        'tools.log_headers.on': False,
        'request.show_tracebacks': False,
        'request.show_mismatched_params': False,
        'log.screen': False,
    },
    'embedded': {
        # For use with CherryPy embedded in another deployment stack.
        'engine.autoreload.on': False,
        'checker.on': False,
        'tools.log_headers.on': False,
        'request.show_tracebacks': False,
        'request.show_mismatched_params': False,
        'log.screen': False,
        'engine.SIGHUP': None,
        'engine.SIGTERM': None,
    },
    'test_suite': {
        'engine.autoreload.on': False,
        'checker.on': False,
        'tools.log_headers.on': False,
        'request.show_tracebacks': True,
        'request.show_mismatched_params': True,
        'log.screen': False,
    },
}
```

如果发现现有环境集（生产，暂存等）过于局限或完全错误，请随时扩展它们或添加新环境：

```python
cherrypy._cpconfig.environments['staging']['log.screen'] = False

cherrypy._cpconfig.environments['Greek'] = {
    'tools.encode.encoding': 'ISO-8859-7',
    'tools.decode.encoding': 'ISO-8859-7',
    }
```
