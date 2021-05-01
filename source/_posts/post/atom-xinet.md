---
title: 统一 MXNet，PyTorch，TensorFlow 接口
lang: zh-CN
tags: [深度学习, Python]
categories: xint
abbrlink: cb8e14f
date: 2021-04-07 22:15:36
---

为了提供一个统一的接口，我在 GitHub 上维护了一个通用 API：[atom](https://github.com/xinetzone/atom)。可以使用 `pip` 安装。本仓库借鉴了 [d2l](https://zh-v2.d2l.ai/) 和 [简单粗暴 TensorFlow 2](https://tf.wiki/zh_hans/)。

首先导入一些库：

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

## 简单的使用 atom

由于 `np` 已经绑定了各自的深度学习环境，且三种框架及其相似，所以，下文如果框架之间没有分歧，统一使用没有指示环境的模式，比如：

```python
n = 10000
a = np.ones(n)
b = np.ones(n)
```

你可以分别查看各自环境的数据：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--4" name="tab-set--1" type="radio">
        <label for="tab-set--0-input--4">MXNet</label>
        <div class="tab-content w3-padding">
            ```python
            a
            ```
            <output>array([1., 1., 1., ..., 1., 1., 1.])</output>
        </div>
        <input id="tab-set--0-input--5" name="tab-set--1" type="radio">
        <label for="tab-set--0-input--5">TensorFlow</label>
        <div class="tab-content w3-padding">
            ```python
            a
            ```
            <output>&lt;ndarray&lt;&lt;tf.Tensor: shape=(10000,), dtype=float64, numpy=array([1., 1., 1., ..., 1., 1., 1.])&gt;&gt;</output>
        </div>
        <input id="tab-set--0-input--6" name="tab-set--1" type="radio">
        <label for="tab-set--0-input--6">PyTorch</label>
        <div class="tab-content w3-padding">
            ```python
            a
            ```
            <output>tensor([1., 1., 1.,  ..., 1., 1., 1.])</output>
        </div>
    </div>
</article>

可以看到不同环境表示的不同的 `np`，可以无缝的使用它。

`atom` 定义了一个计时器 `Timer`，下面测试矢量化的好处。

1. 使用 `for` 循环：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--7" name="tab-set--2" type="radio">
        <label for="tab-set--0-input--7">MXNet</label>
        <div class="tab-content w3-padding">
            ```python
            c = np.zeros(n)
            timer = utils.Timer()
            for i in range(n):
                c[i] = a[i] + b[i]
            f'{timer.stop():.5f} sec'
            ```
            <output>'2.83601 sec'</output>
        </div>
        <input id="tab-set--0-input--8" name="tab-set--2" type="radio">
        <label for="tab-set--0-input--8">TensorFlow</label>
        <div class="tab-content w3-padding">
            ```python
            import tensorflow as tf

            c = tf.Variable(tf.zeros(n))
            timer = utils.Timer()
            for i in range(n):
                c[i].assign(a[i] + b[i])
            f'{timer.stop():.5f} sec'
            ```
            <output>'3.42657 sec'</output>
        </div>
        <input id="tab-set--0-input--9" name="tab-set--2" type="radio">
        <label for="tab-set--0-input--9">PyTorch</label>
        <div class="tab-content w3-padding">
            ```python
            c = np.zeros(n)
            timer = utils.Timer()
            for i in range(n):
                c[i] = a[i] + b[i]
            f'{timer.stop():.5f} sec'
            ```
            <output>'0.13695 sec'</output>
        </div>
    </div>
</article>

2. 使用重载的 `+` 运算符来计算按张量的和。

```python
timer.start()
d = a + b
f'{timer.stop():.9f} sec'
```

输出的时间有点差异，但是都很快：

<article>
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--10" name="tab-set--3" type="radio">
        <label for="tab-set--0-input--10">MXNet</label>
        <div class="tab-content w3-padding">
            <output>'0.0009992 sec'</output>
        </div>
        <input id="tab-set--0-input--11" name="tab-set--3" type="radio">
        <label for="tab-set--0-input--11">TensorFlow</label>
        <div class="tab-content w3-padding">
            <output>'0.000999451 sec'</output>
        </div>
        <input id="tab-set--0-input--12" name="tab-set--3" type="radio">
        <label for="tab-set--0-input--12">PyTorch</label>
        <div class="tab-content w3-padding">
            <output>'0.000944614 sec'</output>
        </div>
    </div>
</article>

可以看出矢量化对运算速度的提升是数量级的。

## 正态分布与平方损失

**正态分布**（normal distribution），也被称为 **高斯分布**（Gaussian distribution），最早由德国数学家高斯（Gauss）应用于天文学研究。简单的说，若随机变量 $x$ 具有均值 $\mu$ 和方差 $\sigma^2$（标准差 $\sigma$），其正态分布概率密度函数如下：

<section>
$$
p(x) = \frac{1}{\sqrt{2 \pi \sigma^2}} \exp\left(-\frac{1}{2 \sigma^2} (x - \mu)^2\right).
$$
</section>

使用 `np` 可以实现：

```python
def normal(x, mu, sigma):
    p = 1 / np.sqrt(2 * np.pi * sigma**2)
    return p * np.exp(-0.5 / sigma**2 * (x - mu)**2)
```

可视化正态分布：

```python
x = np.arange(-7, 7, 0.01)

# 均值和标准差对
params = [(0, 1), (0, 2), (3, 1)]
utils.plot(x, [normal(x, mu, sigma) for mu, sigma in params], xlabel='x',
           ylabel='p(x)', figsize=(4.5, 2.5),
           legend=[f'mean {mu}, std {sigma}' for mu, sigma in params])
```

<output></output>![](output_2_0.svg)

改变均值会产生沿 $x$ 轴的偏移，增加方差将会分散分布、降低其峰值。

利用**均方误差损失**函数（简称**均方损失**）可以用于线性回归的一个原因是：假设观测 $\mathbf{x}$ 中包含噪声，其中噪声服从正态分布。噪声正态分布如下式:

<section>
$$\tag{1.1} y = \mathbf{w}^\top \mathbf{x} + b + \epsilon \text{ where } \epsilon \sim \mathcal{N}(0, \sigma^2).$$
</section>

因此，我们现在可以写出通过给定的观测 $\mathbf{x}$  到特定 $y$ 的似然（likelihood）：

<section>
$$\tag{1.2} P(y \mid \mathbf{x}) = \frac{1}{\sqrt{2 \pi \sigma^2}} \exp\left(-\frac{1}{2 \sigma^2} (y - \mathbf{w}^\top \mathbf{x} - b)^2\right).$$
</section>

根据最大似然估计法，参数 $\mathbf{w}$ 和 $b$ 的最优值是使整个数据集的似然最大的值：

<section>
$$\tag{1.3} P(\mathbf y \mid \mathbf X) = \prod_{i=1}^{n} p(y^{(i)}|\mathbf{x}^{(i)}).$$
</section>

根据最大似然估计法选择的估计量称为**最大似然估计量** 。为了更好的计算，可以**最小化负对数似然** $-\log P(\mathbf y \mid \mathbf X)$。由此可以得到的数学公式是：

<section>
$$\tag{1.4} -\log P(\mathbf y \mid \mathbf X) = \sum_{i=1}^n \frac{1}{2} \log(2 \pi \sigma^2) + \frac{1}{2 \sigma^2} \left(y^{(i)} - \mathbf{w}^\top \mathbf{x}^{(i)} - b\right)^2.$$
</section>

现在我们只需要假设 $\sigma$ 是某个固定常数就可以忽略第一项，因为第一项不依赖于 $\mathbf{w}$ 和 $b$。现在第二项除了常数 $\frac{1}{\sigma^2}$ 外，其余部分和前面介绍的平方误差损失是一样的。因此，<span class="w3-card w3-pale-blue">在高斯噪声的假设下，最小化均方误差等价于对线性模型的最大似然估计</span>。