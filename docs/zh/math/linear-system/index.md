# 线性系统求解

## 目标

了解求解$Ax=b$的方法一览

## 1. 直接求解 

- 求逆: $x=A^{-1}b$
  - **维度比如< 100还行**
  - 复杂度$O(n^3)$，矩阵太大不行
- 因子分解: $A=\begin{pmatrix}&LU, A\ is \ square\ matrix\\&LDL^T,if\ A=A^T\\&LL^T,if\ A=A^T\ and\ A>0\end{pmatrix}$
  - 如果A为方阵，则可以分解为上三角和下三角
  - 如果A为对称阵，可以对角阵分解
  - 如果A为对称阵且正定，可以分解为三角形矩阵和它转秩的矩阵
    - 求解$LL^T x=b$的过程分为两部，第一步是令$y=L^T x$，求$Ly=b$中的$y$，第二步是求$L^T x=y$中的$x$
    - **注意**：求解对角阵的线性系统时，求解过程是前后依赖的，无法使用GPU（或者在使用GPU的程序中运行）
    - **使用**：在建立某个程序的原型时可以使用
  - 时间复杂度
    - 如果A是稠密矩阵，分解的时间复杂度也是$O(n^3)$
  - 所以，当A为稀疏矩阵的时候，该方法比求逆要好一些
    - **维度撑死不能超过100万**

## 2. 迭代求解

- 基于不动点的迭代线性求解器： Stationary iterative linear solvers (求解的过程是平滑的方式)

  - 雅可比迭代 Jacobi 
    - 高斯塞德尔迭代 Gauss-Seidel iterations 
      - 容易实施

  - SOR

  - Multigrid

- Krylov subspace方法

- 共轭梯度 Conjugate gradient (CG) 

  - 当矩阵正定且对称时：$A=A^T$并且$A>0$

  - 保证$n$次迭代一定能收敛

  - 求解效率依赖于一个kappa值：$\kappa=\displaystyle\frac{\lambda_{max}}{\lambda_{min}}$

    - \lambda$是$A$矩阵的特征值，即最大特征值和最小特征值的比值

    - 如果$\kappa$很小，例如等于1的时候，共轭梯度非常非常非常好用

    - 只需要二十行代码就可以搞定:dog:

- biCG

- CR

- MinRes

- GMRes

## 3. 总结

当求解的规模相当大，例如$1e^6$或者更多，比如求解1亿个粒子时，只能使用迭代求解器

## 参考

加