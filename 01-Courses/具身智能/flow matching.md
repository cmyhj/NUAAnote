# Flow Matching：从直觉到公式

## 一、 核心问题：生成模型在做什么？

所有生成模型的目标都是同一个：**学会从简单分布（如高斯噪声）生成复杂数据分布（如图片、机器人动作）**。

不同的方法走不同的路：
- **GAN**：训练一个生成器和判别器对抗博弈
- **VAE**：通过变分推断学习隐空间
- **Diffusion (DDPM)**：学习逐步去噪的逆过程（SDE路径，弯曲）
- **Flow Matching**：学习从噪声到数据的一条**直线流动**（ODE路径，直）

## 二、 从 ODE 视角看生成

Flow Matching 的数学基础是**连续归一化流（CNF）**。核心思想是：存在一个随时间变化的速度场 $v_\theta(x, t)$，它定义了一条从噪声分布 $p_0$ 到数据分布 $p_1$ 的传输路径。

$$\frac{dx}{dt} = v_\theta(x, t), \quad t \in [0, 1]$$

- 在 $t=0$ 时，$x_0 \sim \mathcal{N}(0, I)$，即纯噪声
- 在 $t=1$ 时，$x_1 \sim p_{\text{data}}$，即目标数据
- 中间时刻 $t \in (0,1)$，$x_t$ 是噪声到数据的"过渡态"

**关键问题**：如何训练 $v_\theta$ 来学好这个速度场？

## 三、 Conditional Flow Matching：核心公式推导

### 3.1 朴素思路（不可行）

最直接的想法是定义一个**边际速度场**：

$$\mathcal{L}_{\text{marginal}} = \mathbb{E}_{t, x_t} \left[ \| v_\theta(x_t, t) - u_t(x_t) \|^2 \right]$$

其中 $u_t(x_t)$ 是将 $p_0$ 传输到 $p_1$ 的"真实"速度场。**问题在于**：$u_t(x_t)$ 难以解析计算，因为它依赖于 $p_t$（中间分布）的全局信息。

### 3.2 Conditional Flow Matching（条件流匹配）

Flow Matching 的核心突破在于：**对每个数据点 $x_1$，定义一个条件速度场 $u_t(x | x_1)$，这个条件速度场是可以解析写出的！**

**条件概率路径**定义为：

$$p_t(x | x_1) = \mathcal{N}(x | \mu_t(x_1), \sigma_t(x_1)^2 I)$$

最常用的选择（**高斯概率路径 / 线性插值**）：

$$\boxed{\mu_t(x_1) = t \cdot x_1, \quad \sigma_t(x_1) = 1 - t}$$

这意味着：

$$\boxed{x_t = (1-t)\cdot\epsilon + t\cdot x_1, \quad \epsilon \sim \mathcal{N}(0, I)}$$

这就是**直线插值**：从噪声 $\epsilon$ 到数据 $x_1$ 之间画一条直线，$t$ 从 0 走到 1。

**对应的条件速度场**为：

$$\boxed{u_t(x | x_1) = \frac{x_1 - x}{1 - t} = x_1 - \epsilon}$$

> 推导：对 $x_t = (1-t)\epsilon + t x_1$ 求导：
> $$\frac{dx_t}{dt} = -\epsilon + x_1 = x_1 - \epsilon$$
> 这就是将噪声"推向"数据的速度方向。

**训练目标（条件流匹配损失）**：

$$\boxed{\mathcal{L}_{\text{CFM}} = \mathbb{E}_{t \sim U(0,1),\, x_1 \sim p_{\text{data}},\, \epsilon \sim \mathcal{N}(0,I)} \left[ \left\| v_\theta\big((1-t)\epsilon + t x_1,\, t\big) - (x_1 - \epsilon) \right\|^2 \right]}$$

这个损失函数惊人地简洁：
- 从数据中采样 $x_1$
- 从噪声中采样 $\epsilon$
- 随机选时间 $t$
- 构造插值点 $x_t = (1-t)\epsilon + t x_1$
- 让网络预测速度 $v_\theta(x_t, t)$
- 目标速度就是 $x_1 - \epsilon$

**美妙的性质**：可以证明，优化 $\mathcal{L}_{\text{CFM}}$ 等价于优化真正的边际损失 $\mathcal{L}_{\text{marginal}}$（在梯度期望意义上相等），因此边际速度场可以通过条件速度场的加权平均得到。

### 3.3 推理（采样）

训练好 $v_\theta$ 后，生成新样本只需从噪声出发，沿着速度场做 ODE 积分：

$$\boxed{x_1 = x_0 + \int_0^1 v_\theta(x_t, t)\, dt, \quad x_0 \sim \mathcal{N}(0, I)}$$

实际中用 Euler 方法离散化：

$$x_{t+\Delta t} = x_t + v_\theta(x_t, t) \cdot \Delta t$$

步数通常只需 **10-50 步**即可获得高质量结果。

## 四、 与其他方法的精确对比

| 方法 | 前向过程 | 学习目标 | 推理 |
|------|---------|---------|------|
| **DDPM** | $x_t = \bar\alpha_t x_0 + \sqrt{1-\bar\alpha_t}\,\epsilon$（非直线） | 预测噪声 $\epsilon_\theta(x_t, t)$ | 逐步去噪 SDE |
| **Score Matching (SMLD)** | 类似 DDPM | 预测 score $\nabla_x \log p_t(x)$ | Langevin MCMC |
| **Flow Matching** | $x_t = (1-t)\epsilon + t x_1$（**直线**） | 预测速度 $v_\theta(x_t, t)$ | ODE 积分 |

