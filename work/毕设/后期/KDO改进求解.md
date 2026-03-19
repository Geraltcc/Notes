$$
\frac{\partial k}{\partial t}+\nabla\cdot(\mathrm{u}k)=P_k-\varepsilon+\nabla\cdot[(\nu+\nu_t)\nabla k]
$$
## 一、仅在最密网格上计算
## 二、点隐法
### （1）核心公式
$$
\frac{k^{n+1}-k^n}{\Delta t} + A^n=(P_k^{n+1}-\varepsilon^{n+1})+D^n
$$
源项与耗散项线性化：
$$
P_k^{n+1}\approx P_k^n+\left(\frac{\partial P_k}{\partial k}\right)^n(k^{n+1}-k^n)
$$
! (2025.04.24) 修改，源项线性化对左端矩阵负贡献
!$$\frac{k^{n+1}-k^n}{\Delta t} + A^n=(P_k^{n}-\varepsilon^{n+1})+D^n$$
$$
\varepsilon^{n+1}\approx\varepsilon^n+\left(\frac{\partial \varepsilon}{\partial k}\right)^n(k^{n+1}-k^n)
$$
代回得到
$$
(P_k^{n+1}-\varepsilon^{n+1})\approx(P_k^n-\varepsilon^n)+\left(\frac{\partial P_k}{\partial k}-\frac{\partial \varepsilon}{\partial k}\right)^n(k^{n+1}-k^n)
$$
$$
\left[1-\Delta t\left(\frac{\partial P_k}{\partial k}-\frac{\partial \varepsilon}{\partial k}\right)^n\right]k^{n+1}=k^n+\Delta t(D^n-A^n)+\Delta t(P_k^n-\varepsilon^n)-\Delta t\left(\frac{\partial P_k}{\partial k}-\frac{\partial \varepsilon}{\partial k}\right)^nk^n
$$
!!!修正后
$$
\left[1+\Delta t\left(\frac{\partial \varepsilon}{\partial k}\right)^n\right]k^{n+1}=k^n+\Delta t(D^n-A^n)+\Delta t(P_k^n-\varepsilon^n)+\Delta t\left(\frac{\partial \varepsilon}{\partial k}\right)^nk^n
$$

仅需在当地网格求解，无需全局耦合。
### （2）具体实现
1. stiff源项
$$
P_k = R_bkS  \qquad \frac{\partial P_k}{\partial k}=R_bS(简化) 
$$
$$
\varepsilon=2\nu\left(\frac{\partial\sqrt{k}}{\partial x}+\frac{\partial \sqrt{k}}{\partial y}\right)+\left\{ \begin{array}{rl} 
2A_{\varepsilon}k^{1.5}/d,\quad &Re_k\leq10 \\
A_{\varepsilon}k^{1.5}/d,\quad &Re_k>10\end{array}\right.
$$
由于网格足够细，忽略$\varepsilon_1$项对k的偏导贡献
$$
\frac{\partial \varepsilon}{\partial k}=\left\{\begin{array}{rc}3A_{\varepsilon}\sqrt{k}/d,\quad &Re_k\leq10 \\ 1.5A_{\varepsilon}\sqrt{k}/d, \quad &Re_k>10\end{array}\right.
$$
2. 对流项（二阶迎风）考虑半隐式
	$$
	\left\{\begin{array}{lc}
	\phi_{i+1/2}=\frac{3}{2}\phi_i-\frac{1}{2}\phi_{i-1},\quad &u_{i+1/2}\geq0 \\
	\phi_{i+1/2}=\frac{3}{2}\phi_{i+1}-\frac{1}{2}\phi_{i+2},\quad &u_{i+1/2}<0
	\end{array}\right.
	$$
3. 扩散项
	- 中心差分
## 三、local time stepping
