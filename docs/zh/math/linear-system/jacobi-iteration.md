# 雅可比迭代





## 1. 直接使用A矩阵求解

### 1.1 表示形式

对于矩阵$Ax=b$的第一行，可以写成如下形式：

$a_{11}x_1+a_{12}x_2+\cdots+a_{1n}x_n=b_1$

同理，$Ax=b$通过移向可以表示成如下形式：

$\left\{\begin{aligned}x_1=&\displaystyle\frac{1}{a_{11}}(b_1-a_{12}x_2-\cdots -a_{1n}x_n)\\x_2=&\displaystyle\frac{1}{a_{22}}(b_2-a_{21}x_2-\cdots -a_{2n}x_n)\\&\vdots\\x_n=&\displaystyle\frac{1}{a_{nn}}(b_n-a_{n1}x_2-\cdots -a_{n,n-1}x_{n-1})\end{aligned}\right.$

### 1.2 迭代过程

假设第0步$x$的初始向量：

$x^{(0)}=(x_1^{(0)},x_2^{(0)},\cdots,x_n^{(0)})$

代入上式右边，可以得到迭代后第1步$x$的向量$x^{(1)}$：

$\left\{\begin{aligned}x_1^{(1)}=&\displaystyle\frac{1}{a_{11}}(b_1-a_{12}x_2^{(0)}-\cdots -a_{1n}x_n^{(0)})\\x_2^{(1)}=&\displaystyle\frac{1}{a_{22}}(b_2-a_{21}x_2^{(0)}-\cdots -a_{2n}x_n^{(0)})\\&\vdots\\x_n^{(1)}=&\displaystyle\frac{1}{a_{nn}}(b_n-a_{n1}x_2^{(0)}-\cdots -a_{n,n-1}x_{n-1}^{(0)})\end{aligned}\right.$

同理，迭代k次以后，可以得到$x^{(k+1)}$

$\left\{\begin{aligned}x_1^{(k+1)}=&\displaystyle\frac{1}{a_{11}}(b_1-a_{12}x_2^{(k)}-\cdots -a_{1n}x_n^{(k)})\\x_2^{(k+1)}=&\displaystyle\frac{1}{a_{22}}(b_2-a_{21}x_2^{(k)}-\cdots -a_{2n}x_n^{(k)})\\&\vdots\\x_n^{(k+1)}=&\displaystyle\frac{k+1}{a_{nn}}(b_n-a_{n1}x_2^{(k)}-\cdots -a_{n,n-1}x_{n-1}^{(k)})\end{aligned}\right.$

### 1.3 迭代公式

根据上述迭代过程，雅可比迭代的计算公式为：

$x_i^{k+1}=\displaystyle\frac{1}{a_{ii}}[b_i-\sum_{j=1,j\neq i}^n a_{ij}x_j^{(k)}]$

其中，$i=1,2,...n$代表矩阵维度，$k=0,1,2...$代表迭代次数



## 2. 分解A矩阵求解

### 2.1 表示形式

将$Ax=b$中的$A$矩阵分解为：

- 对角阵$D=\begin{bmatrix}&a_{11}&0&\cdots&0\\&0&a_{22}&\cdots&0\\&\vdots&\vdots&\ddots&\vdots\\&0&0&\cdots&a_{nn}\end{bmatrix}$
- 下三角矩阵$L=D=\begin{bmatrix}&0&0&\cdots&0\\&a_{21}&0&\cdots&0\\&\vdots&\vdots&\ddots&\vdots\\&a_{n1}&a_{n2}&\cdots&0\end{bmatrix}$
- 上三角矩阵$U=D=\begin{bmatrix}&0&a_{12}&\cdots&a_{1n}\\&0&0&\cdots&a_{2n}\\&\vdots&\vdots&\ddots&\vdots\\&0&0&\cdots&0\end{bmatrix}$

可以得到

$A=D+L+U$

则线性方程组可以写成：

$Dx=b-(L+U)x$

### 2.2 迭代公式

$x^{(k+1)}=D^{-1}(b-(L+U)x^{(k)})$

## 3. 迭代停止

> 建设中...

## 学习资料

- [数值分析方法与MATLAB实现](https://zhuanlan.zhihu.com/p/30965284)
- [雅各比迭代Python实现](https://zhuanlan.zhihu.com/p/473872347)