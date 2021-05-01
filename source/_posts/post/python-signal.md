---
title: signal --- 设置异步事件处理程序
lang: zh-CN
tags: signal
categories: Python 编程
abbrlink: e3577543
date: 2021-04-20 13:52:56
---

该模块提供了在 Python 中使用信号处理程序的机制。

## 一般规则

`signal.signal()` 函数允许定义在接收到信号时执行的自定义处理程序。少量的默认处理程序已经设置：`SIGPIPE` 被忽略（因此管道和套接字上的写入错误可以报告为普通的 Python 异常）以及如果父进程没有更改 `SIGINT`，则其会被翻译成 `KeyboardInterrupt` 异常。一旦设置，特定信号的处理程序将保持安装，直到它被显式重置（Python 模拟 BSD 样式接口而不管底层实现），但 `SIGCHLD` 的处理程序除外，它遵循底层实现。

### 执行 Python 信号处理程序

Python 信号处理程序不会在低级（ C ）信号处理程序中执行。相反，低级信号处理程序设置一个标志，告诉 [virtual machine](https://docs.python.org/zh-cn/3.10/glossary.html#term-virtual-machine) 稍后执行相应的 Python 信号处理程序（例如在下一个 bytecode 指令）。这会导致：

- 捕获同步错误是没有意义的，例如 `SIGFPE` 或 `SIGSEGV`，它们是由 C 代码中的无效操作引起的。Python 将从信号处理程序返回到 C 代码，这可能会再次引发相同的信号，导致 Python 显然的挂起。从Python 3.3 开始，你可以使用 [`faulthandler`](https://docs.python.org/zh-cn/3.10/library/faulthandler.html#module-faulthandler) 模块来报告同步错误。
- 纯 C 中实现的长时间运行的计算（例如在大量文本上的正则表达式匹配）可以在任意时间内不间断地运行，而不管接收到任何信号。计算完成后将调用 Python 信号处理程序。

### 信号与线程

Python 信号处理程序总是会在主 Python 主解释器的主线程中执行，即使信号是在另一个线程中接收的。这意味着信号不能被用作线程间通信的手段。你可以改用 [threading](https://docs.python.org/zh-cn/3.10/library/threading.html#module-threading) 模块中的同步原语。

此外，只有主解释器的主线程才被允许设置新的信号处理程序。

更多内容见：[11.3. signal — 同步系统事件](https://learnku.com/docs/pymotw/signal-asynchronous-system-events/3420)