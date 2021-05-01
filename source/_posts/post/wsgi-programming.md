---
title: WSGI 编程
lang: zh-CN
tags: WSGI
categories: 教程
abbrlink: ddf910af
date: 2021-03-09 09:39:32
description:
updated:
---

参考资料：[An Introduction to Web Programming with WSGI](http://www.phyast.pitt.edu/~micheles/python/europython07/talk.html)

## 关键概念

WSGI 是作为 Web 服务器与 Web 应用程序或应用框架之间的一种低级别的接口，以提升可移植 Web 应用开发的共同点。

WSGI 分为两个部分：一为“服务器”或“网关”，另一为“应用程序”或“应用框架”。在处理一个 WSGI 请求时，服务器会为应用程序提供环境信息及一个可回调的迭代器。当应用程序完成处理请求后，通过回调，将结果回传给服务器。

所谓的“**WSGI 中间件**”同时实现了 API 的两方，因此可以在 WSGI 服务器和 WSGI 应用之间起调解作用：从 Web 服务器的角度来说，中间件扮演应用程序，而从应用程序的角度来说，中间件扮演服务器。“中间件”组件可以执行以下功能：

- 重写环境变量后，根据目标 URL，将请求消息路由到不同的应用对象。
- 允许在一个进程中同时运行多个应用程序或应用框架。
- 负载均衡和远程处理，通过在网络上转发请求和响应消息。
- 进行内容后处理，例如应用 XSLT 样式表。

## 简单入门

WSGI 的应用接受参数 `(environ, start_response)`，其中 `environ` 是 dict-like 的环境，`start_response` 是一个 可回调函数，接受两个必须的参数，`status`（HTTP 状态）和 `response_headers`（响应消息的头）。

WSGI 应用必须是可回调的，可迭代的对象。

```python
from wsgiref.simple_server import make_server

# 定义 WSGI 应用
def simple_app(environ, start_response):
    '''一个相对简单的 WSGI 应用程序
    '''
    status = '200 OK'
    headers = [('Content-type', 'text/html; charset=utf-8')]
    start_response(status, headers)
    content = '<h1>欢迎进入 WSGI 世界！</h1>'
    return [content.encode('utf-8')]


port = 8000
# 启动服务器
with make_server('', port, simple_app) as httpd:
    print(f"正在端口 {port} 上服务...")
    httpd.serve_forever()
```

## WSGI 环境变量

下面了解一下 WSGI 的环境变量。

### 标准变量

<article>
    <dl>
        <dt class="w3-yellow">REQUEST_METHOD</dt>
        <dd>HTTP 请求方法，例如 <code>GET</code> 或 <code>POST</code>。这永远不能是一个空字符串，因此始终是必需的。
        </dd>
        <dt class="w3-yellow">SCRIPT_NAME</dt>
        <dd>
            请求 URL 的“path”的初始部分与应用程序对象相对应，以便应用程序知道其虚拟的“location”。如果应用程序对应于服务器的“root”，则它可以是一个空字符串。
        </dd>
        <dt class="w3-yellow">PATH_INFO</dt>
        <dd>
            请求网址的其余“path”，指定应用程序中请求目标的虚拟“location”。如果请求 URL 以应用程序根目录为目标，并且没有尾部斜杠，则该字符串可以为空。
        </dd>
        <dt class="w3-yellow">QUERY_STRING</dt>
        <dd>
            请求网址中“？”之后的部分（如果有）。可能为空或不存在。
        </dd>
        <dt class="w3-yellow">CONTENT_TYPE</dt>
        <dd>
            HTTP 请求中任何 <code>Content-Type</code> 字段的内容。 可能为空或不存在。
        </dd>
        <dt class="w3-yellow">CONTENT_LENGTH</dt>
        <dd>
            HTTP 请求中任何 <code>Content-Length</code> 字段的内容。可能为空或不存在。
        </dd>
        <dt class="w3-yellow">SERVER_NAME</dt>
        <dd></dd>
        <dt class="w3-yellow">SERVER_PORT</dt>
        <dd>
            与 <dfn>SCRIPT_NAME</dfn> 和 <dfn>PATH_INFO</dfn> 结合使用时，这些变量可用于完成 URL。但是请注意，<dfn>HTTP_HOST</dfn（如果存在）应优先于<dfn>SERVER_NAME</dfn> 使用，以重建请求 URL。有关更多详细信息，请参见下面的“URL重构”部分。<dfn>SERVER_NAME</dfn> 和 <dfn>SERVER_PORT</dfn> 永远不能为空字符串，因此始终是必需的。
        </dd>
        <dt class="w3-yellow">SERVER_PROTOCOL</dt>
        <dd>
            客户端用于发送请求的协议版本。通常，这类似于“HTTP/1.0”或“HTTP/1.1”，并且应用程序可以使用它来确定如何处理任何 HTTP 请求标头。（此变量可能应该称为<dfn>REQUEST_PROTOCOL</dfn>，因为它表示请求中使用的协议，不一定是服务器响应中将使用的协议。但是，为了与 CGI 兼容，我们必须保留现有名称。）
        </dd>
        <dt class="w3-yellow">HTTP_ Variables</dt>
        <dd>
            与客户端提供的 HTTP 请求标头相对应的变量（即，名称以 <code>HTTP_</code> 开头的变量）。这些变量的存在与否应与请求中适当的 HTTP 标头的存在与否相对应。
        </dd>
    </dl>
</article>

### WSGI 变量

<article>
    <dl>
        <dt class="w3-yellow">wsgi.version</dt>
        <dd>
            元组 (1, 0)，代表 WSGI 1.0 版。
        </dd>
        <dt class="w3-yellow">wsgi.url_scheme</dt>
        <dd>
            一个字符串，表示在其中调用应用程序的 URL 的“方案”部分。通常，适当时，其值为“http”或“https”。
        </dd>
        <dt class="w3-yellow">wsgi.input</dt>
        <dd>
            可以从中读取 HTTP 请求主体的输入流（类似文件的对象）。（服务器或网关可以根据应用程序的请求按需执行读取，或者可以预先读取客户端的请求主体并将其缓存在内存中或磁盘上，或者使用任何其他技术来提供这样的输入流，根据 偏好）。
        </dd>
        <dt class="w3-yellow">wsgi.errors</dt>
        <dd>
            可以向其中写入错误输出的输出流（类似文件的对象），目的是在标准化且可能集中的位置记录程序或其他错误。这应该是“文本模式”流； 即，应用程序应使用“n”作为行尾，并假定它将被服务器/网关转换为正确的行尾。
        </dd>
        <dd>
            对于许多服务器，<code>wsgi.errors</code> 将是服务器的主要错误日志。或者，它可以是 <code>sys.stderr</code> 或某种日志文件。服务器的文档应包括有关如何配置它或在何处找到记录的输出的说明。如果需要，服务器或网关可以向不同的应用程序提供不同的错误流。
        </dd>
        <dt class="w3-yellow">wsgi.multithread</dt>
        <dd>
            如果应用程序对象可以在同一进程中被另一个线程同时调用，则此值应评估为 <code>true</code>，否则应评估为 <code>false</code>。
        </dd>
        <dt class="w3-yellow">wsgi.multiprocess</dt>
        <dd>
            如果等效的应用程序对象可以同时被另一个进程调用，则此值应评估为 <code>true</code>，否则应评估为 <code>false</code>。
        </dd>
        <dt class="w3-yellow">wsgi.run_once</dt>
        <dd>
            如果服务器或网关期望（但不保证！）该应用程序仅在其包含过程的生命周期内被调用一次，则此值应评估为 <code>true</code>。通常，这仅适用于基于 CGI（或类似内容）的网关。
        </dd>
    </dl>
</article>

## 解析 Get 请求

下面一步步实现 Get 请求。

```python
from wsgiref.simple_server import make_server
from urllib.parse import parse_qs
from io import StringIO


def app(environ, start_response):
    query_str = environ['QUERY_STRING']
    # 解析 `%` 编码的查询字符串；返回查询字典，其中的值是列表
    d = parse_qs(query_str)
    request_method = environ['REQUEST_METHOD']
    with StringIO() as fp:
        print(f"查询字符串：{query_str}\n", file=fp)
        fp.write(f"查询方式：{request_method}\n")
        for k, v in d.items():
            # print(f"{k} = {v}\n", file=fp)
            fp.write(f"{k} = {v}\n")
        output = fp.getvalue()
    status = '200 OK'
    headers = [('Content-type', 'text/plain; charset=utf-8')]
    start_response(status, headers)
    yield output.encode("utf-8")


port = 8000
with make_server('', port, app) as httpd:
    print(f"正在端口 {port} 上服务...")
    httpd.serve_forever()
```

使用 `parse_qs` 函数解析 `%` 编码的查询字符串。返回查询字典，其中的值是列表。

比如 URL `http://localhost:8000/?年龄=10&hobbies=software&hobbies=tunning` 的输出结果：

```html
查询字符串：%E5%B9%B4%E9%BE%84=10&hobbies=software&hobbies=tunning

查询方式：GET

年龄 = ['10']
hobbies = ['software', 'tunning']
```

接着，编写表单的 GET。防止用户的输入产生脚本注入，可以使用 `escape` 函数。