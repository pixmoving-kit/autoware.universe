<a id="mpc-lateral-controller"></a>

# MPC 横向控制器

本文是横向控制器节点的设计文档，
该节点位于 `autoware_trajectory_follower_node` 功能包中。

<a id="purpose-use-cases"></a>

## 目的与使用场景

<!-- Required -->
<!-- Things to consider:
    - Why did we implement this feature? -->

本节点用于生成横向控制命令（转向角和转向角速度），
以跟踪路径。

<a id="design"></a>

## 设计

<!-- Required -->
<!-- Things to consider:
    - How does it work? -->

本节点采用线性模型预测控制（MPC）实现精确的路径跟踪。
MPC 使用车辆模型模拟控制命令所产生的轨迹。
控制命令的优化被表述为二次规划（QP）问题。

实现了不同的车辆模型：

- kinematics：带有转向一阶延迟的自行车运动学模型。
- kinematics_no_delay：不带转向延迟的自行车运动学模型。
- dynamics：考虑侧偏角的自行车动力学模型。
  默认使用运动学模型。更多信息请参阅参考文献 [1]。

优化使用二次规划（QP）求解器，目前实现了两种选项：

<!-- cspell: ignore ADMM -->

- unconstraint_fast：使用 eigen，通过最小二乘法求解无约束 QP。
- [osqp](https://osqp.org/)：运行[以下 ADMM](https://web.stanford.edu/~boyd/papers/admm_distr_stats.html)
  算法（更多信息请参阅
  [引用 OSQP](https://web.stanford.edu/~boyd/papers/admm_distr_stats.html) 一节中的相关论文）：

<a id="trajectory-reference-mode"></a>

### 轨迹参考模式

`trajectory_reference_mode` 参数选择 MPC 跟踪参考轨迹的方式：

- `spatial`（默认）：按距离索引参考轨迹，各预测点的时间步长
  根据距离和速度计算。
- `temporal`：直接使用参考轨迹的 `time_from_start` 字段，自车
  在轨迹上的位置根据预测时间估计（基于上一次最近时间和控制周期），
  并用当前位姿观测到的最近时间进行修正。

此参数通常在 `autoware_trajectory_follower_node` 层声明一次（参见
[trajectory_follower_node.param.yaml](../autoware_trajectory_follower_node/param/trajectory_follower_node.param.yaml)），
并与纵向控制器共享。此处也声明了相同的 `spatial`
默认值，以便本节点单独运行时（例如测试中）仍可工作。默认值
保持现有的空间参考行为不变。

<a id="filtering"></a>

### 滤波

为了有效降低噪声，需要进行滤波。
使用[巴特沃斯滤波器](https://en.wikipedia.org/wiki/Butterworth_filter)处理作为 MPC 输入的偏航误差和横向误差，并对输出转向角进行滤波。
只要降噪性能足够好，也可以考虑其他
滤波方法。
例如，移动平均滤波器并不适用，其结果可能比完全不进行
滤波还差。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

<!-- Required -->

如果参考轨迹的第一个点位于当前自车位置或其前方，则跟踪会不准确。

<a id="inputs-outputs-api"></a>

## 输入、输出与 API

<!-- Required -->
<!-- Things to consider:
    - How do you use the package / API? -->

<a id="inputs"></a>

### 输入

由 [controller_node](../autoware_trajectory_follower_node/README.md) 设置以下内容：

- `autoware_planning_msgs/Trajectory`：需要跟踪的参考轨迹。
- `nav_msgs/Odometry`：当前里程计。
- `autoware_vehicle_msgs/SteeringReport`：当前转向状态。

<a id="outputs"></a>

### 输出

向控制器节点返回包含以下内容的 LateralOutput：

- `autoware_control_msgs/Lateral`
- LateralSyncData
  - 转向角收敛状态。

发布以下消息。

| 名称 | 类型 | 说明 |
| ------------------------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `~/output/predicted_trajectory` | autoware_planning_msgs::Trajectory | MPC 计算的预测轨迹。当控制器发生紧急情况（例如偏离规划轨迹过大）时，轨迹为空。 |

<a id="mpc-class"></a>

### MPC 类

`MPC` 类（定义于 `mpc.hpp`）提供 MPC 算法接口。
设置车辆模型、QP 求解器及待跟踪的参考轨迹后
（使用 `setVehicleModel()`、`setQPSolver()`、`setReferenceTrajectory()`），
即可向 `calculateMPC()` 提供当前转向、速度和位姿，计算横向控制命令。

<a id="parameter-description"></a>

### 参数说明

`param/lateral_controller_defaults.param.yaml` 中的默认参数针对
AutonomouStuff Lexus RX 450h 以低于 40 km/h 的速度行驶进行了调整。

<a id="system"></a>

#### 系统

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/system.json") }}

<a id="path-smoothing"></a>

#### 路径平滑

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/path_smoothing.json") }}

<a id="trajectory-extending"></a>

#### 轨迹延长

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/trajectory_extending.json") }}

<a id="mpc-optimization"></a>

#### MPC 优化

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/mpc_optimization.json") }}

<a id="vehicle-model"></a>

#### 车辆模型

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/vehicle_model.json") }}

<a id="lowpass-filter-for-noise-reduction"></a>

#### 用于降噪的低通滤波器

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/lowpass_filter.json") }}

<a id="stop-state"></a>

#### 停止状态

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/stop_state.json") }}

（stop_state_entry_ego_speed 和 stop_state_entry_target_speed）为防止不必要的转向动作，在停止状态下将转向命令固定为上一次的值。

<a id="steer-offset"></a>

#### 转向偏置

定义于 `steering_offset` 命名空间中。此逻辑尽可能保持简单，并使用最少的设计参数。

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/steering_offset.json") }}

<a id="for-dynamics-model-wip"></a>

##### 用于动力学模型（开发中）

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/dynamics_model.json") }}

<a id="publish-debug-predicted-trajectory-in-frenet-coordinate"></a>

##### 在 Frenet 坐标系中发布调试预测轨迹

{{ json_to_markdown("control/autoware_mpc_lateral_controller/schema/sub/debug_publish.json") }}

<a id="debug"></a>

#### 调试

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------- | :------ | :-------------------------------------------------------------------------------- | :------------ |
| publish_debug_trajectories | boolean | 发布预测轨迹及重采样后的参考轨迹，用于调试 | true |

当 `trajectory_reference_mode` 为 `temporal` 时，`~/debug/nearest_info` 话题和
诊断消息还包含以下字段，用于说明如何确定
当前轨迹上的最近时间：

| 名称 | 说明 |
| :------------------------ | :----------------------------------------------------------------------- |
| temporal_predicted_time | 根据上一次最近时间和控制周期预测出的最近时间 |
| temporal_observed_time | 从当前自车位姿观测到的最近时间（如果找到） |
| temporal_fused_time | 融合预测时间和观测时间后实际使用的最近时间 |
| temporal_observation_used | 是否找到并使用了观测到的最近时间：是为 1.0，否为 0.0 |
| temporal_window_min | 搜索观测时间所用时间窗口的下界 |
| temporal_window_max | 搜索观测时间所用时间窗口的上界 |

<a id="how-to-tune-mpc-parameters"></a>

### 如何调整 MPC 参数

<a id="set-kinematics-information"></a>

#### 设置运动学信息

首先，应正确设置车辆运动学参数，包括表示前后轮之间距离的 `wheelbase`，以及表示最大轮胎转向角的 `max_steering_angle`。这些参数应在 `vehicle_info.param.yaml` 中设置。

<a id="set-dynamics-information"></a>

#### 设置动力学信息

接下来，需要正确设置动力学模型参数，包括转向动力学的时间常数 `steering_tau` 和时间延迟 `steering_delay`，以及速度动力学的最大加速度 `mpc_acceleration_limit` 和时间常数 `mpc_velocity_time_constant`。

<a id="confirmation-of-the-input-information"></a>

#### 确认输入信息

确保输入信息准确也很重要。需要后轮轴中心的速度 [m/s]、轮胎转向角 [rad] 等信息。已有多次因输入信息错误导致性能下降的报告。例如，轮胎半径的意外差异可能使车速出现偏差，转向传动比或中点偏差可能导致轮胎转角测量不准。建议比较多个传感器的信息（例如车速积分与 GNSS 位置、转向角与 IMU 角速度），确保输入 MPC 的信息正确。

<a id="mpc-weight-tuning"></a>

#### MPC 权重调整

然后调整 MPC 权重。一种简单方法是保持横向偏差权重 `weight_lat_error` 不变，调整输入权重 `weight_steering_input`，同时观察转向振荡与控制精度之间的权衡。

其中，`weight_lat_error` 用于抑制路径跟踪中的横向误差，`weight_steering_input` 用于使转向角接近由路径曲率决定的标准值。`weight_lat_error` 较大时，转向会大幅变化以提高精度，可能产生振荡；`weight_steering_input` 较大时，转向对跟踪误差的响应较弱，行驶更稳定，但跟踪精度可能降低。

步骤如下：

1. 设置 `weight_lat_error` = 0.1、`weight_steering_input` = 1.0，其他权重设为 0。
2. 如果车辆行驶时发生振荡，则增大 `weight_steering_input`。
3. 如果跟踪精度较低，则减小 `weight_steering_input`。

如果只想调整高速区间的效果，可以使用 `weight_steering_input_squared_vel`。此参数对应高速区间的转向权重。

<a id="descriptions-for-weights"></a>

#### 权重说明

- `weight_lat_error`：减小横向跟踪误差，作用类似 PID 中的 P 增益。
- `weight_heading_error`：使车辆保持直行，作用类似 PID 中的 D 增益。
- `weight_heading_error_squared_vel_coeff`：使车辆在高速区间保持直行。
- `weight_steering_input`：减小跟踪振荡。
- `weight_steering_input_squared_vel_coeff`：减小高速区间的跟踪振荡。
- `weight_lat_jerk`：减小横向加加速度。
- `weight_terminal_lat_error`：为提高稳定性，建议设置为高于常规横向权重 `weight_lat_error` 的值。
- `weight_terminal_heading_error`：为提高稳定性，建议设置为高于常规航向权重 `weight_heading_error` 的值。

<a id="other-tips-for-tuning"></a>

#### 其他调参建议

调整其他参数时，可参考以下建议：

- 理论上，增大终端权重 `weight_terminal_lat_error` 和 `weight_terminal_heading_error` 可以提高跟踪稳定性。这种方法有时会有效。
- 增大 `prediction_horizon` 并减小 `prediction_sampling_time` 有利于跟踪性能，但会增加计算开销。
- 如果希望根据轨迹曲率调整权重（例如在急弯处增大权重），请使用 `mpc_low_curvature_thresh_curvature` 并调整 `mpc_low_curvature_weight_**` 权重。
- 如果希望根据车速和轨迹曲率调整转向角速度限制，可以修改 `steer_rate_lim_dps_list_by_curvature`、`curvature_list_for_steer_rate_lim`、`steer_rate_lim_dps_list_by_velocity`、`velocity_list_for_steer_rate_lim`。这样可以在高速行驶时收紧转向角速度限制，在转弯时放宽限制。
- 如果目标曲率呈锯齿状，调整 `curvature_smoothing` 对准确计算曲率非常重要。较大的值可使曲率计算更平滑、噪声更小，但可能引入前馈计算延迟，进而降低性能。
- 调整 `steering_lpf_cutoff_hz` 也能有效抑制计算噪声。该参数是输出最后一层二阶巴特沃斯滤波器的截止频率。截止频率越低，降噪越强，但也会引入操作延迟。
- 如果车辆持续横向偏离轨迹，通常是转向传感器或自身定位估计存在偏置。最好在输入 MPC 之前消除这些偏置，也可以在 MPC 内部消除。为此，将 `enable_auto_steering_offset_removal` 设为 true，启用转向偏置消除器。转向偏置估计逻辑在车辆高速行驶且转向接近中位时工作，并应用偏置补偿。
- 如果进入弯道时转向启动过晚，通常是转向模型中的延迟时间和时间常数不正确。请重新检查 `input_delay` 和 `vehicle_model_steer_tau`。此外，MPC 会在调试信息中输出其模型假定的当前转向角，请检查该角度是否与实际值一致。

<a id="references-external-links"></a>

## 参考资料与外部链接

<!-- Optional -->

- [1] Jarrod M. Snider, "Automatic Steering Methods for Autonomous Automobile Path Tracking",
  Robotics Institute, Carnegie Mellon University, February 2009.

<a id="related-issues"></a>

## 相关问题

<!-- Required -->
