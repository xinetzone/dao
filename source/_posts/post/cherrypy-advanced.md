---
title: CherryPy 高级教程
lang: zh-CN
tags: CherryPy
categories: 教程
abbrlink: c90e97b2
date: 2021-03-15 09:21:08
description:
updated:
---

CherryPy 支持这些部分将介绍的更高级的功能。

## 1 为页面处理程序设置别名

`cherrypy.expose` 装饰器提供的一个相当未知但有用的功能是支持别名。

```python
import random
import string

import cherrypy


class StringGenerator:
    @cherrypy.expose(['generer', 'generar'])
    def generate(self, length=8):
        return ''.join(random.sample(string.hexdigits, int(length)))


if __name__ == '__main__':
    cherrypy.quickstart(StringGenerator())
```

在此示例中，我们为页面处理程序创建本地化的别名。这意味着可以通过以下方式访问页面处理程序：

- `/generate`
- `/generer` (French)
- `/generar` (Spanish)

显然，您的别名可以满足您的需要。

<article class="w3-light-grey w3-card">
<p class="w3-pale-red">注意</p>
<p>
别名可以是单个字符串或它们的列表。
</p>
</article>

## 2 RESTful-style dispatching

术语 **RESTful URL** 有时用于谈论友好的 URL，这些 URL 很好地映射到应用程序公开的实体。

<article class="w3-light-grey w3-card">
<p class="w3-pale-red">重要</p>
<p>
我们不会就什么是 restful 的问题进行辩论，但是我们将展示两种在您的 CherryPy 应用程序中实现通常想法的机制。
</p>
</article>

假设您希望创建一个可以显示乐队和唱片的应用程序。您的应用程序可能具有以下 URL：

- `http://hostname/<artist>/`
- `http://hostname/<artist>/albums/<album_title>/`

### 2.1 The special `_cp_dispatch` method

`_cp_dispatch` 是您在任何控制器中声明的一种特殊方法，用于在 CherryPy 处理剩余段之前对剩余段进行处理。这使您能够删除，添加或以其他方式处理您想要的任何段，甚至可以完全更改其余部分。

```python
import cherrypy

class Band:
    def __init__(self):
        self.albums = Album()

    def _cp_dispatch(self, vpath):
        if len(vpath) == 1:
            cherrypy.request.params['name'] = vpath.pop()
            return self

        if len(vpath) == 3:
            cherrypy.request.params['artist'] = vpath.pop(0)  # /band name/
            vpath.pop(0) # /albums/
            cherrypy.request.params['title'] = vpath.pop(0) # /album title/
            return self.albums

        return vpath

    @cherrypy.expose
    def index(self, name):
        return 'About %s...' % name

class Album:
    @cherrypy.expose
    def index(self, artist, title):
        return 'About %s by %s...' % (title, artist)

if __name__ == '__main__':
    cherrypy.quickstart(Band())
```

该方法可以检查和操作段列表，在任何位置删除任何段或添加新段。然后将新的段列表发送到调度程序，该调度程序将使用它来查找适当的资源。

在上面的示例中，您应该能够转到以下 URL：

- `http://localhost:8080/nirvana/`
- `http://localhost:8080/nirvana/albums/nevermind/`

`/nirvana/` 段与乐队相关联， `/nevermind/` 段与专辑相关。

为了实现这一点，我们的 `_cp_dispatch` 方法基于以下想法：默认调度程序将 URL 与页面处理程序签名及其在处理程序树中的位置进行匹配。

在这种情况下，我们在URL中使用动态段（乐队和记录名称），将它们注入到请求参数中，然后将它们从段列表中删除，就好像它们从来没有出现过一样。

换句话说，`_cp_dispatch` 使它就像我们正在处理以下 URL 一样：

- `http://localhost:8080/?artist=nirvana`
- `http://localhost:8080/albums/?artist=nirvana&title=nevermind`

### 2.2 The popargs decorator

`cherrypy.popargs` 更直接，因为它为 CherryPy 否则无法解释的任何段命名。这使段与页面处理程序签名的匹配更加容易，并帮助 CherryPy 了解 URL 的结构。

