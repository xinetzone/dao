---
layout: pytorch
title: PyTorch 讨论
date: 2021-05-10 23:07:12
tags:
---

1. 可以认为 `a.reshape = a.view() + a.contiguous().view()`，所以建议仅仅使用 `reshape` 即可。（参考[PyTorch：view() 与 reshape() 区别详解](https://blog.csdn.net/Flag_ing/article/details/109129752)）