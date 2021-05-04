---
layout: xint
title: 简单的梯度运算
date: 2021-05-04 16:44:49
tags:
---

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
\nabla_{\mathbf{x}} \mathbf{Ax} = \nabla_{\mathbf{x}} \langle  \mathbf{A}^T, \mathbf{x} \rangle = \mathbf{A}^T
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