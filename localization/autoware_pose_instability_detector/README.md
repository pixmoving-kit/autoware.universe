# autoware_pose_instability_detector

`pose_instability_detector` 节点用于监控扩展卡尔曼滤波器（EKF）的输出话题 `/localization/kinematic_state` 的稳定性。

此节点定期触发定时器回调，比较以下两个位姿：

- 以 `timer_period` 秒前取得的 `/localization/kinematic_state` 位姿为起点，通过航位推算得到的位姿。
- `/localization/kinematic_state` 的最新位姿。

比较结果随后输出到 `/diagnostics` 话题。

![概述](./media/pose_instability_detector_overview.png)

![rqt 运行状态监控](./media/rqt_runtime_monitor.png)

如果此节点向 `/diagnostics` 输出 ERROR 消息，表示 EKF 输出与速度积分得到的结果有明显差异。
换言之，ERROR 输出表示车辆已移动到根据速度值预测的范围之外。
这一差异说明估计位姿或输入速度可能存在问题。

下图概述了处理过程：

![处理过程](./media/pose_instabilty_detector_procedure.svg)

<a id="dead-reckoning-algorithm"></a>

## 航位推算算法

航位推算是根据车辆此前的位置和速度估计当前位置的方法。
航位推算步骤如下：

1. 从 `/input/twist` 话题取得所需的速度值。
2. 对速度积分，计算位姿变化。
3. 将位姿变化应用于此前的位姿，得到当前位姿。

<a id="collecting-twist-values"></a>

### 收集速度值

`pose_instability_detector` 节点从 `~/input/twist` 话题收集速度值，以进行航位推算。
理想情况下，`pose_instability_detector` 需要此前位姿与当前位姿之间的速度值。
因此，`pose_instability_detector` 会截取速度缓冲区，并通过插值和外推获取所需时刻的速度值。

![截取所需速度数据](./media/how_to_snip_twist.png)

<a id="linear-transition-and-angular-transition"></a>

### 平移变化与旋转变化

收集速度值后，节点据此计算平移变化和旋转变化，并将其叠加到此前位姿上。

<a id="threshold-definition"></a>

## 阈值定义

`pose_instability_detector` 节点将航位推算得到的位姿与 EKF 的最新输出位姿进行比较。
理想情况下，两者应完全相同；但实际上，由于速度值和位姿观测存在误差，两者会有差异。
如果两个位姿差异明显，且差值的绝对值超过阈值，节点会向 `/diagnostics` 话题输出 WARN 消息。
通过六个阈值（x、y、z、roll、pitch 和 yaw）判断位姿差异是否明显，具体定义见以下小节。

### `diff_position_x`

此阈值检查两个位姿在纵向轴上的差异，判断车辆是否超出预期误差范围。
此阈值为“速度比例因子误差引起的最大纵向误差”与“位姿估计误差容限”之和。

$$
\tau_x = v_{\rm max}\frac{\beta_v}{100} \Delta t + \epsilon_x\\
$$

| 符号 | 说明 | 单位 |
| ------------- | -------------------------------------------------------------------------------- | ----- |
| $\tau_x$ | 纵向轴差异阈值 | $m$ |
| $v_{\rm max}$ | 最大速度 | $m/s$ |
| $\beta_v$ | 最大速度的比例因子容限 | $\%$ |
| $\Delta t$ | 时间间隔 | $s$ |
| $\epsilon_x$ | 位姿估计器（例如 ndt_scan_matcher）在纵向轴上的误差容限 | $m$ |

<a id="diff_position_y-and-diff_position_z"></a>

### `diff_position_y` 和 `diff_position_z`

这些阈值检查两个位姿在横向和垂直轴上的差异，判断车辆是否超出预期误差范围。
`pose_instability_detector` 计算车辆可能到达的范围，并取得标称航位推算位姿与极限位姿之间的最大差异。

![横向阈值计算](./media/lateral_threshold_calculation.png)

此外，`pose_instability_detector` 节点还会考虑位姿估计误差容限，以确定阈值。

$$
\tau_y = l + \epsilon_y
$$

| 符号 | 说明 | 单位 |
| ------------ | ----------------------------------------------------------------------------------------------- | ---- |
| $\tau_y$ | 横向轴差异阈值 | $m$ |
| $l$ | 上图所示的最大横向距离（计算方法见附录） | $m$ |
| $\epsilon_y$ | 位姿估计器（例如 ndt_scan_matcher）在横向轴上的误差容限 | $m$ |

