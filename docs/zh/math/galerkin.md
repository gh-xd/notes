# 伽辽金法

## 目标

认识伽辽金法，了解伽辽金法的基本步骤

## 1. 为什么要用伽辽金法

> 数值求解PDE模型，越快解决越好

## 2. 伽辽金方法直观步骤

1. 给出一个满足边界条件的表达式，表达式是一系列基函数的线性累加（线性插值），基函数满足边界条件
2. 给定的微分方程和每一个在计算区域上加权积分为零（泛函中最小势能原理？），得到一系列加权值的方程
3. 获得线性系统方程组$Ax=b$，求$x$，即求加权值。他们和基函数的点积就是在当前函数表达下的最贴近真实解的位移表达式

## 3. 伽辽金法一维例子 

*关于垂直悬臂梁端点受力的问题，存在解析解，可以显式求解。*

1. 给出一个满足边界条件的表达式，表达式是一系列基函数的线性累加（线性插值），基函数满足边界条件，例如：

   - 设基函数为$\phi_i,\ i=1,2,3$
   - 其中：$\phi_0=x,\ \phi_1=x,\ \phi_2=x^2$

   - 那么位移函数可以表达为线性累加$u(x)=\displaystyle\sum_{j=1}^3 a_j\phi_j$

   - 于是，$u(x)=a_0x+a_1x^2+a_2x^3$

2. 给定的微分方程和每一个在计算区域上加权积分为零（泛函中最小势能原理？），得到一系列加权值的方程

   - 对于物理方程$E\displaystyle\frac{du(x)}{dx}-\frac{f(x)}{A}=0$，第1步中的$u(x)$不一定就是真实函数
   - 所以，需要要求上述函数与各基函数的内积为 0，内积可以理解为在同一个区域$\Omega$的两个函数$f$和$g$的积分：$\displaystyle\int_{\Omega}fgd\Omega=0$
   - 具体得到：$\displaystyle\int_0^L(E\displaystyle\frac{du(x)}{dx}-\frac{F}{A})\phi_jdx=0$
   - 移向得到：$\displaystyle\int_0^L\phi_j du(x)=\frac{1}{EA}\int_0^LF\phi_jdx$
   - 因为$u(x)=a_0x+a_1x^2+a_2x^3$，带入可得$\displaystyle\sum_{k=1}^3a_k\int\phi_jd\phi_k=\frac{1}{EA}\int_0^LF\phi_jdx$

3. 获得线性系统方程组$Ax=b$，求$x$，即求加权值。他们和基函数的点积就是在当前函数表达下的最贴近真实解的位移表达式

   - 根据第2步，可以得到$\displaystyle\begin{bmatrix}\displaystyle\int_0^L\phi_1d\phi_1\quad \int_0^L\phi_1d\phi_2\quad\int_0^L\phi_1d\phi_3\\\displaystyle\int_0^L\phi_2d\phi_1\quad \int_0^L\phi_2d\phi_2\quad\int_0^L\phi_2d\phi_3\\\displaystyle\int_0^L\phi_3d\phi_1\quad \int_0^L\phi_3d\phi_2\quad\int_0^L\phi_3d\phi_3\end{bmatrix}\begin{bmatrix}a_1\\a_2\\a_3\end{bmatrix}=\displaystyle\frac{1}{EA}\begin{bmatrix}\displaystyle\int_0^LF\phi_1dx\\\displaystyle\int_0^LF\phi_2dx\\\displaystyle\int_0^LF\phi_3dx\end{bmatrix}$
   - 求解得到$a_1,a_2,a_3$，获得$u(x)$的表达式

## 4. 伽辽金法二维的例子

*关于薄板挠度和受力的例子，无法显示求解。*

基本步骤和上述类似，但是基函数的选取不同。

为了满足二维边界条件，选取三角函数作为基函数：