```python
import cherrypy

@cherrypy.popargs('band_name')
class Band:
    def __init__(self):
        self.albums = Album()

    @cherrypy.expose
    def index(self, band_name):
        return 'About %s...' % band_name

@cherrypy.popargs('album_title')
class Album:
    @cherrypy.expose
    def index(self, band_name, album_title):
        return 'About %s by %s...' % (album_title, band_name)

if __name__ == '__main__':
    cherrypy.quickstart(Band())
```

它的工作方式与 `_cp_dispatch` 类似，但是如上所述，它更加明确和本地化。 它说：

- 取第一段并将其存储到名为 `band_name` 的参数中
- 再次获取第一段（因为我们删除了之前的第一段）并将其存储在名为 `album_title` 的参数中

请注意，装饰器接受的绑定不仅仅是一个绑定。例如：

```python
@cherrypy.popargs('album_title')
class Album:
    def __init__(self):
        self.tracks = Track()

@cherrypy.popargs('track_num', 'track_title')
class Track:
    @cherrypy.expose
    def index(self, band_name, album_title, track_num, track_title):
        ...
```

这将处理以下 URL：`http://localhost:8080/nirvana/albums/nevermind/tracks/06/polly`

最后，请注意如何将整个段堆栈传递给每个页面处理程序，以便获得完整的上下文。

## 3 Error handling

CherryPy 的 `HTTPError` 类支持在出现错误的情况下立即发出响应。

```python
class Root:
    @cherrypy.expose
    def thing(self, path):
        if not authorized():
            raise cherrypy.HTTPError(401, 'Unauthorized')
        try:
            file = open(path)
        except FileNotFoundError:
            raise cherrypy.HTTPError(404)
```

`HTTPError.handle` 是一个上下文管理器，它支持将应用程序中引发的异常转换为适当的HTTP响应，如第二个示例所示。

```python
class Root:
    @cherrypy.expose
    def thing(self, path):
        with cherrypy.HTTPError.handle(FileNotFoundError, 404):
            file = open(path)
```

## 4 Streaming the response body

