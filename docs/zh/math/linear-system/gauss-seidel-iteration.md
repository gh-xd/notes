# 高斯-塞德尔迭代法

## 目标

- 理解高斯-塞德尔迭代法的过程

## 直接使用A矩阵求解

### 表示形式

和雅可比迭代一样

### 迭代过程

在雅可比迭代的基础上，高斯-塞德尔迭代在每一轮更新中，都会使用最新的$x_{i-1}$

> 高斯-塞德尔迭代不能做并行运算。因为它每个$x_i$的计算都需要使用前一项$x_{i-1}$的结果

假设第0步$x$的初始向量：

$x^{(0)}=(x_1^{(0)},x_2^{(0)},\cdots,x_n^{(0)})$

在计算完$x_1^{(0)}$之后，立马就代入到$x_2^{(1)}$的计算中去：

$\left\{\begin{aligned}x_1^{(1)}=&\displaystyle\frac{1}{a_{11}}(b_1-a_{12}x_2^{(0)}-\cdots -a_{1n}x_n^{(0)})\\x_2^{(1)}=&\displaystyle\frac{1}{a_{22}}(b_2-a_{21}x_1^{(1)}-\cdots -a_{2n}x_n^{(0)})\\&\vdots\\x_n^{(1)}=&\displaystyle\frac{1}{a_{nn}}(b_n-a_{n1}x_2^{(1)}-\cdots -a_{n,n-1}x_{n-1}^{(1)})\end{aligned}\right.$

同理，迭代k次以后，可以得到$x^{(k+1)}$

$\left\{\begin{aligned}x_1^{(k+1)}=&\displaystyle\frac{1}{a_{11}}(b_1-a_{12}x_2^{(k)}-\cdots -a_{1n}x_n^{(k)})\\x_2^{(k+1)}=&\displaystyle\frac{1}{a_{22}}(b_2-a_{21}x_1^{(k+1)}-\cdots -a_{2n}x_n^{(k)})\\&\vdots\\x_n^{(k+1)}=&\displaystyle\frac{1}{a_{nn}}(b_n-a_{n1}x_2^{(k+1)}-\cdots -a_{n,n-1}x_{n-1}^{(k+1)})\end{aligned}\right.$

## 迭代停止

> 建设中...

## 