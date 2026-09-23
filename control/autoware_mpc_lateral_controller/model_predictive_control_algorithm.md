<a id="mpc-algorithm"></a>

# MPC 算法

<a id="introduction"></a>

## 简介

模型预测控制（MPC）是一种控制方法，它在每个控制周期求解优化问题，根据给定车辆模型确定最优控制序列，并使用计算出的控制输入序列控制系统。

简单来说，MPC 控制器计算一系列控制输入，优化状态轨迹和输出轨迹，从而实现期望行为。MPC 控制系统的主要特点如下：

1. 预测未来轨迹：MPC 通过预测未来状态和输出轨迹计算控制序列，将第一个控制输入应用于系统，并在每个控制周期以滚动时域方式重复此过程。
2. 处理约束：MPC 能在优化过程中处理状态和输入变量的约束，确保系统在规定范围内运行。
3. 处理复杂动力学：MPC 算法能够处理复杂的线性或非线性动力学。

选择线性还是非线性模型或约束方程，取决于 MPC 问题的具体形式。如果运动方程或约束中存在非线性表达式，优化问题就成为非线性问题。以下各节逐步说明如何在 MPC 框架内求解线性和非线性优化问题。请注意，本文使用线性化方法处理非线性模型。

<a id="linear-mpc-formulation"></a>

## 线性 MPC 建模

<a id="formulate-as-an-optimization-problem"></a>

### 表述为优化问题

本节介绍线性系统的 MPC。下一节将以车辆路径跟踪为应用，展示问题的建模过程。

在线性 MPC 中，所有运动和约束表达式都是线性的。对于路径跟踪问题，假设系统运动可由方程组（1）描述。状态演化和测量采用离散状态空间形式，其中矩阵 $A$、$B$ 和 $C$ 分别表示状态转移矩阵、控制矩阵和测量矩阵。

$$
\begin{gather}
x_{k+1}=Ax_{k}+Bu_{k}+w_{k}, y_{k}=Cx_{k} \tag{1} \\\
x_{k}\in R^{n},u_{k}\in R^{m},w_{k}\in R^{n}, y_{k}\in R^{l}, A\in R^{n\times n}, B\in R^{n\times m}, C\in R^{l \times n}
\end{gather}
$$

方程（1）是状态空间方程，其中 $x_k$ 表示内部状态，$u_k$ 表示输入，$w_k$ 表示由线性化或问题结构引起的已知扰动，$y_k$ 表示测量值。

MPC 的另一个优点是能够有效处理扰动项 $w$。尽管此处称之为扰动，只要符合方程结构，它可以采用多种形式。

方程（1）中的状态转移和测量方程以迭代方式从时刻 $k$ 推进到 $k+1$。从初始状态与控制输入对 $(x_0, u_0)$ 出发，在指定的 $N$ 步预测时域内递推，便可预测状态和测量轨迹。

为简化讨论，假设初始状态为 $x_0$，此时 $k=0$。

首先，将初始状态代入方程（1），计算 $k=1$ 时的状态 $x_1$。由于我们要求解输入序列，因此在符号表达式中将输入作为决策变量。

$$
\begin{align}
x_{1} = Ax_{0} + Bu_{0} + w_{0} \tag{2}
\end{align}
$$

然后，当 $k=2$ 时，结合方程（2）得到：

$$
\begin{align}
x_{2} & = Ax_{1} + Bu_{1} + w_{1} \\\
& = A(Ax_{0} + Bu_{0} + w_{0}) + Bu_{1} + w_{1} \\\
& = A^{2}x_{0} + ABu_{0} + Aw_{0} + Bu_{1} + w_{1} \\\
& = A^{2}x_{0} + \begin{bmatrix}AB & B \end{bmatrix}\begin{bmatrix}u_{0}\\\ u_{1} \end{bmatrix} + \begin{bmatrix}A & I \end{bmatrix}\begin{bmatrix}w_{0}\\\ w_{1} \end{bmatrix} \tag{3}
\end{align}
$$

当 $k=3$ 时，根据方程（3）：

