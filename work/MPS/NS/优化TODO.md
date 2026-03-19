您的 `MPS` 类实现是非常标准的“Zipper”算法（逐步SVD截断），但在处理张量收缩顺序（Contraction Order）时存在严重的性能陷阱。

目前的实现方式导致了极大的**中间张量膨胀**。简单来说，您是先将两个 Core 完全展开相乘（$O(D^4)$），然后再投影到左边界（$O(D^3)$）。**对于 $D=100$，这导致中间内存占用膨胀了 100 倍，且极其消耗带宽。**

以下是针对 `mps.cpp` 的三个关键优化方案，按收益从高到低排序：

### 1. 核心优化：改变收缩顺序 (Contraction Order)

**这是最致命的性能杀手。**

- **当前做法：** `C = A * B` (生成巨大的 $D^2 \times D^2$ 张量)，然后 `Merged = Left * C`。
    
- **优化做法：** 先将 `Left` 矩阵收缩进 `A`，生成一个临时的 $D \times D$ 张量，再与 `B` 收缩。这样中间结果永远保持在 $O(D^3)$ 级别。
    

**收益：** 内存占用和计算量均从 $O(D^4)$ 降低到 $O(D^3)$。对于 $D=100$，速度提升可达 **10-50倍**。

#### 优化后的 `zip_hadamard` 实现

请用此代码替换原有的 `zip_hadamard` 循环部分：

C++

```
// 优化版 zip_hadamard 循环
for (size_t i = 0; i < mask.L_; ++i) {
    const Tensor& A = mask[i];      // shape: (la, p, ra)
    const Tensor& B = field_copy[i];// shape: (lb, p, rb)

    size_t la = A.shape()[0], p = A.shape()[1], ra = A.shape()[2];
    size_t lb = B.shape()[0], rb = B.shape()[2];

    // left_mat 形状原本是 (new_l, la * lb)
    // 我们将其视为 (new_l, la, lb)
    size_t new_l = left_mat.shape()[0];
    
    // -----------------------------------------------------------
    // 步骤 1: 将 Left 与 A 收缩
    // Left(n, la, lb) * A(la, p, ra) -> Temp(n, lb, p, ra)
    // 复杂度: D^3 * d
    // -----------------------------------------------------------
    Tensor temp({new_l, lb, p, ra});
    
    // 使用 tblis 直接收缩，避免 permute 开销
    // indices: n=new_l, a=la, b=lb, s=p, r=ra
    tblis::mult(
        tblis::single,
        dtype(1.0), left_mat.get_cview().reshape({new_l, la, lb}), "nab",
                    A.get_cview(),                                 "asr",
        dtype(0.0), temp.get_view(),                               "nbsr"
    );

    // -----------------------------------------------------------
    // 步骤 2: 将 Temp 与 B 收缩
    // Temp(n, lb, p, ra) * B(lb, p, rb) -> Merged(n, ra, rb, p)
    // 注意: Hadamard 积要求物理指标 p 相同
    // -----------------------------------------------------------
    // 我们直接生成 SVD 所需的布局: (n * p, ra * rb) -> (n, p, ra, rb)
    Tensor merged({new_l, p, ra, rb});

    // indices: n=new_l, b=lb, s=p, x=ra, y=rb
    tblis::mult(
        tblis::single,
        dtype(1.0), temp.get_cview(), "nbsx",
                    B.get_cview(),    "bsy",
        dtype(0.0), merged.get_view(),"nsxy"
    );

    // -----------------------------------------------------------
    // 步骤 3: SVD 截断
    // -----------------------------------------------------------
    // Reshape 为矩阵进行 SVD: (rows=new_l*p, cols=ra*rb)
    merged = merged.reshape({new_l * p, ra * rb});

    if (i == mask.L_ - 1) {
        // 最后一个点直接作为 core
        Tensor new_core = merged.reshape({new_l, p, 1});
        new_cores.push_back(std::move(new_core));
    } else {
        // 截断 SVD
        auto [U, S, Vt] = linalg::svd(merged, max_rank, cutoff);
        
        // U: (new_l * p, rank) -> new core
        new_cores.push_back(U.reshape({new_l, p, S.size()}));
        
        // 更新 Left 矩阵: S * Vt -> (rank, ra * rb)
        left_mat = Tensor::diag(S) % Vt;
    }
}
```

