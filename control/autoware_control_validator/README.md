<a id="control-validator"></a>

# 控制校验器

`control_validator` 模块用于检查控制组件输出的有效性。可以通过 `/diagnostics` 话题查看校验状态。

![控制校验器](./image/control_validator.drawio.svg)

<a id="supported-features"></a>

## 支持的功能

校验支持以下功能，其阈值可以通过参数设置。
以下所列功能不一定与最新实现完全一致。

| 说明 | 参数 | 诊断公式 |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | :---------------------------------------------------: |
| 速度反向：实测速度与目标速度符号不同。 | 实测速度 $v$、目标速度 $\hat{v}$、速度参数 $c$ | $v \hat{v} < 0, \quad \lvert v \rvert > c$ |
| 超速：实测速率显著超过目标速率。 | 实测速度 $v$、目标速度 $\hat{v}$、比例参数 $r$、偏移参数 $c$ | $\lvert v \rvert > (1 + r) \lvert \hat{v} \rvert + c$ |
| 越过停车点估计：估计即使按假设减速度减速，是否仍会越过停车点。 | 假设减速度、假设延迟 | |

- **横向加加速度**：横向加加速度超过配置阈值时判定无效。校验使用车辆速度和转向角速度计算产生的横向加加速度，计算时假设车辆匀速（加速度为零）。
- **参考轨迹与预测轨迹之间的偏差检查**：预测轨迹与参考轨迹之间的最大偏差大于给定阈值时判定无效。
- **偏航角偏差**：自车偏航角与最近轨迹点的插值偏航角之间的差值大于给定阈值时判定无效。
  - 实现了两个阈值，分别用于触发警告诊断和错误诊断。

![轨迹偏差](./image/trajectory_deviation.drawio.svg)

<a id="inputsoutputs"></a>

## 输入与输出

<a id="inputs"></a>

### 输入

`control_validator` 接收以下输入：

| 名称 | 类型 | 说明 |
| ------------------------------ | --------------------------------- | ------------------------------------------------------------------------------ |
| `~/input/kinematics` | nav_msgs/Odometry | 自车位姿与 twist |
| `~/input/reference_trajectory` | autoware_planning_msgs/Trajectory | 规划模块输出、待跟踪的参考轨迹 |
| `~/input/predicted_trajectory` | autoware_planning_msgs/Trajectory | 控制模块输出的预测轨迹 |

<a id="outputs"></a>

### 输出

输出如下：

| 名称 | 类型 | 说明 |
| ---------------------------- | ---------------------------------------- | ------------------------------------------------------------------------- |
| `~/output/validation_status` | control_validator/ControlValidatorStatus | 校验器状态，说明轨迹有效或无效的原因 |
| `/diagnostics` | diagnostic_msgs/DiagnosticStatus | 用于报告错误的诊断信息 |

<a id="parameters"></a>

## 参数

可以为 `control_validator` 设置以下参数：

<a id="system-parameters"></a>

### 系统参数

| 名称 | 类型 | 说明 | 默认值 |
| :--------------------------- | :--- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `publish_diag` | bool | 若为 true，则发布诊断消息。 | true |
| `diag_error_count_threshold` | int | 连续无效轨迹的数量超过此阈值时，将诊断设为 ERROR。（例如，阈值为 1 时，即使某条轨迹无效，只要下一条轨迹有效，诊断就不会设为 ERROR。） | true |
| `display_on_terminal` | bool | 在终端显示错误消息 | true |

<a id="algorithm-parameters"></a>

### 算法参数

<a id="thresholds"></a>

#### 阈值

当指标超过以下阈值时，输入轨迹被判定为无效。

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------------------------- | :----- | :---------------------------------------------------------------------------------------------------------------- | :------------ |
| `thresholds.max_distance_deviation` | double | 预测路径与参考轨迹之间最大距离偏差的无效判定阈值 [m] | 1.0 |
| `thresholds.lateral_jerk` | double | 转向角速度校验中横向加加速度的无效判定阈值 [m/s^3] | 10.0 |
| `thresholds.rolling_back_velocity` | double | 校验车辆速度的速度阈值 [m/s] | 0.5 |
| `thresholds.over_velocity_offset` | double | 校验车辆速度的速度偏移阈值 [m/s] | 2.0 |
| `thresholds.over_velocity_ratio` | double | 校验车辆速度的比例阈值 [*] | 0.2 |
| `thresholds.overrun_stop_point_dist` | double | 越过停车点的距离阈值 [m] | 0.8 |
| `thresholds.acc_error_offset` | double | 校验车辆加速度的比例阈值 [*] | 0.8 |
| `thresholds.acc_error_scale` | double | 校验车辆加速度的加速度阈值 [m] | 0.2 |
| `thresholds.will_overrun_stop_point_dist` | double | 越过停车点的距离阈值 [m] | 1.0 |
| `thresholds.assumed_limit_acc` | double | 越过停车点估计使用的假设加速度 [m] | 5.0 |
| `thresholds.assumed_delay_time` | double | 越过停车点估计使用的假设延迟 [m] | 0.2 |
| `thresholds.yaw_deviation_error` | double | 校验车辆偏航角相对于最近轨迹偏航角的角度阈值 [rad] | 1.0 |
| `thresholds.yaw_deviation_warn` | double | 触发 WARN 诊断的角度阈值 [rad] | 0.5 |
| `over_velocity.vel_lpf_gain` | double | 过滤车辆速度和目标速度的低通滤波器增益 [*]（采样周期为 30msec 时，时间常数为 2.0s） | 0.985 |