$$
\begin{align}
x_{3} & = Ax_{2} + Bu_{2} + w_{2} \\\
& = A(A^{2}x_{0} + ABu_{0} + Bu_{1} + Aw_{0} + w_{1} ) + Bu_{2} + w_{2} \\\
& = A^{3}x_{0} + A^{2}Bu_{0} + ABu_{1} + A^{2}w_{0} + Aw_{1} + Bu_{2} + w_{2} \\\
& = A^{3}x_{0} + \begin{bmatrix}A^{2}B & AB & B  \end{bmatrix}\begin{bmatrix}u_{0}\\\ u_{1} \\\ u_{2} \end{bmatrix} + \begin{bmatrix} A^{2} & A & I \end{bmatrix}\begin{bmatrix}w_{0}\\\ w_{1} \\\ w_{2} \end{bmatrix} \tag{4}
\end{align}
$$

如果 $k=n$，则：

$$
\begin{align}
x_{n} = A^{n}x_{0} + \begin{bmatrix}A^{n-1}B & A^{n-2}B & \dots  & B  \end{bmatrix}\begin{bmatrix}u_{0}\\\ u_{1} \\\ \vdots  \\\ u_{n-1} \end{bmatrix} + \begin{bmatrix} A^{n-1} & A^{n-2} & \dots & I \end{bmatrix}\begin{bmatrix}w_{0}\\\ w_{1} \\\ \vdots \\\ w_{n-1} \end{bmatrix}
\tag{5}
\end{align}
$$

将方程（2）至（5）组合，可得到以下矩阵方程：

$$
\begin{align}
\begin{bmatrix}x_{1}\\\ x_{2} \\\ x_{3} \\\ \vdots  \\\ x_{n} \end{bmatrix} = \begin{bmatrix}A^{1}\\\ A^{2} \\\ A^{3} \\\ \vdots  \\\ A^{n} \end{bmatrix}x_{0} + \begin{bmatrix}B & 0 & \dots  & & 0 \\\ AB & B & 0 & \dots & 0  \\\ A^{2}B & AB & B & \dots & 0 \\\ \vdots & \vdots & & & 0 \\\ A^{n-1}B & A^{n-2}B & \dots & AB & B \end{bmatrix}\begin{bmatrix}u_{0}\\\ u_{1} \\\ u_{2} \\\ \vdots  \\\ u_{n-1} \end{bmatrix} \\\ +
\begin{bmatrix}I & 0 & \dots  & & 0 \\\ A & I & 0 & \dots & 0  \\\ A^{2} & A & I & \dots & 0 \\\ \vdots & \vdots & & & 0 \\\ A^{n-1} & A^{n-2} & \dots & A & I \end{bmatrix}\begin{bmatrix}w_{0}\\\ w_{1} \\\ w_{2} \\\ \vdots  \\\ w_{n-1} \end{bmatrix}
\tag{6}
\end{align}
$$

此时，测量值（输出）为 $y_{k}=Cx_{k}$，因此：

$$
\begin{align}
\begin{bmatrix}y_{1}\\\ y_{2} \\\ y_{3} \\\ \vdots  \\\ y_{n} \end{bmatrix} = \begin{bmatrix}C & 0 & \dots  & & 0 \\\ 0 & C & 0 & \dots & 0  \\\ 0 & 0 & C & \dots & 0 \\\ \vdots & & & \ddots & 0 \\\ 0 & \dots & 0 & 0 & C \end{bmatrix}\begin{bmatrix}x_{1}\\\ x_{2} \\\ x_{3} \\\ \vdots  \\\ x_{n} \end{bmatrix} \tag{7}
\end{align}
$$

可以将方程（6）和（7）组合为以下形式：

$$
\begin{align}
X = Fx_{0} + GU +SW, Y=HX \tag{8}
\end{align}
$$

这种形式与原始状态空间方程（1）类似，但引入了新的矩阵：状态转移矩阵 $F$、控制矩阵 $G$、扰动矩阵 $W$ 和测量矩阵 $H$。其中，$X$ 表示预测状态，即 $\begin{bmatrix}x_{1} & x_{2} & \dots & x_{n} \end{bmatrix}^{T}$。

由于 $G$、$S$、$W$ 和 $H$ 已知，可以将未来 $n$ 步的输出行为 $Y$ 表示为输入 $U$ 的函数。这样就能计算控制输入 $U$，使 $Y(U)$ 跟随目标轨迹 $Y_{ref}$。

