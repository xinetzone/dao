---
layout: xint
title: 简单的梯度运算
date: 2021-05-04 16:44:49
tags: 深度学习
categories: xint
---

## 理论

有矩阵 $\mathbf{A} = [\mathbf{a}_1, \mathbf{a}_2, \cdots, \mathbf{a}_m]^T \in \mathbb{R}^{m \times n}$，和向量 $\mathbf{x} = [x_1, x_2, \cdots, x_n]^T \in \mathbb{R}^{n}$，则有：

<div>
$$
\mathbf{Ax} = \begin{bmatrix}
   \mathbf{a}_1^T \mathbf{x} \\
   \mathbf{a}_2^T \mathbf{x} \\
   \vdots \\
   \mathbf{a}_m^T \mathbf{x}
\end{bmatrix}
$$
</div>

又有：

<div>
$$
\nabla_{\mathbf{x}} \mathbf{a}_i^T \mathbf{x} = \mathbf{a}_i
$$
$$
 \mathbf{1}^T \mathbf{A} \mathbf{x} = \langle \mathbf{1}, \mathbf{Ax} \rangle = \langle \mathbf{A}^T \mathbf{1}, \mathbf{x} \rangle
$$
</div>

所以，

<div>
$$
\nabla_{\mathbf{x}} \mathbf{1}^T \mathbf{A} \mathbf{x} = 
\nabla_{\mathbf{x}} \sum_{i=1}^m \mathbf{a}_i^T \mathbf{x} = \sum_{i=1}^m \mathbf{a}_i = \mathbf{A}^T \mathbf{1}_{m \times 1} = \mathbf{A}^T \mathbf{1}
$$
</div>

这样，有：

<div>
$$
\nabla_{\mathbf{x}} \mathbf{Ax} = \nabla_{\mathbf{x}} \langle  \mathbf{A}^T, \mathbf{x} \mathbf{1}^T \rangle = \mathbf{A}^T
$$
$$
\nabla_{\mathbf{x}} \mathbf{x}^\top \mathbf{A} = \mathbf{A}
$$
$$
\nabla_{\mathbf{x}} \mathbf{x}^\top \mathbf{A} \mathbf{x} = (\mathbf{A} + \mathbf{A}^\top)\mathbf{x}
$$
$$
\nabla_{\mathbf{x}} \|\mathbf{x} \|^2 = \nabla_{\mathbf{x}} \mathbf{x}^\top \mathbf{x} = 2\mathbf{x}
$$
$$
\nabla_{\mathbf{X}} \|\mathbf{X} \|_F^2 = 2\mathbf{X}
$$
</div>

即：

<div>
$$
\nabla_{\mathbf{x}} \langle f(\mathbf{x}), g(\mathbf{x}) \rangle = \langle \nabla{_\mathbf{x}} f(\mathbf{x}), g(\mathbf{x}) \rangle + \langle f(\mathbf{x}), \nabla_{\mathbf{x}} g(\mathbf{x}) \rangle
$$
</div>

下面看一个例子：

## 一个例子

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--1" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--1">MXNet</label>
        <div class="tab-content w3-padding">
            ```python
            from xint import utils
            from xint import mxnet as xint

            np = xint.np
            ```
        </div>
        <input id="tab-set--0-input--2" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--2">TensorFlow</label>
        <div class="tab-content w3-padding">
            ```python
            from xint import utils
            from xint import tensorflow as xint

            np = xint.np
            ```
        </div>
        <input id="tab-set--0-input--3" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--3">PyTorch</label>
        <div class="tab-content w3-padding">
            ```python
            from xint import utils
            from xint import torch as xint

            np = xint.np
            ```
        </div>
    </div>
</article>

创建张量 $\mathbf{x}$：

```python
x = np.arange(4.0).reshape(4, 1)
```

计算函数 $y = 2\mathbf{x}^{\top}\mathbf{x}$ 的梯度：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--1-input--1" name="tab-set--1" type="radio">
        <label for="tab-set--1-input--1">MXNet</label>
        <div class="tab-content w3-padding">
            ```python
            # 我们通过调用`attach_grad`来为一个张量的梯度分配内存
            x.attach_grad()
            # 在我们计算关于`x`的梯度后，我们将能够通过'grad'属性访问它，它的值被初始化为0
            x.grad
            ```
        </div>
        <input id="tab-set--1-input--2" name="tab-set--1" type="radio">
        <label for="tab-set--1-input--2">TensorFlow</label>
        <div class="tab-content w3-padding">
            ```python
            x = tf.Variable(x)
            ```
        </div>
        <input id="tab-set--1-input--3" name="tab-set--1" type="radio">
        <label for="tab-set--1-input--3">PyTorch</label>
        <div class="tab-content w3-padding">
            ```python
            x.requires_grad_(True)  # 等价于 `x = torch.arange(4.0, requires_grad=True)`
            x.grad  # 默认值是None
            ```
        </div>
    </div>
</article>

现在让计算 $y$：