#### 优化后的 `zip_apply` (MPO x MPS)

同样的逻辑适用于 `zip_apply`。先收缩 `Left` 和 `MPS Core`，再收缩 `MPO Core`。

C++

```
// 在 zip_apply 循环中...
    // MPO W: (lw, rw, pin, pout)
    // MPS A: (lpsi, pin, rpsi)
    // Left:  (new_l, lpsi, lw)

    // 1. Temp = Left * A  => Indices: (new_l, lw, pin, rpsi)
    // Contraction: sum(lpsi) Left(n, lpsi, lw) * A(lpsi, pin, rpsi)
    Tensor temp({new_l, lw, p_in, r_psi});
    tblis::mult(..., left_mat, "naw", A, "api", temp, "nwpi");

    // 2. Merged = Temp * W => Indices: (new_l, pout, rpsi, rw)
    // Contraction: sum(lw, pin) Temp(n, lw, pin, rpsi) * W(lw, rw, pin, pout)
    // 注意目标布局直接为了SVD: (n, pout, rpsi, rw)
    Tensor merged({new_l, p_out, r_psi, r_w});
    tblis::mult(..., temp, "nwpi", W, "wrjo", merged, "noir");

    // 3. Reshape & SVD
    merged = merged.reshape({new_l * p_out, r_psi * r_w});
    // ... 后续 SVD 逻辑不变 ...
```

---

### 2. 内存复用：原地加法 (In-place Addition)

问题： 您的 operator- 实现是 *this + (other * -1.0)。这会创建 other 的完整副本，乘 -1，再做加法，产生大量临时对象。

解决： 实现一个 axpy_inplace (y = a*x + y)。

在 `MPS` 类中添加：

C++

```
// mps.hpp
void add_inplace(const MPS& other, dtype alpha = 1.0);

// mps.cpp
void MPS::add_inplace(const MPS& other, dtype alpha) {
    if (L_ != other.L_) throw std::invalid_argument("Size mismatch");
    
    // 只有第一个 core 需要乘以 alpha (标量乘法的线性性质)
    // 第一个 core 的合并逻辑:
    // NewCore[0] = Concat(CoreA[0], alpha * CoreB[0], axis=2)
    
    // 中间 core: 直接 Concat
    // 最后一个 core: Concat axis=0
    
    // 这是一个简化描述。由于您的加法是 Direct Sum (增加 Bond Dimension)，
    // "原地" 意味着不创建全新的 vector<Tensor> 结构，而是直接修改当前的 cores_。
    
    // 实际上，由于 Direct Sum 会改变张量形状，完全的 "In-place" 很难避免重分配。
    // 但您可以避免 (other * -1.0) 这一步的整链复制。
    
    // 优化后的 operator-:
    // 不要调用 operator*(-1)，而是传入一个 flag 或直接在加法构建时处理系数。
}
```

**实用建议：** 鉴于 MPS 加法本身就会重新分配内存（因为维度变了），最简单的优化是修改 `operator+` 接受一个标量系数 `MPS::add(const MPS& other, dtype alpha)`，这样在做减法时调用 `u.add(v, -1.0)`，省去一次全拷贝。

---

### 3. 去除 `std::vector` 初始化

您在 `mps.cpp` 中多次使用：

C++

```
Tensor new_result({r}, 0.0); // 分配并填0
```

在 TBLIS mult 中，如果 beta=0.0，它会覆盖输出，所以不需要先初始化为 0。

优化： 确保您的 Tensor 构造函数支持 uninitialized 模式（即只 malloc 不 memset），或者在调用 tblis::mult 前不要手动填 0。

### 总结

请优先实施 方案 1 (改变收缩顺序)。

在 TT/MPS 算法中，$O(D^4) \to O(D^3)$ 的算法优化是任何代码级微调都无法比拟的。实施后，您的 tblis::pack 和 gemm 时间将大幅下降，因为处理的数据量减少了两个数量级。