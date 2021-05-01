---
title: 使用协程书写多任务合作代码
lang: zh-CN
tags: asyncio
categories: Python 编程
abbrlink: 97a8b773
date: 2021-04-21 08:36:33
---

协程（Coroutine）是为并发操作而设计的语言构造。协程函数在被调用时会创建一个协程对象，然后调用者可以使用协程的 `send()` 方法运行该函数的代码。一个协程可以将 `await` 关键字与另一个协程一起暂停执行。暂停时，协程的状态得以维持，使其在下次唤醒时可以从中断的位置恢复。

## 事件循环

事件循环是每个 `asyncio` 应用的核心。事件循环会运行异步任务和回调，执行网络 IO 操作，以及运行子进程。

应用开发者通常应当使用高层级的 `asyncio` 函数，例如 `asyncio.run()`，应当很少有必要引用`loop`对象或调用其方法。

以下低层级函数可被用于获取、设置或创建事件循环：

1. `asyncio.get_running_loop()` 返回当前 OS 线程中正在运行的事件循环。如果没有正在运行的事件循环则会引发 `RuntimeError`。此函数只能由协程或回调来调用。
2. `asyncio.get_event_loop()` 获取当前事件循环。如果当前 OS 线程没有设置当前事件循环，该 OS 线程为主线程，并且 `set_event_loop()` 还没有被调用，则 `asyncio` 将创建一个新的事件循环并将其设为当前事件循环。由于此函数具有相当复杂的行为（特别是在使用了自定义事件循环策略的时候），更推荐在协程和回调中使用 `get_running_loop()` 函数而非 `get_event_loop()`。<span class="w3-pale-green">应该考虑使用 <code>asyncio.run()</code> 函数而非使用低层级函数来手动创建和关闭事件循环。</span>
3. `asyncio.set_event_loop(loop)` 将 `loop` 设置为当前 OS 线程的当前事件循环。
4. `asyncio.new_event_loop()` 创建一个新的事件循环。