<div class="tab-set w3-light-grey">
    <input checked="True" id="tab-set--2-input--1" name="tab-set--2" type="radio">
    <label for="tab-set--2-input--1">MXNet</label>
    <div class="tab-content w3-padding">
    ```python
    from mxnet import autograd
    # 把代码放到`autograd.record`内，以建立计算图
    with autograd.record():
        y = 2 * x.T @ x
    float(y)
    ```
    </div>
    <input id="tab-set--2-input--2" name="tab-set--2" type="radio">
    <label for="tab-set--2-input--2">TensorFlow</label>
    <div class="tab-content w3-padding">
    ```python
    # 把所有计算记录在磁带上
    with tf.GradientTape() as t:
        y = 2 * tf.transpose(x) @ x
    float(y)
    ```
    </div>
    <input id="tab-set--2-input--3" name="tab-set--2" type="radio">
    <label for="tab-set--2-input--3">PyTorch</label>
    <div class="tab-content w3-padding">
    ```python
    y = 2 * x.T @ x
    float(y)
    ```
    </div>
</div>

<output>28.0</output>

接下来，我们可以通过调用反向传播函数来自动计算 $y$ 关于 $\mathbf{x}$ 每个分量的梯度，并打印这些梯度：

<div class="tab-set w3-light-grey">
    <input checked="True" id="tab-set--3-input--1" name="tab-set--3" type="radio">
    <label for="tab-set--3-input--1">MXNet</label>
    <div class="tab-content w3-padding">
    ```python
    y.backward()
    x.grad
    ```
    <output>
    array([[ 0.],
       [ 4.],
       [ 8.],
       [12.]])</output>
    </div>
    <input id="tab-set--3-input--2" name="tab-set--3" type="radio">
    <label for="tab-set--3-input--2">TensorFlow</label>
    <div class="tab-content w3-padding">
    ```python
    # 把所有计算记录在磁带上
    with tf.GradientTape() as t:
        y = 2 * tf.transpose(x) @ x
    float(y)
    ```
    <output>
    &lt;tf.Tensor: shape=(4, 1), dtype=float64, numpy=
    array([[ 0.],
        [ 4.],
        [ 8.],
        [12.]])></output>
    </div>
    <input id="tab-set--3-input--3" name="tab-set--3" type="radio">
    <label for="tab-set--3-input--3">PyTorch</label>
    <div class="tab-content w3-padding">
    ```python
    y.backward()
    x.grad
    ```
    <output>
    tensor([[ 0.],
        [ 8.],
        [16.],
        [24.]])</output>
    </div>
</div>

可以计算 $\mathbf{x}$ 的另一个函数：

<div class="tab-set w3-light-grey">
    <input checked="True" id="tab-set--4-input--1" name="tab-set--4" type="radio">
    <label for="tab-set--4-input--1">MXNet</label>
    <div class="tab-content w3-padding">
    ```python
    with autograd.record():
        y = x.sum()
    y.backward()
    x.grad  # 被新计算的梯度覆盖
    ```
    <output>
    array([[1.],
       [1.],
       [1.],
       [1.]])
    </output>
    </div>
    <input id="tab-set--4-input--2" name="tab-set--4" type="radio">
    <label for="tab-set--4-input--2">TensorFlow</label>
    <div class="tab-content w3-padding">
    ```python
    with tf.GradientTape() as t:
        y = tf.reduce_sum(x)
    t.gradient(y, x)  # 被新计算的梯度覆盖
    ```
    <output>
    &lt;tf.Tensor: shape=(4, 1), dtype=float64, numpy=
    array([[1.],
        [1.],
        [1.],
        [1.]])>
    </output>
    </div>
    <input id="tab-set--4-input--3" name="tab-set--4" type="radio">
    <label for="tab-set--4-input--3">PyTorch</label>
    <div class="tab-content w3-padding">
    ```python
    # 在默认情况下，PyTorch会累积梯度，我们需要清除之前的值
    x.grad.zero_()
    y = x.sum()
    y.backward()
    x.grad
    ```
    <output>
    tensor([[1.],
        [1.],
        [1.],
        [1.]])
    </output>
    </div>
</div>

注意：对于非标量变量的反向传播，MXNet/TensorFlow 直接调用相应的函数即可获得梯度，但是 Pytorch 不支持直接对非标量进行反向传播，故而需要先对其求和，再求梯度。比如：

<div class="tab-set w3-light-grey">
    <input checked="True" id="tab-set--5-input--1" name="tab-set--5" type="radio">
    <label for="tab-set--5-input--1">MXNet</label>
    <div class="tab-content w3-padding">
    ```python
    # 当我们对向量值变量`y`（关于`x`的函数）调用`backward`时，
    # 将通过对`y`中的元素求和来创建一个新的标量变量。然后计算这个标量变量相对于`x`的梯度
    with autograd.record():
        y = x * x  # `y`是一个向量
    y.backward()
    x.grad  # 等价于y = sum(x * x)
    ```
    </div>
    <input id="tab-set--5-input--2" name="tab-set--5" type="radio">
    <label for="tab-set--5-input--2">TensorFlow</label>
    <div class="tab-content w3-padding">
    ```python
    with tf.GradientTape() as t:
        y = x * x
    t.gradient(y, x)  # 等价于 `y = tf.reduce_sum(x * x)`
    ```
    </div>
    <input id="tab-set--5-input--3" name="tab-set--5" type="radio">
    <label for="tab-set--5-input--3">PyTorch</label>
    <div class="tab-content w3-padding">
    ```python
    # 对非标量调用`backward`需要传入一个`gradient`参数，该参数指定微分函数关于`self`的梯度。
    ## 在我们的例子中，我们只想求偏导数的和，所以传递一个1的梯度是合适的
    x.grad.zero_()
    y = x * x
    # 等价于y.backward(torch.ones(len(x)))
    y.sum().backward()
    x.grad
    ```
    </div>
