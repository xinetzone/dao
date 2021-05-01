---
title: CherryPy 插件
lang: zh-CN
abbrlink: 2b067799
date: 2021-03-16 09:43:02
description:
updated:
tags: CherryPy
categories: 教程
---

CherryPy 确实是一个开放框架，您可以在服务器端或根据每个请求随意扩展和插入新功能。无论哪种方式，CherryPy 都可以帮助您通过简单的模式来构建应用程序并支持体系结构。

## 服务器的函数

CherryPy 可以视为 HTTP 库，也可以视为 Web 应用程序框架。在后一种情况下，其体系结构提供了支持整个服务器实例上的操作的机制。这提供了一个强大的画布，可以执行持久性操作，因为服务器范围的函数可以在请求处理本身之外进行。只要总线（bus）存在，它们就可以在整个过程中使用。

典型用例：

- 保持与外部服务器的连接池，以便您无需在每个请求（例如数据库连接）上重新打开它们。
- 后台处理（例如，您需要在不阻止整个请求本身的情况下完成工作）。

### Publish/Subscribe pattern

CherryPy 的骨干网由一个总线系统（bus system）组成，该总线系统实现了简单的发布/订阅消息传递模式。简而言之，在 CherryPy 中，所有内容都通过该总线进行控制。可以很容易地将 bus 描绘成寿司店的传送带，如下图所示。

