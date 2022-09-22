# 弹簧质点系统

## 目标

- 理解弹簧质点系统

## 1. 弹簧质点系统中的受力

重力 - $f_i^{gravity}=-m_ig$

弹簧力 - $f_{ij}^{spring}=-k(\lVert x_i-x_j\lVert_2-l_{ij})d = -Y(\frac{||x_i-x_j||_2}{l_{ij}}-1)d \qquad Hooke's Law$

阻尼 - $f_i^{drag}=-\alpha v_i$

减震器 - $f_{ij}^{dashpot}=-[\beta (v_i-v_j)\cdot \textbf{d}]\textbf{d}$



$k$ -弹簧刚度 stiffness

$Y$ - 杨氏模量

$l_{ij}$ - 弹簧静止长度

$\textbf{d}=\widehat{(x_i-x_j)}$ - 从i到j的方向向量的单位向量

## 2. 显式积分

### 2.1 显式积分公式

$$
\begin{align*}
& v_{t+1}=v_t+\Delta t\frac{f_t}{m}
\\
& x_{t+1}=x_t+\Delta t v_t
\end{align*}
$$

意思是：**下一时刻的速度，取决于现在的速度与加速度；下一刻的位置，取决于现在的位置和现在的速度**。

**显式积分方法**：

- Forward Euler

- Sympletic Euler

- RK4

容易实施，容易爆炸，不适合刚度很大的材料

爆炸的意思是，计算误差无法随着时间衰减，而是随着时间爆炸性增长

## 3. 隐式积分

### 3.1 隐式积分公式

$$
\begin{align*}
& x_{t+1}=x_t+\Delta tv_{t+1}\quad (1)
\\
\\
& v_{t+1}=v_t+\Delta tM^{-1}f(x_{t+1})\quad (2)
\end{align*}
$$

隐式时间积分

- $(1)$下一个时间的位置取决于下一个时间的速度

- $(2)$下一个时间的速度取决于下一个时间的位置（力）

**隐式积分方法**：

- Backward Euler
- Middle-point

> 提示：如果解**线性系统方程组**，那么计算会更昂贵，求解复杂，无法做向量化

### 3.2 后向欧拉

通过$(1)$和$(2)$得到：
$$
v_{t+1}=v_t+\Delta t M^{-1}f(x_t+\Delta tv_{t+1}) \quad (3)
$$
线性化：由于公式$(3)$是一个非线性系统，对f()进行一届泰勒展开（牛顿方法的一个步骤）

由于$x_t$和$v_t$都已知，需要求解的是$v_{t+1}$，所以得到一个线性系统$(5)$

$\begin{aligned}& v_{t+1}=v_t+\Delta t M^{-1}[f(x_t)+\frac{\partial f}{\partial x}(x_t)\Delta t v_{t+1}] \quad (4)\\ & [I-\Delta t^2 M^{-1	}\frac{\partial f}{\partial x}(x_t)]v_{t+1} = v_t+\Delta t M^{-1}f(x_t) \quad (5)
\end{aligned}$
$$
\begin{aligned}

& v_{t+1}=v_t+\Delta t M^{-1}[f(x_t)+\frac{\partial f}{\partial x}(x_t)\Delta t v_{t+1}] \quad (4)
\\
& [I-\Delta t^2 M^{-1	}\frac{\partial f}{\partial x}(x_t)]v_{t+1} = v_t+\Delta t M^{-1}f(x_t) \quad (5)
\end{aligned}
$$

## 4. 积分器通用公式

对公式$(5)$ 做一点小改动，加入$\beta$ 得到$(6)$

$[I-\beta\Delta t^2 M^{-1	}\frac{\partial f}{\partial x}(x_t)]v_{t+1} = v_t+\Delta t M^{-1}f(x_t) \quad (6)$
$$
[I-\beta\Delta t^2 M^{-1	}\frac{\partial f}{\partial x}(x_t)]v_{t+1} = v_t+\Delta t M^{-1}f(x_t) \quad (6)
$$
不同的$\beta$可以实现不同的积分器：

- $\beta=0$ 时，$v_{t+1}=v_t+\Delta tM^{-1}f(x_t)$，属于前向欧拉显式积分
- $\beta=1/2$ 时，为middle point方法，精度比隐式积分可能好些 
- $\beta=1$ 时，属于后向欧拉法

## 5. 复杂弹簧系统

如果有成千上万个弹簧：

- 稀疏矩阵
- 共轭梯度
- Preconditioning
- [Position-based dynamics (PBD)](...)

或者使用Fast mass-spring system solver [Fast simulation of mass-spring system](...)



## 6. DEMO案例

Taichi

1. Open a window
2. Free falling and fixed particles
3. Springs
4. Wall collision
5. Attractor
6. Drag damping
7. Dashpot damping