# autoware_control_performance_analysis

<a id="purpose"></a>

## 目的

`autoware_control_performance_analysis` 功能包用于分析控制模块的跟踪性能，并监控车辆行驶状态。

本功能包用于量化控制模块的结果，
因此不会干预自动驾驶的核心逻辑。

它根据规划、控制和车辆的各类输入，以本功能包定义的 `autoware_control_performance_analysis::msg::ErrorStamped` 消息发布分析结果。

`ErrorStamped` 消息中的所有结果均在曲线的 Frenet 坐标系中计算。误差及误差变化速度的计算参考以下论文。

<!-- cspell: ignore Werling Moritz Groell Lutz Bretthauer Georg -->

`Werling, Moritz & Groell, Lutz & Bretthauer, Georg. (2010). Invariant Trajectory Tracking With a Full-Size Autonomous Road Vehicle. IEEE Transactions on Robotics. 26. 758 - 765. 10.1109/TRO.2010.2052325.`

如果对计算方法感兴趣，可查阅 `C. Asymptotical Trajectory Tracking With Orientation Control` 一节中的误差及误差变化速度计算。

误差加速度基于上述速度计算得到，其计算方式如下。

![误差加速度公式](https://user-images.githubusercontent.com/45468306/169027099-ef15b306-2868-4084-a350-0e2b652c310f.png)

<a id="input-output"></a>

## 输入与输出

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| --------------------------------- | ------------------------------------------ | ------------------------------------------- |
| `/planning/trajectory` | autoware_planning_msgs::msg::Trajectory | 规划模块输出的轨迹。 |
| `/control/command/control_cmd` | autoware_control_msgs::msg::Control | 控制模块输出的控制命令。 |
| `/vehicle/status/steering_status` | autoware_vehicle_msgs::msg::SteeringReport | 车辆转向信息。 |
| `/localization/kinematic_state` | nav_msgs::msg::Odometry | 使用里程计中的 twist。 |
| `/tf` | tf2_msgs::msg::TFMessage | 从 tf 中提取自车位姿。 |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| --------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------- |
| `/control_performance/performance_vars` | autoware_control_performance_analysis::msg::ErrorStamped | 性能分析结果。 |
| `/control_performance/driving_status` | autoware_control_performance_analysis::msg::DrivingMonitorStamped | 行驶状态监控（加速度、加加速度等）。 |

<a id="outputs"></a>

### 输出

#### autoware_control_performance_analysis::msg::DrivingMonitorStamped

| 名称 | 类型 | 说明 |
| ---------------------------- | ----- | --------------------------------------------------------------------- |
| `longitudinal_acceleration` | float | $[ \mathrm{m/s^2} ]$ |
| `longitudinal_jerk` | float | $[ \mathrm{m/s^3} ]$ |
| `lateral_acceleration` | float | $[ \mathrm{m/s^2} ]$ |
| `lateral_jerk` | float | $[ \mathrm{m/s^3} ]$ |
| `desired_steering_angle` | float | $[ \mathrm{rad} ]$ |
| `controller_processing_time` | float | 最近两条控制命令消息之间的时间间隔 $[ \mathrm{ms} ]$ |

#### autoware_control_performance_analysis::msg::ErrorStamped

| 名称 | 类型 | 说明 |
| ------------------------------------------ | ----- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `lateral_error` | float | $[ \mathrm{m} ]$ |
| `lateral_error_velocity` | float | $[ \mathrm{m/s} ]$ |
| `lateral_error_acceleration` | float | $[ \mathrm{m/s^2} ]$ |
| `longitudinal_error` | float | $[ \mathrm{m} ]$ |
| `longitudinal_error_velocity` | float | $[ \mathrm{m/s} ]$ |
| `longitudinal_error_acceleration` | float | $[ \mathrm{m/s^2} ]$ |
| `heading_error` | float | $[ \mathrm{rad} ]$ |
| `heading_error_velocity` | float | $[ \mathrm{rad/s} ]$ |
| `control_effort_energy` | float | $[ \mathbf{u}^\top \mathbf{R} \mathbf{u} ]$，简化为 $[ R \cdot u^2 ]$ |
| `error_energy` | float | $e_{\text{lat}}^2 + e_\theta^2$（横向误差平方 + 航向误差平方） |
| `value_approximation` | float | $V = \mathbf{x}^\top \mathbf{P} \mathbf{x}$；由 DARE 李雅普诺夫矩阵 $\mathbf{P}$ 得到的价值函数 |
| `curvature_estimate` | float | $[ \mathrm{1/m} ]$ |
| `curvature_estimate_pp` | float | $[ \mathrm{1/m} ]$ |
| `vehicle_velocity_error` | float | $[ \mathrm{m/s} ]$ |
| `tracking_curvature_discontinuity_ability` | float | 衡量跟踪曲率变化的能力 $\frac{\lvert \Delta(\text{curvature}) \rvert}{1 + \lvert \Delta(e_{\text{lat}}) \rvert}$ |

<a id="parameters"></a>

## 参数

| 名称 | 类型 | 说明 |
| ------------------------------------- | ---------------- | ----------------------------------------------------------------- |
| `curvature_interval_length` | double | 用于估计当前曲率 |
| `prevent_zero_division_value` | double | 避免除零的值，默认为 `0.001` |
| `odom_interval` | unsigned integer | odom 消息之间的间隔，增大可使曲线更平滑 |
| `acceptable_max_distance_to_waypoint` | double | 轨迹点与车辆之间的最大距离 [m] |
| `acceptable_max_yaw_difference_rad` | double | 轨迹点与车辆之间的最大偏航角差 [rad] |
| `low_pass_filter_gain` | double | 低通滤波器增益 |

<a id="usage"></a>

## 使用方法

- 启动仿真和控制模块后，启动 `control_performance_analysis.launch.xml`。
- 应能在话题中看到行驶监控与误差变量。
- 如需将结果可视化，可使用 `Plotjuggler`，并将 `config/controller_monitor.xml` 作为布局。
- 导入布局后，请指定下列话题。

> - /localization/kinematic_state
> - /vehicle/status/steering_status
> - /control_performance/driving_status
> - /control_performance/performance_vars

- 在 `Plotjuggler` 中，可以将统计值（最大值、最小值、平均值）导出为 csv 文件，并使用这些统计结果比较控制模块。

<a id="future-improvements"></a>

## 后续改进

- 通过截止频率、微分方程和离散状态空间更新实现低通滤波器（LPF）。
