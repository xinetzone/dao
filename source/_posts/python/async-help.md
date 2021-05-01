---
layout: python
title: Python 异步编程技巧
date: 2021-04-22 08:04:00
tags: asyncio
categories: Python 编程
---

经验告诉我们永远不要面向用户的接口暴露 `Future` 对象，同时建议使用 [`loop.create_future()`](https://docs.python.org/zh-cn/3.10/library/asyncio-eventloop.html#asyncio.loop.create_future "asyncio.loop.create_future") 来创建 `Future` 对象。这种方法可以让 `Future` 对象使用其它的事件循环实现，它可以注入自己的优化实现。