---
title: 并发执行任务
lang: zh-CN
abbrlink: 34d3d930
date: 2021-04-21 12:42:44
tags: asyncio
categories: Python 编程
---

任务（`Task`）是与事件循环进行交互的主要方式之一。任务包装协程并跟踪它们何时完成。任务是 `Future` 的子类，因此其他协程可以等待它们，并且每个协程都有可以在任务完成后检索的结果。

## 创建任务

要启动任务，请使用 `create_task()` 创建一个 `Task` 实例。只要循环正在运行且协程不返回，结果任务将作为事件循环管理的并发操作的一部分运行。

```python
# asyncio_create_task.py
import asyncio


async def task_func():
    print('in task_func')
    return 'the result'


async def main(loop):
    print('creating task')
    task = loop.create_task(task_func())
    print(f'waiting for {task}')
    return_value = await task
    print(f'task completed {task}')
    print(f'return value: {return_value}')


event_loop = asyncio.get_event_loop()
try:
    event_loop.run_until_complete(main(event_loop))
finally:
    event_loop.close()
```

本示例在 `main()` 函数退出之前等待任务返回结果。

```shell
$ python3 asyncio_create_task.py

creating task
waiting for <Task pending coro=<task_func() running at
asyncio_create_task.py:12>>
in task_func
task completed <Task finished coro=<task_func() done, defined at
asyncio_create_task.py:12> result='the result'>
return value: 'the result'
```

## 取消任务

通过保留从 `create_task()` 返回的 `Task` 对象，可以在任务完成之前取消其操作。

```python
# asyncio_cancel_task.py
import asyncio


async def task_func():
    print('in task_func')
    return 'the result'


async def main(loop):
    print('creating task')
    task = loop.create_task(task_func())

    print('canceling task')
    task.cancel()

    print(f'canceled task {task}')
    try:
        await task
    except asyncio.CancelledError:
        print('caught error from canceled task')
    else:
        print(f'task result: {task.result()}')


event_loop = asyncio.get_event_loop()
try:
    event_loop.run_until_complete(main(event_loop))
finally:
    event_loop.close()
```

本示例在启动事件循环之前创建并取消任务。结果是来自 `run_until_complete()` 的 `CancelledError` 异常。

```shell
$ python3 asyncio_cancel_task.py

creating task
canceling task
canceled task <Task cancelling coro=<task_func() running at
asyncio_cancel_task.py:12>>
caught error from canceled task
```

如果任务在等待另一个并发操作时被取消，则通过在任务等待时引发 `CancelledError` 异常来通知该任务已取消。

```python
# asyncio_cancel_task2.py
import asyncio


async def task_func():
    print('in task_func, sleeping')
    try:
        await asyncio.sleep(1)
    except asyncio.CancelledError:
        print('task_func was canceled')
        raise
    return 'the result'


def task_canceller(t):
    print('in task_canceller')
    t.cancel()
    print('canceled the task')


async def main(loop):
    print('creating task')
    task = loop.create_task(task_func())
    loop.call_soon(task_canceller, task)
    try:
        await task
    except asyncio.CancelledError:
        print('main() also sees task as canceled')


event_loop = asyncio.get_event_loop()
try:
    event_loop.run_until_complete(main(event_loop))
finally:
    event_loop.close()
```

如有必要，捕获异常可提供清理已完成工作的机会。

```shell
$ python3 asyncio_cancel_task2.py

creating task
in task_func, sleeping
in task_canceller
canceled the task
task_func was canceled
main() also sees task as canceled
```

## 从协程中创建任务

`ensure_future()` 函数返回与协程的执行相关的 `Task`。然后可以将该 `Task` 实例传递给其他代码，后者可以在不知道原始协程如何构造或调用的情况下等待它。

```python
# asyncio_ensure_future.py
import asyncio


async def wrapped():
    print('wrapped')
    return 'result'


async def inner(task):
    print('inner: starting')
    print(f'inner: waiting for {task}')
    result = await task
    print(f'inner: task returned {result}')


async def starter():
    print('starter: creating task')
    task = asyncio.ensure_future(wrapped())
    print('starter: waiting for inner')
    await inner(task)
    print('starter: inner returned')


event_loop = asyncio.get_event_loop()
try:
    print('entering event loop')
    result = event_loop.run_until_complete(starter())
finally:
    event_loop.close()
```

请注意，直到有某种使用 `await` 执行的协程才启动给 `ensure_future()` 的协程。

```shell
$ python3 asyncio_ensure_future.py

entering event loop
starter: creating task
starter: waiting for inner
inner: starting
inner: waiting for <Task pending coro=<wrapped() running at
asyncio_ensure_future.py:12>>
wrapped
inner: task returned 'result'
starter: inner returned
```

