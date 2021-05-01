---
title: Python 异步编程
lang: zh-CN
abbrlink: b13fe0ba
date: 2021-04-19 13:19:49
tags: asyncio
categories: Python 编程
---

`future` 是代表尚未完成的工作结果的数据结构。事件循环可以监视将 `Future` 对象设置为完成的状态，从而允许应用程序的一部分等待另一部分完成某些工作。除 `future` 外，`asyncio` 还包括其他并发原语，例如锁和信号量（semaphores）。

`Task` 是 `Future` 的子类，它知道如何包装和管理协程的执行。可以使用事件循环调度任务，以在所需资源可用时运行它们，并产生可以被其他协程消耗的结果。

## Awaitables

若一个对象可以被用于 `await` 表达式，则可被称为<dfn class="xin-term">可等待</dfn>（awaitable）对象。<dfn class="xin-term">可等待</dfn> 对象有三种主要类型: <dfn class="xin-term">协程</dfn>（coroutines）, <dfn class="xin-term">任务</dfn>（Tasks） 和 <dfn class="xin-term">Futures</dfn>。

### 协程

<dfn class="xin-term">协程</dfn> 通过 `async/await` 语法进行声明，是编写 asyncio 应用的推荐方式。例如，以下代码段（需要 Python 3.7+）会打印 "hello"，等待 1 秒，再打印 "world"：

```python
>>> import asyncio

>>> async def main():
...     print('hello')
...     await asyncio.sleep(1)
...     print('world')

>>> asyncio.run(main())
hello
world
```

注意：简单地调用一个协程并不会使其被调度（schedule）执行

```python
>>> main()
<coroutine object main at 0x1053bb7c8>
```

要真正运行一个协程，asyncio 提供了三种主要机制:

- `asyncio.run()` 函数用来运行最高层级的入口点 "main()" 函数 (参见上面的示例。)
- 等待一个协程。以下代码段会在等待 1 秒后打印 "hello"，然后 再次 等待 2 秒后打印 "world"：

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

预期的输出：

```shell
started at 17:13:52
hello
world
finished at 17:13:55
```

- `asyncio.create_task()` 函数用来并发运行作为 asyncio **任务** 的多个协程。

修改以上示例，**并发**（concurrently）运行两个 `say_after` 协程:

```python
async def main():
    task1 = asyncio.create_task(
        say_after(1, 'hello'))

    task2 = asyncio.create_task(
        say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    # Wait until both tasks are completed (should take
    # around 2 seconds.)
    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")
```

注意，预期的输出显示代码段的运行时间比之前快了 1 秒：

```shell
started at 17:14:32
hello
world
finished at 17:14:34
```

Python <i>协程</i> 属于 可等待 对象，因此可以在其他协程中被等待：

```python
import asyncio

async def nested():
    return 42

async def main():
    # Nothing happens if we just call "nested()".
    # A coroutine object is created but not awaited,
    # so it *won't run at all*.
    nested()

    # Let's do it differently now and await it:
    print(await nested())  # will print "42".

asyncio.run(main())
```

<span class="w3-yellow">重要</span> ：<dfn class="xin-term">协程</dfn> 一般有两个紧密关联的概念:

- <dfn class="xin-term">协程函数</dfn>: 定义形式为 `async def` 的函数;
- <dfn class="xin-term">协程对象</dfn>: 调用 协程函数 所返回的对象。

### 任务

<dfn class="xin-term">任务</dfn> 被用来“并发的”（concurrently）调度协程。当一个协程通过 `asyncio.create_task()` 等函数被封装为一个 **任务**，该协程会被自动调度执行：

```python
import asyncio

async def nested():
    return 42

async def main():
    # Schedule nested() to run soon concurrently
    # with "main()".
    task = asyncio.create_task(nested())

    # "task" can now be used to cancel "nested()", or
    # can simply be awaited to wait until it is complete:
    await task

asyncio.run(main())
```

### Future 对象

<dfn class="xin-term">Future</dfn> 是一种特殊的 **低层级** 可等待对象，表示一个异步操作的 **最终结果**。

当一个 Future 对象 被等待，这意味着协程将保持等待直到该 Future 对象在其他地方操作完毕。

在 asyncio 中需要 <dfn class="xin-term">Future</dfn> 对象以便允许通过 async/await 使用基于回调的代码。

通常情况下 **没有必要** 在应用层级的代码中创建 <dfn class="xin-term">Future</dfn> 对象。

