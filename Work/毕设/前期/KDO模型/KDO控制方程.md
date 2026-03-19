$$\frac{\partial k}{\partial t} + \nabla \cdot (\mathrm{u}k) = P_k - \epsilon + \nabla \cdot [(\nu + \nu_t)\cdot \nabla k]$$
$$

\begin{split}
P_k & = \quad 2\nu_t S_{ij}S_{ij} \\

\varepsilon \hspace{0.5em}& = \quad 2\nu\frac{\partial \sqrt k}{\partial x_j}\frac{\partial \sqrt k}{\partial x_j} + \frac{A_{\varepsilon}k^{\frac{3}{2}}}{d} \\

A_{\varepsilon} & = \quad 2.68(\frac{Re_k}{0.25})^{-0.8}[1+(\frac{Re_k}{0.25})^{1.5}]^{\frac{0.45}{1.5}}[1+(\frac{Re_k}{2.4})^{1.5}]^{-\frac{0.1}{1.5}}\\

A_{\varepsilon} & = \quad min(A_{\varepsilon} , 0.8) \\

\end{split}
$$