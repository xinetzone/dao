---
title: Future 以异步方式生成数据
tags: asyncio.Future
categories: Python 编程
abbrlink: ffb8ca57
date: 2021-04-21 11:21:45
---

`Future` 对象用来链接 **底层回调式代码** 和高层异步/等待式代码。

## Future 对象

class `asyncio.Future(*, loop=None)`：一个 `Future` 代表一个异步运算的最终结果。线程不安全。

`Future` 是一个 [awaitable](https://docs.python.org/zh-cn/3.10/glossary.html#term-awaitable) 对象。协程可以等待 `Future` 对象直到它们有结果或异常集合或被取消。

通常 `Future` 用于支持底层回调式代码(例如在协议实现中使用 `asyncio` [transports](https://docs.python.org/zh-cn/3.10/library/asyncio-protocol.html#asyncio-transports-protocols)) 与高层异步/等待式代码交互。

经验告诉我们永远不要面向用户的接口暴露 `Future` 对象，同时建议使用 [`loop.create_future()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.create_future "asyncio.loop.create_future") 来创建 `Future` 对象。这种方法可以让 `Future` 对象使用其它的事件循环实现，它可以注入自己的优化实现。

### `set_result(result)`

将 `Future` 标记为 *完成* 并设置结果。

如果 `Future` 已经 *完成* 则抛出一个 [`InvalidStateError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.InvalidStateError "asyncio.InvalidStateError") 错误。

### `result()`

返回 `Future` 的结果。

如果 `Future` 状态为 *完成* ，并由 [`set_result()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.set_result "asyncio.Future.set_result") 方法设置一个结果，则返回这个结果。

如果 `Future` 状态为 *完成* ，并由 [`set_exception()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.set_exception "asyncio.Future.set_exception") 方法设置一个异常，那么这个方法会引发异常。

如果 `Future` 已 *取消*，方法会引发一个 [`CancelledError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.CancelledError "asyncio.CancelledError") 异常。

如果 `Future` 的结果还不可用，此方法会引发一个 [`InvalidStateError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.InvalidStateError "asyncio.InvalidStateError") 异常。

### `set_exception(exception)`

将 F`uture` 标记为 *完成* 并设置一个异常。

如果 `Future` 已经 *完成* 则抛出一个 [`InvalidStateError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.InvalidStateError "asyncio.InvalidStateError") 错误。

### `done()`

如果 `Future` 为已 *完成* 则返回 `True`。

如果 `Future` 为 *取消* 或调用 [`set_result()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.set_result "asyncio.Future.set_result") 设置了结果或调用 [`set_exception()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.set_exception "asyncio.Future.set_exception") 设置了异常，那么它就是 *完成*。

### `cancelled()`

如果 `Future` 已 取消 则返回 `True`。

这个方法通常在设置结果或异常前用来检查 `Future` 是否已 *取消*。

```python
if not fut.cancelled():
    fut.set_result(42)
```

### `add_done_callback(callback, *, context=None)`

添加一个在 `Future` *完成* 时运行的回调函数。

调用 *`callback`* 时，`Future` 对象是它的唯一参数。

调用这个方法时 `Future` 已经 *完成* , 回调函数已被 [`loop.call_soon()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.call_soon "asyncio.loop.call_soon") 调度。

可选键值类的参数 *`context`* 允许 *`callback`* 运行在一个指定的自定义 [`contextvars.Context`](https://docs.python.org/zh-cn/3.10/library/contextvars.html#contextvars.Context "contextvars.Context") 对象中。如果没有提供 *`context`* ，则使用当前上下文。

可以用 [`functools.partial()`](https://docs.python.org/zh-cn/3.10/library/functools.html#functools.partial "functools.partial") 给回调函数传递参数，例如：

```python
# Call 'print("Future:", fut)' when "fut" is done.
fut.add_done_callback(
    functools.partial(print, "Future:"))
```

### `remove_done_callback(callback)`

从回调列表中移除 `callback`。

返回被移除的回调函数的数量，通常为 1，除非一个回调函数被添加多次。

### `cancel(msg=None)`

取消 `Future` 并调度回调函数。

如果 `Future` 已经 *完成* 或 *取消*，返回 `False`。否则将 `Future` 状态改为 *取消* 并在调度回调函数后返回 `True`。

### `exception()`

返回 `Future` 已设置的异常。

只有 `Future` 在 *完成* 时才返回异常（或者 `None` ，如果没有设置异常）。

如果 `Future` 已 *取消*，方法会引发一个 [`CancelledError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.CancelledError "asyncio.CancelledError") 异常。

如果 `Future` 还没 *完成* ，这个方法会引发一个 [`InvalidStateError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.InvalidStateError "asyncio.InvalidStateError") 异常。

### `get_loop()`

返回 `Future` 对象已绑定的事件循环。

## Future 函数

### `asyncio.isfuture(obj)`

如果 `obj` 为下面任意对象，返回 `True`：

* 一个 [`asyncio.Future`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future "asyncio.Future") 类的实例，
* 一个 [`asyncio.Task`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.Task "asyncio.Task") 类的实例，
* 带有 `_asyncio_future_blocking` 属性的类似 `Future` 的对象。

### `asyncio.ensure_future(obj, *, loop=None)` 

返回：

* *`obj`* 参数会是保持原样，如果 *`obj`* 是 [`Future`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future "asyncio.Future")、 [`Task`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.Task "asyncio.Task") 或 类似 Future 的对象( [`isfuture()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.isfuture "asyncio.isfuture") 用于测试。)

* 封装了 *`obj`* 的 [`Task`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.Task "asyncio.Task") 对象，如果 *`obj`* 是一个协程 (使用 [`iscoroutine()`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.iscoroutine "asyncio.iscoroutine") 进行检测)；在此情况下该协程将通过 `ensure_future()` 加入执行计划。

* 等待 *`obj`* 的 [`Task`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.Task "asyncio.Task") 对象，如果 *`obj`* 是一个可等待对象( [`inspect.isawaitable()`](https://docs.python.org/zh-cn/3.10/library/inspect.html#inspect.isawaitable "inspect.isawaitable") 用于测试)

如果 `obj` 不是上述对象会引发一个 `TypeError` 异常。

### `asyncio.wrap_future(future, *, loop=None)` 

将一个 [`concurrent.futures.Future`](https://docs.python.org/zh-cn/3.10/library/concurrent.futures.html#concurrent.futures.Future "concurrent.futures.Future") 对象封装到 [`asyncio.Future`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future "asyncio.Future") 对象中。

## Waiting for a Future

`Future` 的行为就像协程，因此，任何用于等待协程的有用技术也可以用来等待 `future` 被标记为完成。本示例将 `future` 传递给事件循环的 `run_until_complete()` 方法。

```python
# asyncio_future_event_loop.py
import asyncio


def mark_done(future, result):
    print(f'setting future result to {result!r}')
    future.set_result(result)


event_loop = asyncio.get_event_loop()
try:
    all_done = asyncio.Future()

    print('scheduling mark_done')
    event_loop.call_soon(mark_done, all_done, 'the result')

    print('entering event loop')
    result = event_loop.run_until_complete(all_done)
    print(f'returned result: {result!r}')
finally:
    print('closing event loop')
    event_loop.close()

print(f'future result: {all_done.result()}')
```

当调用 `set_result()` 时，`Future` 的状态更改为完成，并且 `Future` 实例保留提供给该方法的结果供以后检索。

```shell
$ python3 asyncio_future_event_loop.py

scheduling mark_done
entering event loop
setting future result to 'the result'
returned result: 'the result'
closing event loop
future result: 'the result'
```

`Future` 也可以与 `await` 关键字一起使用。

```python
# asyncio_future_await.py
import asyncio


def mark_done(future, result):
    print(f'setting future result to {result!r}')
    future.set_result(result)


async def main(loop):
    all_done = asyncio.Future()

    print('scheduling mark_done')
    loop.call_soon(mark_done, all_done, 'the result')

    result = await all_done
    print(f'returned result: {result!r}')


event_loop = asyncio.get_event_loop()
try:
    event_loop.run_until_complete(main(event_loop))
finally:
    event_loop.close()
```

`Future` 的结果是由 `await` 返回的，因此通常可以在常规协程和 `Future` 实例中使用相同的代码。

```shell
$ python3 asyncio_future_await.py

scheduling mark_done
setting future result to 'the result'
returned result: 'the result'
```

## Future Callbacks

除了像协程一样工作，`Future` 还可以在完成时调用回调。回调按注册顺序调用。

```python
# asyncio_future_callback.py
import asyncio
import functools


def callback(future, n):
    print(f'{n}: future done: {future.result()!r}')


async def register_callbacks(all_done):
    print('registering callbacks on future')
    all_done.add_done_callback(functools.partial(callback, n=1))
    all_done.add_done_callback(functools.partial(callback, n=2))


async def main(all_done):
    await register_callbacks(all_done)
    print('setting result of future')
    all_done.set_result('the result')


event_loop = asyncio.get_event_loop()
try:
    all_done = asyncio.Future()
    event_loop.run_until_complete(main(all_done))
finally:
    event_loop.close()
```

回调应包含一个参数，即 `Future` 实例。要将其他参数传递给回调，请使用 `functools.partial()` 创建包装器。

```shell
$ python3 asyncio_future_callback.py

registering callbacks on future
setting result of future
1: future done: 'the result'
2: future done: 'the result'
```

## 使用事件循环创建 Future

这个例子创建一个 `Future` 对象，创建和调度一个异步任务去设置 `Future` 结果，然后等待其结果：

```python
async def set_after(fut, delay, value):
    # Sleep for *delay* seconds.
    await asyncio.sleep(delay)

    # Set *value* as a result of *fut* Future.
    fut.set_result(value)

async def main():
    # Get the current event loop.
    loop = asyncio.get_running_loop()

    # Create a new Future object.
    fut = loop.create_future()

    # Run "set_after()" coroutine in a parallel Task.
    # We are using the low-level "loop.create_task()" API here because
    # we already have a reference to the event loop at hand.
    # Otherwise we could have just used "asyncio.create_task()".
    loop.create_task(
        set_after(fut, 1, '... world'))

    print('hello ...')

    # Wait until *fut* has a result (1 second) and print it.
    print(await fut)

asyncio.run(main())
```

**重要**：该 `Future` 对象是为了模仿 `concurrent.futures.Future` 类。主要差异包含：

* 与 asyncio 的 Future 不同，[`concurrent.futures.Future`](https://docs.python.org/zh-cn/3.10/library/concurrent.futures.html#concurrent.futures.Future "concurrent.futures.Future") 实例不是可等待对象。

* [`asyncio.Future.result()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.result "asyncio.Future.result") 和 [`asyncio.Future.exception()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.exception "asyncio.Future.exception") 不接受 *timeout* 参数。

* Future 没有 *完成* 时 [`asyncio.Future.result()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.result "asyncio.Future.result") 和 [`asyncio.Future.exception()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.exception "asyncio.Future.exception") 抛出一个 [`InvalidStateError`](https://docs.python.org/zh-cn/3.10/library/asyncio-exceptions.html#asyncio.InvalidStateError "asyncio.InvalidStateError") 异常。

* 使用 [`asyncio.Future.add_done_callback()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.add_done_callback "asyncio.Future.add_done_callback") 注册的回调函数不会立即调用，而是被 [`loop.call_soon()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.call_soon "asyncio.loop.call_soon") 调度。

* `asyncio.Future` 不能兼容 [`concurrent.futures.wait()`](https://docs.python.org/zh-cn/3.10/library/concurrent.futures.html#concurrent.futures.wait "concurrent.futures.wait") 和 [`concurrent.futures.as_completed()`](https://docs.python.org/zh-cn/3.10/library/concurrent.futures.html#concurrent.futures.as_completed "concurrent.futures.as_completed") 函数。

* [`asyncio.Future.cancel()`](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future.cancel "asyncio.Future.cancel") 接受一个可选的 `msg` 参数，但 `concurrent.futures.cancel()` 无此参数。