<dfn class="xin-term">Future</dfn> 对象有时会由库和某些 asyncio API 暴露给用户，用作可等待对象：

```python
async def main():
    await function_that_returns_a_future_object()

    # this is also valid:
    await asyncio.gather(
        function_that_returns_a_future_object(),
        some_python_coroutine()
    )
```

一个很好的返回对象的低层级函数的示例是 [`loop.run_in_executor()`](https://docs.python.org/zh-cn/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor)。

## 在 Jupyter Notebook 中使用异步编程

在 Jupyter Notebook 中使用 `asyncio.run()` 时会触发异常：

```shell
---------------------------------------------------------------------------
RuntimeError                              Traceback (most recent call last)
<ipython-input-5-eaadc83a82ea> in <module>
----> 1 asyncio.run(main())

~\anaconda3\envs\ui\lib\asyncio\runners.py in run(main, debug)
     31     """
     32     if events._get_running_loop() is not None:
---> 33         raise RuntimeError(
     34             "asyncio.run() cannot be called from a running event loop")
     35 

RuntimeError: asyncio.run() cannot be called from a running event loop
```

这是因为 Jupyter 已经运行了 事件循环，无需自己激活，采用上文中的 `await()` 调用即可。

比如，如下程序：

```python
import asyncio


async def main():
    print('hello')
    await asyncio.sleep(1)
    print('world')
```

在 Jupyter Notebook 和 Python Shell 中分别为：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--1" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--1">Jupyter Notebook</label>
        <div class="tab-content w3-padding">
```python
await main()
```
        </div>
        <input id="tab-set--0-input--2" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--2">Python Shell</label>
        <div class="tab-content w3-padding">
```python
asyncio.run(main())
```
        </div>
    </div>
</article>

## 运行 asyncio 程序

`asyncio.run(coro, *, debug=False)` 执行 [coroutine](https://docs.python.org/zh-cn/3.10/glossary.html#term-coroutine) coro 并返回结果。此函数会运行传入的协程，负责管理 `asyncio` 事件循环，终结异步生成器，并关闭线程池。

当有其他 `asyncio` 事件循环在同一线程中运行时，此函数不能被调用。

如果 `debug` 为 `True`，事件循环将以调试模式运行。此函数总是会创建一个新的事件循环并在结束时关闭之。它应当被用作 `asyncio` 程序的主入口点，理想情况下应当只被调用一次。

示例：

```python
async def main():
    await asyncio.sleep(1)
    print('hello')

asyncio.run(main())
```

## 创建任务

`asyncio.create_task(coro, *, name=None)` 将 `coro` 协程 封装为一个 `Task` 并调度其执行。返回 `Task` 对象。`name` 不为 `None`，它将使用 `Task.set_name()` 来设为任务的名称。

该任务会在 `get_running_loop()` 返回的循环中执行，如果当前线程没有在运行的循环则会引发 `RuntimeError`。此函数 在 Python 3.7 中被加入。在 Python 3.7 之前，可以改用低层级的 `asyncio.ensure_future()` 函数。

## 休眠

coroutine `asyncio.sleep(delay, result=None)` 阻塞 `delay` 指定的秒数。如果指定了 `result`，则当协程完成时将其返回给调用者。`sleep()` 总是会挂起当前任务，以允许其他任务运行。

将 `delay` 设置为 `0` 提供了优化的路径，以允许其他任务运行。长时间运行的函数可以使用它来避免在函数调用的整个过程中阻塞事件循环。

以下协程示例运行 5 秒，每秒显示一次当前日期：

```python
import asyncio
import datetime

async def display_date():
    loop = asyncio.get_running_loop()
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

asyncio.run(display_date())
```

## 并发运行任务 

awaitable `asyncio.gather(*aws, return_exceptions=False)` 并发 运行 `aws` 序列中的 <dfn class="xin-term">可等待对象</dfn>。如果 `aws` 中的某个可等待对象为<dfn class="xin-term">协程</dfn>，它将自动被作为一个任务调度。如果所有<dfn class="xin-term">可等待对象</dfn>都成功完成，结果将是一个由所有返回值聚合而成的列表。结果值的顺序与 `aws` 中可等待对象的顺序一致。

如果 `return_exceptions` 为 `False` (默认)，所引发的首个异常会立即传播给等待 `gather()` 的任务。`aws` 序列中的其他可等待对象 **不会被取消** 并将继续运行。如果 `return_exceptions` 为 `True`，异常会和成功的结果一样处理，并聚合至结果列表。

如果 `gather()` 被取消，所有被提交 (尚未完成) 的可等待对象也会 被取消。如果 `aws` 序列中的任一 Task 或 Future 对象 被取消，它将被当作引发了 CancelledError 一样处理 -- 在此情况下 `gather()` 调用 不会 被取消。这是为了防止一个已提交的 Task/Future 被取消导致其他 Tasks/Future 也被取消。

示例：

```python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number + 1):
        print(f"Task {name}: Compute factorial({i})...")
        await asyncio.sleep(1)
        f *= i
    print(f"Task {name}: factorial({number}) = {f}")

async def main():
    # Schedule three calls *concurrently*:
    await asyncio.gather(
        factorial("A", 2),
        factorial("B", 3),
        factorial("C", 4),
    )

asyncio.run(main())
```

<output>
Task A: Compute factorial(2)...
Task B: Compute factorial(2)...
Task C: Compute factorial(2)...
Task A: factorial(2) = 2
Task B: Compute factorial(3)...
Task C: Compute factorial(3)...
Task B: factorial(3) = 6
Task C: Compute factorial(4)...
Task C: factorial(4) = 24
</output>

## 屏蔽取消操作

awaitable `asyncio.shield(aw)` 保护一个 <dfn class="xin-term">可等待对象</dfn> 防止其被 取消。如果 `aw` 是一个协程，它将自动被作为任务调度。以下语句：

```python
res = await shield(something())
```

相当于：

```python
res = await something()
```

不同之处：如果包含它的协程被取消，在 `something()` 中运行的任务不会被取消。从 `something()` 的角度看来，取消操作并没有发生。然而其调用者已被取消，因此 `"await"` 表达式仍然会引发 `CancelledError`。如果通过其他方式取消 `something()` (例如在其内部操作) 则 `shield()` 也会取消。如果希望完全忽略取消操作 (不推荐) 则 `shield()` 函数需要配合一个 `try/except` 代码段，如下所示：

```python
try:
    res = await shield(something())
except CancelledError:
    res = None
```

## 超时

coroutine `asyncio.wait_for(aw, timeout)` 等待 `aw` 可等待对象 完成，指定 `timeout` 秒数后超时。如果 `aw` 是一个协程，它将自动被作为任务调度。`timeout` 可以为 `None`，也可以为 `float` 或 `int` 型数值表示的等待秒数。如果 `timeout` 为 `None`，则等待直到完成。

如果发生超时，任务将取消并引发 `asyncio.TimeoutError`。要避免任务 被取消，可以加上 `shield()`。此函数将等待直到 `Future` 确实被取消，所以总等待时间可能超过 `timeout`。如果在取消期间发生了异常，异常将会被传播。如果等待被取消，则 `aw` 指定的对象也会被取消。

示例：

```python
async def eternity():
    # Sleep for one hour
    await asyncio.sleep(3600)
    print('yay!')

async def main():
    # Wait for at most 1 second
    try:
        await asyncio.wait_for(eternity(), timeout=1.0)
    except asyncio.TimeoutError:
        print('timeout!')

asyncio.run(main())
```

<output>
timeout!
</output>

Python 3.8 版后已移除: `asyncio.wait`，所以尽量使用 `asyncio.wait_for`。

## as_completed

`asyncio.as_completed(aws, *, timeout=None)` 并发地运行 `aws` 可迭代对象中的 可等待对象。返回一个协程的迭代器。所返回的每个协程可被等待以从剩余的可等待对象的可迭代对象中获得最早的下一个结果。

如果在所有 `Future` 对象完成前发生超时则将引发 `asyncio.TimeoutError`。示例:

```python
for coro in as_completed(aws):
    earliest_result = await coro
    # ...
```

## 在线程中运行

`coroutine asyncio.to_thread(func, /, *args, **kwargs)` 在不同的线程中异步地运行函数 `func`。

向此函数提供的任何 `*args` 和 `**kwargs` 会被直接传给 `func`。并且，当前 `contextvars.Context` 会被传播，允许在不同的线程中访问来自事件循环的上下文变量。返回一个可被等待以获取 `func` 的最终结果的协程。

这个协程函数主要是用于执行在其他情况下会阻塞事件循环的 IO 密集型函数/方法。例如：

```python
def blocking_io():
    print(f"start blocking_io at {time.strftime('%X')}")
    # Note that time.sleep() can be replaced with any blocking
    # IO-bound operation, such as file operations.
    time.sleep(1)
    print(f"blocking_io complete at {time.strftime('%X')}")

async def main():
    print(f"started main at {time.strftime('%X')}")

    await asyncio.gather(
        asyncio.to_thread(blocking_io),
        asyncio.sleep(1))

    print(f"finished main at {time.strftime('%X')}")


asyncio.run(main())
```

<output>
started main at 14:55:04
start blocking_io at 14:55:04
blocking_io complete at 14:55:05
finished main at 14:55:05
</output>

在任何协程中直接调用 `blocking_io()` 将会在调用期间阻塞事件循环，导致额外的 1 秒运行时间。而通过改用 `asyncio.to_thread()`，我们可以在不同的线程中运行它从而不会阻塞事件循环。

## 跨线程调度

`asyncio.run_coroutine_threadsafe(coro, loop)` 向指定事件循环提交一个协程。（线程安全）

返回一个 `concurrent.futures.Future` 以等待来自其他 OS 线程的结果。

此函数应该从另一个 OS 线程中调用，而非事件循环运行所在线程。示例：

```python
# Create a coroutine
coro = asyncio.sleep(1, result=3)

# Submit the coroutine to a given loop
future = asyncio.run_coroutine_threadsafe(coro, loop)

# Wait for the result with an optional timeout argument
assert future.result(timeout) == 3
```

如果在协程内产生了异常，将会通知返回的 Future 对象。它也可被用来取消事件循环中的任务：

```python
try:
    result = future.result(timeout)
except asyncio.TimeoutError:
    print('The coroutine took too long, cancelling the task...')
    future.cancel()
except Exception as exc:
    print(f'The coroutine raised an exception: {exc!r}')
else:
    print(f'The coroutine returned: {result!r}')
```

不同与其他 `asyncio` 函数，此函数要求显式地传入 `loop` 参数。

## 内省

`asyncio.current_task(loop=None)` 返回当前运行的 `Task` 实例，如果没有正在运行的任务则返回 `None`。如果 `loop` 为 `None` 则会使用 `asyncio.get_running_loop()` 获取当前事件循环。

`asyncio.all_tasks(loop=None)` 返回事件循环所运行的未完成的 `Task` 对象的集合。如果 `loop` 为 `None`，则会使用 `asyncio.get_running_loop()` 获取当前事件循环。

## Task 对象

`class asyncio.Task(coro, *, name=None)` 与 [Future 类似](https://docs.python.org/zh-cn/3/library/asyncio-future.html#asyncio.Future)，可运行 Python 协程。非线程安全。

Task 对象被用来在事件循环中运行协程。如果一个协程在等待一个 `Future` 对象，`Task` 对象会挂起该协程的执行并等待该 `Future` 对象完成。当该 `Future` 对象 完成，被打包的协程将恢复执行。事件循环使用协同日程调度: 一个事件循环每次运行一个 Task 对象。而一个 Task 对象会等待一个 Future 对象完成，该事件循环会运行其他 Task、回调或执行 IO 操作。

使用高层级的 `asyncio.create_task()` 函数来创建 `Task` 对象，也可用低层级的 `asyncio.loop.create_task()` 或 `asyncio.ensure_future()` 函数。不建议手动实例化 `Task` 对象。

要取消一个正在运行的 `Task` 对象可使用 `asyncio.Task.cancel()` 方法。调用此方法将使该 `Task` 对象抛出一个 `CancelledError` 异常给打包的协程。如果取消期间一个协程正在等待一个 `Future` 对象，该 `Future` 对象也将被取消。

`asyncio.Task.cancelled()` 可被用来检测 `Task` 对象是否被取消。如果打包的协程没有抑制 `CancelledError` 异常并且确实被取消，该方法将返回 `True`。

`asyncio.Task` 从 `Future` 继承了其除 `Future.set_result()` 和 `Future.set_exception()` 以外的所有 API。

`Task` 对象支持 [contextvars](https://docs.python.org/zh-cn/3/library/contextvars.html#module-contextvars) 模块。当一个 `Task` 对象被创建，它将复制当前上下文，然后在复制的上下文中运行其协程。

### `cancel(msg=None)`

请求取消 `Task` 对象。这将安排在下一轮事件循环中抛出一个 `CancelledError` 异常给被封包的协程。协程在之后有机会进行清理甚至使用 `try ... ... except CancelledError ... finally` 代码块抑制异常来拒绝请求。不同于 `Future.cancel()`，`Task.cancel()` 不保证 `Task` 会被取消，虽然抑制完全取消并不常见，也很不鼓励这样做。

以下示例演示了协程是如何侦听取消请求的：

```python
async def cancel_me():
    print('cancel_me(): before sleep')

    try:
        # Wait for 1 hour
        await asyncio.sleep(3600)
    except asyncio.CancelledError:
        print('cancel_me(): cancel sleep')
        raise
    finally:
        print('cancel_me(): after sleep')

async def main():
    # Create a "cancel_me" Task
    task = asyncio.create_task(cancel_me())

    # Wait for 1 second
    await asyncio.sleep(1)

    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("main(): cancel_me is cancelled now")

asyncio.run(main())
```

<output>
cancel_me(): before sleep
cancel_me(): cancel sleep
cancel_me(): after sleep
main(): cancel_me is cancelled now
</output>

### `cancelled()`

如果 `Task` 对象 被取消 则返回 `True`。当使用 `cancel()` 发出取消请求时 `Task` 会被 取消，其封包的协程将传播被抛入的 `CancelledError` 异常。

### `done()`

如果 `Task` 对象 已完成 则返回 `True`。当 `Task` 所封包的协程返回一个值、引发一个异常或 `Task` 本身被取消时，则会被认为 已完成。

### `result()`

返回 `Task` 的结果。如果 `Task` 对象 已完成，其封包的协程的结果会被返回 (或者当协程引发异常时，该异常会被重新引发)。如果 `Task` 对象 被取消，此方法会引发一个 `CancelledError` 异常。如果 `Task` 对象的结果还不可用，此方法会引发一个 `InvalidStateError` 异常。

### `exception()`

返回 `Task` 对象的异常。如果所封包的协程引发了一个异常，该异常将被返回。如果所封包的协程正常返回则该方法将返回 `None`。如果 `Task` 对象 被取消，此方法会引发一个 `CancelledError` 异常。如果 `Task` 对象尚未 完成，此方法将引发一个 `InvalidStateError` 异常。

### `add_done_callback(callback, *, context=None)`

添加一个回调，将在 `Task` 对象 完成 时被运行。此方法应该仅在低层级的基于回调的代码中使用。要了解更多细节请查看 [Future.add_done_callback()](https://docs.python.org/zh-cn/3/library/asyncio-future.html#asyncio.Future.add_done_callback) 的文档。

### `remove_done_callback(callback)`

从回调列表中移除 `callback` 指定的回调。此方法应该仅在低层级的基于回调的代码中使用。

要了解更多细节请查看 [Future.remove_done_callback()](https://docs.python.org/zh-cn/3/library/asyncio-future.html#asyncio.Future.remove_done_callback) 的文档。

### `get_stack(*, limit=None)`

返回此 `Task` 对象的栈 frame 列表。如果所封包的协程未完成，这将返回其挂起所在的栈。如果协程已成功完成或被取消，这将返回一个空列表。如果协程被一个异常终止，这将返回回溯 frame 列表。

frame 总是从按从旧到新排序。

每个被挂起的协程只返回一个栈 frame。

可选的 `limit` 参数指定返回 frame 的数量上限；默认返回所有 frame。返回列表的顺序要看是返回一个栈还是一个回溯：栈返回最新的 frame，回溯返回最旧的frame。(这与 traceback 模块的行为保持一致。)

### `print_stack(*, limit=None, file=None)`

打印此 `Task` 对象的栈或回溯。

此方法产生的输出类似于 traceback 模块通过 get_stack() 所获取的框架。

`limit` 参数会直接传递给 `get_stack()`。

`file` 参数是输出所写入的 I/O 流；默认情况下输出会写入 `sys.stderr`。

### `get_coro()`

返回由 Task 包装的协程对象。

### get_name()

返回 Task 的名称。

如果没有一个 Task 名称被显式地赋值，默认的 asyncio Task 实现会在实例化期间生成一个默认名称。

#### `set_name(value)`

设置 Task 的名称。

`value` 参数可以为任意对象，它随后会被转换为字符串。

在默认的 Task 实现中，名称将在任务对象的 `repr()` 输出中可见