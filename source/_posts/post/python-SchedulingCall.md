---
title: Scheduling Calls 设定定时器
lang: zh-CN
abbrlink: af2cec0c
date: 2021-04-21 10:11:36
tags: asyncio
categories: Python 编程
---

除了管理协程和 I/O 回调外，`asyncio` 事件循环还可以根据循环中保留的计时器值来调度对常规函数的调用。

## Scheduling a Callback "Soon"

如果回调的时间无关紧要，则可以使用 `call_soon()` 为循环的下一次迭代安排调用。调用该函数后，该函数之后的所有其他位置参数都将传递给该回调。要将关键字参数传递给回调，请使用 [functools](https://pymotw.com/3/functools/index.html#module-functools) 模块中的 `partial()`。

```python
# asyncio_call_soon.py
import asyncio
import functools


def callback(arg, *, kwarg='default'):
    print(f'callback invoked with {arg} and {kwarg}')


async def main(loop):
    print('registering callbacks')
    loop.call_soon(callback, 1)
    wrapped = functools.partial(callback, kwarg='not default')
    loop.call_soon(wrapped, 2)

    await asyncio.sleep(0.1)

event_loop = asyncio.get_event_loop()
try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loop')
    event_loop.close()
```

回调按调度的顺序被调用。

```shell
$ python3 asyncio_call_soon.py

entering event loop
registering callbacks
callback invoked with 1 and default
callback invoked with 2 and not default
closing event loop
```

注解：大多数 [`asyncio`](https://docs.python.org/zh-cn/3.10/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O.") 的调度函数不让传递关键字参数。为此，请使用 [`functools.partial()`](https://docs.python.org/zh-cn/3.10/library/functools.html#functools.partial "functools.partial") ：

```python
# will schedule "print("Hello", flush=True)"
loop.call_soon(
    functools.partial(print, "Hello", flush=True))
```

使用 `partial` 对象通常比使用 `lambda` 更方便，`asyncio` 在调试和错误消息中能更好的呈现 `partial` 对象。

更加友好的写法是：

```python
# asyncio_call_soon.py
import asyncio
import functools


def callback(arg, *, kwarg='default'):
    print(f'callback invoked with {arg} and {kwarg}')

async def call_soon(loop):
    print('registering callbacks')
    loop.call_soon(callback, 1)
    wrapped = functools.partial(callback, kwarg='not default')
    loop.call_soon(wrapped, 2)
    await asyncio.sleep(0.1)

async def main():
    event_loop = asyncio.get_event_loop()
    print('entering event loop')
    await call_soon(event_loop)
    print('closing event loop')

asyncio.run(main())
```

### 安排回调

1. `loop.call_soon(callback, *args, context=None)` 安排 `callback` 在事件循环的下一次迭代时附带 `args` 参数被调用。回调按其注册顺序被调用。每个回调仅被调用一次。可选的仅关键字型参数 `context` 允许为要运行的 `callback` 指定一个自定义 `contextvars.Context`。如果没有提供 `context`，则使用当前上下文。返回一个能用来取消回调的 [asyncio.Handle](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.Handle) 实例。这个方法不是线程安全的。
2. `loop.call_soon_threadsafe(callback, *args, context=None)` 是 `call_soon()` 的线程安全变体。必须被用于安排 来自其他线程 的回调。查看 [并发和多线程](https://docs.python.org/zh-cn/3.10/library/asyncio-dev.html#asyncio-multithreading) 章节的文档。

## 调度延迟回调

事件循环提供安排调度函数在将来某个时刻调用的机制。事件循环使用单调时钟来跟踪时间。`loop.call_later(delay, callback, *args, context=None)` 安排 `callback` 在给定的 `delay` 秒（可以是 int 或者 float）后被调用。返回一个 asyncio.[TimerHandle](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.TimerHandle) 实例，该实例能用于取消回调。

`callback` 只被调用一次。如果两个回调被安排在同样的时间点，执行顺序未限定。可选的位置参数 *`args`* 在被调用的时候传递给 *`callback`* 。如果你想把关键字参数传递给 *`callback`* ，请使用 [`functools.partial()`](https://docs.python.org/zh-cn/3.10/library/functools.html#functools.partial "functools.partial") 。

可选的仅关键字型参数 *context* 允许为要运行的 *callback* 指定一个自定义 [`contextvars.Context`](https://docs.python.org/zh-cn/3.10/library/contextvars.html#contextvars.Context "contextvars.Context")。如果没有提供 *`context`* ，则使用当前上下文。

```python
# asyncio_call_later.py
import asyncio


def callback(n):
    print(f'callback {n} invoked')


async def main(loop):
    print('registering callbacks')
    loop.call_later(0.2, callback, 1)
    loop.call_later(0.1, callback, 2)
    loop.call_soon(callback, 3)

    await asyncio.sleep(0.4)


event_loop = asyncio.get_event_loop()
try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loop')
    event_loop.close()
```

在此示例中，使用不同的参数将相同的回调函数调度了几次不同的时间。使用 `call_soon()` 的最终实例导致在任何定时实例之前使用参数 3 调用回调，这表明“soon”通常意味着最小的延迟。

```python
$ python3 asyncio_call_later.py

entering event loop
registering callbacks
callback 3 invoked
callback 2 invoked
callback 1 invoked
closing event loop
```

## 安排特定时间的回调

也可以安排在特定时间进行回调。该 `loop` 使用单调时钟（monotonic clock）而不是挂钟时间（wall-clock tim），以确保“now”的值永不回归。要为计划的回调选择时间，必须使用 `loop` 的 `time()` 方法从该时钟的内部状态开始。

`loop.call_at(when, callback, *args, context=None)` 安排 `callback` 在给定的绝对时间戳 `when` (`int` 或 `float`) 被调用，使用与 `loop.time()` 同样的时间参考。这个函数的行为与 [`call_later()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.call_later "asyncio.loop.call_later") 相同。返回一个 [`asyncio.TimerHandle`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.TimerHandle "asyncio.TimerHandle") 实例，该实例能用于取消回调。

`loop.time()` 根据时间循环内部的单调时钟，返回当前时间为一个 `float` 值。参见 [asyncio.sleep()](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.sleep) 函数。

```python
# asyncio_call_at.py
import asyncio
import time


def callback(n, loop):
    print(f'callback {n} invoked at {loop.time()}')


async def main(loop):
    now = loop.time()
    print(f'clock time: {time.time()}')
    print(f'loop  time: {now}')

    print('registering callbacks')
    loop.call_at(now + 0.2, callback, 1, loop)
    loop.call_at(now + 0.1, callback, 2, loop)
    loop.call_soon(callback, 3, loop)

    await asyncio.sleep(1)


event_loop = asyncio.get_event_loop()
try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loop')
    event_loop.close()
```

请注意，对应的 `loop` 的时间与 `time.time()` 返回的值不匹配。

```shell
$ python3 asyncio_call_at.py

entering event loop
clock time: 1618973483.6534503
loop  time: 3016024.109
registering callbacks
callback 3 invoked at 3016024.109
callback 2 invoked at 3016024.218
callback 1 invoked at 3016024.312
closing event loop
```

