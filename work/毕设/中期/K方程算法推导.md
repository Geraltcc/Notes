# 湍动能控制方程
$$\frac{\partial k}{\partial t}+\frac{\partial}{\partial x_j}\left(u_jk\right)=P_k-\varepsilon +\frac{\partial}{\partial x_j}\left[(\nu +\nu_t)k_{,j}\right]$$
## 分量写法
$$\frac{\partial k}{\partial t}+u\frac{\partial k}{\partial x}+v\frac{\partial k}{\partial y}+w\frac{\partial k}{\partial z}=P_k-\varepsilon +\frac{\partial}{\partial x}\left[(\nu +\nu_t)\frac{\partial k}{\partial x}\right]+\frac{\partial}{\partial y}\left[(\nu+\nu_t)\frac{\partial k}{\partial y}\right]+\frac{\partial }{\partial z}\left[(\nu+\nu_t)\frac{\partial k}{\partial z}\right]$$
## 隐式离散
Production lagged in time:
	$$\frac{k^{n+1}-k^n}{\Delta t}+C^{n+1}=P_k^n-\varepsilon ^{n}+D^{n+1}$$
	$$\Delta k=k^{(n+1)}-k^{(n)}$$
	$$\frac{\Delta k}{\Delta t}=P_k^n+D^{(n+1)}-C^{(n+1)}-\varepsilon ^{(n)}=\mathrm{RHS}_k$$
## AF线性化RHS at time level$(n+1)$
1. 对流项：
	$$-u_j\frac{\partial k}{\partial x_j}^{(n+1)}=-u_j\frac{\partial (k^{(n)}+\Delta k)}{\partial x_j}=-u_j\frac{\partial k}{\partial x_j}^{(n)}-u_j\frac{\partial (\Delta k)}{\partial x_j}$$
	$$=-u_j\frac{\partial k^n}{\partial x_j}-u_j\delta ^{upwind}(\Delta k)$$
2. 扩散项：
$$\frac{\partial}{\partial x_j}\left[\Gamma \frac{\partial k}{\partial x_j}\right]^{(n+1)}=\frac{\partial}{\partial x_j}\left[\Gamma \frac{\partial (k^{(n)}+\Delta k)}{\partial x_j}\right]=\frac{\partial }{\partial x_j}\left[\Gamma \left(\frac{\partial k^{(n)}}{\partial x_j}+\frac{\partial (\Delta k)}{\partial x_j}\right)\right]$$
	$$=\frac{\partial}{\partial x_j}\left[\Gamma \frac{\partial k}{\partial x_j}\right]^{(n)}+\frac{\partial }{\partial x_j}\left[\Gamma \frac{\partial (\Delta k)}{\partial x_j}\right]
	$$
3. 整理得：
$$
\left[I+\Delta t\left(u_j\frac{\partial }{\partial x_j}-\frac{\partial }{\partial x_j}\left(\Gamma \frac{\partial}{\partial x_j}\right)\right)\right]\Delta k=\Delta t\cdot\left[P_k^n-\varepsilon^n - u_j\frac{\partial k}{\partial x_j}^{(n)}+\frac{\partial }{\partial x_j}\left(\Gamma \frac{\partial k}{\partial x_j}^{(n)}\right)\right]=\Delta t\cdot\mathrm{RHS}
$$
## LHS离散方案
1. 对流项::一阶迎风
	- 若$U>0$，则$\delta_j^{upwind}(\Delta k)=\Delta k_i-\Delta k_{i-1}$，$\Delta tu_j\delta_j^{upwind}=\frac{\Delta t}{\Delta x_j}u_j(\Delta k_i-\Delta k_{i-1})$
		$$
		=-(\frac{\Delta t}{\Delta x_j}u_j)\Delta k_{i-1}+(\frac{\Delta t}{\Delta x_j}u_j)\Delta k_{i}
		$$
	- 若$U<0$，则$\delta_j^{upwind}(\Delta k)=\Delta k_{i+1}-\Delta k_{i}$，$\Delta tu_j\delta_j^{upwind}=\frac{\Delta t}{\Delta x_j}u_j(\Delta k_{i+1}-\Delta k_{i})$
		$$=\left(\frac{\Delta t}{\Delta x_j}u_j\right)\Delta k_{i+1} -\left(\frac{\Delta t}{\Delta x_j}u_j\right)\Delta k_{i}$$
	- 合并得：
	$$
	\text{Adv}=-\left[\text{max}(u_j,0)\frac{\Delta t}{\Delta x_j}\right]\Delta k_{i-1} +\left[\text{abs}(u_j)\frac{\Delta t}{\Delta x_j}\right]\Delta k_{i}+\left[\text{min}(u_j,0)\frac{\Delta t}{\Delta x_j}\right]\Delta k_{i+1}
	$$