</div>

## 分离计算

有时，我们希望将某些计算移动到记录的计算图之外。例如，假设 $\mathbf{y}$ 是作为 $\mathbf{x}$ 的函数计算的，而 $\mathbf{z}$ 则是作为 $\mathbf{y}$ 和 $\mathbf{x}$ 的函数计算的。现在，想象一下，我们想计算 $\mathbf{z}$ 关于 $\mathbf{x}$ 的梯度，但由于某种原因，我们希望将 $\mathbf{y}$ 视为一个常数，并且只考虑到 $\mathbf{x}$ 在 $\mathbf{y}$ 被计算后发挥的作用。

在这里，我们可以分离 $\mathbf{y}$ 来返回一个新变量 $u$，该变量与 $\mathbf{y}$ 具有相同的值，但截断计算图中关于如何计算 $\mathbf{y}$ 的任何信息。换句话说，梯度不会向后流经 $u$ 到 $\mathbf{x}$。因此，下面的反向传播函数计算 $\mathbf{z} = u * \mathbf{x}$ 关于 $\mathbf{x}$ 的偏导数，同时将 $u$ 作为常数处理，而不是 $\mathbf{z} = \mathbf{x} * \mathbf{x} * \mathbf{x}$ 关于 $\mathbf{x}$ 的偏导数。

<div class="tab-set w3-light-grey">
    <input checked="True" id="tab-set--6-input--1" name="tab-set--6" type="radio">
    <label for="tab-set--6-input--1">MXNet</label>
    <div class="tab-content w3-padding">
    ```python
    with autograd.record():
        y = x * x
        u = y.detach()
        z = u * x
    z.backward()
    x.grad == u
    ```
    </div>
    <input id="tab-set--6-input--2" name="tab-set--6" type="radio">
    <label for="tab-set--6-input--2">TensorFlow</label>
    <div class="tab-content w3-padding">
    ```python
    # 设置 `persistent=True` 来运行 `t.gradient`多次
    with tf.GradientTape(persistent=True) as t:
        y = x * x
        u = tf.stop_gradient(y)
        z = u * x

    x_grad = t.gradient(z, x)
    x_grad == u
    ```
    </div>
    <input id="tab-set--6-input--3" name="tab-set--6" type="radio">
    <label for="tab-set--6-input--3">PyTorch</label>
    <div class="tab-content w3-padding">
    ```python
    x.grad.zero_()
    y = x * x
    u = y.detach()
    z = u * x

    z.sum().backward()
    x.grad == u
    ```
    </div>
</div>

## 通用微分函数

令 $f: \mathbb{R}^n \rightarrow \mathbb{R}$，$\mathbf{x} = [x_1, x_2, \ldots, x_n]^\top$，有

<div>
$$
\nabla_{\mathbf{x}} f(\mathbf{x}) = \bigg[\frac{\partial f(\mathbf{x})}{\partial x_1}, \frac{\partial f(\mathbf{x})}{\partial x_2}, \ldots, \frac{\partial f(\mathbf{x})}{\partial x_n}\bigg]^\top
$$
</div>

若有 $\mathbf{y} = [y_1, y_2, \ldots, y_m]^\top$，$x \in \mathbb{R}$，则：

<div>
$$
\frac{\partial \mathbf{y}}{\partial x} = \bigg[\frac{\partial y_1}{\partial x}, \frac{\partial y_2}{\partial x}, \ldots, \frac{\partial y_m}{\partial x}\bigg]^\top
$$
</div>

还有，

<div>
$$
\frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \bigg[\frac{\partial y_1}{\partial \mathbf{x}}, \frac{\partial y_2}{\partial \mathbf{x}}, \ldots, \frac{\partial y_m}{\partial \mathbf{x}}\bigg]^\top = \begin{bmatrix} \frac{\partial y_1}{\partial x_1} & \frac{\partial y_1}{\partial x_2} &\cdots &\frac{\partial y_1}{\partial x_n} \\
\frac{\partial y_2}{\partial x_1} & \frac{\partial y_2}{\partial x_2} & \cdots & \frac{\partial y_2}{\partial x_n}\\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial y_m}{\partial x_1} & \frac{\partial y_m}{\partial x_2} & \cdots & \frac{\partial y_m}{\partial x_n}
\end{bmatrix}
$$
</div>