下一步是定义代价函数。代价函数通常采用以下二次形式：

$$
\begin{align}
J = (Y - Y_{ref})^{T}Q(Y - Y_{ref}) + (U - U_{ref})^{T}R(U - U_{ref}) \tag{9}
\end{align}
$$

其中，$U_{ref}$ 是目标输入或稳态输入，也是系统针对 $U$ 进行线性化时的参考输入。

该代价函数与 LQR 控制器相同。$J$ 的第一项惩罚与参考轨迹的偏差，第二项惩罚与参考（或稳态）控制轨迹的偏差。$Q$ 和 $R$ 是代价权重矩阵，分别具有正定和半正定性质。

注意：有时会使用 $U_{ref}=0$，但这可能意味着即使车辆正在转弯，也要求转向角设为 $0$。因此，此处使用 $U_{ref}$ 进行说明。$U_{ref}$ 可以根据目标轨迹的曲率或稳态分析预先计算。

由于轨迹输出现为 $Y=Y(x_{0}, U)$，代价函数仅取决于 U 和初始状态条件，即 $J=J(x_{0}, U)$。下面求使其最小的 $U$。

将方程（8）代入方程（9），并按 $U$ 整理。

$$
\begin{align}
J(U) &= (H(Fx_{0}+GU+SW)-Y_{ref})^{T}Q(H(Fx_{0}+GU+SW)-Y_{ref})+(U-U_{ref})^{T}R(U-U_{ref}) \\\
& =U^{T}(G^{T}H^{T}QHG+R)U+2\lbrace\{(H(Fx_{0}+SW)-Y_{ref})^{T}QHG-U_{ref}^{T}R\rbrace\}U +(\rm{constant}) \tag{10}
\end{align}
$$

此方程是关于 $U$ 的二次型（即 $U^{T}AU+B^{T}U$）。

由于 $Q$ 和 $R$ 的正定与半正定要求，$U$ 的二次项系数矩阵 $G^{T}C^{T}QCG+R$ 为正定矩阵。因此，代价函数是关于 U 的凸二次函数，可以通过凸优化高效求解。

<a id="apply-to-vehicle-path-following-problem-nonlinear-problem"></a>

### 应用于车辆路径跟踪问题（非线性问题）

基于车辆运动学模型的路径跟踪问题是非线性的，因此不能直接使用上一节中的线性 MPC 方法。处理非线性的方法有多种，例如使用非线性优化求解器。此处沿参考轨迹对非线性车辆模型进行线性化，将其转换为线性时变模型。

对于非线性车辆运动学模型，离散时间更新方程如下：

$$
\begin{align}
x_{k+1} &= x_{k} + v\cos\theta_{k} \text{d}t \\\
y_{k+1} &= y_{k} + v\sin\theta_{k} \text{d}t \\\
\theta_{k+1} &= \theta_{k} + \frac{v\tan\delta_{k}}{L} \text{d}t \tag{11} \\\
\delta_{k+1} &= \delta_{k} - \tau^{-1}\left(\delta_{k}-\delta_{des}\right)\text{d}t
\end{align}
$$

![车辆运动学](./image/vehicle_kinematics.png)

车辆参考点位于后轴中心，所有状态均在此点测量。状态、参数和控制变量如下表所示。

| 符号 | 含义 |
| -------------- | ------------------------------------------------------------- |
| $v$ | 后轴中心测得的车速 |
| $\theta$ | 全局坐标系中的偏航角（航向角） |
| $\delta$ | 车辆转向角 |
| $\delta_{des}$ | 车辆目标转向角 |
| $L$ | 车辆轴距（后轴与前轴之间的距离） |
| $\tau$ | 一阶转向动力学的时间常数 |

本例假设 MPC 只生成转向控制，车辆沿轨迹的速度由轨迹生成器提供。

车辆运动学模型的离散更新方程包含 sin 和 cos 等三角函数，车辆坐标 $x$、$y$ 及偏航角均位于全局坐标系。在路径跟踪应用中，通常将模型改写为误差动力学，把控制问题转换为目标值为零（零误差）的调节问题。

![车辆误差运动学](./image/vehicle_error_kinematics.png)

