# 微分算子

## 目标

了解什么是微分算子。

## 1. 梯度

Gradient $\nabla:\mathbb{R}^1\rightarrow \mathbb{R}^3$

对一个标量进行梯度计算，从一维到三维

$grad\ s=\nabla s = \left[\displaystyle\frac{\partial s}{\partial x}, \displaystyle\frac{\partial s}{\partial y}, \displaystyle\frac{\partial s}{\partial z}\right]^T$

例如一个速度，速度梯度？

## 2. 散度

Divergence $\nabla\cdot:\mathbb{R}^3\rightarrow \mathbb{R}^1$  

对一个三维向量进行散度计算，从三维到一个一维的标量

$div\ v = \nabla \cdot v = \displaystyle\frac{\partial v_x}{\partial x}+\displaystyle\frac{\partial v_y}{\partial y}+\displaystyle\frac{\partial v_z}{\partial z}$

例如一个不可压缩流体，在流出的时候，如果x方向的导数过大（x方向流出过多），那么y方向的导数相应要很小（y方向流出为负）。这样可以描述流体在流动过程中总的**体积**不变。

速度的散度点乘密度的话，$\rho(\nabla \cdot v)$，等于描述了流体在流动过程中总的**质量**不变。

## 3. 旋度

Curl $\nabla \times:\mathbb{R}^3\rightarrow\mathbb{R}^3$

$curl\ v=\nabla \times v =\left[\displaystyle\frac{\partial v_z}{\partial y}-\displaystyle\frac{\partial v_y}{\partial z}, \displaystyle\frac{\partial v_x}{\partial x}-\displaystyle\frac{\partial v_x}{\partial z}, \displaystyle\frac{\partial v_y}{\partial x}-\displaystyle\frac{\partial v_x}{\partial y}\right]^T$

## 4. 拉普拉斯 算子

$Laplace\ \Delta=\nabla^2=\nabla \cdot \nabla:\mathbb{R}^n\rightarrow\mathbb{R}^n$

$laplace\ s=div(grad\ s)=\nabla^2s=\displaystyle\frac{\partial^2 s}{\partial x^2}+\displaystyle\frac{\partial^2 s}{\partial y^2}+\displaystyle\frac{\partial^2 s}{\partial z^2}$

例如扩散

## 问题

- 几个学科中的应用