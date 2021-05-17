---
layout: event
title: scheduler
date: 2021-05-17 11:17:08
tags: sched
---

`sched` 模块实现了一个通用事件调度程序，用于在特定时间运行任务。`scheduler` 类使用 `time` 函数来学习当前时间，使用  `delay` 函数等待特定的一段时间。实际的时间单位并不重要，这使得接口足够灵活，可以用于许多目的。

## 运行有延迟的事件

事件可以计划在延迟之后运行，或在特定时间运行。要使用延迟调度它们，可以使用`scheduler.enter(delay, priority, action, argument=(), kwargs={})`方法，该方法有四个主要参数。

- 表示延迟的数字
- 优先级值
- 要调用的函数
- 函数的参数元组

这个示例分别安排两个不同的事件在 $2$ 秒和 $3$ 秒后运行。当事件的时间出现时，将调用 `print_event()` 并打印当前时间和传递给事件的 `name` 参数。

```python
# sched_basic.py
import sched
import time

scheduler = sched.scheduler(time.time, time.sleep)


def print_event(name, start):
    now = time.time()
    elapsed = int(now - start)
    print(f'EVENT: {now} elapsed={elapsed} name={name}')


start = time.time()
print('START:', time.ctime(start))
scheduler.enter(2, 1, print_event, ('first', start))
scheduler.enter(3, 1, print_event, ('second', start))

scheduler.run()
```

<output>
START: Mon May 17 13:34:28 2021
EVENT: Mon May 17 13:34:30 2021 elapsed=2 name=first
EVENT: Mon May 17 13:34:31 2021 elapsed=3 name=second
</output>

为第一个事件打印的时间是在开始后 $2$ 秒，为第二个事件打印的时间是在开始后 $3$ 秒。

## 重叠的事件

对`run()`的调用会阻塞，直到所有事件都被处理完。每个事件都在同一个线程中运行，所以如果一个事件的运行时间比事件之间的延迟时间长，就会出现重叠。通过推迟后面的事件来解决重叠问题。事件不会丢失，但是有些事件可能会在计划时间之后被调用。在下一个例子中，`long_event()` 会休眠，但它可以通过执行长时间的计算或阻塞I/O来轻易地延迟。

```python
# sched_overlap.py
import sched
import time

scheduler = sched.scheduler(time.time, time.sleep)


def long_event(name):
    print('BEGIN EVENT :', time.ctime(time.time()), name)
    time.sleep(2)
    print('FINISH EVENT:', time.ctime(time.time()), name)


print('START:', time.ctime(time.time()))
scheduler.enter(2, 1, long_event, ('first',))
scheduler.enter(3, 1, long_event, ('second',))

scheduler.run()
```

输出：

<output>
START: Mon May 17 13:43:54 2021
BEGIN EVENT : Mon May 17 13:43:56 2021 first
FINISH EVENT: Mon May 17 13:43:58 2021 first
BEGIN EVENT : Mon May 17 13:43:58 2021 second
FINISH EVENT: Mon May 17 13:44:00 2021 second
</output>

结果是在第一个事件结束后立即运行第二个事件，因为第一个事件花费了足够长的时间来推动时钟超过第二个事件的期望开始时间。

## 事件优先级

如果计划在同一时间调度多个事件，则将使用它们的优先级值来确定它们的运行顺序。

```python
# sched_priority.py
import sched
import time

scheduler = sched.scheduler(time.time, time.sleep)


def print_event(name):
    print('EVENT:', time.ctime(time.time()), name)


now = time.time()
print('START:', time.ctime(now))
scheduler.enterabs(now + 2, 2, print_event, ('first',))
scheduler.enterabs(now + 2, 1, print_event, ('second',))

scheduler.run()
```

输出：

<output>
START: Mon May 17 13:50:50 2021
EVENT: Mon May 17 13:50:52 2021 second
EVENT: Mon May 17 13:50:52 2021 first
</output>

这个示例需要确保它们被安排在完全相同的时间，因此使用 `enterabs()` 方法而不是 `enter()`。`enterabs()` 的第一个参数是运行事件的时间，而不是延迟的时间。

## 取消事件

`enter()` 和 `enterabs()` 都返回一个对事件的引用，可以在以后用来取消该事件。因为 `run()` 会阻塞，所以必须在不同的线程中取消该事件。对于本例，将启动一个线程来运行调度程序，并使用主处理线程来取消事件。

```python
# sched_cancel.py
import sched
import threading
import time

scheduler = sched.scheduler(time.time, time.sleep)

# Set up a global to be modified by the threads
counter = 0


def increment_counter(name):
    global counter
    print('EVENT:', time.ctime(time.time()), name)
    counter += 1
    print('NOW:', counter)


print('START:', time.ctime(time.time()))
e1 = scheduler.enter(2, 1, increment_counter, ('E1',))
e2 = scheduler.enter(3, 1, increment_counter, ('E2',))

# Start a thread to run the events
t = threading.Thread(target=scheduler.run)
t.start()

# Back in the main thread, cancel the first scheduled event.
scheduler.cancel(e1)

# Wait for the scheduler to finish running in the thread
t.join()

print('FINAL:', counter)
```

原定有两项活动，但第一项后来被取消。只有第二个事件运行，因此计数器变量只增加一次。

<output>
START: Mon May 17 13:55:10 2021
EVENT: Mon May 17 13:55:13 2021 E2
NOW: 1
FINAL: 1
</output>