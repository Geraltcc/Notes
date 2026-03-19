#有限体积法离散
# 一、通用格式
$$
\frac{\partial k}{\partial t}+\frac{\partial}{\partial x_j}\left(u_jk\right)=P_k-\varepsilon +\frac{\partial}{\partial x_j}\left[(\nu +\nu_t)k_{,j}\right]
$$
1. 积分：
$$
\int_{\Omega}\frac{\partial k}{\partial t}+\int_{\Omega}\nabla\cdot(\vec{u}k)=\int_{\Omega}(P_k-\varepsilon)+\int_{\Omega}\nabla \cdot(\Gamma \nabla k)
$$
2. Gauss散度定理
$$
\frac{k^{(n+1)}-k^n}{\Delta t}V_{\Omega}+\sum_{nb}^{f}k(\vec{u}\cdot\vec{S_f})=(P_k-\varepsilon)V_{\Omega}+\sum_{nb}^{f}(\Gamma \nabla k)\cdot \vec{S_f}
$$
# 二、AF方法
### (1).控制方程：
$$
\frac{\partial \phi}{\partial t}+\nabla \cdot (\vec{u}\phi)=P_k-\varepsilon+\nabla \cdot (\Gamma^{\phi}\nabla \phi)
$$
### (2).半离散：
$$
\frac{\phi^{n+1}-\phi^{n}}{\Delta t}+\nabla\cdot(\vec{u}\phi)^{n+1}=(P_k-\varepsilon)^{n}+\nabla\cdot(\Gamma^{\phi}\nabla\phi)^{n+1}
$$
### (3).线性化at time level (n+1)
$$
\phi^{n+1}=\phi^{n}+\Delta\phi
$$
1. 对流项
$$
\nabla\cdot(\vec{u}\phi)^{n+1}=\left(\frac{\partial (u\phi)}{\partial x}+\frac{\partial (v\phi)}{\partial y}+\frac{\partial (w\phi)}{\partial z}\right)^{n+1}
$$
$$
=\left(\frac{\partial [u(\phi+\Delta\phi)]}{\partial x}+\frac{\partial [v(\phi+\Delta\phi)]}{\partial y}+\frac{\partial [w(\phi+\Delta\phi)]}{\partial z}\right)^{n}
$$
$$
=\left(\frac{\partial (u\phi)}{\partial x}+\frac{\partial (v\phi)}{\partial y}+\frac{\partial (w\phi)}{\partial z}\right)^{n}+\left(\frac{\partial (u\Delta\phi)}{\partial x}+\frac{\partial (v\Delta\phi)}{\partial y}+\frac{\partial (w\Delta\phi)}{\partial z}\right)
$$
$$
=\nabla\cdot(\vec{u}\phi^{n})+\nabla\cdot(\vec{u}\Delta\phi)
$$
2. 扩散项
$$
\nabla\cdot(\Gamma^{\phi}\nabla\phi^{n+1})=\frac{\partial}{\partial x}\left(\Gamma^{\phi}\frac{\partial \phi}{\partial x}\right)^{n+1}+\frac{\partial}{\partial y}\left(\Gamma^{\phi}\frac{\partial \phi}{\partial y}\right)^{n+1}+\frac{\partial}{\partial z}\left(\Gamma^{\phi}\frac{\partial \phi}{\partial z}\right)^{n+1}
$$
$$
=\left(\frac{\partial \Gamma^{\phi}}{\partial x}\frac{\partial\phi}{\partial x}+\Gamma^{\phi}\frac{\partial^2 \phi}{\partial x^2}\right)+\left(\frac{\partial \Gamma^{\phi}}{\partial y}\frac{\partial\phi}{\partial y}+\Gamma^{\phi}\frac{\partial^2 \phi}{\partial y^2}\right)+\left(\frac{\partial \Gamma^{\phi}}{\partial z}\frac{\partial\phi}{\partial z}+\Gamma^{\phi}\frac{\partial^2 \phi}{\partial z^2}\right)
$$
$$
=\nabla\cdot(\Gamma^{\phi}\nabla\phi^{n})+\nabla\cdot\left[\Gamma^{\phi}\nabla(\Delta \phi)\right]
$$
### (4).组装方程
$$
\frac{\phi^{n+1}-\phi^{n}}{\Delta t}+\nabla\cdot(\vec{u}\phi)^{n+1}=(P_k-\varepsilon)^{n}+\nabla\cdot(\Gamma^{\phi}\nabla\phi)^{n+1}
$$
1. 代入得
$$
\frac{\Delta\phi}{\Delta t}+\nabla\cdot(\vec{u}\phi^{n})+\nabla\cdot(\vec{u}\Delta\phi)=(P_k-\varepsilon)^{n}+\nabla\cdot(\Gamma^{\phi}\nabla\phi^{n})+\nabla\cdot\left[\Gamma^{\phi}\nabla(\Delta \phi)\right]
$$
2. 移项得
$$
\frac{\Delta\phi}{\Delta t}+\nabla\cdot(\vec{u}\Delta\phi)-\nabla\cdot\left[\Gamma^{\phi}\nabla(\Delta\phi)\right]=(P_k-\varepsilon)^{n}+\nabla\cdot(\Gamma^{\phi}\nabla\phi^{n})-\nabla\cdot(\vec{u}\phi^n)
$$
3. 在单元$\Omega$内积分得
$$
\int_{\Omega}\frac{\Delta \phi}{\Delta t}+\int_{\Omega}\nabla\cdot(\vec{u}\Delta\phi)-\int_{\Omega}\nabla\cdot\left[\Gamma^{\phi}\nabla(\Delta\phi)\right]=\int_{\Omega}(P_k-\varepsilon)^{n}+\int_{\Omega}\nabla\cdot(\Gamma^{\phi}\nabla\phi^{n})-\int_{\Omega}\nabla\cdot(\vec{u}\phi^n)
$$
4. Gauss散度定理
$$
\frac{\Delta \phi}{\Delta t}V_{\Omega}+\sum_{nb}^{f}\Delta\phi(\vec{u}\cdot\vec{S_f})-\sum_{nb}^{f}\Gamma^{\phi}\left[\nabla(\Delta\phi)\cdot\vec{S_f}\right]=(P_k-\varepsilon)^nV_{\Omega}+\sum_{nb}^{f}\Gamma^{\phi}(\nabla\phi^n\cdot\vec{S_f})-\sum_{nb}^{f}\phi^n(\vec{u}\cdot\vec{S_f})
$$