DDPM 的路径是**弯曲**的（受 $\bar\alpha_t$ 噪声调度控制），而 Flow Matching 是**直线**的。直线路径属于最优传输（Optimal Transport）的解，理论上更高效。

## 五、 具体例子

### 例1：一维高斯混合 → 一维高斯

**任务**：学习从高斯噪声 $\mathcal{N}(0,1)$ 到数据分布 $p_{\text{data}} = 0.5\cdot\mathcal{N}(-3, 0.3^2) + 0.5\cdot\mathcal{N}(3, 0.3^2)$ 的映射。

**训练过程**：
```
重复 N 次:
  1. 从数据分布采样 x₁ = 3.2 （假设这次采到了右侧峰）
  2. 从噪声分布采样 ε = -0.5
  3. 随机选 t = 0.3
  4. 计算插值点 x_t = 0.7 × (-0.5) + 0.3 × 3.2 = 0.61
  5. 目标速度 = x₁ - ε = 3.2 - (-0.5) = 3.7
  6. 网络预测 v_θ(0.61, 0.3)，最小化与 3.7 的 MSE
```

**推理过程**：
```
  1. 采样 x₀ = -0.3（随机噪声）
  2. 用学到的速度场积分：
     t=0.0: x = -0.3,  v_θ = 3.5  → 推向右侧
     t=0.2: x = 0.4,   v_θ = 3.1
     t=0.4: x = 1.02,  v_θ = 2.8
     t=0.6: x = 1.58,  v_θ = 2.5
     t=0.8: x = 2.08,  v_θ = 2.3
     t=1.0: x = 2.54   → 最终样本落在右侧峰附近
```

不同的噪声起点 $x_0$ 会被导向不同的模式，从而自然地生成两个峰的分布。

### 例2：二维数据点云（瑞士卷）

**任务**：生成二维瑞士卷形状的数据点。

**训练**：对每对 $(\epsilon, x_1)$，直线插值并学习速度场。

**可视化**：在二维平面上，速度场形成了从各个噪声点出发、**汇聚并卷曲成瑞士卷形状**的流线。靠近瑞士卷中心的噪声点需要"转弯"，而外部的则相对直线到达。

```
噪声空间 (t=0):          数据空间 (t=1):
  . . . . . .            ╭─╮
  . . . . . .           ╭╯ ╰─╮
  . . . . . .    →→→   ╭╯    ╰─╮
  . . . . . .          │  🌀   │
  . . . . . .          ╰─╮  ╭──╯
                         ╰──╯
  均匀噪声点     流动→    瑞士卷分布
```

### 例3：机器人动作生成（π0 场景）

**任务**：给定视觉观察和语言指令"把杯子放到盘子上"，生成一段机械臂动作序列。

**数据**：$x_1 \in \mathbb{R}^{H \times D}$ 是一个动作 chunk，$H$ 步动作，每步 $D$ 维（如 7 个关节角度）。

**训练**：
```
  x₁ = [a₁, a₂, ..., aₕ]  // 真实动作序列，维度 H×D
  ε ~ N(0, I)               // 同维度噪声
  t ~ U(0, 1)
  
  x_t = (1-t)ε + t·x₁      // 插值
  v_θ(x_t, t, c) ≈ x₁ - ε  // c 是 VLM 条件特征
```

**推理**：
```
  采样 ε ~ N(0, I)
  for t = 0, Δt, 2Δt, ..., 1:
      x_{t+Δt} = x_t + v_θ(x_t, t, c) · Δt
  输出: x₁ = 动作 chunk → 机器人执行前 k 步
```

## 六、 Flow Matching 的变体

除了上述最基础的 **Conditional Flow Matching (CFM)**，还有一些重要变体：

### 6.1 Optimal Transport (OT) Flow Matching

选择配对 $(\epsilon, x_1)$ 时，不是随机配对，而是使用**最优传输**来寻找最小化总传输距离的配对方案，使得直线路径更"直"、训练更高效。

$$\min_{\pi} \int \|x_1 - \epsilon\|^2 \, d\pi(\epsilon, x_1)$$

### 6.2 Rectified Flow

通过 **reflow 操作**，反复用训练好的模型对数据重新配对，逐步让路径变得更直：

$$\text{第 } k \text{ 次 reflow: } \quad x_1^{(k)} = x_0 + \int_0^1 v_{\theta}^{(k)}(x_t, t)\, dt$$

然后用新的配对 $(x_0, x_1^{(k)})$ 训练 $v_{\theta}^{(k+1)}$。每次 reflow 路径更直，最终可能只需 1 步推理。

### 6.3 条件注入（π0 的做法）

在 π0 中，VLM 特征 $c$ 通过 **attention 层**注入 flow matching 网络：

$$v_\theta(x_t, t, c) = \text{DiT}(x_t, t, c)$$

其中 DiT（Diffusion Transformer）的每一层通过交叉注意力融合 $c$，使得速度场的生成受到语言和视觉条件的引导。

## 七、 总结

Flow Matching 的哲学可以用一句话概括：

> **"在噪声和数据之间画一条直线，然后学会沿着这条直线推。"**

它的优势在于：
- 公式简洁（一个 MSE 损失）
- 路径最短（直线 = 最优传输）
- 采样高效（少步数 ODE 求解）
- 与条件信息融合自然（VLM 特征通过 attention 注入）

这使得它成为当前机器人基础模型（如 π0）最偏爱的动作生成范式。