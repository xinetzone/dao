---
title: wsgiref
lang: zh-CN
tags: WSGI
categories: 手册
abbrlink: bf07bad1
date: 2021-03-08 14:07:37
description:
updated:
---

WSGI（Web Server Gateway Interface, Web 服务器网关接口）是描述 Web 服务器如何与 Web 应用程序通信以及如何将 Web 应用程序链接在一起以处理一个 Request 的规范。

只有 Web 服务器和编程框架的作者才需要了解 WSGI 设计的每个细节和特殊情况。您无需了解 WSGI 的每个细节，仅安装 WSGI 应用程序或使用现有框架编写 Web 应用程序即可。

[wsgiref](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref) 是 WSGI 规范的参考实现，可用于将 WSGI 支持添加到 Web 服务器或框架。它提供了用于处理 WSGI 环境变量和响应标头的实用程序，用于实现 WSGI 服务器的基类，为 WSGI 应用程序提供服务的演示 HTTP 服务器以及用于检查 WSGI 服务器和应用程序是否符合 WSGI 规范的验证工具（[PEP 3333](https://www.python.org/dev/peps/pep-3333)）。

有关 WSGI 的更多信息，请参见 [wsgi.readthedocs.io](https://wsgi.readthedocs.io/)，以及教程和其他资源的链接。

Python 目前拥有大量的 Web 框架，比如 Zope, Quixote, Webware, SkunkWeb, PSO, 和 Twisted Web。大量的选择使得新手无所适从，因为总得来说，框架的选择都会限制 Web 服务器的选择。

WSGI 的目的是使得 Web 框架和 Web 服务器之间轻松互连，而不是创建一套新的 Web 框架。

## PEP3333 概述

WSGI 不是服务器，Python 模块，框架，API 或任何类型的软件。它只是服务器和应用程序进行通信的接口规范。在 PEP 3333 中指定了服务器和应用程序接口端。如果将应用程序（或框架或工具包）写入 WSGI 规范，则它将在写入该规范的任何服务器上运行。

可以堆叠 WSGI 应用程序（意味着符合 WSGI）。那些位于堆栈中间的组件称为**中间件**（middleware，一般为 app 的 Python 装饰器），它们必须同时实现 WSGI 接口的两端：应用程序和服务器。对于最上面的应用程序，它将充当服务器，对于下面的应用程序，则它将充当应用程序。

WSGI 服务器（意味着符合 WSGI）仅接收来自客户端的请求，将其传递给应用程序，然后将应用程序返回的响应发送给客户端。它什么也没做。所有 gory 详细信息必须由应用程序或中间件提供。

无需学习 WSGI 规范即可在框架或工具包之上构建应用程序。要使用中间件，除非对中间件已经集成在框架中，或者框架提供了某种包装器以将那些未集成的中间件集成在一起，否则必须对如何将它们与应用程序或框架堆叠在一起有最低限度的了解。

## [wsgiref.util](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.util) -- WSGI environment utilities

如果要在单个主机和端口上提供多个应用程序，则应创建一个 WSGI 应用程序，该应用程序解析 `PATH_INFO` 以选择为每个请求调用哪个应用程序。（例如，使用 `wsgiref.util` 中的 `shift_path_info` 函数。）

该模块提供了用于 WSGI 环境的各种实用程序函数。WSGI 环境是一个包含 HTTP 请求变量的字典，如 [PEP 3333](https://www.python.org/dev/peps/pep-3333) 中所述。所有带有 `environ` 参数的函数都希望提供符合 WSGI 的字典；请参阅 PEP 3333 了解详细规格。

- `wsgiref.util.guess_scheme(environ)`：通过检查 `environ` 字典中的 `HTTPS` 环境变量，返回有关 `wsgi.url_scheme` 应为 `"http"` 还是 `"https"` 的猜测。返回值是字符串。当创建包装 CGI 或类似 CGI 的协议（如 FastCGI）的网关时，此功能很有用。通常，当通过 SSL 接收到请求时，提供此类协议的服务器将包含一个 `HTTPS` 变量，其值为 `"1"`, `"yes"`, 或者 `"on"`。因此，如果找到该值，此函数将返回 `"https"`，否则返回 `"http"`。
- `wsgiref.util.application_uri(environ)`：返回应用程序的基本 URI（无 `PATH_INFO` 或 `QUERY_STRING`）
- `wsgiref.util.request_uri(environ, include_query=True)`：使用 PEP 3333 的 "URL Reconstruction" 部分中找到的算法，返回完整的请求 URI（可选包括查询字符串）。如果 `include_query` 为 `false`，则查询字符串不包含在结果 URI 中。
- `wsgiref.util.shift_path_info(environ)`：将单个名称从 `PATH_INFO` 移至 `SCRIPT_NAME`，然后返回该名称。`environ` 字典就地修改；如果您需要保留原始的 `PATH_INFO` 或 `SCRIPT_NAME`，请使用 `copy`。如果 `PATH_INFO` 中没有剩余的路径段，则返回 `None`。通常，此例程（routine）用于处理请求 URI 路径的每个部分，例如，将该路径视为一系列字典键。该例程修改了传入的环境，使其适合于调用位于目标 URI 处的另一个 WSGI 应用程序。例如，如果在 `/foo` 处有一个 WSGI 应用程序，并且请求 URI 路径是 `/foo/bar/baz`，在 `/foo` 处的 WSGI 应用程序调用 `shift_path_info` 函数，它将接收字符串 "bar"，并且环境将被更新以适合传递到 `/foo/bar/` 的 WSGI 应用程序。也就是说，`SCRIPT_NAME` 将从 `/foo` 更改为 `/foo/bar/`，`PATH_INFO` 将从 `/bar/baz` 更改为 `/baz`。当 `PATH_INFO` 只是 `"/"` 时，此例程将返回一个空字符串，并在 `SCRIPT_NAME` 后面附加一个斜杠，即使通常忽略空路径段，并且 `SCRIPT_NAME` 通常也不以斜杠结尾。这是故意的行为，以确保应用程序在使用此例程进行对象遍历时可以区分以 `/x` 结尾的 URI 与以 `/x/` 结尾的 URI 之间的区别。
- `wsgiref.util.setup_testing_defaults(environ)`：为了测试目的，使用简单的默认值更新环境。此例程添加了 WSGI 所需的各种参数，包括 `HTTP_HOST`，`SERVER_NAME`，`SERVER_PORT`，`REQUEST_METHOD`，`SCRIPT_NAME`，`PATH_INFO` 以及所有 PEP 3333 定义的 `wsgi.*` 变量。它仅提供默认值，并且不替换这些变量的任何现有设置。该例程旨在使 WSGI 服务器和应用程序的单元测试更容易设置虚拟环境。实际的 WSGI 服务器或应用程序都不应使用它，因为数据是伪造的！

用法示例：

```python
from wsgiref.util import setup_testing_defaults
from wsgiref.simple_server import make_server

# A relatively simple WSGI application. It's going to print out the
# environment dictionary after being updated by setup_testing_defaults


def simple_app(environ, start_response):
    setup_testing_defaults(environ)

    status = '200 OK'
    headers = [('Content-type', 'text/plain; charset=utf-8')]

    start_response(status, headers)

    ret = [(f"{key}: {value}\n").encode("utf-8")
           for key, value in environ.items()]
    return ret


with make_server('', 8000, simple_app) as httpd:
    print("Serving on port 8000...")
    httpd.serve_forever()
```

- `wsgiref.util.is_hop_by_hop(header_name)`：如果 `'header_name'` 是 [RFC 2616](https://tools.ietf.org/html/rfc2616.html) 定义的 HTTP/1.1 "Hop-by-Hop" Header，则返回 `True`。
- `class wsgiref.util.FileWrapper(filelike, blksize=8192)`：用于将类似文件的对象转换为迭代器的包装器（wrapper）。生成的对象支持 `__getitem __()` 和 `__iter __()` 迭代样式，以与 Python 2.1 和 Jython 兼容。当对象被迭代时，可选的 `blksize` 参数将反复传递给类似文件的对象的 `read()` 方法，以获取要产生的字节串。当 `read()` 返回一个空字节串时，迭代结束且不可恢复。如果 filelike 具有 `close` 方法，则返回的对象也将具有 `close` 方法，并且在调用时将调用该 Filelike 对象的 `close` 方法。

用法示例：

```python
from io import StringIO
from wsgiref.util import FileWrapper

# We're using a StringIO-buffer for as the file-like object
filelike = StringIO("This is an example file-like object"*10)
wrapper = FileWrapper(filelike, blksize=5)

for chunk in wrapper:
    print(chunk)
```


### 应用/框架端

应用对象（application object）就是一个简单的接受两个参数的可调用对象。不要混淆术语"object"就真的是一个对象实例。Python 中的函数、方法、类、实现了 `__call__` 的实例都是可以接受的应用对象。应用对象必须可以被多次调用，因为实际上所有服务器/网关（除了 CGI 网关）都会重复地调用它。

注意：我们总是讲 应用对象，不要误解为应用开发者需要使用 WSGI 作为 web 编程 API！应用开发者可以继续使用已经存在的、高级框架服务去开发他们的应用。WSGI 是一个为框架开发者和服务器开发者准备的工具，应用开发者不需要直接使用 WSGI。

应用对象必须：

1. accept two positional parameters:
    - A dictionary containing CGI like variables; and
    - a callback function that will be used by the application to send HTTP status code/message and HTTP headers to the server.
2. return the response body to the server as strings wrapped in an iterable.

比如：

```python
def simple_app(environ, start_response):
    """最简单的应用对象
    参数
    =======
    environ: 指向包含类似于 CGI 的环境变量的字典，该字典由服务器针对客户端收到的每个请求填充
    start_response: 服务器提供的回调函数，它以 HTTP 状态和标头为参数
    """
    # 使用提供的 environ 字典构建响应主体
    response_body = f'Request method: {environ["REQUEST_METHOD"]}'
    # HTTP 响应代码和消息
    status = '200 OK'
    # 客户端期望的 HTTP 标头必须将它们包装为元组对的列表：[(Header name, Header value)]
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(response_body)))]
    # 使用提供的函数将它们发送到服务器
    start_response(status, response_headers)
    # 返回响应主体。请注意，尽管它可以迭代，但它包装在列表中。
    return [response_body.encode("utf-8")]
```

下面是两个应用对象(application object)的示例。一个是函数(function)，一个是类(class)：

```python
HELLO_WORLD = b"Hello world!\n"


def simple_app(environ, start_response):
    """最简单的应用对象"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [HELLO_WORLD]


class AppClass:
    """产生相同的输出，但是用类实现。

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield HELLO_WORLD
```

## 服务器/网关端

服务器或者网关每次从 HTTP 客户端收到一个请求，就调用一次应用对象。为了描述方便，以下是一个简单的 CGI 网关，用 Python 函数实现，接收应用对象。注意这个简单的示例在错误处理方面相当简单，因为默认情况下，未捕获的异常会被 `dump` 到 `sys.stderr`，并且被 Web 服务器记入日志。

```python
import os
import sys

enc, esc = sys.getfilesystemencoding(), 'surrogateescape'


def unicode_to_wsgi(u):
    # Convert an environment variable to a WSGI "bytes-as-unicode" string
    return u.encode(enc, esc).decode('iso-8859-1')


def wsgi_to_bytes(s):
    return s.encode('iso-8859-1')


def run_with_cgi(application):
    environ = {k: unicode_to_wsgi(v) for k, v in os.environ.items()}
    environ['wsgi.input'] = sys.stdin.buffer
    environ['wsgi.errors'] = sys.stderr
    environ['wsgi.version'] = (1, 0)
    environ['wsgi.multithread'] = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once'] = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        out = sys.stdout.buffer

        if not headers_set:
            raise AssertionError("write() before start_response()")

        elif not headers_sent:
            # Before the first output, send the stored headers
            status, response_headers = headers_sent[:] = headers_set
            out.write(wsgi_to_bytes('Status: %s\r\n' % status))
            for header in response_headers:
                out.write(wsgi_to_bytes('%s: %s\r\n' % header))
            out.write(wsgi_to_bytes('\r\n'))

        out.write(data)
        out.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[1].with_traceback(exc_info[2])
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]

        # Note: error checking on the headers should happen here,
        # *after* the headers are set.  That way, if an error
        # occurs, start_response can only be re-called with
        # exc_info set.

        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```

## [wsgiref.headers](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.headers) -- WSGI response header tools

该模块提供了一个单独的类 `Headers`，用于使用类似于映射的接口方便地操作 WSGI 响应标头。

`class wsgiref.headers.Headers([headers])`：创建一个类似于映射的对象来包装响应的 headers，该标头必须是如 PEP 3333 中所述的标头 **名称/值** 元组的列表。标头的默认值为空列表。标头对象支持典型的映射操作，包括 `__getitem__()`, `get()`, `__setitem__()`, `setdefault()`, `__delitem__()` 和 `__contains__()`。对于这些方法中的每一个，键都是标头名称（不区分大小写地对待），并且值是与该标头名称关联的第一个值。设置 header 会删除该 header 的所有现有值，然后在包装的 header 列表的末尾添加一个新值。通常保留 header 的现有顺序，将新的 header 添加到包装列表的末尾。

与字典不同，当您尝试获取或删除包装的标头列表中没有的键时，标头对象不会引发错误。获取不存在的标头只会返回 `None`，而删除不存在的标头则不会执行任何操作。

`Headers` 对象还支持 `keys`，`values` 和 `items` 方法。如果存在多值标头，则 `keys` 和 `items` 方法返回的列表可以多次包含同一个键。`Headers` 对象的 `len` 方法与其 `items` 方法的长度相同，也与包装的 haeder 列表的长度相同。实际上，`items` 方法仅返回包装的 header 列表的副本。

在 `Headers` 对象上调用 `bytes()` 返回适合于作为 HTTP 响应标头传输的格式化字节串。每个标头均以其值放在一行中，并用冒号和空格分隔。每行以回车和换行符结尾，字节串以空行结尾。

除了其映射接口和格式设置功能之外，`Headers` 对象还具有以下方法用于查询和添加多值标头以及添加带有 MIME 参数的标头：

- `get_all(name)`：返回命名标头的所有值的列表。返回的列表将按照它们在原始 header 列表中出现或添加到此实例的顺序进行排序，并且可能包含重复项。删除并重新插入的所有字段始终附加到 header 列表中。如果不存在具有给定 `name` 的字段，则返回一个空列表。
- `add_header(name, value, **_params)`：添加一个（可能是多值的）header，并通过关键字参数指定可选的 MIME 参数。`name` 是要添加的 header 字段。关键字参数可用于设置标头字段的 MIME 参数。每个参数必须是字符串或 `None`。参数名称中的下划线会转换为破折号，因为破折号在 Python 标识符中是非法的，但是许多 MIME 参数名称都包含破折号。如果参数值是字符串，则以 `name="value"` 的形式将其添加到标头值参数中。如果为 `None`，则仅添加参数名称。（这用于没有值的 MIME 参数。）用法示例：

```python
h.add_header('content-disposition', 'attachment', filename='bud.gif')
```

上面将添加一个 header，如下所示：

```html
Content-Disposition: attachment; filename="bud.gif"
```


## [wsgiref.simple_server](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.simple_server) -- a simple WSGI HTTP server

该模块实现了一个简单的 HTTP 服务器（基于 `http.server`），该服务器为 WSGI 应用程序服务。每个服务器实例在给定的主机和端口上服务单个 WSGI 应用程序。

用法示例：

```python
from wsgiref.simple_server import make_server, demo_app

port = 8000  # 端口号
host = '' # IP 地址
# demo_app 是 处理函数
# 创建一个服务器
with make_server(host, port, demo_app) as httpd:
    print(f"在端口 {port} 上提供 HTTP 服务...")
    # 开始监听 HTTP 请求，响应请求直到进程终止
    httpd.serve_forever()
    # 备选方案：处理一个请求，然后退出
    httpd.handle_request()
```

下面详细说明此示例。

`wsgiref.simple_server.make_server(host, port, app, server_class=WSGIServer, handler_class=WSGIRequestHandler)` 创建一个侦听 `host` 和 `port` 的新 WSGI 服务器，接受 `app` 的连接。返回值是提供的 `server_class` 的实例，并将使用指定的 `handler_class` 处理请求。`app` 必须是 PEP 3333 定义的 WSGI 应用程序对象。

`wsgiref.simple_server.demo_app(environ, start_response)`：该函数是一个小型但完整的 WSGI 应用程序，它返回一个包含消息 `"Hello world!"` 的文本页面。以及 `environ` 参数中提供的键/值对的列表。这对于验证 WSGI 服务器（例如 `wsgiref.simple_server`）是否能够正确运行简单的 WSGI 应用程序很有用。

`class wsgiref.simple_server.WSGIServer(server_address, RequestHandlerClass)` 创建一个 `WSGIServer` 实例。`server_address` 应该是一个 `(host,port)` 元组，而 `RequestHandlerClass` 应该是将用于处理请求的 [http.server.BaseHTTPRequestHandler](https://docs.python.org/zh-cn/3.10/library/http.server.html#http.server.BaseHTTPRequestHandler) 的子类。通常不需要调用此构造函数，因为 `make_server` 函数可以为您处理所有详细信息。`WSGIServer` 是 `http.server.HTTPServer` 的子类，因此它的所有方法（例如 `serve_forever` 和`handle_request`）都可用。`WSGIServer` 还提供以下特定于 WSGI 的方法：

- `set_app(application)`：将可调用的应用程序设置为将接收请求的 WSGI 应用程序。
- `get_app()`：返回当前设置的可调用应用程序。

但是，通常不需要使用这些其他方法，因为 `set_app` 通常由 `make_server` 调用，而 `get_app` 主要是为了请求处理程序实例而存在。

`class wsgiref.simple_server.WSGIRequestHandler(request, client_address, server)` 为给定 `request`（即 `socket`），`client_address`（`(host,port)` 元组）和 `server`（`WSGIServer` 实例）创建 HTTP handler。您无需直接创建此类的实例； 它们由 `WSGIServer` 对象根据需要自动创建。但是，您可以将该类作为子类，并将其作为 `handler_class` 提供给 `make_server` 函数。在子类中重写的一些可能相关的方法：

- `get_environ()`：返回包含请求的 WSGI 环境的字典。缺省实现复制 WSGIServer 对象的 `base_environ` 词典属性的内容，然后添加从 HTTP 请求派生的各种标头。每次对此方法的调用都应返回一个新字典，其中包含 PEP 3333 中指定的所有相关 CGI 环境变量。
- `get_stderr()` 返回应该用作 `wsgi.errors` 流的对象。默认实现只是返回 `sys.stderr`。
- `handle()` 处理 HTTP 请求。缺省实现使用 [`wsgiref.handlers`](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.handlers) 类创建一个处理程序实例，以实现实际的 WSGI 应用程序接口。

## 中间件：可以与两端交互的组件

中间件（Middleware）就是一个简单对象：既可以作为服务端角色，响应应用对象；也可以作为应用对象，与服务器交互。除此之外，还有一些其他功能：

- 重写 environ，然后基于 URL，将请求对象路由给不同的应用对象。
- 支持多个应用或者框架顺序地运行于同一个进程中。
- 通过转发请求和响应，支持负载均衡和远程处理。
- 支持对内容做后处理(postprocessing)，比如处理一个 XSL 样式表文件。


中间件的灵魂是：对 WSGI 接口的服务器/网关端和 应用/框架端是透明的，不需要其他条件。

希望将中间件合并进应用的用户，将这个中间件传递给服务器即可，就好像这个中间件是一个应用对象；或者让中间件去调用应用对象，好像这个中间件就是服务器。当然，被中间件包装(wrap)的应用对象，实际上可能是另一个包装了另一个应用的中间件，以此类推，就创建了一个中间件栈（middleware stack）。

最重要的，中间件必须同时满足服务端和应用端的限制和条件。然而，在有些情况下，中间件需要的条件比单纯的服务端或者应用端更严格，这些点会在下面予以说明。

以下是一个中间件示例。他用 Joe Strout 的 `piglatin.py` 将 `text/plain` 的响应转换成 pig latin（注意：真正的中间件应该使用更加健壮的方式——应该检查内容的类型和内容的编码，同样这个简单的例子还忽略了一个单词可能被分割到一个块边界的可能性）。

```python
from foo_app import foo_app
from piglatin import piglatin


class LatinIter:

    """Transform iterated output to piglatin, if it's okay to do so

    Note that the "okayness" can change until the application yields
    its first non-empty bytestring, so 'transform_ok' has to be a mutable
    truth value.
    """

    def __init__(self, result, transform_ok):
        if hasattr(result, 'close'):
            self.close = result.close
        self._next = iter(result).__next__
        self.transform_ok = transform_ok

    def __iter__(self):
        return self

    def __next__(self):
        if self.transform_ok:
            return piglatin(self._next())   # call must be byte-safe on Py3
        else:
            return self._next()


class Latinator:

    # by default, don't transform output
    transform = False

    def __init__(self, application):
        self.application = application

    def __call__(self, environ, start_response):

        transform_ok = []

        def start_latin(status, response_headers, exc_info=None):

            # Reset ok flag, in case this is a repeat call
            del transform_ok[:]

            for name, value in response_headers:
                if name.lower() == 'content-type' and value == 'text/plain':
                    transform_ok.append(True)
                    # Strip content-length if present, else it'll be wrong
                    response_headers = [(name, value)
                                        for name, value in response_headers
                                        if name.lower() != 'content-length'
                                        ]
                    break

            write = start_response(status, response_headers, exc_info)

            if transform_ok:
                def write_latin(data):
                    write(piglatin(data))   # call must be byte-safe on Py3
                return write_latin
            else:
                return write

        return LatinIter(self.application(environ, start_latin), transform_ok)


# Run foo_app under a Latinator's control, using the example CGI gateway
run_with_cgi(Latinator(foo_app))
```

## [wsgiref.validate](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.validate) --- WSGI conformance checker

创建新的 WSGI 应用程序对象，框架，服务器或中间件（middleware）时，使用 `wsgiref.validate` 验证新代码的一致性可能很有用。该模块提供了创建 WSGI 应用程序对象的函数，该函数验证 WSGI 服务器或网关与 WSGI 应用程序对象之间的通信，以检查双方的协议一致性。

请注意，该实用程序不能保证完全符合 PEP 3333；该模块中没有错误并不一定意味着不存在错误。但是，如果此模块确实产生错误，则实际上可以确定服务器或应用程序不是 $100\%$ 兼容的。

该模块基于 Ian Bicking 的“Python Paste”库中的 `paste.lint` 模块。

`wsgiref.validate.validator(application)` 包装应用程序并返回一个新的 WSGI 应用程序对象。返回的应用程序会将所有请求转发到原始应用程序，并将检查该应用程序和调用它的服务器是否都符合 WSGI 规范和 RFC 2616。

任何检测到的不符合都会导致引发 `AssertionError`；但是请注意，如何处理这些错误取决于服务器。例如，`wsgiref.simple_server` 和其他基于 `wsgiref.handlers` 的服务器（不会重写错误处理方法以执行其他操作）将仅输出一条消息，指出发生了错误，并将回溯信息转储到 `sys.stderr` 或其他错误流。

该包装器还可以使用 [`warnings`](https://docs.python.org/zh-cn/3.10/library/warnings.html#module-warnings) 模块生成输出，以指示可疑的行为，但实际上可能不会被 PEP 3333 禁止。除非使用 Python 命令行选项或警告 API 禁止它们，否则任何此类警告都会写入 `sys.stderr`（不是 `wsgi.errors`，除非它们恰好是同一对象）。

用法示例：

```python
from wsgiref.validate import validator
from wsgiref.simple_server import make_server

# Our callable object which is intentionally not compliant to the
# standard, so the validator is going to break
def simple_app(environ, start_response):
    status = '200 OK'  # HTTP Status
    headers = [('Content-type', 'text/plain')]  # HTTP Headers
    start_response(status, headers)

    # This is going to break because we need to return a list, and
    # the validator is going to inform us
    return b"Hello World"

# This is the application wrapped in a validator
validator_app = validator(simple_app)

with make_server('', 8000, validator_app) as httpd:
    print("Listening on port 8000....")
    httpd.serve_forever()
```

## [wsgiref.handlers](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.handlers) -- server/gateway base classes

该模块提供用于实现 WSGI 服务器和网关的基本处理程序类。这些基类可以处理与 WSGI 应用程序进行通信的大部分工作，只要它们具有类似 CGI 的环境以及输入，输出和错误流即可。

