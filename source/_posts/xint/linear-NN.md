---
title: 深度学习基础：线性回归
lang: zh-CN
tags: 线性回归
categories: xint
description: 使用 TensorFlow，MXNet，PyTorch 实现线性回归
updated:
---

对 <span class="w3-yellow">样本</span> 做如下约定：

<article>
$$
\tag{1} \mathbf{X} = \begin{bmatrix}
\mathbf{x}_1^T \\ \mathbf{x}_2^T \\ \vdots \\ \mathbf{x}_m^T
\end{bmatrix} \in \mathbb{R}^{m \times n} 
$$
$$
\begin{matrix}
\tag{2}
\mathbf{x}_i = \begin{bmatrix}
x_{i1} \\ x_{i2} \\ \vdots \\ x_{in}
\end{bmatrix} \in \mathbb{R}^n, & i \in \{1, \cdots, m\}
\end{matrix}
$$
</article>

## 模型定义

若有权重 $\mathbf{w} = (w_1, w_2, \cdots, w_n)^T \in \mathbb{R}^n$，偏置 $b \in \mathbb{R}$，则<span class="w3-yellow">线性模型</span>可以表示为：

<article>
$$
\tag{3} \hat{\mathbf{y}} = \mathbf{Xw} + b \in \mathbb{R}^m
$$
</article>

展开公式 (3)，即：

<article>
$$
\begin{cases}
\tag{4} \hat{\mathbf{y}} = (\hat{y}_1, \hat{y}_2, \cdots, \hat{y}_m)^T\\
\hat{y}_i = \mathbf{x}_i^T \mathbf{w} + b = \langle \mathbf{x}_i, \mathbf{w} \rangle + b,&i \in \{1, \cdots, m\}
\end{cases}
$$
</article>

## 损失函数


已知样本 $\(\mathbf{x}_i, y_i\) _{i=1}^{m}$，且 $\mathbf{x}_i$ 的预测值为 $\hat{y_i}$，则定义可单个样本是损失函数：

<article>
$$
\tag{5}
l^{(i)}(\mathbf{w}, b) = \frac 1 2 (\hat{y}_i - y_i)^2, i \in \{1, \cdots, m\}
$$
</article>

总损失函数定义为：

<article>
$$
\tag{6}
L(\mathbf{w}, b) = {\frac 1 m} \sum_{i=1}^m l^{(i)}(\mathbf{w}, b) = {\frac 1 {2m}} \lVert \mathbf{Xw} + b - \mathbf{y} \rVert ^2
$$
</article>

在训练模型时，我们希望寻找一组参数 $\(\mathbf{w}^*, b^*\)$，这组参数能最小化在所有训练样本上的总损失。如下式：

<article>
$$
\tag{7}
\mathbf{w}^{\ast}, b^{\ast} = \argmin_{\mathbf{w}^{\ast}, b^{\ast}} L(\mathbf{w}, b)
$$
</article>

可以求得解析解：

将 $\mathbf{w}$ 与 $b$ 合并为 $\overline{\mathbf{w}}$，$\overline{\mathbf{X}} = \(\mathbf{X}, \mathbf{1}\)$，则公式 (6)，可以写作：

<article>
$$
\tag{8}
L(\mathbf{w}, b) = {\frac 1 {2m}} \lVert \overline{\mathbf{X}} \overline{\mathbf{w}} - \mathbf{y} \rVert ^2
$$
</article>

这很容易求得解析解：

<article>
$$
\tag{9}
\overline{\mathbf{w}}^{\ast} = (\overline{\mathbf{X}}^T \overline{\mathbf{X}})^{-1} \overline{\mathbf{X}}^T \mathbf{y}
$$
</article>

对于实际问题，往往模型很复杂很难求得解析解，大都仅仅求得其近似解。

## 梯度下降

由计算梯度得：

<article>
$$
\tag{10}
\nabla_{\overline{\mathbf{w}}} L = {\cfrac 1 m} \overline{\mathbf{X}}^T (\overline{\mathbf{X}} \overline{\mathbf{w}} - y)
$$
</article>

所以，参数更新：

<article>
$$
\tag{11}
\begin{cases}
\mathbf{w} \leftarrow \mathbf{w} - {\cfrac \eta m} \mathbf{X}^T (\mathbf{Xw} + b - \mathbf{y}) \\
b \leftarrow b - {\frac \eta m} \mathbf{1}^T (\mathbf{Xw} + b - \mathbf{y})
\end{cases}
$$
</article>

其中 $\eta$ 表示学习率。
