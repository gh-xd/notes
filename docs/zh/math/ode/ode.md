# 常微分方程

## 目标

- 了解常用的常微分方程求解过程

## 1. 显式欧拉方法

### 1.1 前向欧拉法

$u_{n+1}=u_n+h*f(x_n,u_n) \qquad (1)$

属于单步线性法。线性方程，直接迭代

### 1.2 改进欧拉法

欧拉法用梯形公式表达$(1)$

$u_{n+1}=u_n+\displaystyle\frac{h}{2}(f(x_n,u_n)+f(x_{n+1},u_{n+1}))\qquad (2)$ 

精度高，但是要解非线性方程

属于单步法

迭代法解$(2)$：

$u^{k+1}_{n+1}=u_n+\displaystyle\frac{h}{2}(f(x_n,u_n)+f(x_{n+1},u^{k}_{n+1}))\qquad k=0,1,2,...$

### 1.3 龙格库塔RK4

显式更新用到的公式为：

$y_{n+1}=y_n + \displaystyle\frac{h}{6}(k_1 + 2k_2 + 2k_3 + k4)$

其中：

$\begin{cases}
k_1 = f(y_n, t_n)
\\
k_2=f(y_n + h\displaystyle\frac{k_1}{2}, t_n+\frac{h}{2})
\\
k_3=f(y_n + h\displaystyle\frac{k_2}{2}, t_n+\frac{h}{2})
\\
k_4=f(y_n+hk_3, t_n + h)
\end{cases}$

## 2. 隐式欧拉方法



## x. 线性差分方程