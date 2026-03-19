考虑微分方程$\frac{\partial \phi}{\partial t}+\nabla \cdot F=S$在等号两边进行空间积分得
$$
\int_{\Omega}\frac{\partial \phi}{\partial t}d\Omega+\int_{\partial \Omega}Fd\sigma=\int_{\Omega}Sd\Omega
$$
每个体积元$\Omega$都满足上述方程，考虑一维情况，并假设无源：
$$
q_t(x,t)+f(q(x,t))_x=0
$$

![[Pasted image 20250402210645.png]]
在小区间内求出q的积分均值:
$$
Q_i^n=\frac{1}{\Delta x}\int_{x_{i-\frac{1}{2}}}^{x_{i+\frac{1}{2}}}q(x,t_n)dx=\frac{1}{\Delta x}\int_{C_i}q(x,t_n)dx
$$