在下面推导线性方程时，采用小角度假设。基于非线性动力学，并省略纵向坐标 $x$，可得到以下方程组：

$$
\begin{align}
y_{k+1} &= y_{k} + v\sin\theta_{k} \text{d}t \\\
\theta_{k+1} &= \theta_{k} + \frac{v\tan\delta_{k}}{L} \text{d}t - \kappa_{r}v\cos\theta_{k}\text{d}t
\tag{12} \\\
\delta_{k+1} &= \delta_{k} - \tau^{-1}\left(\delta_{k}-\delta_{des}\right)\text{d}t
\end{align}
$$

其中，$\kappa_{r}\left(s\right)$ 是以弧长为参数的轨迹曲率。

更新方程中有三个表达式需要线性近似：横向偏差（或横向坐标）$y$、航向角（或航向角误差）$\theta$，以及转向角 $\delta$。可以对航向角 $\theta$ 采用小角度假设。

在路径跟踪问题中，轨迹曲率 $\kappa_{r}$ 预先已知。在较低速度下，可以通过阿克曼公式近似参考转向角 $\theta_{r}$（对应前文的 $U_{ref}$）。阿克曼转向关系可写为：

$$
\begin{align}
\delta_{r} = \arctan\left(L\kappa_{r}\right)
\end{align}
$$

车辆沿弯曲路径行驶时，转向角 $\delta$ 应接近 $\delta_{r}$。因此，$\delta$ 可以表示为：

$$
\begin{align}
\delta = \delta_{r} + \Delta \delta, \Delta\delta \ll 1
\end{align}
$$

将此式代入方程（12），并假设 $\Delta\delta$ 很小，进行近似。

$$
\begin{align}
\tan\delta &\simeq \tan\delta_{r} + \frac{\text{d}\tan\delta}{\text{d}\delta} \Biggm|_{\delta=\delta_{r}}\Delta\delta \\\
&= \tan \delta_{r} + \frac{1}{\cos^{2}\delta_{r}}\Delta\delta \\\
&= \tan \delta_{r} + \frac{1}{\cos^{2}\delta_{r}}\left(\delta-\delta_{r}\right) \\\
&= \tan \delta_{r} - \frac{\delta_{r}}{\cos^{2}\delta_{r}} + \frac{1}{\cos^{2}\delta_{r}}\delta
\end{align}
$$

由此，$\theta_{k+1}$ 可表示为：

$$
\begin{align}
\theta_{k+1} &= \theta_{k} + \frac{v\tan\delta_{k}}{L}\text{d}t - \kappa_{r}v\cos\delta_{k}\text{d}t \\\
&\simeq \theta_{k} + \frac{v}{L}\text{d}t\left(\tan\delta_{r} - \frac{\delta_{r}}{\cos^{2}\delta_{r}} + \frac{1}{\cos^{2}\delta_{r}}\delta_{k} \right) - \kappa_{r}v\text{d}t \\\
&= \theta_{k} + \frac{v}{L}\text{d}t\left(L\kappa_{r} - \frac{\delta_{r}}{\cos^{2}\delta_{r}} + \frac{1}{\cos^{2}\delta_{r}}\delta_{k} \right) - \kappa_{r}v\text{d}t \\\
&= \theta_{k} + \frac{v}{L}\frac{\text{d}t}{\cos^{2}\delta_{r}}\delta_{k} - \frac{v}{L}\frac{\delta_{r}\text{d}t}{\cos^{2}\delta_{r}}
\end{align}
$$

最终，线性化后的时变模型方程为：

$$
\begin{align}
\begin{bmatrix} y_{k+1} \\\ \theta_{k+1} \\\ \delta_{k+1} \end{bmatrix} = \begin{bmatrix} 1 & v\text{d}t & 0 \\\ 0 & 1 & \frac{v}{L}\frac{\text{d}t}{\cos^{2}\delta_{r}} \\\ 0 & 0 & 1 - \tau^{-1}\text{d}t \end{bmatrix} \begin{bmatrix} y_{k} \\\ \theta_{k} \\\ \delta_{k} \end{bmatrix} + \begin{bmatrix} 0 \\\ 0 \\\ \tau^{-1}\text{d}t \end{bmatrix}\delta_{des} + \begin{bmatrix} 0 \\\ -\frac{v}{L}\frac{\delta_{r}\text{d}t}{\cos^{2}\delta_{r}} \\\ 0 \end{bmatrix}
\end{align}
$$

