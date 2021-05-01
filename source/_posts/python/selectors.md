---
layout: python
title: 高级 I/O 复用库
date: 2021-04-22 12:57:48
tags: asyncio
categories: Python 编程
---

参考自：[selectors — High-level I/O multiplexing](https://docs.python.org/3.10/library/selectors.html)

`selectors` 模块允许高层级且高效率的 I/O 复用，它建立在 `select` 模块原型的基础之上，提供基于 `select` 模块的 I/O 复用的平台无关的抽象。推荐使用 `selectors` 模块，除非希望对所使用的 OS 层级原型进行精确控制。它定义了一个 `BaseSelector` 抽象基类，以及多个实际的实现 ([`KqueueSelector`](https://docs.python.org/zh-cn/3.10/library/selectors.html#selectors.KqueueSelector "selectors.KqueueSelector"), [`EpollSelector`](https://docs.python.org/zh-cn/3.10/library/selectors.html#selectors.EpollSelector "selectors.EpollSelector")...)，它们可被用于在多个文件对象上等待 I/O 就绪通知。 在下文中，"文件对象" 是指任何具有 `fileno()` 方法的对象，或是一个原始文件描述器。参见 [file object](https://docs.python.org/3.10/glossary.html#term-file-object)。

<div class="w3-pale-green w3-card w3-padding">

<dfn class="xin-term">文件对象</dfn>（file object）：对外提供面向文件 API 以使用下层资源的对象（带有 <code>read()</code> 或 <code>write()</code> 这样的方法）。根据其创建方式的不同，文件对象可以处理对真实磁盘文件，对其他类型存储，或是对通讯设备的访问（例如标准输入/输出、内存缓冲区、套接字、管道等等）。文件对象也被称为 **file-like objects** 或 **流**（streams）。

实际上共有三种类别的文件对象: 原始 [二进制文件](https://docs.python.org/3.10/glossary.html#term-binary-file), 被缓冲的 [二进制文件](https://docs.python.org/3.10/glossary.html#term-binary-file) 以及 [文本文件](https://docs.python.org/3.10/glossary.html#term-text-file)。它们的接口定义均在 <code>io</code> 模块中。创建文件对象的规范方式是使用 <code>[open()](https://docs.python.org/3.10/library/functions.html#open)</code> 函数。
</div>

`DefaultSelector` 是一个指向当前平台上可用的最高效实现的别名：这应为大多数用户的默认选择。


```python
if _can_use('kqueue'):
    DefaultSelector = KqueueSelector
elif _can_use('epoll'):
    DefaultSelector = EpollSelector
elif _can_use('devpoll'):
    DefaultSelector = DevpollSelector
elif _can_use('poll'):
    DefaultSelector = PollSelector
else:
    DefaultSelector = SelectSelector
```

下文中，`events` 一个位掩码，指明哪些 I/O 事件要在给定的文件对象上执行等待。它可以是以下模块级常量的组合:

常量|解释
:-:|:-:
EVENT_READ|可读
EVENT_WRITE|可写

## class selectors.SelectorKey

`SelectorKey` 是一个 [namedtuple](https://docs.python.org/zh-cn/3.10/library/collections.html#collections.namedtuple)，用来将文件对象关联到其隐含的文件描述器、选定事件掩码和附加数据等。它会被某些 `BaseSelector` 方法返回。

<dl class="w3-pale-yellow w3-card-4 w3-padding">
 <dt class="w3-pale-green w3-card-4">fileobj</dt>
 <dd>已注册的文件对象</dd>
 <dt class="w3-pale-green w3-card-4">fd</dt>
 <dd>隐含的的文件描述器（Underlying file descriptor）</dd>
 <dt class="w3-pale-green w3-card-4">events</dt>
 <dd>必须在此文件对象上被等待的事件</dd>
 <dt class="w3-pale-green w3-card-4">data</dt>
 <dd>可选的关联到此文件对象的不透明数据：例如，这可被用来存储各个客户端的会话 ID</dd>
</dl>

## class selectors.BaseSelector

一个 `BaseSelector`，用来在多个文件对象上等待 I/O 事件就绪。它支持文件流注册、注销，以及在这些流上等待 I/O 事件的方法。它是一个抽象基类，因此不能被实例化。请改用 `DefaultSelector`，或者 `SelectSelector`, `KqueueSelector` 等。如果你想要指明使用某个实现，并且你的平台支持它的话。`BaseSelector` 及其具体实现支持 [context manager](https://docs.python.org/3.10/glossary.html#term-context-manager) 协议。

<dl class="w3-pale-yellow w3-card-4 w3-padding">
 <dt class="w3-pale-green w3-card-4">abstractmethod register(fileobj, events, data=None)</dt>
 <dd>注册一个用于选择的文件对象，在其上监视 I/O 事件。</dd>
 <dd><code>fileobj</code> 是要监视的文件对象。它可以是整数形式的文件描述符或者具有 <code>fileno()</code> 方法的对象。<code>events</code> 是要监视的事件的位掩码。<code>data</code> 是一个不透明对象。</dd>
 <dd>这将返回一个新的 <code>SelectorKey</code> 实例，或在出现无效事件掩码或文件描述符时引发 <code>ValueError</code>，或在文件对象已被注册时引发 <code>KeyError</code>。</dd>

 <dt class="w3-pale-green w3-card-4">abstractmethod unregister(fileobj)</dt>
 <dd>注销对一个文件对象的选择，移除对它的监视。在文件对象被关闭之前应当先将其注销。</dd>
 <dd><code>fileobj</code>必须是之前已注册的文件对象。</dd>
 <dd>这将返回已关联的 <code>SelectorKey</code> 实例，或者如果 <code>fileobj</code> 未注册则会引发 <code>KeyError</code>。 如果 <code>fileobj</code> 无效（例如它没有 <code>fileobj()</code> 方法或其 <code>fileobj()</code> 方法返回无效值），则返回 <code>ValueError</code></dd>
 
 <dt class="w3-pale-green w3-card-4">modify(fileobj, events, data=None)(fileobj)</dt>
 <dd>更改已注册文件对象所监视的事件或所附带的数据。</dd>
 <dd>这等价于 <code>BaseSelector.unregister(fileobj)()</code> 加 <code>BaseSelector.register(fileobj, events, data)()</code>，区别在于它可以被更高效地实现。</dd>
 <dd>这将返回一个新的 <code>SelectorKey</code> 实例，或在出现无效事件掩码或文件描述符时引发 <code>ValueError</code>，或在文件对象未被注册时引发 <code>KeyError</code>。</dd>

 <dt class="w3-pale-green w3-card-4">abstractmethod select(timeout=None)</dt>
 <dd>等待直到有已注册的文件对象就绪，或是超过时限。</dd>
 <dd>如果 <code>timeout > 0</code>，这指定以秒数表示的最大等待时间。如果 <code>timeout <= 0</code>，调用将不会阻塞，并将报告当前就绪的文件对象。如果 <code>timeout</code> 为 <code>None</code>，调用将阻塞直到某个被监视的文件对象就绪。</dd>
 <dd>返回由 <code>(key, events)</code> 元组构成的列表，每项各表示一个就绪的文件对象。</dd>
 <dd><code>key</code> 是对应于就绪文件对象的 <code>SelectorKey</code> 实例。<code>events</code> 是在此文件对象上等待的事件位掩码。</dd>
 <dd class="w3-card-4 w3-light-grey w3-padding"><span class="w3-text-blue">注解</span>：如果当前进程收到一个信号（<code>signal</code>），此方法可在任何文件对象就绪之前或超出时限时返回：在此情况下，将返回一个空列表。<dd>
 <dd>在 3.5 版更改: 现在当被某个信号中断时，如果信号处理程序没有引发异常，选择器会用重新计算的超时值进行重试（理由请查看 <a href="https://www.python.org/dev/peps/pep-0475">PEP 475</a> ），而不是在超时之前返回空的事件列表。</dd>

 <dt class="w3-pale-green w3-card-4">close()</dt>
 <dd>关闭选择器（selector）。</dd>
 <dd>必须调用这个方法以确保下层资源会被释放。选择器被关闭后将不可再使用。</dd>

 <dt class="w3-pale-green w3-card-4">get_key(fileobj)</dt>
 <dd>返回关联到某个已注册文件对象的键。</dd>
 <dd>此方法将返回关联到文件对象的 <code>SelectorKey</code> 实例，或在文件对象未注册时引发 <code>KeyError</code>。</dd>

 <dt class="w3-pale-green w3-card-4">abstractmethod get_map()</dt>
 <dd>返回从文件对象到选择器键的映射。</dd>
 <dd>返回一个将已注册文件对象映射到与其相关联的<code>SelectorKey</code> 实例的 <a href="https://docs.python.org/zh-cn/3.10/library/collections.abc.html#collections.abc.Mapping">Mapping</a> 实例。</dd>
</dl>

## 一个示例

下面是一个简单的`echo`服务器实现：

```python
import selectors
import socket
# 生成一个 select 对象
sel = selectors.DefaultSelector()

def accept(sock, mask):
    conn, addr = sock.accept()  # Should be ready
    print('accepted', conn, 'from', addr)
    conn.setblocking(False) # 设定非阻塞
    # 新连接注册 read 回调函数
    sel.register(conn, selectors.EVENT_READ, read)

def read(conn, mask):
    data = conn.recv(1000)  # Should be ready
    if data:
        print('echoing', repr(data), 'to', conn)
        conn.send(data)  # Hope it won't block
    else:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()

sock = socket.socket()
sock.bind(('localhost', 1234))
sock.listen(100)
sock.setblocking(False)
# 把刚生成的sock连接对象注册到select连接列表中，并交给accept函数处理
sel.register(sock, selectors.EVENT_READ, accept)

while True:
    events = sel.select() # 默认是阻塞，有活动连接就返回活动的连接列表
    for key, mask in events:
        callback = key.data # 去调accept函数
        callback(key.fileobj, mask)  # key.fileobj就是readable中的一个socket连接对象
```

https://pymotw.com/3/selectors/