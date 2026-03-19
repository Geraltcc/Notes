#cfl3d #implicit-approximate-factorization
## Discritization Scheme
1. advection term: first-order upwind
2. source term     :explicit, lagged in time(to obtain positive  LHS Matrix)
### Example SST model
$$\frac{\partial k}{\partial t}=-u_j\frac{\partial k}{\partial x_j}+\frac{1}{\rho}P_k\left(\frac{M_{\infty}}{Re}\right)-\beta ^{\prime}k\omega \left(\frac{Re}{M_{\infty}}\right)+\frac{1}{\rho}\frac{\partial}{\partial x_j}\left[\left(\mu +\frac{\mu _T}{\sigma _k}\right)\frac{\partial k}{\partial x_j}\right]\left(\frac{M_{\infty}}{Re}\right)=RHS_k$$
linearize the right-hand-side terms at time$(n+1)$
$$\Delta k=k^{(n+1)}-k^{(n)}$$
$$-u_j\frac{\partial k^{(n+1)}}{\partial x_j}\cong -u_j\frac{\partial k^{(n)}}{\partial x_j}-\frac{\partial}{\partial k}\left(u_j\frac{\partial k}{\partial x_j}\right)(\Delta k)$$
$$\cong -u_j\frac{\partial k^{(n)}}{\partial x_j}-\frac{\partial}{\partial k}\left(u_j\frac{\partial k}{\partial x_j}\right)(\Delta k)$$
$$-u_j\frac{\partial k^{(n+1)}}{\partial x_j}\cong -u_j\frac{\partial k^{(n)}}{\partial x_j}-u_j\delta ^{upwind}(\Delta k)$$
