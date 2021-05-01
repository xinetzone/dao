---
title: CherryPy 源码解析
lang: zh-CN
abbrlink: cdbf627f
date: 2021-03-16 14:26:39
description:
updated:
tags: CherryPy
categories: 教程
---

## cherrypy.process.wspbus.Bus

`cherrypy.process.wspbus.Bus` 处理用于 HTTP 站点部署的状态机（state-machine）和 Messenger。

即使同一频道上的其他监听器失败，也可以确保调用给定频道的所有监听器。记录每个失败，但是执行继续到下一个侦听器。停止从侦听器内部进行所有处理的唯一方法是提高 SystemExit 并停止整个服务器。