![](https://docs.cherrypy.org/en/latest/_images/sushibelt.JPG)

您可以订阅和发布到 bus 上的通道。通道有点像总线中的唯一标识符。当消息发布到某个通道时，总线会将消息分发给该通道的所有订户。

pubsub 模式的一个有趣方面是，它促进了调用者和被调用者之间的解耦。发布的消息最终将生成响应，但是发布者不知道该响应来自何处。

由于这种解耦，CherryPy 应用程序可以轻松访问功能，而不必保留对提供该功能的实体的引用。取而代之的是，该应用程序只是发布到总线上，并会收到适当的响应，这很重要。

#### 典型模式

让我们来看看以下虚拟应用程序：

```python
import cherrypy

class ECommerce:
    def __init__(self, db):
        self.mydb = db

    @cherrypy.expose
    def save_kart(self, cart_data):
        cart = Cart(cart_data)
        self.mydb.save(cart)

if __name__ == '__main__':
    cherrypy.quickstart(ECommerce(), '/')
```

该应用程序具有对数据库的引用，但这在数据库提供程序和应用程序之间建立了相当强的耦合。

解决耦合问题的另一种方法是使用 pubsub 工作流程：

```python
import cherrypy

class ECommerce:
    @cherrypy.expose
    def save_kart(self, cart_data):
        cart = Cart(cart_data)
        cherrypy.engine.publish('db-save', cart)

if __name__ == '__main__':
    cherrypy.quickstart(ECommerce(), '/')
```

在此示例中，我们将 `cart` 车实例发布到 `db-save` 通道。然后，一个或多个订阅者可以对该消息做出反应，而应用程序不必知道这些消息。

<p class="w3-pale-yellow">
这种方法不是强制性的，您可以自行决定如何设计实体互动。
</p>

#### 实现细节

CherryPy 的总线实现非常简单，因为它向通道注册了函数。每当消息发布到通道时，每个注册函数都会应用该消息作为参数传递。

整个行为是同步发生的，从这个意义上说，如果一个订户花费太长时间来处理一条消息，则其余订户将被延迟。

CherryPy 的总线不是由 [zeromq](http://zeromq.org/) 或 [RabbitMQ](https://www.rabbitmq.com/) 提供的高级 pubsub 消息传递代理系统。在使用它的前提下，可能会产生成本。

#### 引擎作为 pubsub 总线

如前所述，CherryPy 是围绕 pubsub 总线构建的。框架在运行时管理的所有实体都在单个总线实例（称为引擎（`engine`））上运行。

因此，总线实现提供了一组描述应用程序生命周期的通用通道：

```sh
                 O
                 |
                 V
STOPPING --> STOPPED --> EXITING -> X
   A   A         |
   |    \___     |
   |        \    |
   |         V   V
 STARTED <-- STARTING
```

各状态的转换触发了要发布到的通道，以便订户可以对其做出反应。

一个很好的例子是 HTTP 服务器，它将在消息发布到 [start](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpmodpy.html#cherrypy._cpmodpy.ModPythonServer.start) 通道时从“STOPPED”状态转换为“STARTED”状态。

#### 内置通道

为了支持其生命周期，CherryPy 定义了一组公共通道（channels），这些 channels 将在各个 state 发布：

- “start”：当总线处于 “STARTING” 状态时
- “main”：定期从 CherryPy 的主循环中获取
- “stop”：当总线处于 “STOPPING” 状态时
- “graceful”：当总线请求重新加载 subscribers 时
- “exit”：当总线处于 “EXITING” 状态时

Channel 将由 **engine** 自动发布。因此，注册需要对所有订阅者 **engine** 的 transition changes 做出反应。

此外，在请求处理期间还会发布一些其他通道。

- “before_request”：在 CherryPy 处理请求之前
- “after_request”：在处理之后

另外，从 [cherrypy.process.plugins.ThreadManager](https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.plugins.html#cherrypy.process.plugins.ThreadManager) 插件中：

- “acquire_thread”
- “start_thread”
- “stop_thread”
- “release_thread”

#### Bus API

为了使用总线(bus)，该实现提供了以下简单的 API：

<dl>
  <dt><a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.wspbus.html#cherrypy.process.wspbus.Bus.publish">cherrypy.engine.publish(channel, *args)</a></dt>
  <dd><code>channel</code> 参数是一个字符串，用于标识将消息发送到的信道
  </dd>
  <dd><code>*args</code> 是消息，可能包含任何有效的 Python 值或对象。</dd>
  <dt><a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.wspbus.html#cherrypy.process.wspbus.Bus.subscribe">cherrypy.engine.subscribe(channel, callable)</a></dt>
  <dd><code>channel</code> 参数是一个字符串，用于标识可回调对象将被注册到的信道。
  </dd>
  <dd><code>callable</code> 是一个 Python 函数或方法，其签名必须与将要发布的签名匹配。</dd>

  <dt><a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.wspbus.html#cherrypy.process.wspbus.Bus.unsubscribe">cherrypy.engine.unsubscribe(channel, callable)(channel, callable)</a></dt>
  <dd><code>channel</code> 参数是一个字符串，用于标识可回调对象将被注册到的信道。
  </dd>
  <dd><code>callable</code> 是已注册的 Python 函数或方法。</dd>
</dl>

### Plugins

简而言之，插件是可以通过发布或订阅通道（通常同时在同一时间）与总线进行交互的实体。

<article class="w3-green">
只要您具有以下功能，插件就非常有用：

- 在整个应用程序服务器中可用
- 与应用程序的生命周期相关
- 您要避免与应用程序紧密耦合
</article>

#### 创建插件

一个典型的插件如下所示：

```python
import cherrypy
from cherrypy.process import wspbus, plugins

class DatabasePlugin(plugins.SimplePlugin):
    def __init__(self, bus, db_klass):
        plugins.SimplePlugin.__init__(self, bus)
        self.db = db_klass()

    def start(self):
        self.bus.log('Starting up DB access')
        self.bus.subscribe("db-save", self.save_it)

    def stop(self):
        self.bus.log('Stopping down DB access')
        self.bus.unsubscribe("db-save", self.save_it)

    def save_it(self, entity):
        self.db.save(entity)
```

[cherrypy.process.plugins.SimplePlugin](https://docs.cherrypy.org/en/latest/pkg/cherrypy.process.plugins.html#cherrypy.process.plugins.SimplePlugin) 是 CherryPy 提供的帮助程序类，该类将自动将您的[`start`](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpmodpy.html#cherrypy._cpmodpy.ModPythonServer.start "cherrypy._cpmodpy.ModPythonServer.start")和[`stop`](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpmodpy.html#cherrypy._cpmodpy.ModPythonServer.stop "cherrypy._cpmodpy.ModPythonServer.stop")方法订阅到相关通道。

发布[`start`](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpmodpy.html#cherrypy._cpmodpy.ModPythonServer.start "cherrypy._cpmodpy.ModPythonServer.start")和[`stop`](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpmodpy.html#cherrypy._cpmodpy.ModPythonServer.stop "cherrypy._cpmodpy.ModPythonServer.stop")通道时，将相应地调用这些方法。

请注意，接下来我们的插件如何订阅 `db-save` 通道，以便总线可以向插件分发消息。

#### 启用插件

要启用该插件，必须将其注册到总线，如下所示：

```python
DatabasePlugin(cherrypy.engine, SQLiteDB).subscribe()
```

这里的 `SQLiteDB` 是用作我们的数据库提供程序的伪类。

#### 禁用插件

您还可以按以下步骤取消注册插件：

```python
someplugin.unsubscribe()
```

当您希望阻止默认的 HTTP 服务器由 CherryPy 启动时，例如，如果您在其他 HTTP 服务器（支持 WSGI）上运行，通常会使用此方法：

```python
cherrypy.server.unsubscribe()
```

让我们来看一个使用此默认应用程序的示例：

```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "hello world"

if __name__ == '__main__':
    cherrypy.quickstart(Root())
```

这是运行此应用程序时会看到的内容：

```sh
[27/Apr/2014:13:04:07] ENGINE Listening for SIGHUP.
[27/Apr/2014:13:04:07] ENGINE Listening for SIGTERM.
[27/Apr/2014:13:04:07] ENGINE Listening for SIGUSR1.
[27/Apr/2014:13:04:07] ENGINE Bus STARTING
[27/Apr/2014:13:04:07] ENGINE Started monitor thread 'Autoreloader'.
[27/Apr/2014:13:04:08] ENGINE Serving on http://127.0.0.1:8080
[27/Apr/2014:13:04:08] ENGINE Bus STARTED
```

现在，让我们退订 HTTP 服务器：

```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "hello world"

if __name__ == '__main__':
    cherrypy.server.unsubscribe()
    cherrypy.quickstart(Root())
```

这是我们得到的：

```log
[27/Apr/2014:13:08:06] ENGINE Listening for SIGHUP.
[27/Apr/2014:13:08:06] ENGINE Listening for SIGTERM.
[27/Apr/2014:13:08:06] ENGINE Listening for SIGUSR1.
[27/Apr/2014:13:08:06] ENGINE Bus STARTING
[27/Apr/2014:13:08:06] ENGINE Started monitor thread 'Autoreloader'.
[27/Apr/2014:13:08:06] ENGINE Bus STARTED
```

如您所见，服务器未启动。消失了：

```sh
[27/Apr/2014:13:04:08] ENGINE Serving on http://127.0.0.1:8080
```

## 每个请求函数

Web应用程序开发中最常见的任务之一是根据运行时上下文调整请求的处理。

在CherryPy中，这是通过所谓的[Tools](https://docs.cherrypy.org/en/latest/extend.html#tools)执行的。如果您熟悉 Django 或 WSGI 中间件，CherryPy 工具在本质上是相似的。它们添加了在请求/响应处理期间应用的功能。

### Hook point

挂接点（hook point）是请求/响应处理期间的一个点。

这是“挂接点”的简要概述，您可以将其挂在工具上：

- “on_start_resource””：最早的钩子；Request-Line 和 request 标头已处理，并且调度程序已设置 `request.handler` 和 `request.config`。
- “before_request_body”：连接到此处的工具将在处理请求正文之前运行。
- “before_handler”：在request.handler（调度程序发现的公开的可调用对象）被调用之前。
- “before_finalize”：在处理页面处理程序之后以及CherryPy格式化最终响应对象之前，将立即调用此钩子。例如，它可以帮助您检查页面处理程序可能返回的内容，并在需要时更改某些标头。
- “on_end_resource”：处理完成-可以返回响应了。这并不总是意味着request.handler（公开的页面处理程序）已执行！它可能是一个发电机。如果在页面处理程序生成响应主体之后绝对需要运行您的工具，则需要使用on_end_request代替，或者将response.body包装在生成器中，该生成器将在生成响应主体时应用您的工具。
- “before_error_response”：在设置错误响应（状态代码，正文）之前调用。
- “after_error_response”：在设置了错误响应（状态代码，主体）之后，并在错误响应最终确定之前立即调用。
- “on_end_request”：请求/响应对话已结束，所有数据均已写入客户端，仅此而已，请继续。

### Tools

工具是连接到挂钩点（hook point）的简单可调用对象（函数，方法，实现 `__call__` 方法的对象）。

下面是一个简单的工具，该工具附加到 `before_finalize` 挂接点，因此在调用页面处理程序之后：

```python
@cherrypy.tools.register('before_finalize')
def logit():
   print(cherrypy.request.remote.ip)
```

也可以手动创建和分配工具。装饰器注册等效于：

```python
cherrypy.tools.logit = cherrypy.Tool('before_finalize', logit)
```

使用该工具非常简单，如下所示：

```python
class Root:
    @cherrypy.expose
    @cherrypy.tools.logit()
    def index(self):
        return "hello world"
```

显然，可以使用 [其他常用方法](https://docs.cherrypy.org/en/latest/basics.html#perappconf) 声明该工具。

<p class="w3-pale-red">
工具的名称（技术上设置为 cherrypy.tools 的属性）不必与可调用名称匹配。但是，在配置中将使用该名称来引用该工具。
</p>

#### Stateful tools

工具机制确实非常灵活，并且可以实现丰富的按请求功能。

上一节中所示的 Straight 工具通常就足够了。但是，如果您的工作流在请求处理期间需要某种状态，则可能需要基于类的方法：

```python
import time

import cherrypy

class TimingTool(cherrypy.Tool):
    def __init__(self):
        cherrypy.Tool.__init__(self, 'before_handler',
                               self.start_timer,
                               priority=95)

    def _setup(self):
        cherrypy.Tool._setup(self)
        cherrypy.request.hooks.attach('before_finalize',
                                      self.end_timer,
                                      priority=5)

    def start_timer(self):
        cherrypy.request._time = time.time()

    def end_timer(self):
        duration = time.time() - cherrypy.request._time
        cherrypy.log("Page handler took %.4f" % duration)

cherrypy.tools.timeit = TimingTool()
```

该工具计算页面处理程序针对给定请求所花费的时间。它存储处理程序即将被调用的时间，并在处理程序返回其结果后立即记录时间差。

导入位是 [cherrypy.Tool](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cptools.html#cherrypy._cptools.Tool) 构造函数允许您注册到挂钩点，但是，要将同一个工具附加到另一个挂钩点，必须使用 [cherrypy.request.hooks.attach](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cprequest.html#cherrypy._cprequest.HookMap.attach) 方法。将工具应用于请求时，CherryPy 自动调用 [cherrypy.Tool._setup](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cptools.html#cherrypy._cptools.Tool._setup) 方法。

接下来，让我们看看如何使用我们的工具：

```python
class Root:
    @cherrypy.expose
    @cherrypy.tools.timeit()
    def index(self):
        return "hello world"
```

#### Tools ordering

由于您可以在同一个挂钩点上注册许多工具，因此您可能想知道将以什么顺序应用它们。

CherryPy 提供了确定性但又如此简单的机制。只需将优先级属性设置为1到100之间的值，较低的值将提供较高的优先级。

如果为多个工具设置了相同的优先级，则会按照在配置中声明它们的顺序来调用它们。

#### Toolboxes

所有内置的 CherryPy 工具都收集在一个名为 `cherrypy.tools` 的工具箱中。它响应 `"tools"` 名称空间中的配置条目。您可以如上所述将自己的工具添加到此工具箱。

如果需要更多的模块化，也可以制作自己的工具箱。例如，您可能创建了多个使用 JSON 的工具，或者可能发布了一套涵盖身份验证和授权的工具，每个人都可以从中受益（提示，提示）。创建一个新的工具箱很简单：

```python
import cherrypy

# Create a new Toolbox.
newauthtools = cherrypy._cptools.Toolbox("newauth")

# Add a Tool to our new Toolbox.
@newauthtools.register('before_request_body')
def check_access(default=False):
    if not getattr(cherrypy.request, "userid", default):
        raise cherrypy.HTTPError(401)
```

然后，在您的应用程序中，就像使用 `cherrypy.tools` 一样使用它，并带有向应用程序注册工具箱的附加步骤。注意，这样做会自动注册“newauth”配置名称空间。您可以在下面查看正在使用的配置条目：

```python
import cherrypy

class Root:
    @cherrypy.expose
    def default(self):
        return "Hello"

conf = {
   '/demo': {
       'newauth.check_access.on': True,
       'newauth.check_access.default': True,
    }
}

app = cherrypy.tree.mount(Root(), config=conf)
```

### Request parameters manipulation

HTTP 使用字符串在两个端点之间传送数据。但是，您的应用程序可能会更好地利用更丰富的对象类型。让每个页面处理程序反序列化数据并不是很容易理解，也不是关于维护的好主意，因此，将这种功能委托给工具是一种常见的模式。

例如，假设您在查询字符串中有一个用户 ID，并将一些用户数据存储到数据库中。您可以检索数据，创建一个对象并将其传递给页面处理程序，而不是传递给用户 ID。

```python
import cherrypy

class UserManager(cherrypy.Tool):
    def __init__(self):
        cherrypy.Tool.__init__(self, 'before_handler',
                               self.load, priority=10)

    def load(self):
        req = cherrypy.request

        # let's assume we have a db session
        # attached to the request somehow
        db = req.db

        # retrieve the user id and remove it
        # from the request parameters
        user_id = req.params.pop('user_id')
        req.params['user'] = db.get(int(user_id))

cherrypy.tools.user = UserManager()


class Root:
    @cherrypy.expose
    @cherrypy.tools.user()
    def index(self, user):
        return "hello %s" % user.name
```

换句话说，CherryPy 使您能够：

- 将不属于初始请求的数据注入到页面处理程序中
- 以及删除数据
- 将数据转换为另一个更有用的对象，以减轻页面处理程序本身的负担

## Tailored dispatchers

调度是为给定请求定位适当的页面处理程序的艺术。通常，分派基于请求的 URL，查询字符串以及有时基于请求的方法（GET，POST 等）进行。

基于此，CherryPy 已经附带了各种调度程序。

但是，在某些情况下，您将需要更多。这是一个调度程序的示例，该调度程序将始终确保传入的 URL 导致一个小写的页面处理程序。

```python
import random
import string

import cherrypy
from cherrypy._cpdispatch import Dispatcher

class StringGenerator:
   @cherrypy.expose
   def generate(self, length=8):
       return ''.join(random.sample(string.hexdigits, int(length)))

class ForceLowerDispatcher(Dispatcher):
    def __call__(self, path_info):
        return Dispatcher.__call__(self, path_info.lower())

if __name__ == '__main__':
    conf = {
        '/': {
            'request.dispatch': ForceLowerDispatcher(),
        }
    }
    cherrypy.quickstart(StringGenerator(), '/', conf)
```

运行此代码段后，请转到：

- `http://localhost:8080/generate?length=8`
- `http://localhost:8080/GENerAte?length=8`

在这两种情况下，您都将被引导到“生成页面处理程序”。如果没有我们的自制调度程序，第二个调度程序将失败并返回404错误（[RFC 7231＃section-6.5.4](https://tools.ietf.org/html/rfc7231.html#section-6.5.4)）。

### Tool or dispatcher?

在前面的示例中，为什么不简单地使用工具？好了，总是可以在找到页面处理程序之后尽快调用工具。在我们的示例中，为时已晚，因为默认调度程序甚至都找不到 `/GENerAte` 的匹配项。

通常存在一个调度程序，以确定为请求的资源提供服务的最佳页面处理程序。

另一方面，工具可以使请求的处理适应应用程序的运行时上下文和请求的内容。

通常，仅当您有非常特定的用例来定位最合适的页面处理程序时，才需要编写调度程序。否则，默认值可能就足够了。

## Request body processors

自从 3.2 版本发布以来，CherryPy 提供了一种非常优雅而强大的机制，可以根据其模仿类型来处理请求的正文。请参阅 [cherrypy._cpreqbody](https://docs.cherrypy.org/en/latest/pkg/cherrypy._cpreqbody.html#module-cherrypy._cpreqbody) 模块以了解如何实现自己的处理器。