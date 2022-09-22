# Python常微分方程求解器

## 目标

- 使用python构建一个简单的ODE求解器

## 常微分方程

### 1. 标准形式

一阶常微分方程的标准形式，如$(1)$：
$$
y^{'}=f(x,y) \qquad (1)
$$
如果考虑时间为自变量，则可以写成：
$$
u^{'}(t) = f(u(t),t) \qquad (2)
$$
在构建求解器时，**求解的对象是$u(t)$，即一个关于$t$的函数**。

但是，函数无法用代码来表示。于是，可以理解为求解得到的具体形式为一个数组。

例如，假设$t$的形式为$(0,1,2,...,n-1)$一共n个的离散的时间点，那么，求解出来对应的$u(t)$的形式也是一个n维的数组，其形式为$(5.3, 5.4, 5.5,...8.3)$。从可视化的角度来看，**这两个数组共同构成了一个二维坐标的x轴和y轴**，也就可以用这条离散的曲线来表达求解的对象$u(t$)。

对于二阶，可以写成：
$$
y^{''}=f(x,y,y^{'}), \quad y(x_0)=y_0, \quad y^{'}(x_0)=u_0
$$
用中间变量来表示：
$$
\begin{cases}
u=y^{'}, \quad u(x_0)=u_0\\
u^{'}=f(x,y,u), \quad y(x_0)=y_0
\end{cases}
$$


### 2. 数值方法

- 显式，欧拉前向 Forward euler method
- 隐式欧拉
- 龙格库塔 Runge-Kutta
  - 龙格库塔四阶 RK4
- 多步法 Multistep
  - Adams Bashforth Moulton


### 3. 一阶常微分方程例子

#### 3.1 欧拉前向方法Python

针对一阶常微分方程的标准表达形式，将微分方程改写：
$$
y^{'}=f(x,y)\quad \rightarrow \quad \frac{dy}{dx} = f(x, y) \qquad (3)
$$
根据$(3)$，欧拉前向方法的**更新**可以表示为：
$$
\frac{y_{n+1} - y_n}{h}\approx f(x_n,y_n) \quad \rightarrow \quad y_{n+1} \approx y_n + f(x_n, y_n)h \qquad (4)
$$
还需要一个初始条件：
$$
y_0 = Y_0 \qquad (5)
$$

> **求解微分方程就是，已知一个方程$f$微分后的模样，求解这个方程$f$本身。**

根据公式$(4)$和(5)，求解器的输入需要：

- 写出求导后的方程$f$，即$y^{'}$，其它写法可以是定义$u^{'}(t)$
- 方程的初始值$y_0$
- 步长$h$，其它写法可以是，$\Delta t$ / 或者次数$n$
- 自变量区间，其它写法可以是$T$ / 或者次数$n$

Python代码（仅展示功能）

```python
def forward_euler(f, U0, T, n):
    """ Solve u' = f(u, t), where u(0) = U0 with n steps until T """
    t = np.zeros(n + 1) # 自变量array
    u = np.zeros(n + 1) # 方程的值array

    u[0] = U0 # 方程初始值
    t[0] = 0 # 自变量初始值
    dt = T/n # 步长可以算出来，从区间T，除以更新次数n。dt, T, n 三者取其二作为输入即可。
    
    # n 为更新次数
    # 每次更新两个值：自变量和方程
    for k in range(n):
        t[k + 1] = t[k] + dt # 根据t0更新t1,以此类推
        u[k + 1] = u[k] + dt * f(u[k], t[k]) # 根据u0，更新u1，以此类推
        
    # 返回u和t的array，可以进一步可视化等等
    return u, t
  
// 定义f
def f(t):
  return u + t
```



### 3.2 龙格库塔4阶 RK4

根据$(3)$，RK4方法的**更新**可以表示为：
$$
y_{n+1}=y_n + \displaystyle\frac{h}{6}(k_1 + 2k_2 + 2k_3 + k4) \qquad (6)
\\
\\
\begin{cases}
k_1 = f(y_n, t_n)
\\
k_2=f(y_n + h\frac{k_1}{2}, t_n+\frac{h}{2})
\\
k_3=f(y_n + h\frac{k_2}{2}, t_n+\frac{h}{2})
\\
k_4=f(y_n+hk_3, t_n + h)
\end{cases}
$$

## 4. 二阶常微分方程例子

求解ODE系统：[Solving ODEs in Python 3: System of Equations](https://www.youtube.com/watch?v=ycxg3snaJZE)

Rust