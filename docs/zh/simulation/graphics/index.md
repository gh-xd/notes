# 图形学中物理仿真方法概览

物理仿真中涉及的方法可以粗略地分为：

- 基于网格的 - Mesh based
  - 有限差分法 - Finite Difference Method (FDM)
  - 有限元法 - Finite Element Method (FEM)
  - 有限体积法 - Finite Volume Method (FVM)
- 无网格的 - Mesh-free / meshless
  - 离散元法 - Discrete Element Method (DEM)
  - 物质点法 - Material Point Method (MPM)
  - 光滑粒子流体动力学 - Smoothed Particle Hydrodynamics (SPH)

### DEM简述

原理：DEM方法对每一个颗粒的运动进行跟踪，通过硬球模型或软球模型（通常用软球模型）来计算粒子间的碰撞过程，也考虑粒子旋转

应用：可以处理颗粒形状、扭曲、粘连，？

### MPM简述

原理：物质点法将需要仿真的材料离散成粒子质点，用网格收集粒子的物理量、对物理量做处理（计算、求导等），再将物理量返回到粒子身上

应用：适合模拟涉及材料特大变形和断裂破碎，例如奶酪的切割，沙子的搅拌等

常见方法：

- 粒子平动范式
  - Particle-in-Cell (PIC), Affine Particle-in-Cell (APIC)
  - Polynomial PIC (PolyPIC)
  - Fluid Implicit Particles (FLIP)
- 物质点
  - Material Point Method (MPM)
  - Moving Least Squares MPM (MLS-MPM)
  - ...

精度：？

### SPH简述

原理：SPH中颗粒并不会发生直接相互作用，而是通过**核函数**实现不同颗粒间的相互作用。

应用：一般用于流体仿真，但是可以做固液耦合

常见方法：

- 弱可压缩SPH - Weakly Compressible SPH (WCSPH)
- Predictive-corrective Incompressible SPH (PCISPH)
- 隐式不可压缩SPH - Implicit Incompressible SPH (IISPH)
- 无散SPH - Divergence-free SPH (DFSPH)

精度：？