$\left\{\begin{aligned}\phi_1(x,y)&=(1+cos\displaystyle\frac{\pi x}{a})(1+cos\displaystyle\frac{\pi y}{b})\\\phi_2(x,y)&=(1+cos\displaystyle\frac{3\pi x}{a})(1+cos\displaystyle\frac{\pi y}{b})\\\phi_3(x,y)&=(1+cos\displaystyle\frac{3\pi x}{a})(1+cos\displaystyle\frac{3\pi y}{b})\end{aligned}\right.$

以及在写出线性系统方程组时，需要简写：

记：$\displaystyle\int_{-a}^b\int_{-a}^bf(x,y)dxdy=H(f)$

则线性方程组为：

$\displaystyle\begin{bmatrix}H(\phi_1\nabla^4\phi_1)\quad H(\phi_1\nabla^4\phi_2)\quad H(\phi_1\nabla^4\phi_3)\\H(\phi_2\nabla^4\phi_1)\quad H(\phi_2\nabla^4\phi_2)\quad H(\phi_2\nabla^4\phi_3)\\H(\phi_3\nabla^4\phi_1)\quad H(\phi_3\nabla^4\phi_2)\quad H(\phi_3\nabla^4\phi_3)\end{bmatrix}\begin{bmatrix}a_1\\a_2\\a_3\end{bmatrix}=\displaystyle\frac{q_0}{D}\begin{bmatrix}H(\phi_2)\\H(\phi_1)\\H(\phi_3)\end{bmatrix}$

然而，这样可能会带来的问题：

以上例子，都是在整个计算域上用一个函数表达式来进行描述，固然很方便。但带来的问题会有精度上的不足，而且解得最终效果也很依赖于基函数的选取。

解决方案：

将整个区域分成小块，在小块上进行一些基函数近似(FEM?)

*参考*

- [伽辽金法学习（一）](https://zhuanlan.zhihu.com/p/543086700)

## 5. 线段上使用伽辽金法

### 5.1 一小个区间的线性方程组

假设有一个待求解的方程$Au(x)=f$，取一个区间$[x_k,x_{k+1}]$，则步长为$h_k=x_{k+1}-x_k$

假设在$x_k$点的解$u(x_k)=u_k$，在$x_{k+1}$点的解$u(x_{k+1})=u_{k+1}$

则伽辽金法在局部区间上的的线性插值函数为：$\tilde u(x)=\displaystyle\frac{x_{k+1}-x}{h_k}u_k+\frac{x-x_k}{h_k}u_{k+1}$

它满足边界条件：$\left\{\begin{aligned}&\tilde u(x_k)=u_k\\&\tilde u(x_{k+1})=u_{k+1}\end{aligned}\right.$

基函数可以记为：$\left\{\begin{aligned}&\phi_0^k=\displaystyle\frac{x_{k+1}-x}{h_k}\\&\phi_1^k=\frac{x-x_k}{h_k}\end{aligned}\right.$，$k$在此处的意思是，某一个区间任意取的第$k$段

线性插值函数则可以表达为：$\tilde u(x)=\phi_0^ku_k+\phi_1^ku_{k+1},\ x_k<x<x_{k+1}$

注意，这里线性插值函数表达式和之前的例子类似：$\tilde u(x)=a_0x+a_1x^2+a_2x^3$

它们都是线性形式：$\tilde u(x)=\displaystyle\sum a_j\phi_j$，只不过这个例子中，不再对全局进行近似，而是对局部的$k$段进行近似，于是基函数就有了上标$k$

然后对$[x_k,x_{k+1}]$这个小区间使用Galerkin方法的分析（最小势能原理？）：

$\displaystyle\int_{x_k}^{x_{k+1}}(A\tilde u(x)-f)\phi_j^kdx=0,\ j=0,1$

同样，记 $H_k(f)=\displaystyle\int_{x_k}^{x_{k+1}}f(x)dx$

可以得到针对一小个区间的线性方程组的形式：

$\begin{bmatrix}H_k(\phi_0^kA\phi_0^k)\quad H_k(\phi_0^kA\phi_1^k)\\H_k(\phi_1^kA\phi_0^k)\quad H_k(\phi_1^kA\phi_1^k)\end{bmatrix}\begin{bmatrix}u_k\\u_{k+1}\end{bmatrix}=\begin{bmatrix}H_k(f\phi_0^k)\\H_k(f\phi_1^k)\end{bmatrix}$

但是这还不够，需要对全局的矩阵进行组装

### 5.2 对全局进行矩阵的组装

首先，上述线性方程组可以写成如下形式：$A_{ij}^ku_k=b_i^k$，其中：

$\left\{\begin{aligned}&A_{ij}^k=H_k(\phi_i^kA\phi_j^k),\quad i,j\in {0,1}\\&b_i^k=H_k(f\phi_i^k),\quad i=0,1\end{aligned}\right.$，$u_k$则是方程第$k$段的解，需要求解

其中$i,j,k$的含义是：

- $k$是第几个单元
- 根据基函数的定义，$A_{ij}^k$表示节点$j$对节点$i$处的贡献
- $b_i^k$表示节点$i$处所获得的贡献值

假设，我们把全局分成了3段，则有4个节点，即$k=0,1,2,3$

$|_{u_0}\_\_\_Unit\_0\_\_\_|_{u_1}\_\_\_Unit\_1\_\_\_|_{u_2}\_\_\_Unit\_2\_\_\_|_{u_3}$

拿$u_1$这个点来举例，因为它横跨了$Unit0$和$Unit1$

1. 在$Unit0$上
   - 节点$u_0$对$u_0$自己的贡献为$A_{00}^0u_0$
   - 节点$u_0$对节点$u_1$有贡献$A_{10}^0u_0$
   - 节点$u_1$对$u_1$自己的贡献为$A_{11}^0u_1$
   - $b$则是该节点上的总贡献值
   - $\begin{bmatrix}A_{00}^0\quad A_{01}^0\\A_{10}^0\quad A_{11}^0\end{bmatrix}\begin{bmatrix}u_0\\u_1\end{bmatrix}=\begin{bmatrix}b_0^0\\b_1^0\end{bmatrix}$
2. 在$Unit1$上
   - $u_1$对$u_1$自己的贡献为$A_{00}^1u_1$
   - 节点$u_2$对节点$u_1$有贡献$A_{01}^1u_2$
   - $\begin{bmatrix}A_{00}^1\quad A_{01}^1\\A_{10}^1\quad A_{11}^1\end{bmatrix}\begin{bmatrix}u_1\\u_2\end{bmatrix}=\begin{bmatrix}b_0^1\\b_1^1\end{bmatrix}$
3. 在$Unit2$上
   - 同上
   - $\begin{bmatrix}A_{00}^2\quad A_{01}^2\\A_{10}^2\quad A_{11}^2\end{bmatrix}\begin{bmatrix}u_2\\u_3\end{bmatrix}=\begin{bmatrix}b_0^2\\b_1^2\end{bmatrix}$

关于节点$u_k$的矩阵可以总结为：

$\begin{bmatrix}&u_k点对u_k点自己的贡献\quad u_{k+1}点对u_k点的贡献\\&u_k点对u_{k+1}点的贡献\quad u_{k+1}点对u_{k+1}点自己的贡献\end{bmatrix}\begin{bmatrix}u_k\\u_{k+1}\end{bmatrix}=\begin{bmatrix}u_k处的总贡献值\\u_{k+1}处的总贡献值\end{bmatrix}$

将上述3个矩阵根据$u_k$进行拼接，组装起来可以得到线性方程组最终的形式$Ax=b$：

$\begin{bmatrix}&A_{00}^0\quad &A_{01}^0\quad &0 \quad &0\\&A_{10}^0\quad &A_{11}^0+A_00^1 \quad &A_{01}^1 \quad &0\\&0 \quad &A_{10}^1 \quad &A_{11}^1+A_{00}^2 \quad &0\\&0 \quad &0 \quad &A_{10}^2 \quad &A_{11}^2\end{bmatrix}\begin{bmatrix}u_0\\u_1\\u_2\\u_3\end{bmatrix}=\begin{bmatrix}b_0^0\\b_1^0 + b_0^1\\b_0^2+b_1^1\\b_1^2\end{bmatrix}$

解线性方程组，求得的$u_0,u_1,u_2,u_3$就是之前假设在$u(x_0),u(x_1),u(x_2),u(x_3)$上的解。

以上为局部使用Galerkin方法获取弱解形式的全部流程，因为把整个区间划分成了有限个区域，所以也叫有限元方法。

### 5.3 积分换元

令 $\xi=\displaystyle\frac{x-x_k}{h_k}$，$h_kd\xi=dx$

则 $\left\{\begin{aligned}&\phi_0^k=1-\xi\\&\phi_1^k=\xi\end{aligned}\right.$

那么，$H_k(\phi_i^kA\phi_j^k)=\displaystyle\int_{x_k}^{x_{k+1}}\phi_i^kA\phi_j^kdx=h_k\int_0^1\phi_i^kA\phi_j^kd\xi$

并且，$H_k(f\phi_j^k)=\displaystyle\int_{x_k}^{x_{k+1}}f(x)\phi_j^kdx=h_k\int_0^1f(h_k\xi+x_k)\phi_j^kd\xi$

当微分算子$A$已知的时候，$H_k(\phi_i^kA\phi_j^k)$都可以显式地写出来，但是，由于$f(x)$往往没有能够直接写出来的显式积分表达式，但是在$h_k$较小的情况下，可以用Simpson公式/简单的梯形公式进行近似：

$H_k(f\phi_0^k)=\displaystyle\frac{h_k}{6}\displaystyle[f(x)+2f(x_k+\frac{h_k}{2})]$

$H_k(f\phi_1^k)=\displaystyle\frac{h_k}{6}\displaystyle[2f(x_k+\frac{h_k}{2})+f(x_{k+1})]$

### 5.4 求解微分方程的例子

求解例子：$\displaystyle\frac{du}{dx}=x^3e^x$

可以得到微分算子$A=\displaystyle\frac{d}{dx}=\frac{1}{h_k}\frac{d}{d\xi}$，将微分算子代入公式可得：

$\left\{\begin{aligned}&H_k(\phi_0^kA\phi_0^k)=h_k\displaystyle\int_0^1(1-\xi)(\frac{1}{h_k}\frac{d(1-\xi)}{d\xi})d\xi=-\int_0^1(1-\xi)d\xi=-\frac{1}{2}\\&H_k(\phi_0^kA\phi_1^k)=h_k\displaystyle\int_0^1(1-\xi)(\frac{1}{h_k}\frac{d\xi}{d\xi})d\xi=\int_0^1(1-\xi)d\xi=\frac{1}{2}\\&H_k(\phi_1^kA\phi_0^k)=h_k\displaystyle\int_0^1\xi(\frac{1}{h_k}\frac{d(1-\xi)}{d\xi})d\xi=-\int_0^1\xi d\xi=-\frac{1}{2}\\&H_k(\phi_1^kA\phi_1^k)=h_k\displaystyle\int_0^1\xi(\frac{1}{h_k}\frac{d\xi}{d\xi})d\xi=\int_0^1\xi d\xi=\frac{1}{2}\end{aligned}\right.$

由此可得$Ax=b$为：

$\begin{bmatrix}&\displaystyle-\frac{1}{2}&\displaystyle\frac{1}{2}&0&0&\cdots&0\\&\displaystyle-\frac{1}{2}&0&\displaystyle\frac{1}{2}&0&\cdots &0\\&0&\displaystyle-\frac{1}{2}&0&\displaystyle\frac{1}{2}&\cdots &\vdots\\&\vdots&\vdots&\vdots&\ddots&\cdots&0\\&0&0&0&0&\displaystyle-\frac{1}{2}&\displaystyle\frac{1}{2}\end{bmatrix}\begin{bmatrix}u_0\\u_1\\u_2\\\vdots\\u_{k-1}\\u_k\end{bmatrix}=\begin{bmatrix}\displaystyle\frac{6}{h_0}[f(x_0)+2f(x_0+\displaystyle\frac{h_0}{2})\\\displaystyle\frac{6}{h_0}[2f(x_0+\displaystyle\frac{h_0}{2})+f(x_1)]+\displaystyle\frac{6}{h_1}[f(x_1)+2f(x_1+\displaystyle\frac{h_1}{2})\\\vdots\\\displaystyle\frac{6}{h_{k-1}}[2f(x_{k-1}+\displaystyle\frac{h_{k-1}}{2})+f(x_{k-1})]+\displaystyle\frac{6}{h_{k}}[f(x_{k})+2f(x_{k}+\displaystyle\frac{h_{k}}{2})\\?\end{bmatrix}$

*Tips*

> 经过他人的努力发现，对于$u^{'}=f$的方程，用有限元进行划分，本质上和有限差分法（FDM）求解的过程是类似的。虽然Galerkin方法在每个单元的操作有所不同，但最终和FDM一样，都是在节点处用差商代替微元，在边界处开始一步步递进运算。

*参考*

- [伽辽金法学习（二）](https://zhuanlan.zhihu.com/p/544599506)



### 4. 更好的线性插值

划分单元的伽辽金法中，划分出来各段的最终函数值都是一条直线，一条由单元两端函数控制的直线。

但是单元内函数呈线性变化，且单元间两条直线斜率不一样会产生“折角”，不够光滑。

可否使插值函数变得光滑（若干介导数值相同）？

答案：样条函数，样条插值方法。

> 三次样条，即三弯矩方程，是使用得最广泛的。因为其能保证二阶导数值相同，从而平滑。



## 问题

- 空间上的
- 代码

## 参考资料

- 伽辽金法学习（一）
- 伽辽金法学习（二）