注意，`pose_instability_detector` 为垂直轴设置与横向轴相同的阈值，仅位姿估计器误差容限不同。

<a id="diff_angle_x-diff_angle_y-and-diff_angle_z"></a>

### `diff_angle_x`、`diff_angle_y` 和 `diff_angle_z`

这些阈值检查两个位姿在滚转角、俯仰角和横摆角上的差异。
此阈值为“速度比例因子误差和偏置误差引起的最大角度误差”与“位姿估计误差容限”之和。

$$
\tau_\phi = \tau_\theta = \tau_\psi = \left(\omega_{\rm max}\frac{\beta_\omega}{100} + b \right) \Delta t + \epsilon_\psi
$$

| 符号 | 说明 | 单位 |
| ------------------ | ------------------------------------------------------------------------ | ------------- |
| $\tau_\phi$ | 滚转角差异阈值 | ${\rm rad}$ |
| $\tau_\theta$ | 俯仰角差异阈值 | ${\rm rad}$ |
| $\tau_\psi$ | 横摆角差异阈值 | ${\rm rad}$ |
| $\omega_{\rm max}$ | 最大角速度 | ${\rm rad}/s$ |
| $\beta_\omega$ | 最大角速度的比例因子容限 | $\%$ |
| $b$ | 角速度偏置容限 | ${\rm rad}/s$ |
| $\Delta t$ | 时间间隔 | $s$ |
| $\epsilon_\psi$ | 位姿估计器（例如 ndt_scan_matcher）在横摆角上的误差容限 | ${\rm rad}$ |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("localization/autoware_pose_instability_detector/schema/pose_instability_detector.schema.json") }}

<a id="input"></a>

## 输入

| 名称 | 类型 | 说明 |
| ------------------ | ---------------------------------------------- | --------------------- |
| `~/input/odometry` | nav_msgs::msg::Odometry | EKF 估计的位姿 |
| `~/input/twist` | geometry_msgs::msg::TwistWithCovarianceStamped | 速度 |

<a id="output"></a>

## 输出

| 名称 | 类型 | 说明 |
| ------------------- | ------------------------------------- | ----------- |
| `~/debug/diff_pose` | geometry_msgs::msg::PoseStamped | diff_pose |
| `/diagnostics` | diagnostic_msgs::msg::DiagnosticArray | 诊断信息 |

<a id="appendix"></a>

## 附录

计算最大横向距离 $l$ 时，`pose_instability_detector` 节点会估计以下位姿。

| 位姿 | 前进速度 $v$ | 角速度 $\omega$ |
| ------------------------------- | ------------------------------------------------ | -------------------------------------------------------------- |
| 标称航位推算位姿 | $v_{\rm max}$ | $\omega_{\rm max}$ |
| 角点 A 的航位推算位姿 | $\left(1+\frac{\beta_v}{100}\right) v_{\rm max}$ | $\left(1+\frac{\beta_\omega}{100}\right) \omega_{\rm max} + b$ |
| 角点 B 的航位推算位姿 | $\left(1-\frac{\beta_v}{100}\right) v_{\rm max}$ | $\left(1+\frac{\beta_\omega}{100}\right) \omega_{\rm max} + b$ |
| 角点 C 的航位推算位姿 | $\left(1-\frac{\beta_v}{100}\right) v_{\rm max}$ | $\left(1-\frac{\beta_\omega}{100}\right) \omega_{\rm max} - b$ |
| 角点 D 的航位推算位姿 | $\left(1+\frac{\beta_v}{100}\right) v_{\rm max}$ | $\left(1-\frac{\beta_\omega}{100}\right) \omega_{\rm max} - b$ |

给定前进速度 $v$ 和角速度 $\omega$，相对于此前位姿的二维理论变化量计算如下：

$$
\begin{align*}
\left[
    \begin{matrix}
    \Delta x\\
    \Delta y
    \end{matrix}
\right]
&=
\left[
    \begin{matrix}
    \int_{0}^{\Delta t} v \cos(\omega t) dt\\
    \int_{0}^{\Delta t} v \sin(\omega t) dt
    \end{matrix}
\right]
\\
&=
\left[
    \begin{matrix}
    \frac{v}{\omega} \sin(\omega \Delta t)\\
    \frac{v}{\omega} \left(1 - \cos(\omega \Delta t)\right)
    \end{matrix}
\right]
\end{align*}
$$

对每个角点计算这一变化量，并比较标称航位推算位姿与各角点位姿之间的距离，得到横向距离 $l$ 的最大值。
