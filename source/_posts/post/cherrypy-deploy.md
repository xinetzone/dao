---
title: CherryPy 部署
lang: zh-CN
tags: CherryPy
categories: 教程
abbrlink: 5de53779
date: 2021-03-16 10:50:15
description:
updated:
---

CherryPy 独立存在，但作为应用程序服务器，它通常位于共享或复杂的环境中。因此，在反向代理后面运行 CherryPy 或使用其他服务器托管应用程序的情况并不少见。

多年来，CherryPy 的服务器已被证明可靠且速度足够快。如果您收到的访问量是平均水平，那么它就可以很好地完成自己的任务。尽管如此，通常还是将静态内容的提供委托给功能更强大的服务器，例如 [nginx](http://nginx.org/) 或 CDN。

## Run as a daemon

CherryPy 使您可以使用传统的双叉（double-fork）轻松地将当前流程与父环境脱钩：

```python
from cherrypy.process.plugins import Daemonizer
d = Daemonizer(cherrypy.engine)
d.subscribe()
```

[engine plugin](https://docs.cherrypy.org/en/latest/extend.html#busplugins) 仅在提供 `fork()` 的 Unix 和类似系统上可用。

如果在派生的子代中发生启动错误，则父进程的返回代码仍将为 `0`。初始守护进程中的错误仍将返回正确的退出代码，但派生后的错误将不会返回。因此，如果您使用此插件进行守护进程，请不要使用返回码作为该过程是否已完全启动的准确指示。实际上，该返回码仅指示该进程是否成功完成了第一个派生。

插件采用可选参数来重定向标准流：`stdin`，`stdout` 和 `stderr`。默认情况下，所有这些都重定向到 `/dev/null`，但是您可以随意将它们发送到日志文件或其他地方。

<p class="w3-pale-red">
您应该小心，不要在插件运行之前启动任何线程。如果这样做，插件将发出警告，因为“...在 fork() 调用与 exec 函数调用之间需要某些资源的调用函数的效果未定义”（<a href="http://www.opengroup.org/onlinepubs/000095399/functions/fork.html">ref</a>）。因此，服务器插件以优先级75运行（启动工作线程），该优先级高于守护程序的默认优先级 65。
</p>

## Run as a different user

使用此 [engine plugin](https://docs.cherrypy.org/en/latest/extend.html#busplugins) 以 root 用户身份启动 CherryPy 网站（例如，在特权端口（如 80）上侦听），然后将特权降低到更受限制的位置。

此插件的“start”侦听器的优先级略高于 `server.start` 的优先级，以便于最常见的使用：从低端口（需要 root）启动，然后移交给其他用户。

```python
DropPrivileges(cherrypy.engine, uid=1000, gid=1000).subscribe()
```

## PID files

PIDFile 引擎插件非常简单：它在启动时将进程 ID 写入文件，并在退出时将其删除。您必须提供“pidfile”参数，最好是绝对路径：

```python
PIDFile(cherrypy.engine, '/var/run/myapp.pid').subscribe()
```

## Systemd socket activation

套接字激活是一项 systemd 功能，它允许设置系统，以便 systemd 可以坐在端口上并“on demand”启动服务（有点像 inetd 和 xinetd 一样）。

CherryPy 具有内置的套接字激活支持，如果从 systemd 服务文件运行，它将检测 **LISTEN_PID** 环境变量，以知道应将 fd 3 视为传递的套接字。

要了解有关套接字激活的更多信息，请访问：<http://0pointer.de/blog/projects/socket-activation.html>。

## Control via Supervisord

[Supervisord](http://supervisord.org/) 是一个功能强大的过程控制和管理工具，可以围绕过程监视执行许多任务。

以下是您的 CherryPy 应用程序的简单supervisor配置。

```conf
[unix_http_server]
file=/tmp/supervisor.sock

[supervisord]
logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock

[program:myapp]
command=python server.py
environment=PYTHONPATH=.
directory=.
```

这可以通过 `server.py` 模块作为应用程序入口点来控制您的服务器。

```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "Hello World!"


cherrypy.config.update({'server.socket_port': 8090,
                        'engine.autoreload.on': False,
                        'log.access_file': './access.log',
                        'log.error_file': './error.log'})
cherrypy.quickstart(Root())
```

要考虑配置（假设配置已保存在名为 `supervisor.conf` 的文件中）：

```sh
$ supervisord -c supervisord.conf
$ supervisorctl update
```

现在，您可以将浏览器指向 `http://localhost:8090/ `，它将显示 Hello World!。

要停止 supervisor，请键入：

```sh
$ supervisorctl shutdown
```

显然，这将关闭您的应用程序。

### SSL support

您可能要使用 [Qualys, Inc](https://www.ssllabs.com/ssltest/index.html) 的服务来测试服务器的 SSL。

CherryPy 可以使用 SSL 加密连接以创建 https 连接。这样可以确保您的网络流量安全。这是如何做。

1. 生成私钥。我们将使用 openssl 并遵循 [OpenSSL Keys HOWTO](https://www.openssl.org/docs/HOWTO/keys.txt)。

```sh
$ openssl genrsa -out privkey.pem 2048
```

您可以创建需要使用密码的密钥，也可以创建没有密码的密钥。使用密码保护私钥更为安全，但是每次使用密钥时都需要输入密码。例如，启动或重新启动CherryPy服务器时，您可能必须输入密码。根据您的设置，这可能可行，也可能不可行。

如果要输入密码，请将-aes128，-aes192或-aes256开关之一添加到上述命令。您不应该使用任何DES，3DES或SEED算法来保护密码，因为它们不安全。

SSL实验室建议使用2048位RSA密钥以提高安全性（请参阅最后的参考部分）。

2.生成证书。我们将使用 openssl 并遵循 [OpenSSL Certificates HOWTO](https://www.openssl.org/docs/HOWTO/certificates.txt)。让我们从一个自签名证书开始进行测试：

```sh
$ openssl req -new -x509 -days 365 -key privkey.pem -out cert.pem
```

然后，openssl 会问您一系列问题。您可以输入任何适用的值，或将大多数字段保留为空白。您必须填写的一个字段是“Common Name”：输入用于访问您的网站的主机名。如果您只是创建要在自己的计算机上进行测试的证书，并且通过在浏览器中键入“localhost”来访问服务器，请输入通用名称“localhost”。

3. 确定您要使用 Python 的内置 SSL 库还是 pyOpenSSL 库。CherryPy 都支持。
    - 内置的。 要使用 Python 的内置 SSL，请将以下行添加到 CherryPy 配置中：
    ```sh
    cherrypy.server.ssl_module = 'builtin'
    ```
    - pyOpenSSL。因为在首次创建 CherryPy 时 Python 没有内置的 SSL 库，所以默认设置是使用 pyOpenSSL。要使用它，您需要安装它（我们建议您先安装 cython）：

    ``sh
    $ pip install cython, pyOpenSSL
    ```

4. 在 CherryPy 配置中添加以下行，以指向您的证书文件：

```python
cherrypy.server.ssl_certificate = "cert.pem"
cherrypy.server.ssl_private_key = "privkey.pem"
```

5. 如果手边有证书链，也可以指定它：

```python
cherrypy.server.ssl_certificate_chain = "certchain.perm"
```

6. 正常启动 CherryPy 服务器。请注意，如果您在本地调试和/或使用自签名证书，则浏览器可能会向您显示安全警告。

## WSGI servers

### Embedding into another WSGI framework

尽管 CherryPy 带有非常可靠且足够快的 HTTP 服务器，但是您可能希望将 CherryPy 应用程序集成到其他框架中。为此，我们将受益于 [**PEP 333**](https://www.python.org/dev/peps/pep-0333) 和 [**PEP 3333**](https://www.python.org/dev/peps/pep-3333) 中定义的 WSGI 接口。

请注意，在将 CherryPy 嵌入第三方 WSGI 服务器中时，应遵循一些基本规则：

- 如果您依赖发布的“main”channel（就像发生在 CherryPy 的 mainloop 中一样），则应该找到一种在其他框架的 mainloop 中发布到该channel的方法。
- 启动 CherryPy 的引擎。这将发布到总线的“start”通道。

```python
cherrypy.engine.start()
```

- 停止 CherryPy 的引擎。 这将发布到总线的“stop”通道。

```python
cherrypy.engine.stop()
```

- 不要调用 `cherrypy.engine.block()`。
- 禁用内置的 HTTP 服务器，因为它将不被使用。

```python
cherrypy.server.unsubscribe()
```

- 禁用 `autoreload`。通常其他框架对此反应不佳，或者有时会提供相同的功能。

```python
cherrypy.config.update({'engine.autoreload.on': False})
```

- 禁用 CherryPy 信号处理。可能不需要这样做，这取决于其他框架如何处理它们。

```python
cherrypy.engine.signals.subscribe()
```

- 使用 `"embedded"` 环境配置方案。

```python
cherrypy.config.update({'environment': 'embedded'})
```

本质上，这将禁用以下功能：

- Stdout logging
- Autoreloader
- Configuration checker
- Headers logging on error
- Tracebacks in error
- Mismatched params error during dispatching
- Signals (SIGHUP, SIGTERM)

### Tornado

您可以按照以下方式使用 [tornado](http://www.tornadoweb.org/) HTTP 服务器：

```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "Hello World!"

if __name__ == '__main__':
    import tornado
    import tornado.httpserver
    import tornado.wsgi

    # our WSGI application
    wsgiapp = cherrypy.tree.mount(Root())

    # Disable the autoreload which won't play well
    cherrypy.config.update({'engine.autoreload.on': False})

    # let's not start the CherryPy HTTP server
    cherrypy.server.unsubscribe()

    # use CherryPy's signal handling
    cherrypy.engine.signals.subscribe()

    # Prevent CherryPy logs to be propagated
    # to the Tornado logger
    cherrypy.log.error_log.propagate = False

    # Run the engine but don't block on it
    cherrypy.engine.start()

    # Run thr tornado stack
    container = tornado.wsgi.WSGIContainer(wsgiapp)
    http_server = tornado.httpserver.HTTPServer(container)
    http_server.listen(8080)
    # Publish to the CherryPy engine as if
    # we were using its mainloop
    tornado.ioloop.PeriodicCallback(lambda: cherrypy.engine.publish('main'), 100).start()
    tornado.ioloop.IOLoop.instance().start()
```

### Twisted

您可以按照以下方式使用 [Twisted](https://twistedmatrix.com/) HTTP 服务器：

```python
import cherrypy

from twisted.web.wsgi import WSGIResource
from twisted.internet import reactor
from twisted.internet import task

# Our CherryPy application
class Root:
    @cherrypy.expose
    def index(self):
        return "hello world"

# Create our WSGI app from the CherryPy application
wsgiapp = cherrypy.tree.mount(Root())

# Configure the CherryPy's app server
# Disable the autoreload which won't play well
cherrypy.config.update({'engine.autoreload.on': False})

# We will be using Twisted HTTP server so let's
# disable the CherryPy's HTTP server entirely
cherrypy.server.unsubscribe()

# If you'd rather use CherryPy's signal handler
# Uncomment the next line. I don't know how well this
# will play with Twisted however
#cherrypy.engine.signals.subscribe()

# Publish periodically onto the 'main' channel as the bus mainloop would do
task.LoopingCall(lambda: cherrypy.engine.publish('main')).start(0.1)

# Tie our app to Twisted
reactor.addSystemEventTrigger('after', 'startup', cherrypy.engine.start)
reactor.addSystemEventTrigger('before', 'shutdown', cherrypy.engine.exit)
resource = WSGIResource(reactor, reactor.getThreadPool(), wsgiapp)
```

请注意，我们是如何将总线方法附加到 Twisted 自己的生命周期的。

将该代码保存到名为 `cptw.py` 的模块中，然后如下运行：

```sh
$ twistd -n web --port 8080 --wsgi cptw.wsgiapp
```

### uwsgi

您可以按照以下方式使用 [uwsgi](http://projects.unbit.it/uwsgi/) HTTP 服务器：

```python
import cherrypy

# Our CherryPy application
class Root:
    @cherrypy.expose
    def index(self):
        return "hello world"

cherrypy.config.update({'engine.autoreload.on': False})
cherrypy.server.unsubscribe()
cherrypy.engine.start()

wsgiapp = cherrypy.tree.mount(Root())
```

将其保存到一个名为 `mymod.py` 的 Python 模块中，并如下运行：

```sh
$ uwsgi --socket 127.0.0.1:8080 --protocol=http --wsgi-file mymod.py --callable wsgiapp
```

## Virtual Hosting

CherryPy 支持虚拟主机。它是通过一个调度程序来完成的，该调度程序根据请求的域来定位适当的资源。

下面是一个简单的示例：

```python
import cherrypy

class Root:
    def __init__(self):
        self.app1 = App1()
        self.app2 = App2()

class App1:
    @cherrypy.expose
    def index(self):
        return "Hello world from app1"

class App2:
    @cherrypy.expose
    def index(self):
        return "Hello world from app2"

if __name__ == '__main__':
    hostmap = {
        'company.com:8080': '/app1',
        'home.net:8080': '/app2',
    }

    config = {
        'request.dispatch': cherrypy.dispatch.VirtualHost(**hostmap)
    }

    cherrypy.quickstart(Root(), '/', {'/': config})
```

在此示例中，我们声明两个域及其端口：

- `company.com:8080`
- `home.net:8080`

多亏了 `cherrypy.dispatch.VirtualHost` 调度程序，我们告诉 CherryPy 当请求到达时要调度到哪个应用程序。调度程序查找请求的域并调用相应的应用程序。

要测试此示例，只需将以下规则添加到您的 `hosts` 文件中：

```hosts
127.0.0.1       company.com
127.0.0.1       home.net
```

## Reverse-proxying

nginx 是一种快速，现代化的 HTTP 服务器，占地面积小。作为对诸如 CherryPy 之类的应用程序服务器的反向代理，它是一种流行的选择。

本节将不介绍 nginx 提供的全部功能。相反，它只会为您提供一个基本的配置，可以作为一个很好的起点。

```nginx
upstream apps {
   server 127.0.0.1:8080;
   server 127.0.0.1:8081;
}

gzip_http_version 1.0;
gzip_proxied      any;
gzip_min_length   500;
gzip_disable      "MSIE [1-6]\.";
gzip_types        text/plain text/xml text/css
                  text/javascript
                  application/javascript;

server {
   listen 80;
   server_name  www.example.com;

   access_log  /app/logs/www.example.com.log combined;
   error_log  /app/logs/www.example.com.log;

   location ^~ /static/  {
      root /app/static/;
   }

   location / {
      proxy_pass         http://apps;
      proxy_redirect     off;
      proxy_set_header   Host $host;
      proxy_set_header   X-Real-IP $remote_addr;
      proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Host $server_name;
   }
}
```

编辑此配置以匹配您自己的路径。然后，将此配置保存到 `/etc/nginx/conf.d/` 下的文件中（假设Ubuntu）。文件名无关。然后运行以下命令：

```sh
$ sudo service nginx stop
$ sudo service nginx start
```

希望这足以将达到 nginx 前端的请求转发到您的 CherryPy 应用程序。`upstream` 块定义了 CherryPy 实例的地址。

它表明您可以在两个应用程序服务器之间进行负载平衡。请参阅 nginx 文档以了解如何实现。

```nginx
upstream apps {
   server 127.0.0.1:8080;
   server 127.0.0.1:8081;
}
```

稍后，此块用于定义反向代理部分。

现在，让我们看一下我们的应用程序：


```python
import cherrypy

class Root:
    @cherrypy.expose
    def index(self):
        return "hello world"

if __name__ == '__main__':
    cherrypy.config.update({
        'server.socket_port': 8080,
        'tools.proxy.on': True,
        'tools.proxy.base': 'http://www.example.com'
    })
    cherrypy.quickstart(Root())
```


如果您运行此代码的两个实例，在 nginx 部分中定义的每个端口上运行一个实例，您将能够通过 nginx 进行的负载平衡来访问这两个实例。

注意我们如何定义代理工具。它不是强制性的，仅用于让 CherryPy 请求知道真实客户的地址。否则，它将只知道 nginx 自己的地址。这在日志中最明显。

`base` 属性应与 Nginx 配置的 `server_name` 部分匹配。