2. 扩散项::中心差分
	$$
	\frac{\partial}{\partial x_j}\left[\Gamma \frac{\partial k}{\partial x_j}\right]=\frac{1}{\Delta x^2}\left[\Gamma_{j,i+1/2}(\Delta k_{i+1}-\Delta k_i)-\Gamma_{j,i-1/2}(\Delta k_{i}-\Delta k_{i-1})\right]
	$$
	$$
	=\frac{\Gamma_{j,i-1/2}}{\Delta x^2}\Delta k_{i-1}-\frac{\Gamma_{j,i+1/2}+\Gamma_{j,i-1/2}}{\Delta x^2}\Delta k_{i}+\frac{\Gamma_{j,i+1/2}}{\Delta x^2}\Delta k_{i+1}
	$$
3. RHS计算
	- 生成项：二阶格式
		$$
		P_k=2\nu_tS_{ij}S_{ij}=2\nu_t \sum_{ij} S_{ij}^2
		$$
		$$
		S_{ij}=\frac{1}{2}\left(\frac{\partial u_i}{\partial x_j}+\frac{\partial u_j}{\partial x_i}\right)=\left[\begin{array}{ccc}\frac{\partial u}{\partial x} & \frac{1}{2}\left(\frac{\partial u}{\partial y}+\frac{\partial v}{\partial x}\right) & \frac{1}{2}\left(\frac{\partial u}{\partial z}+\frac{\partial w}{\partial x}\right) \\ \frac{1}{2}\left(\frac{\partial v}{\partial x}+\frac{\partial u}{\partial y}\right) & \frac{\partial v}{\partial y} & \frac{1}{2}\left(\frac{\partial v}{\partial z}+\frac{\partial w}{\partial y}\right) \\ \frac{1}{2}\left(\frac{\partial w}{\partial x}+\frac{\partial u}{\partial z}\right) & \frac{1}{2}\left(\frac{\partial w}{\partial y}+\frac{\partial v}{\partial z}\right) & \frac{\partial w}{\partial z}\end{array}\right]
		$$
	- 耗散项：
		$$
		\varepsilon=\varepsilon_1+\varepsilon_2
		$$
		$$
		\varepsilon_1=2\nu\frac{\partial \sqrt{k}}{\partial x_j}\frac{\partial \sqrt{k}}{\partial x_j}=2\nu \sum_{j}\left(\frac{\partial \sqrt{k}}{\partial x_j}\right)^2
		$$
		$$
		\varepsilon_2=\frac{A_{\varepsilon}k^{\frac{3}{2}}}{d}::\left\{\begin{array}{l}A_{\varepsilon}=2.68\left(Re_k/0.25\right)^{-0.8}\left(1+(Re_k/0.25)^{1.5}\right)^{0.45/1.5}(1+(Re_k/2.4)^{1.5})^{-0.1/1.5} \\ A_{\varepsilon}=min(A_{\varepsilon},0.8) \\ Re_k=\rho \frac{\sqrt{k}d}{\mu} \\ d\gets \text{distance to wall}\end{array}\right.
		$$
		
	- 对流项：
	- 扩散项：
4. 组合矩阵
	$$
	A_i\Delta k_{i-1}^*+B_i\Delta k_i^*+C_i\Delta k_{i+1}^*=\mathrm{RHS}_i
	$$
	其中：
	$$
	\left\{
	\begin{array}{l} 
	A_i=-\Delta t\left(\frac{\text{max}(u_j,0)}{\Delta x_j}+\frac{\Gamma_{j,i-1/2}}{\Delta x^2}\right) \\
	B_i= 1+\Delta t\left[\text{abs}(u_j)+\frac{\Gamma_{j,i+1/2}+\Gamma_{j,i-1/2}}{\Delta x^2}\right] \\ 
	C_i=\Delta t\left(\frac{\text{min}(u_j,0)}{\Delta x_j}-\frac{\Gamma_{j,i+1/2}}{\Delta x^2}\right)
	\end{array} \right.
	$$

## 算法
$$\underset{\boldsymbol{A}}{\left[\begin{array}{ccccc}b_1 & c_1 & 0 & \cdots & 0 \\ a_2 & b_2 & c_2 & \cdots & 0 \\ \vdots & \vdots & & \ddots & \vdots \\ 0 & \cdots & a_{N-1} & b_{N-1} & c_{N-1} \\ 0 & 0 & \cdots & a_N & b_N\end{array}\right]} \underset{\boldsymbol{u}}{\left[\begin{array}{c}u_1 \\ u_2 \\ \vdots \\ u_{N-1} \\ u_N\end{array}\right]}=\underset{\boldsymbol{d}}{\left[\begin{array}{c}d_1 \\ d_2 \\ \vdots \\ d_{N-1} \\ d_N\end{array}\right]}\tag{3}$$