请注意 [`get_event_loop()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.get_event_loop "asyncio.get_event_loop")，[`set_event_loop()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.set_event_loop "asyncio.set_event_loop") 以及 [`new_event_loop()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.new_event_loop "asyncio.new_event_loop") 函数的行为可以通过 [设置自定义事件循环策略](https://docs.python.org/zh-cn/3.10/library/asyncio-policy.html#asyncio-policies) 来改变。

## 启动协程

有几种不同的方法可以使 `asyncio` 事件循环启动协程。最简单的方法是使用 `run_until_complete()`，将协程直接传递给它。

```python
# asyncio_coroutine.py
import asyncio


async def coroutine():
    print('in coroutine')

# 获取对事件循环的引用
event_loop = asyncio.get_event_loop()
try:
    print('starting coroutine')
    coro = coroutine()
    print('entering event loop')
    event_loop.run_until_complete(coro)
finally:
    print('closing event loop')
    event_loop.close()
```

第一步是获取对事件循环的引用。可以使用默认的循环类型，或者可以实例化特定的循环类。在此示例中，使用默认循环。`run_until_complete()` 方法使用协程对象启动循环，并在协程退出返回时停止循环。

```shell
$ python asyncio_coroutine.py
starting coroutine
entering event loop
in coroutine
closing event loop
```

### 运行和停止循环

1. `loop.run_until_complete(future)` 运行直到 future([Future](https://docs.python.org/zh-cn/3.10/library/asyncio-future.html#asyncio.Future) 的实例) 被完成。如果参数是 [coroutine object](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#coroutine)，将被隐式调度为 [asyncio.Task](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.Task) 来运行。
2. `loop.run_forever()` 运行事件循环直到 `stop()` 被调用。如果 [`stop()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.stop "asyncio.loop.stop") 在调用 [`run_forever()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.run_forever "asyncio.loop.run_forever") 之前被调用，循环将轮询一次 I/O 选择器并设置超时为零，再运行所有已加入计划任务的回调来响应 I/O 事件（以及已加入计划任务的事件），然后退出。如果 [`stop()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.stop "asyncio.loop.stop") 在 [`run_forever()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.run_forever "asyncio.loop.run_forever") 运行期间被调用，循环将运行当前批次的回调然后退出。请注意在此情况下由回调加入计划任务的新回调将不会运行；它们将会在下次 [`run_forever()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.run_forever "asyncio.loop.run_forever") 或 [`run_until_complete()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.run_until_complete "asyncio.loop.run_until_complete") 被调用时运行。
3. `loop.stop()` 停止事件循环。
4. `loop.is_running()` 返回 `True` 如果事件循环当前正在运行。
5. `loop.is_closed()`如果事件循环已经被关闭，返回 `True`。
6. `loop.close()` 关闭事件循环。当这个函数被调用的时候，循环必须处于非运行状态。`pending` 状态的回调将被丢弃。此方法清除所有的队列并立即关闭执行器，不会等待执行器完成。这个方法是幂等的和不可逆的（idempotent and irreversible）。事件循环关闭后，不应调用其他方法。
7. coroutine `loop.shutdown_asyncgens()` 安排所有当前打开的 [asynchronous generator](https://docs.python.org/zh-cn/3.10/glossary.html#term-asynchronous-generator) 对象通过 [`aclose()`](https://docs.python.org/zh-cn/3.10/reference/expressions.html#agen.aclose "agen.aclose") 调用来关闭。在调用此方法后，如果有新的异步生成器被迭代事件循环将会发出警告。这应当被用来可靠地完成所有已加入计划任务的异步生成器。请注意当使用 [`asyncio.run()`](https://docs.python.org/zh-cn/3.10/library/asyncio-task.html#asyncio.run "asyncio.run") 时不必调用此函数。

示例:

```python
try:
    loop.run_forever()
finally:
    loop.run_until_complete(loop.shutdown_asyncgens())
    loop.close()
```

8. `coroutine loop.shutdown_default_executor()` 安排默认执行器的关闭并等待它合并 `ThreadPoolExecutor` 中的所有线程。在调用此方法后，如果在使用默认执行器期间调用了 [loop.run_in_executor()](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.run_in_executor) 则将会引发 `RuntimeError`。请注意当使用 `asyncio.run()` 时不必调用此函数。

## 从协程返回值

协程的返回值被传递回启动并等待它的代码。

```python
# asyncio_coroutine_return.py
import asyncio


async def coroutine():
    print('in coroutine')
    return 'result'


event_loop = asyncio.get_event_loop()
try:
    return_value = event_loop.run_until_complete(
        coroutine()
    )
    print(f'it returned: {return_value}')
finally:
    event_loop.close()
```

在这种情况下，`run_until_complete()` 还返回它正在等待的协程的结果。

更加简便的写法是：

```python
# asyncio_coroutine_return.py
import asyncio


async def coroutine():
    print('in coroutine')
    return 'result'


async def main():
    return_value = await coroutine()
    print(f'it returned: {return_value}')
    
asyncio.run(main())
```

```shell
$ python asyncio_coroutine_return.py

in coroutine
it returned: 'result
```

## 协程链

一个协程可以启动另一个协程并等待结果。这使得将任务分解为可重用的部分变得更加容易。以下示例具有必须按顺序执行的两个阶段，但是可以与其他操作同时运行。

```python
# asyncio_coroutine_chain.py
import asyncio


async def outer():
    print('in outer')
    print('waiting for result1')
    result1 = await phase1()
    print('waiting for result2')
    result2 = await phase2(result1)
    return (result1, result2)


async def phase1():
    print('in phase1')
    return 'result1'


async def phase2(arg):
    print('in phase2')
    return f'result2 derived from {arg}'


async def main():
    return_value = await outer()
    print(f'return value: {return_value}')

asyncio.run(main())
```

使用 `await` 关键字而不是将新的协程添加到循环中，因为控制流已经在由循环管理的协程内部，因此不必告诉循环来管理新的协程。

```shell
$ python3 asyncio_coroutine_chain.py
in outer
waiting for result1
in phase1
waiting for result2
in phase2
return value: ('result1', 'result2 derived from result1')
```