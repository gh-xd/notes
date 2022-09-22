# 欧拉公式

## 目标

理解一下欧拉公式的基本概念。

## 1. 麦克劳林级数

从泰勒级数，到麦克劳林级数

级数=逼近

举一个四次方的例子$(1)$

$f(x)=Ax^4+Bx^3+Cx^2+Dx+E \qquad (1)$

$\rightarrow f(0)=E$

$\rightarrow f^{'}(x)=4Ax^3+3Bx^2+2Cx+D\quad \rightarrow f^{'}(0)=D$

同理：

$f^{''}(0)=2C$

$f^{'''}(0)=6B$

则从$(1)$得到：

$f(x)=\displaystyle\frac{f^{(4)}(0)}{(4\times3\times2\times1)}x^4+\frac{f^{(3)}(0)}{(3\times2\times1)}x^3+\frac{f^{(2)}(0)}{(2\times1)}x^2+\frac{f^{(1)}(0)}{(1)}x^1+f(0)x^0$

阶乘 factorial

**麦克劳林级数$(3)$是泰勒级数在$x=0$展开的特殊情况**

$f(x)=\displaystyle\sum_{n=0}^{\infty}\frac{f^{(n)}(0)}{n!}x^n \qquad (3)$

## 2. 三角函数

$f(x)=sin(x)$求导$f^{'}(x)=cos(x)$

使用麦克劳林级数对$f(x)=sin(x)$展开，得到

$f(x)=sin(x)=f(0)+f^{'}(0)x+\displaystyle\frac{f^{''}(0)}{2!}x^2 + \frac{f^{'''}(0)}{3!}x^3...$

即

$sinx=0+1x+0\displaystyle\frac{x^2}{2!}+(-1)\frac{x^3}{3!}+...\qquad (4)$

再使用麦克劳林级数对$f(x)=cos(x)$展开，得到

$cosx=1+0x+(-1)\displaystyle\frac{x^2}{2!}+0\frac{x^3}{3!}+...\qquad (5)$

再使用麦克劳林级数对$f(x)=e^x$展开

$e^x=1+x+\displaystyle\frac{x^2}{2!}+\frac{x^3}{3!}+...\qquad (6)$

如果把公式$(4)$和公式$(5)$相加，得到一个近似公司$(6)$的公式，但是有些项的符号为负。

为了消除负号，欧拉引入了$i^2=-1$

根据公式$(6)$，有如下公式

$e^{xi}=1+xi+i^2\displaystyle\frac{x^2}{2!}+i^2\frac{x^3i}{3!}+...\qquad (7)$

调和公式$(4)$和公式$(5)$，$i\ast sinx+cosx$，可以得到公式$(7)$

于是得到欧拉公式：

$e^{xi}=cosx+i\ast sinx$

## 3. 欧拉公式的应用

[链接](https://mathvault.ca/euler-formula/)

[链接2](https://ozanerhansha.medium.com/applications-of-eulers-formula-857bf60ba32d)

- 复数的幂$\rightarrow$复数：$(a+bi)^n=r^ne^{in\theta}$
- 含复数的指数$\rightarrow$复数：$5^{3+2i}=5^3(cos(2ln5)+i\ast sin(2ln5))\approx-124.64+0.65i$
- 

---

## 问题

Euler's identity 欧拉恒等式？

复数的指数形式

三角函数和双曲线函数的代替 Alternate definitions of trigonometric and hyperbolic functions

指数和对数函数对复数的推广==复数的指数==

**棣莫弗定理**和三角加法相同点的替代证明 Alternate proofs of de Moivre’s theorem and trigonometric additive identities

$(a+bi)^n$可以通过欧拉公式的形式化简

上面推到过，则

$(re^{i\theta})^n\ =\ (r)^n(e^{i\theta})^n$

于是

$r^ne^{in\theta}\ =\ r^n(cos(n\theta)+i\ast sin(n\theta))$