此方程与线性 MPC 假设中的方程（1）形式相同，但矩阵 $A$、$B$ 和 $w$ 随坐标变换而变化。为明确这一点，将整个方程写为：

$$
\begin{align}
x_{k+1} = A_{k}x_{k} + B_{k}u_{k}+w_{k}
\end{align}
$$

与方程（1）相比，$A \rightarrow A_{k}$。这意味着 $A$ 矩阵是在 $k$ 步之后（即 $k* \text{d}t$ 秒后）轨迹附近的线性近似；如果轨迹预先已知，就可以求得该矩阵。

使用此方程，按与（2）至（6）相同的方式写出更新方程：

$$
\begin{align}
\begin{bmatrix}
 x_{1} \\\ x_{2} \\\ x_{3} \\\ \vdots \\\ x_{n}
\end{bmatrix}
= \begin{bmatrix}
 A_{1} \\\ A_{1}A_{0} \\\ A_{2}A_{1}A_{0} \\\ \vdots \\\ \prod_{i=0}^{n-1} A_{k}
\end{bmatrix}
x_{0} +
\begin{bmatrix}
 B_{0} & 0 & \dots & & 0 \\\ A_{1}B_{0} & B_{1} & 0 & \dots & 0 \\\ A_{2}A_{1}B_{0} & A_{2}B_{1} & B_{2} & \dots & 0 \\\ \vdots & \vdots & &\ddots & 0 \\\ \prod_{i=1}^{n-1} A_{k}B_{0} & \prod_{i=2}^{n-1} A_{k}B_{1} & \dots & A_{n-1}B_{n-1} & B_{n-1}
\end{bmatrix}
\begin{bmatrix}
 u_{0} \\\ u_{1} \\\ u_{2} \\\ \vdots \\\ u_{n-1}
\end{bmatrix} +
\begin{bmatrix}
I & 0 & \dots & & 0 \\\ A_{1} & I & 0 & \dots & 0 \\\ A_{2}A_{1} & A_{2} & I & \dots & 0 \\\ \vdots & \vdots & &\ddots & 0 \\\ \prod_{i=1}^{n-1} A_{k} & \prod_{i=2}^{n-1} A_{k} & \dots & A_{n-1} & I
\end{bmatrix}
\begin{bmatrix}
 w_{0} \\\ w_{1} \\\ w_{2} \\\ \vdots \\\ w_{n-1}
\end{bmatrix}
\end{align}
$$

由于其形式与方程（6）相同，因此与前一节模型一样，可以应用凸优化。

<a id="the-cost-functions-and-constraints"></a>

## 代价函数与约束

本节详细说明如何设置代价函数和约束条件。

<a id="the-cost-function"></a>

### 代价函数

<a id="weight-for-error-and-input"></a>

#### 误差和输入的权重

MPC 的状态和控制权重以类似 LQR（9）的方式出现在代价函数中。对于前述车辆路径跟踪问题，如果 C 是单位矩阵，则输出为 $y = x = \left[y, \theta, \delta\right]$。（为避免与 y 方向偏差混淆，此处使用 $e$ 表示横向偏差。）

例如，对于预测步数为 $n=2$ 的系统，可以如下确定评价函数的权重矩阵 $Q_{1}$。

$$
\begin{align}
Q_{1} = \begin{bmatrix} q_{e} & 0 & 0 & 0 & 0& 0 \\\ 0 & q_{\theta} & 0 & 0 & 0 & 0 \\\ 0 & 0 & 0 & 0 & 0 & 0 \\\ 0 & 0 & 0 & q_{e} & 0 & 0 \\\ 0 & 0 & 0 & 0 & q_{\theta} & 0 \\\ 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
\end{align}
$$

当 $n=2$ 时，代价函数（9）的第一项如下（将 $Y_{ref}$ 设为 $0$）：

$$
\begin{align}
q_{e}\left(e_{0}^{2} + e_{1}^{2} \right) + q_{\theta}\left(\theta_{0}^{2} + \theta_{1}^{2} \right)
\end{align}
$$

