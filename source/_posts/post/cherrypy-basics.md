---
title: CherryPy 基础
lang: zh-CN
abbrlink: 5b871e48
date: 2021-03-12 09:11:05
description:
updated:
tags: CherryPy
categories: 教程
---

翻译自 [CherryPy Basics](https://docs.cherrypy.org/en/latest/basics.html#id3)

以下各节将引导您完成 CherryPy 应用程序的基础知识，并介绍一些基本概念。

## 1 一分钟的应用示例

您可以用 CherryPy 编写的最基本的应用程序几乎涉及其所有核心概念。

```python
import cherrypy


class Root:
    @cherrypy.expose
    def index(self):
        return "Hello World!"


if __name__ == '__main__':
    cherrypy.quickstart(Root(), '/')
```

首先，对于大多数任务，您将只需要第 1 行中所示的单个 `import` 语句即可。在讨论这些内容之前，让我们跳到第 11 行，该行显示如何使用 CherryPy 服务器应用程序托管您的应用程序，以及如何在 `/` 路径中将其与内置的 HTTP 服务器一起使用。

现在回到实际的应用程序。即使 CherryPy 没有强制要求，大多数时候您的应用程序仍将被编写为 Python 类。这些类的方法将由 CherryPy 调用以响应客户端请求。但是，CherryPy 需要意识到可以使用这种方法，我们说该方法需要公开。这正是 `cherrypy.expose` 装饰器在第 5 行中所做的。

执行此程序，在你的浏览器定位到：`http://127.0.0.1:8080` 可以预览效果。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
CherryPy 是一个小型框架，专注于一项任务：接收 HTTP 请求并找到与请求的 URL 匹配的最合适的 Python 函数或方法。与其他知名框架不同，CherryPy 不提供对数据库访问，HTML 模板或任何其他中间件漂亮功能的内置支持。
</p>
<p>
简而言之，一旦 CherryPy 找到并调用了公开的方法，作为开发人员，您就应自行提供工具来实现应用程序的逻辑。
</p>
<p>
CherryPy 认为您（开发人员）最了解。
</p>
</article>

<article class="w3-card w3-padding w3-margin-top w3-pale-red">
<p class="w3-text-yellow w3-wide w3-large">警告</p>
<p>
前面的示例演示了 CherryPy 接口的简单性，但是您的应用程序可能还会包含其他一些细节：静态服务，更复杂的结构，数据库访问等。这将在教程部分中进行开发。
<p>
</article>

CherryPy 是一个微型框架，但不是一个裸露的框架，它带有一些基本工具来涵盖您期望的常用用法。

## 2 [托管一个或多个应用程序](https://docs.cherrypy.org/en/latest/basics.html#id5)

Web 应用程序需要访问 HTTP 服务器。 CherryPy 提供了自己的，可投入生产的 HTTP 服务器。有两种方法来托管应用程序。

### 2.1 单一应用

最直接的方法是使用 `cherrypy.quickstart` 函数。它需要至少一个参数，即要托管的应用程序实例。另外两个设置是可选的。首先，可以从中访问应用程序的基本路径。其次，使用配置字典或文件来配置您的应用程序。

```python
cherrypy.quickstart(Blog())
cherrypy.quickstart(Blog(), '/blog')
cherrypy.quickstart(Blog(), '/blog', {'/': {'tools.gzip.on': True}})
```

第一个意味着您的应用程序将在 ` http://hostname:port/` 上可用，而另两个将使您的博客应用程序在 `http://hostname:port/blog` 上可用。此外，最后一个为应用程序提供了特定的设置。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
注意在第三种情况下，设置如何仍然相对于应用程序，而不是在何处可用，因此使用 {'/': ... } 而不是 {'/blog': ... }。
</p>
</article>

### 2.2 多元应用

`cherrypy.quickstart` 方法适用于单个应用程序，但缺乏使用服务器托管多个应用程序的能力。 为此，必须使用 `cherrypy.tree.mount` 函数，如下所示：

```python
cherrypy.tree.mount(Blog(), '/blog', blog_conf)
cherrypy.tree.mount(Forum(), '/forum', forum_conf)

cherrypy.engine.start()
cherrypy.engine.block()
```

本质上，`cherrypy.tree.mount` 具有与 `cherrypy.quickstart` 相同的参数：应用程序，托管路径段和配置。最后两行只是启动应用程序服务器。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">重要</p>
<p>
    <code>cherrypy.quickstart</code> 和 <code>cherrypy.tree.mount</code> 不是唯一的。例如，前几行可以写成：
</p>

```python
cherrypy.tree.mount(Blog(), '/blog', blog_conf)
cherrypy.quickstart(Forum(), '/forum', forum_conf)
```
</article>

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
您也可以 <a href="https://docs.cherrypy.org/en/latest/advanced.html#hostwsgiapp"> 托管外部 WSGI 应用程序</a>。
</p>
</article>

## 3 [Logging](https://docs.cherrypy.org/en/latest/basics.html#id8)

日志记录（Logging）是任何应用程序中的重要任务。CherryPy 将记录所有传入的请求以及协议错误。

为此，CherryPy 管理着两个记录器：

1. 记录每个传入请求的访问权限
2. 跟踪错误或其他应用程序级别消息的应用程序/错误日志

您的应用程序可以通过调用 `cherrypy.log` 来利用第二个记录器。

```python
cherrypy.log("hello there")
```

您还可以记录异常：

```python
try:
   ...
except Exception:
   cherrypy.log("kaboom!", traceback=True)
```

这两个日志都将写入由配置中的以下键标识的文件：

- 使用[通用日志格式](http://en.wikipedia.org/wiki/Common_Log_Format)的传入请求的 `log.access_file`
- 其他日志的 `log.error_file`

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">也可以参考</p>
<p>
有关 CherryPy 的日志记录体系结构的更多详细信息，请参阅 <a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy._cplogging.html#module-cherrypy._cplogging">cherrypy._cplogging</a> 模块。
</p>
</article>

### 3.1 [Disable logging](https://docs.cherrypy.org/en/latest/basics.html#id9)

您可能有兴趣禁用某个日志。

要禁用文件日志记录，只需在[全局配置](https://docs.cherrypy.org/en/latest/basics.html#globalsettings)中为 `log.access_file` 或 `log.error_file` 键对应更多值设置一个空字符串。

要禁用控制台日志记录，请将 `log.screen` 设置为 `False`。

```python
cherrypy.config.update({'log.screen': False,
                        'log.access_file': '',
                        'log.error_file': ''})
```

### 3.2 [与您的其他记录器一起玩](https://docs.cherrypy.org/en/latest/basics.html#id10)

您的应用程序可能显然已经在使用日志记录模块来跟踪应用程序级别的消息。下面是一个简单的设置示例。

```python
import logging
import logging.config

import cherrypy

logger = logging.getLogger()
db_logger = logging.getLogger('db')

LOG_CONF = {
    'version': 1,

    'formatters': {
        'void': {
            'format': ''
        },
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
    },
    'handlers': {
        'default': {
            'level':'INFO',
            'class':'logging.StreamHandler',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'cherrypy_console': {
            'level':'INFO',
            'class':'logging.StreamHandler',
            'formatter': 'void',
            'stream': 'ext://sys.stdout'
        },
        'cherrypy_access': {
            'level':'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'formatter': 'void',
            'filename': 'access.log',
            'maxBytes': 10485760,
            'backupCount': 20,
            'encoding': 'utf8'
        },
        'cherrypy_error': {
            'level':'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'formatter': 'void',
            'filename': 'errors.log',
            'maxBytes': 10485760,
            'backupCount': 20,
            'encoding': 'utf8'
        },
    },
    'loggers': {
        '': {
            'handlers': ['default'],
            'level': 'INFO'
        },
        'db': {
            'handlers': ['default'],
            'level': 'INFO' ,
            'propagate': False
        },
        'cherrypy.access': {
            'handlers': ['cherrypy_access'],
            'level': 'INFO',
            'propagate': False
        },
        'cherrypy.error': {
            'handlers': ['cherrypy_console', 'cherrypy_error'],
            'level': 'INFO',
            'propagate': False
        },
    }
}

class Root:
    @cherrypy.expose
    def index(self):

        logger.info("boom")
        db_logger.info("bam")
        cherrypy.log("bang")

        return "hello world"

if __name__ == '__main__':
    cherrypy.config.update({'log.screen': False,
                            'log.access_file': '',
                            'log.error_file': ''})
cherrypy.engine.unsubscribe('graceful', cherrypy.log.reopen_files)
    logging.config.dictConfig(LOG_CONF)
    cherrypy.quickstart(Root())
```

在此代码段中，我们创建一个[配置字典](https://docs.python.org/2/library/logging.config.html#logging.config.dictConfig)，然后将其传递到 `logging` 模块以配置记录器：

- 默认的根记录器与单个流处理程序关联
- db 后端的记录器，还有一个流处理程序

另外，我们重新配置 CherryPy 记录器：

- 顶级 `cherrypy.access` 记录器，将请求记录到文件中
- `cherrypy.error` 记录器，将其他所有内容记录到文件中并登录到控制台

当自动重新加载程序启动时，我们还阻止 CherryPy 尝试打开其日志文件。由于我们甚至都不让 CherryPy 首先打开它们，因此这不是严格要求的。但是，这样可以避免浪费时间在无用的东西上。

## 4 [Configuring](https://docs.cherrypy.org/en/latest/basics.html#id11)

CherryPy 带有细粒度的配置机制，可以在各种级别上进行设置。

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">也可以参考</p>
<p>
复习了基础知识后，请参考有关配置的<a href="https://docs.cherrypy.org/en/latest/config.html#configindepth">深入讨论</a>。
</p>
</article>

### 4.1 [Global server configuration](https://docs.cherrypy.org/en/latest/basics.html#id12)

要配置 HTTP 和应用程序服务器，请使用 `cherrypy.config.update` 方法。

```python
cherrypy.config.update({'server.socket_port': 9090})
```

`cherrypy.config` 对象是一个字典，`update` 方法将传递的字典合并到其中。您也可以改为传递文件（假设使用 `server.conf` 文件）：

```conf
[global]
server.socket_port: 9090
```

```python
cherrypy.config.update("server.conf")
```

<article class="w3-light-grey">
<p class="w3-red">警告</p>
<code>cherrypy.config.update</code> 并非用于配置应用程序。这是一个常见的错误。它用于配置服务器和引擎。
</article>

### 4.2 [Per-application configuration](https://docs.cherrypy.org/en/latest/basics.html#id13)

要配置您的应用程序，请在将应用程序与服务器关联时传入字典或文件。

```python
cherrypy.quickstart(myapp, '/', {'/': {'tools.gzip.on': True}})
```

或通过文件（例如，称为 `app.conf`）：

```conf
[/]
tools.gzip.on: True
```

```python
cherrypy.quickstart(myapp, '/', "app.conf")
```

尽管您可以全局方式定义大多数配置，但有时在代码中应用它们的位置定义它们很方便。

```python
class Root:
    @cherrypy.expose
    @cherrypy.tools.gzip()
    def index(self):
        return "hello world!"
```

上面的变体符号：

```python
class Root:
    @cherrypy.expose
    def index(self):
        return "hello world!"
    index._cp_config = {'tools.gzip.on': True}
```

两种方法具有相同的效果，因此请选择最适合您的样式的方法。

### 4.3 [Additional application settings](https://docs.cherrypy.org/en/latest/basics.html#id14)

您可以添加非特定于请求URL的设置，并从页面处理程序中检索它们，如下所示：

```conf
[/]
tools.gzip.on: True

[googleapi]
key = "..."
appid = "..."
```

```python
class Root:
    @cherrypy.expose
    def index(self):
        google_appid = cherrypy.request.app.config['googleapi']['appid']
        return "hello world!"

cherrypy.quickstart(Root(), '/', "app.conf")
```

## 5 [Cookies](https://docs.cherrypy.org/en/latest/basics.html#id15)

CherryPy 使用 Python 中的 `Cookie` 模块，尤其是 `Cookie.SimpleCookie` 对象类型来处理 Cookie。

- 要将 Cookie 发送到浏览器，请设置 `cherrypy.response.cookie[key] = value`。
- 要检索浏览器发送的 cookie，请使用 `cherrypy.request.cookie[key]`。
- 要删除 cookie（在客户端），必须发送其有效时间设置为 `0` 的 cookie：

```python
cherrypy.response.cookie[key] = value
cherrypy.response.cookie[key]['expires'] = 0
```

请务必注意，请求 Cookie 不会自动复制到响应 Cookie 中。客户端将在每个请求上发送相同的 cookie，因此每次都应填充 `cherrypy.request.cookie`。
但是服务器不需要在每次响应时都发送相同的 cookie；因此，`cherrypy.response.cookie` 通常为空。故而，当您希望“delete”（过期）cookie 时，必须首先设置 
`cherrypy.response.cookie[key] = value`，然后将其 `expires` 属性设置为 `0`。

扩展示例：

```python

class MyCookieApp:
    @cherrypy.expose
    def set(self):
        cookie = cherrypy.response.cookie
        cookie['cookieName'] = 'cookieValue'
        cookie['cookieName']['path'] = '/'
        cookie['cookieName']['max-age'] = 3600
        cookie['cookieName']['version'] = 1
        return "<html><body>Hello, I just sent you a cookie</body></html>"

    @cherrypy.expose
    def read(self):
        cookie = cherrypy.request.cookie
        res = """<html><body>Hi, you sent me %s cookies.<br />
                Here is a list of cookie names/values:<br />""" % len(cookie)
        for name in cookie.keys():
            res += "name: %s, value: %s<br>" % (name, cookie[name].value)
        return res + "</body></html>"

if __name__ == '__main__':
    cherrypy.quickstart(MyCookieApp(), '/cookie')
```

## 6 [Using sessions](https://docs.cherrypy.org/en/latest/basics.html#id16)

会话是开发人员用来识别用户并同步其活动的最常用机制之一。默认情况下，CherryPy 不激活会话，因为它不是必需的功能，要使其启用，只需在配置中添加以下设置：

```conf
[/]
tools.sessions.on: True
```

```python
cherrypy.quickstart(myapp, '/', "app.conf")
```

默认情况下，会话存储在 RAM 中，因此，如果重新启动服务器，则所有当前会话都将丢失。您可以将它们存储在 memcached 或文件系统中。

在应用程序中使用会话的操作如下：

```python
import cherrypy

@cherrypy.expose
def index(self):
    if 'count' not in cherrypy.session:
       cherrypy.session['count'] = 0
    cherrypy.session['count'] += 1
```

在此代码段中，每次调用索引页面处理程序时，当前用户的会话的 `'count'` 键都增加 1。

CherryPy 通过检查与请求一起发送的 cookie 来知道要使用哪个会话。此 Cookie 包含 CherryPy 用于从存储中加载用户会话的会话标识符。

<article class="w3-light-grey">
<p class="w3-yellow">也可以看看</p>
<p>
有关会话接口和实现的更多详细信息，请参阅 <a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy.lib.sessions.html#module-cherrypy.lib.sessions">cherrypy.lib.sessions</a> 模块。值得注意的是，您将了解会话到期。
</p>
</article>

### 6.1 [Filesystem backend](https://docs.cherrypy.org/en/latest/basics.html#id17)

使用文件系统很简单，不会在重新启动之间丢失会话。每个会话都保存在给定目录中的自己的文件中。

```conf
[/]
tools.sessions.on: True
tools.sessions.storage_class = cherrypy.lib.sessions.FileSession
tools.sessions.storage_path = "/some/directory"
```

### 6.2 [Memcached backend](https://docs.cherrypy.org/en/latest/basics.html#id18)

[Memcached](http://memcached.org/) 是 RAM 上流行的密钥库，它是分布式的，如果您想在运行 CherryPy 的进程之外共享会话，它是一个不错的选择。要求安装 Python [memcached 软件包](https://pypi.org/project/memcached)，这可以通过安装 `cherrypy[memcached_session]` 来指示。

```conf
[/]
tools.sessions.on: True
tools.sessions.storage_class = cherrypy.lib.sessions.MemcachedSession
```

### 6.3 [Other backends](https://docs.cherrypy.org/en/latest/basics.html#id19)

任何其他库都可以实现会话后端。只需将 `cherrypy.lib.sessions.Session` 子类化，并将该子类表示为 `tools.sessions.storage_class`。

## 7 [Static content serving](https://docs.cherrypy.org/en/latest/basics.html#id20)

CherryPy 可以提供您的静态内容，例如图像，JavaScript 和 CSS 资源等。

<article class="w3-light-grey w3-card">
<p class="w3-pale-red">笔记</p>
<p>
CherryPy 使用 <code>mimetypes</code> 模块来确定服务特定资源的最佳内容类型。如果选择无效，则可以如下设置更多的媒体类型：

```python
import mimetypes
mimetypes.types_map['.csv'] = 'text/csv'
```
</p>
</article>

### 7.1 [Serving a single file](https://docs.cherrypy.org/en/latest/basics.html#id21)

您可以按以下方式提供单个文件：

```conf
[/style.css]
tools.staticfile.on = True
tools.staticfile.filename = "/home/site/style.css"
```

CherryPy 将自动响应 URL，例如 `http://hostname/style.css`。

### 7.2 [Serving a whole directory](https://docs.cherrypy.org/en/latest/basics.html#id22)

服务整个目录类似于单个文件：

```conf
[/static]
tools.staticdir.on = True
tools.staticdir.dir = "/home/site/static"
```

假设您在 `static/js/my.js` 中有一个文件，CherryPy 将自动响应 URL，例如 `http://hostname/static/js/my.js`。

<article class="w3-light-grey w3-card">
<p class="w3-pale-yellow">注意</p>
<p>
CherryPy 始终需要将要服务的文件或目录的绝对路径。如果要配置多个静态部分，但它们位于同一根目录中，则可以使用以下快捷方式：
</p>

```conf
[/]
tools.staticdir.root = "/home/site"

[/static]
tools.staticdir.on = True
tools.staticdir.dir = "static"
```
</article>


### 7.3 [指定 `index` 文件](https://docs.cherrypy.org/en/latest/basics.html#id23)

默认情况下，指示未找到路径“/”的静态目录的根，CherryPy 将响应 404 错误。要指定索引文件，可以使用以下命令：

```conf
[/static]
tools.staticdir.on = True
tools.staticdir.dir = "/home/site/static"
tools.staticdir.index = "index.html"
```

假设您在 `static/index.html` 上有一个文件，CherryPy 将通过返回其内容自动响应 URL，例如 `http://hostname/static/`。

### 7.4 [允许下载文件](https://docs.cherrypy.org/en/latest/basics.html#id24)

使用 `"application/x-download"` 响应内容类型，您可以告诉浏览器应该将资源下载到用户的计算机上而不是显示。

例如，您可以编写一个页面处理程序，如下所示：

```python
from cherrypy.lib.static import serve_file

@cherrypy.expose
def download(self, filepath):
    return serve_file(filepath, "application/x-download", "attachment")
```

假设文件路径是您计算机上的有效路径，那么浏览器会将响应视为可下载的内容。

<article class="w3-light-grey w3-card">
<p class="w3-pale-red">警告</p>
<p>
上面的页面处理程序本身就有安全风险，因为可以访问服务器的任何文件（如果运行服务器的用户对其具有权限）。
</p>
</article>

## 8 [Dealing with JSON](https://docs.cherrypy.org/en/latest/basics.html#id25)

CherryPy 具有对请求和/或响应的 JSON 编码和解码的内置支持。

### 8.1 [Decoding request](https://docs.cherrypy.org/en/latest/basics.html#id26)

要使用 JSON 自动解码请求的内容，请执行以下操作：

```python
class Root:
    @cherrypy.expose
    @cherrypy.tools.json_in()
    def index(self):
        data = cherrypy.request.json
```

附加到请求的 `json` 属性包含解码后的内容。

### 8.2 [Encoding response](https://docs.cherrypy.org/en/latest/basics.html#id27)

要使用 JSON 自动编码响应的内容，请执行以下操作：

```python
    @cherrypy.expose
    @cherrypy.tools.json_out()
    def index(self):
        return {'key': 'value'}
```

CherryPy 将使用 JSON 对您的页面处理程序返回的所有内容进行编码。并非所有类型的对象都可以本地编码。

## 9 [Authentication](https://docs.cherrypy.org/en/latest/basics.html#id28)

CherryPy 支持以下两种非常简单的基于 HTTP 的身份验证机制，在 [RFC 7616](https://tools.ietf.org/html/rfc7616.html) 和 [RFC 7617](https://tools.ietf.org/html/rfc7617.html)（已淘汰 [RFC 2617](https://tools.ietf.org/html/rfc2617.html)）中进行了描述：Basic 和 Digest。众所周知，它们会触发浏览器的弹出窗口，询问用户其名称和密码。

### 9.1 [Basic](https://docs.cherrypy.org/en/latest/basics.html#id29)

基本身份验证是最简单的身份验证形式，但是由于用户的凭据已嵌入到请求中，因此它不是安全的形式。除非您在 SSL 上或封闭的网络中运行，否则我们建议不要使用它。

```python
from cherrypy.lib import auth_basic

USERS = {'jon': 'secret'}

def validate_password(realm, username, password):
    if username in USERS and USERS[username] == password:
       return True
    return False

conf = {
   '/protected/area': {
       'tools.auth_basic.on': True,
       'tools.auth_basic.realm': 'localhost',
       'tools.auth_basic.checkpassword': validate_password,
       'tools.auth_basic.accept_charset': 'UTF-8',
    }
}

cherrypy.quickstart(myapp, '/', conf)
```

简而言之，您必须提供一个由 CherryPy 调用的函数，该函数传递从请求中解码的用户名和密码。

该函数可以从其必须具有的任何源中读取其数据：文件，数据库，内存等。

### 9.2 [Digest](https://docs.cherrypy.org/en/latest/basics.html#id30)

摘要式身份验证的不同之处在于，凭据不是由请求携带的，因此它比基本身份验证更为安全。

CherryPy 的摘要支持具有与上述基本支持类似的界面。

```python
from cherrypy.lib import auth_digest

USERS = {'jon': 'secret'}

conf = {
   '/protected/area': {
        'tools.auth_digest.on': True,
        'tools.auth_digest.realm': 'localhost',
        'tools.auth_digest.get_ha1': auth_digest.get_ha1_dict_plain(USERS),
        'tools.auth_digest.key': 'a565c27146791cfb',
        'tools.auth_digest.accept_charset': 'UTF-8',
   }
}

cherrypy.quickstart(myapp, '/', conf)
```

### 9.3 [SO_PEERCRED](https://docs.cherrypy.org/en/latest/basics.html#id31)

UNIX 文件和抽象套接字还具有低级身份验证。这是启用它的方式：

```conf
[global]
server.peercreds: True
server.peercreds_resolve: True
server.socket_file: /var/run/cherrypy.sock
```

`server.peercreds` 允许查找连接的进程 ID，用户 ID 和组ID。它们可以作为 WSGI 环境变量进行访问：

- X_REMOTE_PID
- X_REMOTE_UID
- X_REMOTE_GID

`server.peercreds_resolve` 将其解析为用户名和组名。它们可以作为 WSGI 环境变量进行访问：

- X_REMOTE_USER and REMOTE_USER
- X_REMOTE_GROUP

## 10 [Favicon](https://docs.cherrypy.org/en/latest/basics.html#id32)

CherryPy 使用静态文件工具将其自己的甜红色 cherrypy 作为默认图标提供服务。您可以按以下方式提供自己的网站图标：

```python
import cherrypy

class HelloWorld:
   @cherrypy.expose
   def index(self):
       return "Hello World!"

if __name__ == '__main__':
    cherrypy.quickstart(HelloWorld(), '/',
        {
            '/favicon.ico':
            {
                'tools.staticfile.on': True,
                'tools.staticfile.filename': '/path/to/myfavicon.ico'
            }
        }
    )
```

有关更多详细信息，请参阅[静态服务](https://docs.cherrypy.org/en/latest/basics.html#staticontent)部分。

您还可以使用文件进行配置：

<article>
    <link rel="stylesheet" href="https://xinetzone.github.io/w3css/4/w3.css">
    <link rel="stylesheet" href="https://xinetzone.github.io/xinet-css/tabs.css">
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--1" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--1">conf</label>
        <div class="tab-content w3-padding">
            ```conf
            [/favicon.ico]
            tools.staticfile.on: True
            tools.staticfile.filename: "/path/to/myfavicon.ico"
            ```
        </div>
        <input id="tab-set--0-input--2" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--2">Python</label>
        <div class="tab-content w3-padding">
            ```python
            import cherrypy

            class HelloWorld:
            @cherrypy.expose
            def index(self):
                return "Hello World!"

            if __name__ == '__main__':
                cherrypy.quickstart(HelloWorld(), '/', "app.conf")
            ```
        </div>
    </div>
</article>