CherryPy 处理 HTTP 请求，打包和解压缩低级详细信息，然后将控制权传递给应用程序的[页面处理程序](https://docs.cherrypy.org/en/latest/glossary.html#term-page-handler)，从而生成响应的正文。CherryPy 允许您以各种类型返回正文内容：字符串，字符串列表，文件。CherryPy 还允许您产生内容，而不是返回内容。当您使用“ yield”时，您还可以选择流式传输输出。

通常，不流输出更安全，更容易。因此，流输出默认情况下处于关闭状态。流输出并使用会话需要对会话锁的工作方式有充分的了解。

### 4.1 The “normal” CherryPy response process

当您从页面处理程序提供内容时，CherryPy 将管理 HTTP 服务器与您的代码之间的对话，如下所示：

![](https://docs.cherrypy.org/en/latest/_images/cpreturn.gif)

请注意，HTTP 服务器首先收集所有输出，然后立即将所有内容写入客户端：状态，标头和正文。这适用于静态页面或简单页面，因为可以在应用程序代码中或通过 CherryPy 框架随时更改整个响应。

### 4.2 CherryPy 如何使用“流输出”

当您将配置条目“response.stream”设置为True（并使用“yield”）时，CherryPy 将管理 HTTP 服务器与您的代码之间的对话，如下所示：

![](https://docs.cherrypy.org/en/latest/_images/cpyield.gif)

流式传输时，您的应用程序不会立即将原始内容传递回 CherryPy 或 HTTP 服务器。而是将其传递回生成器。那时，CherryPy 在生成器被消耗或产生任何输出之前完成状态和标头的确定。这是必要的，以允许 HTTP 服务器在头文件和正文部分可用时发送它们。

CherryPy 设置了状态和标头后，便将其发送到 HTTP 服务器，然后由 HTTP 服务器将其写出到客户端。从那时起，CherryPy 框架基本上不再使用，HTTP 服务器实际上直接从您的应用程序代码（您的页面处理程序方法）中请求内容。

因此，在流式传输时，如果页面处理程序中发生错误，CherryPy 将不会捕获它-HTTP 服务器将捕获它。因为标头（可能还有主体的一部分）已经被写入客户端，所以服务器无法知道一种安全的错误处理方法，因此只能关闭连接（当前的内置服务器实际上会写出一个简短错误） 消息，但可以更改此消息，并且不能保证可能与 CherryPy 一起使用的所有 HTTP 服务器的行为）。

此外，如果该处理程序方法是流生成器，则无法手动修改页面处理程序中的状态或标头，因为在将标头写入客户端之前，不会迭代该方法。这包括引发异常，例如 HTTPError，NotFound，InternalRedirect 和 HTTPRedirect。要在修改标题时使用流式生成器，您将必须返回与页面处理程序分离（或嵌入到页面处理程序中）的生成器。 例如：

```python
class Root:
    @cherrypy.expose
    def thing(self):
        cherrypy.response.headers['Content-Type'] = 'text/plain'
        if not authorized():
            raise cherrypy.NotFound()
        def content():
            yield "Hello, "
            yield "world"
        return content()
    thing._cp_config = {'response.stream': True}
```

流生成器很性感，但是它们会对 HTTP 造成破坏。CherryPy 允许您针对特定情况流输出：需要花费几分钟才能生成的页面，或需要部分内容的页面会立即输出到客户端。由于上面概述的问题，通常最好扁平化（缓冲）内容而不是流内容。仅当流式传输的好处胜过风险时，否则请执行其他操作。

## 5 响应时间

CherryPy 响应包括一个属性：`response.time`：响应开始的 `time.time()`

## 6 Deal with signals

[引擎插件](https://docs.cherrypy.org/en/latest/extend.html#busplugins)将自动实例化为 `cherrypy.engine.signal_handler`。但是，它仅由 `cherrypy.quickstart` 自动订阅。因此，如果您要进行信号处理并回调：

```python
tree.mount()
engine.start()
engine.block()
```

您必须自己添加，然后才能启动引擎：

```python
engine.signals.subscribe()
```

### 6.1 Windows Console Events

Microsoft Windows 使用控制台事件来传达某些信号，例如 Ctrl-C。在 Windows 平台上部署 CherryPy 需要 [Python for Windows Extensions](http://sourceforge.net/projects/pywin32/)，它会自动安装，并带有环境标记，从而具有额外的依赖性。 安装该程序后，CherryPy 将自动处理 Ctrl-C 和其他控制台事件（CTRL_C_EVENT，CTRL_LOGOFF_EVENT，CTRL_BREAK_EVENT，CTRL_SHUTDOWN_EVENT 和 CTRL_CLOSE_EVENT），从而关闭总线以准备退出进程。


## 7 保护服务器安全

本部分不作为保护 Web 应用程序或生态系统的完整指南。请查看 [OWASP](https://www.owasp.org/index.php/Main_Page) 提供的各种指南。

可以启用多种设置以使 CherryPy 页面更安全。这些包括：

- 传输数据：使用安全 Cookie
- 渲染页面：
    - 设置 HttpOnly cookie
    - 设置 XFrame 选项
    - 启用 XSS 保护
    - 设置内容安全策略（Content Security Policy）

一种简单的方法是使用工具设置标题，并用它包装整个 CherryPy 应用程序：

```python
import cherrypy

# set the priority according to your needs if you are hooking something
# else on the 'before_finalize' hook point.
@cherrypy.tools.register('before_finalize', priority=60)
def secureheaders():
    headers = cherrypy.response.headers
    headers['X-Frame-Options'] = 'DENY'
    headers['X-XSS-Protection'] = '1; mode=block'
    headers['Content-Security-Policy'] = "default-src 'self';"
```

了解有关[这些 headers](https://www.owasp.org/index.php/List_of_useful_HTTP_headers)的更多信息。

在配置文件（或您要启用该工具的任何其他位置）中：

```conf
[/]
tools.secureheaders.on = True
```

如果您使用会话，则还可以启用以下设置：

```conf
[/]
tools.sessions.on = True
# increase security on sessions
tools.sessions.secure = True
tools.sessions.httponly = True
```

如果使用 SSL，则还可以启用严格传输安全性：

```conf
# add this to secureheaders():
# only add Strict-Transport headers if we're actually using SSL; see the ietf spec
# "An HSTS Host MUST NOT include the STS header field in HTTP responses
# conveyed over non-secure transport"
# http://tools.ietf.org/html/draft-ietf-websec-strict-transport-sec-14#section-7.2
if (cherrypy.server.ssl_certificate != None and cherrypy.server.ssl_private_key != None):
    headers['Strict-Transport-Security'] = 'max-age=31536000'  # one year
```

接下来，您可能应该使用 [SSL](https://docs.cherrypy.org/en/latest/deploy.html#ssl)。

## 8 多个 HTTP 服务器支持

每当您启动引擎时，CherryPy 都会启动其自己的 HTTP 服务器。在某些情况下，您可能希望将应用程序托管在多个端口上。这很容易实现：

```python
from cherrypy._cpserver import Server
server = Server()
server.socket_port = 8090
server.subscribe()
```

您可以根据需要创建任意数量的服务器服务器实例，一旦订阅，它们将遵循 CherryPy 引擎的生命周期。

## 9 支持 WSGI 

CherryPy 支持 [PEP 333](https://www.python.org/dev/peps/pep-0333) 中定义的 WSGI 接口以及 [PEP 3333](https://www.python.org/dev/peps/pep-3333) 中的更新。这意味着：

- 您可以使用 CherryPy 服务器托管外部 WSGI 应用程序
- CherryPy 应用程序可以由另一个 WSGI 服务器托管

### 9.1 使您的 CherryPy 应用程序成为 WSGI 应用程序

可以从您的应用程序中获取 WSGI 应用程序，如下所示：

```python
import cherrypy
wsgiapp = cherrypy.Application(StringGenerator(), '/', config=myconf)
```

只需在任何支持 WSGI 的服务器中使用 `wsgiapp` 实例。

### 9.2 在 CherryPy 中托管外部 WSGI 应用程序

假设您具有可识别 WSGI 的应用程序，则可以使用 `cherrypy.tree.graft` 工具将其托管在 CherryPy 服务器中。

您不能将工具与外部WSGI应用程序一起使用。但是，您仍然可以从 [CherryPy bus](https://docs.cherrypy.org/en/latest/extend.html#buspattern) 中受益。

### 9.3 不需要 WSGI 接口吗？

默认的 CherryPy HTTP 服务器支持 PEP 333 和 PEP 3333 中定义的 WSGI 接口。但是，如果您的应用程序是纯 CherryPy 应用程序，则可以切换到完全绕过 WSGI 层的 HTTP 服务器。它将提供轻微的性能提升。

```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "Hello World!"

if __name__ == '__main__':
    from cherrypy._cpnative_server import CPHTTPServer
    cherrypy.server.httpserver = CPHTTPServer(cherrypy.server)

    cherrypy.quickstart(Root(), '/')
```

使用本地服务器，您将无法移植上一节中显示的WSGI应用程序。这样做会在运行时导致服务器错误。

## 10 支持 WebSocket

[WebSocket](http://tools.ietf.org/html/rfc6455) 是 HTML5 工作组应运而生的最新应用程序协议，可满足双向通信的需求。已经提出了各种黑客手段，例如 Comet，polling 等。

WebSocket 是从 HTTP 升级请求开始的套接字。执行升级后，基础套接字将保持打开状态，但不再在 HTTP 上下文中使用。相反，两个连接的端点都可以使用套接字将数据推送到另一端。

CherryPy 本身不支持 WebSocket，但是该功能由名为 [ws4py](https://github.com/Lawouach/WebSocket-for-Python) 的外部库提供。

## 11 Database 支持

CherryPy 不会捆绑任何数据库访问权限，但是它的体系结构使集成通用数据库接口（例如 [PEP 249](https://www.python.org/dev/peps/pep-0249) 中指定的 DB-API）变得容易。您也可以使用 [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)（例如 [SQLAlchemy](http://sqlalchemy.readthedocs.org/) 或 [SQLObject](https://pypi.python.org/pypi/SQLObject/)）。

您将在 [cherrypy-recipes](https://bitbucket.org/Lawouach/cherrypy-recipes/src/tip/web/database/sql_alchemy/) 上找到一个食谱，其中介绍了如何使用[插件](https://docs.cherrypy.org/en/latest/extend.html#busplugins)和[工具](https://docs.cherrypy.org/en/latest/extend.html#tools) 的组合来集成 SQLAlchemy。

## 12 HTML 模板支持

CherryPy不提供任何 HTML 模板，但其体系结构使集成一个模板变得容易。最受欢迎的是 [Mako](http://www.makotemplates.org/) 或 [Jinja2](http://jinja.pocoo.org/docs/)。

您将在[此处](https://bitbucket.org/Lawouach/cherrypy-recipes/src/tip/web/templating/)找到有关如何使用混合[插件](https://docs.cherrypy.org/en/latest/extend.html#busplugins)和[工具](https://docs.cherrypy.org/en/latest/extend.html#tools)进行集成的秘诀。

## 13 测试您的应用程序

与其他任何类型的代码一样，Web 应用程序也必须经过测试。CherryPy 提供了一个帮助程序类，以简化编写功能测试的过程。

这是基本回显应用程序的一个简单示例：

```python
import cherrypy
from cherrypy.test import helper

class SimpleCPTest(helper.CPWebCase):
    def setup_server():
        class Root:
            @cherrypy.expose
            def echo(self, message):
                return message

        cherrypy.tree.mount(Root())
    setup_server = staticmethod(setup_server)

    def test_message_should_be_returned_as_is(self):
        self.getPage("/echo?message=Hello%20world")
        self.assertStatus('200 OK')
        self.assertHeader('Content-Type', 'text/html;charset=utf-8')
        self.assertBody('Hello world')

    def test_non_utf8_message_will_fail(self):
        """
        CherryPy defaults to decode the query-string
        using UTF-8, trying to send a query-string with
        a different encoding will raise a 404 since
        it considers it's a different URL.
        """
        self.getPage("/echo?message=A+bient%F4t",
                     headers=[
                         ('Accept-Charset', 'ISO-8859-1,utf-8'),
                         ('Content-Type', 'text/html;charset=ISO-8859-1')
                     ]
        )
        self.assertStatus('404 Not Found')
```

如您所见，`test` 继承自该帮助程序类。您应该设置您的应用程序并将其按常规方式安装。然后，定义您的各种测试，并调用辅助方法 `getPage` 方法以执行请求。只需使用各种专门的 `assert*` 方法来验证您的工作流程和数据。

然后可以使用 `py.test` 运行测试，如下所示：

```sell
$ py.test -s test_echo_app.py
```

`-s` 是必需的，因为 CherryPy 类还包装了 `stdin` 和 `stdout`。如果没有该标志，则测试可能会在失败的断言上挂起，等待输入。

避免此问题的另一种选择（例如，如果您正在 IDE 中运行测试）是禁用默认情况下启用的交互模式。可以将 `WEBTEST_INTERACTIVE` 环境变量设置为 `False` 或 `0` 来禁用它。

如果您不想更改环境变量以仅运行一组测试，则也可以将帮助程序类作为子类，在该类中设置 `helper.CPWebCase.interactive = False`，然后从您的自定义类派生所有测试类：

```python
import cherrypy
from cherrypy.test import helper

class TestsBase(helper.CPWebCase):
    helper.CPWebCase.interactive = False
```

尽管它们是使用`unittest`模块支持的典型模式编写的，但它们并不是裸露的单元测试。实际上，将为您启动整个 CherryPy 堆栈并运行您的应用程序。如果要真正对 CherryPy 应用程序进行单元测试（即不必启动服务器），则可能需要看一下此[菜谱](https://bitbucket.org/Lawouach/cherrypy-recipes/src/tip/testing/unit/serverless/)。

`helper` 类是从 `unittest.TestCase` 类派生的。因此，从 pytest 运行时，相对于标准 pytest 测试存在一些限制，尤其是在将测试分组到测试类中时。您可以在[此页面](pytest-docs:unittest.html#pytest-features-in-unittest-testcase-subclasses)上找到更多详细信息。