这表明 $q_{e}$ 是横向误差权重，$q$ 是角度误差权重。本例中，$q_{e}$ 对横向跟踪误差的作用类似比例 P 增益，$q_{\theta}$ 类似微分 D 增益。这些因素（包括 R）之间的平衡需要通过实际实验确定。

<a id="weight-for-non-diagonal-term"></a>

#### 非对角项的权重

MPC 可以在计算中处理非对角项（只要最终矩阵为正定矩阵）。

例如，对于 $n=2$ 的系统，将 $Q_{2}$ 写为：

$$
\begin{align}
Q_{2} = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 \\\ 0 & 0 & 0 & 0 & 0 & 0 \\\ 0 & 0 & q_{d} & 0 & 0 & -q_{d} \\\ 0 & 0 & 0 & 0 & 0 & 0 \\\ 0 & 0 & 0 & 0 & 0 & 0 \\\ 0 & 0 & -q_{d} & 0 & 0 & q_{d} \end{bmatrix}
\end{align}
$$

使用 $Q_{2}$ 展开评价函数的第一项：

$$
\begin{align}
q_{d}\left(\delta_{0}^{2} -2\delta_{0}\delta_{1} + \delta_{1}^{2} \right) = q_{d}\left( \delta_{0} - \delta_{1}\right)^{2}
\end{align}
$$

$q_{d}$ 对 $\delta$ 的变化量加权，可防止轮胎快速转动。添加这一项后，系统便可权衡跟踪精度与方向盘转角变化。

由于权重矩阵可以线性相加，最终权重可设为 $Q = Q_{1} + Q_{2}$。

此外，MPC 在一段时间范围内进行优化，因此也可以在优化中考虑时变权重。

<a id="constraints"></a>

### 约束

<a id="input-constraint"></a>

#### 输入约束

MPC 控制器的主要优点是能够处理状态或输入约束。约束可表示为区间约束，例如“轮胎转角必须在 ±30 度以内”，写成以下形式：

$$
\begin{align}
u_{min} < u < u_{max}
\end{align}
$$

在线性 MPC 应用中，约束必须是线性且凸的。

<a id="constraints-on-the-derivative-of-the-input"></a>

#### 输入导数的约束

还可以对输入的变化施加约束。转向角的导数为 $\dot{u}$，其区间约束为：

$$
\begin{align}
\dot u_{min} < \dot u < \dot u_{max}
\end{align}
$$

将 $\dot{u}$ 离散化为 $\left(u_{k} - u_{k-1}\right)/\text{d}t$，并将两边乘以 dt，所得约束是线性且凸的：

$$
\begin{align}
\dot u_{min}\text{d}t < u_{k} - u_{k-1} < \dot u_{max}\text{d}t
\end{align}
$$

在预测或控制时域内，例如设 $n=3$：

$$
\begin{align}
\dot u_{min}\text{d}t < u_{1} - u_{0} < \dot u_{max}\text{d}t \\\
\dot u_{min}\text{d}t < u_{2} - u_{1} < \dot u_{max}\text{d}t
\end{align}
$$

将不等号方向统一：

$$
\begin{align}
u_{1} - u_{0} &< \dot u_{max}\text{d}t \\\
- u_{1} + u_{0} &< -\dot u_{min}\text{d}t \\\
u_{2} - u_{1} &< \dot u_{max}\text{d}t \\\
- u_{2} + u_{1} &< - \dot u_{min}\text{d}t
\end{align}
$$

可将得到的约束方程写为以下矩阵形式：

$$
\begin{align}
Ax \leq b
\end{align}
$$

因此，将此不等式整理为上述形式，就能以一阶近似方式加入对 $\dot{u}$ 的约束。

$$
\begin{align}
\begin{bmatrix} -1 & 1 & 0 \\\ 1 & -1 & 0 \\\ 0 & -1 & 1 \\\ 0 & 1 & -1 \end{bmatrix}\begin{bmatrix} u_{0} \\\ u_{1} \\\ u_{2} \end{bmatrix} \leq \begin{bmatrix} \dot u_{max}\text{d}t \\\ -\dot u_{min}\text{d}t \\\ \dot u_{max}\text{d}t \\\ -\dot u_{min}\text{d}t \end{bmatrix}
\end{align}
$$
