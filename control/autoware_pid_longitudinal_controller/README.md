<a id="pid-longitudinal-controller"></a>

# PID 纵向控制器

<a id="purpose-use-cases"></a>

## 目的与使用场景

longitudinal_controller 使用前馈与反馈控制，计算目标加速度，以达到目标轨迹各点设定的目标速度。

它还包含考虑道路坡度信息的坡度力补偿，以及延迟补偿功能。
此处假设计算出的目标加速度能够由车辆接口正确实现。

请注意，如果车辆支持“目标速度”接口，Autoware 并不强制要求使用本模块。

<a id="design-inner-workings-algorithms"></a>

## 设计、内部工作原理与算法

<a id="states"></a>

### 状态

为在特定情况下进行特殊处理，本模块具有下述四种状态及其切换。

- **DRIVE**
  - 通过 PID 控制跟踪目标速度。
  - 同时应用延迟补偿和坡度补偿。
- **STOPPING**
  - 控制即将停车前的运动。
  - 执行特殊控制序列，实现准确、平稳的停车。
- **STOPPED**
  - 执行停止状态下的操作（例如保持制动）。
- **EMERGENCY**。
  - 满足特定条件时进入紧急状态（例如车辆越过停止线一定距离）。
  - 恢复条件（是否保持紧急状态直到车辆完全停止）及紧急状态下的减速度由参数定义。

状态切换图如下。

![纵向控制器状态切换](./media/LongitudinalControllerStateTransition.drawio.svg)

<a id="logics"></a>

### 逻辑

<a id="control-block-diagram"></a>

#### 控制框图

![纵向控制器框图](./media/LongitudinalControllerDiagram.drawio.svg)

<a id="feedforward-ff"></a>

#### 前馈（FF）

将轨迹中设定的参考加速度和坡度补偿项作为前馈输出。在不存在建模误差的理想条件下，仅此 FF 项就应足以完成速度跟踪。

由建模或离散化误差造成的跟踪误差通过反馈控制消除（目前使用 PID）。

<a id="brake-keeping"></a>

##### 保持制动

从乘坐舒适性来看，以零加速度停稳很重要，因为这样可以减小制动冲击。但如果停车时目标加速度为零，车辆可能因模型误差或坡度估计误差越过停止线，或在停止线前轻微加速。

为确保可靠停车，停车时将前馈系统计算出的目标加速度限制为负值。

![保持制动示意图](./media/BrakeKeeping.drawio.svg)

<a id="slope-compensation"></a>

#### 坡度补偿

根据坡度信息，向目标加速度添加补偿项。

坡度信息有两个来源，可通过参数切换。

- 估计自车位姿的俯仰角（默认）
  - 根据估计自车位姿的俯仰角计算当前坡度。
  - 优点：容易获取。
  - 缺点：受车辆振动影响，无法提取准确的坡度信息。
- 轨迹上的 Z 坐标
  - 根据目标轨迹中前后轮位置的 z 坐标差计算道路坡度。
  - 优点：如果路线的 z 坐标维护正确，则比俯仰角信息更准确。
  - 优点：可与延迟补偿配合使用（尚未实现）。
  - 缺点：需要高精地图的 z 坐标。
  - 缺点：目前不支持自由空间规划。

也提供了根据行驶条件在两者之间切换的选项。

**说明：**此功能仅适用于底层控制系统中没有加速度反馈的车辆系统。

此补偿会向目标加速度添加重力修正，使输出值不再等于自动驾驶系统期望的目标加速度，因此会与底层控制器的加速度反馈作用冲突。
例如，车辆尝试以 `1.0 m/s^2` 的加速度起步，如果应用 `-1.0 m/s^2` 的重力修正，输出值将为 `0`。如果错误地将该输出值当作目标加速度，车辆就不会起步。

适合使用坡度补偿的车辆系统示例是：将 longitudinal_controller 输出的加速度转换为目标油门或制动踏板输入，且不进行任何反馈。在这种情况下，输出加速度仅作为计算目标踏板输入的前馈项，因此不会出现上述问题。

注意：坡度角在上坡时定义为正，而自车位姿的俯仰角在车头向上时定义为负，两者的符号约定相反。

![坡度定义](./media/slope_definition.drawio.svg)

<a id="pid-control"></a>

#### PID 控制

对于前馈控制无法处理的偏差（例如模型误差），使用 PID 控制构建反馈系统。

PID 控制根据当前自车速度与目标速度之间的偏差计算目标加速度。

此 PID 逻辑对各项输出均设有上限，以防止以下情况：

- 积分项过大，产生用户不期望的行为。
- 意外噪声使微分项输出过大。

注意：默认情况下，车辆静止时控制系统不累积积分项。这是为了避免在 Autoware 认为车辆已接管、但外部系统为执行启动流程而限制车辆运动的场景中，积分项发生意外累积。

不过，有些情况下，例如车辆起步时遇到路面凹陷，或坡度补偿估计偏低，可能导致车辆无法起步。为处理这些情况，可以将 `enable_integration_at_low_speed` 设为 true，使车辆静止时也能进行误差积分。

当 `enable_integration_at_low_speed` 为 true 时，如果车辆在 `time_threshold_before_pid_integration` 指定的时间内一直未超过 `current_vel_threshold_pid_integration` 设定的最低速度，PID 控制器就会开始对加速度误差积分。

`time_threshold_before_pid_integration` 对实际 PID 调参很重要。车辆静止或低速时进行误差积分，可能使调参更复杂。此参数在积分项生效之前引入延迟，防止积分立即介入，使 PID 调整更可控、更有效。

目前采用 PID 控制，是在开发和维护成本与性能之间权衡的结果。
后续开发中可能将其替换为性能更高的控制器（例如自适应控制或鲁棒控制）。

<a id="time-delay-compensation"></a>

#### 时间延迟补偿

高速行驶时，油门和制动等执行器系统的延迟会显著影响驾驶精度。
根据车辆执行机构原理，实际控制油门和制动的机械装置通常存在约一百毫秒的延迟。

本控制器计算延迟时间之后的预测自车速度和目标速度，并用于反馈，以处理时间延迟问题。

<a id="slope-compensation_1"></a>

### 坡度补偿

根据坡度信息，向目标加速度添加补偿项。

坡度信息有两个来源，可通过参数切换。

- 估计自车位姿的俯仰角（默认）
  - 根据估计自车位姿的俯仰角计算当前坡度。
  - 优点：容易获取。
  - 缺点：受车辆振动影响，无法提取准确的坡度信息。
- 轨迹上的 Z 坐标
  - 根据目标轨迹中前后轮位置的 z 坐标差计算道路坡度。
  - 优点：如果路线的 z 坐标维护正确，则比俯仰角信息更准确。
  - 优点：可与延迟补偿配合使用（尚未实现）。
  - 缺点：需要高精地图的 z 坐标。
  - 缺点：目前不支持自由空间规划。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

1. 轨迹中应设置平滑后的目标速度及其加速度。
   1. 控制器内部不会平滑速度命令（只可能去除噪声）。
   2. 对于阶跃式目标信号，会尽可能快速地跟踪。
2. 车辆速度必须是正确的值。
   1. 自车速度必须带符号，以对应前进和后退方向。
   2. 自车速度应经过适当的噪声处理。
   3. 如果自车速度中存在大量噪声，跟踪性能会显著降低。
3. 本控制器的输出必须由后续模块（例如车辆接口）实现。
   1. 如果车辆接口不提供目标速度或加速度接口（例如仅提供油门和制动踏板接口），则必须在本控制器之后进行适当转换。

<a id="inputs-outputs-api"></a>

## 输入、输出与 API

<a id="input"></a>

### 输入

由 [controller_node](../autoware_trajectory_follower_node/README.md) 设置以下内容：

- `autoware_planning_msgs/Trajectory`：需要跟踪的参考轨迹。
- `nav_msgs/Odometry`：当前里程计。

<a id="output"></a>

### 输出

向控制器节点返回包含以下内容的 LongitudinalOutput：

- `autoware_control_msgs/Longitudinal`：控制车辆纵向运动的命令，包含目标速度和目标加速度。
- LongitudinalSyncData
  - 速度收敛状态（目前未使用）。

<a id="pidcontroller-class"></a>

### PIDController 类

`PIDController` 类的使用很直接。
首先，使用 `setGains()` 和 `setLimits()`，为比例（P）、积分（I）和微分（D）项设置增益和限制。
然后，向 `calculate()` 函数提供当前误差和时间步长，即可计算速度。

<a id="parameter-description"></a>

## 参数说明

`param/lateral_controller_defaults.param.yaml` 中的默认参数针对
AutonomouStuff Lexus RX 450h 以低于 40 km/h 的速度行驶进行了调整。

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------------------------ | :----- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| delay_compensation_time | double | 纵向控制延迟 [s] | 0.17 |
| enable_smooth_stop | bool | 是否启用向 STOPPING 状态的切换 | true |
| enable_overshoot_emergency | bool | 自车越过停止线一定距离时，是否启用向 EMERGENCY 状态的切换。参见 `emergency_state_overshoot_stop_dist`。 | true |
| enable_large_tracking_error_emergency | bool | 因轨迹与自车位姿偏差过大而无法找到最近轨迹点时，是否启用向 EMERGENCY 状态的切换。 | true |
| enable_slope_compensation | bool | 是否修改输出加速度以补偿坡度。坡度角可来自自车位姿或轨迹角度。参见 `use_trajectory_for_pitch_calculation`。 | true |
| enable_brake_keeping_before_stop | bool | 自车停止前，是否在 DRIVE 状态保持一定加速度。参见[保持制动](#brake-keeping)。 | false |
| enable_keep_stopped_until_steer_convergence | bool | 是否保持停止状态直到转向收敛。 | true |
| max_acc | double | 输出加速度的最大值 [m/s^2] | 3.0 |
| min_acc | double | 输出加速度的最小值 [m/s^2] | -5.0 |
| max_jerk | double | 输出加速度的最大加加速度 [m/s^3] | 2.0 |
| min_jerk | double | 输出加速度的最小加加速度 [m/s^3] | -5.0 |
| use_trajectory_for_pitch_calculation | bool | 若为 true，则根据轨迹 z 坐标估计坡度；否则使用自车位姿的俯仰角。 | false |
| lpf_pitch_gain | double | 俯仰角估计的低通滤波器增益 | 0.95 |
| max_pitch_rad | double | 估计俯仰角的最大值 [rad] | 0.1 |
| min_pitch_rad | double | 估计俯仰角的最小值 [rad] | -0.1 |

<a id="state-transition"></a>

### 状态切换

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------------------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| drive_state_stop_dist | double | 距停车点的距离大于 `drive_state_stop_dist` + `drive_state_offset_stop_dist` 时，切换为 DRIVE。[m] | 0.5 |
| drive_state_offset_stop_dist | double | 距停车点的距离大于 `drive_state_stop_dist` + `drive_state_offset_stop_dist` 时，切换为 DRIVE。[m] | 1.0 |
| stopping_state_stop_dist | double | 距停车点的距离小于 `stopping_state_stop_dist` 时，切换为 STOPPING。[m] | 0.5 |
| stopped_state_entry_vel | double | 切换到 STOPPED 状态时的自车速度阈值 [m/s] | 0.01 |
| stopped_state_entry_acc | double | 切换到 STOPPED 状态时的自车加速度阈值 [m/s^2] | 0.1 |
| emergency_state_overshoot_stop_dist | double | 如果 `enable_overshoot_emergency` 为 true，且自车越过停车点 `emergency_state_overshoot_stop_dist` 米，则切换为 EMERGENCY。[m] | 1.5 |

<a id="drive-parameter"></a>

### DRIVE 参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------------------ | :----- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| kp | double | 纵向控制的 p 增益 | 1.0 |
| ki | double | 纵向控制的 i 增益 | 0.1 |
| kd | double | 纵向控制的 d 增益 | 0.0 |
| max_out | double | DRIVE 状态下 PID 输出加速度的最大值 [m/s^2] | 1.0 |
| min_out | double | DRIVE 状态下 PID 输出加速度的最小值 [m/s^2] | -1.0 |
| max_p_effort | double | p 项加速度的最大值 | 1.0 |
| min_p_effort | double | p 项加速度的最小值 | -1.0 |
| max_i_effort | double | i 项加速度的最大值 | 0.3 |
| min_i_effort | double | i 项加速度的最小值 | -0.3 |
| max_d_effort | double | d 项加速度的最大值 | 0.0 |
| min_d_effort | double | d 项加速度的最小值 | 0.0 |
| lpf_vel_error_gain | double | 速度误差低通滤波器的增益 | 0.9 |
| enable_integration_at_low_speed | bool | 车速低于 `current_vel_threshold_pid_integration` 时，是否启用加速度误差积分。 | false |
| current_vel_threshold_pid_integration | double | 仅当当前速度的绝对值大于此参数时，才对速度误差积分以计算 I 项。[m/s] | 0.5 |
| time_threshold_before_pid_integration | double | 车辆保持不动多长时间后启用 PID 误差积分。[s] | 5.0 |
| ff_scale_min | double | 时间到弧长转换期间应用的前馈缩放系数下限。 | 0.5 |
| ff_scale_max | double | 时间到弧长转换期间应用的前馈缩放系数上限。 | 2.0 |
| brake_keeping_acc | double | 若 `enable_brake_keeping_before_stop` 为 true，则自车停止前在 DRIVE 状态保持一定加速度 [m/s^2]。参见[保持制动](#brake-keeping)。 | 0.2 |

<a id="stopping-parameter-smooth-stop"></a>

### STOPPING 参数（平滑停车）

如果 `enable_smooth_stop` 为 true，则启用平滑停车。
平滑停车时，先输出较强的减速加速度（`strong_acc`）以降低自车速度。
然后输出较弱的减速加速度（`weak_acc`），通过降低自车加加速度实现平稳停车。
如果自车在一定时间内仍未停止，或越过停车点一定距离，则输出用于立即停车的较弱加速度（`weak_stop_acc`）。
如果自车仍在运动，则输出用于立即停车的较强加速度（`strong_stop_acc`）。

| 名称 | 类型 | 说明 | 默认值 |
| :--------------------------- | :----- | :------------------------------------------------------------------------------------------------------------------- | :------------ |
| smooth_stop_max_strong_acc | double | 强减速阶段的最大加速度 [m/s^2] | -0.5 |
| smooth_stop_min_strong_acc | double | 强减速阶段的最小加速度 [m/s^2] | -0.8 |
| smooth_stop_weak_acc | double | 弱减速加速度 [m/s^2] | -0.3 |
| smooth_stop_weak_stop_acc | double | 用于立即停车的较弱加速度 [m/s^2] | -0.8 |
| smooth_stop_strong_stop_acc | double | 自车越过停车点 `smooth_stop_strong_stop_dist` 米时输出的较强加速度。[m/s^2] | -3.4 |
| smooth_stop_max_fast_vel | double | 判定自车快速行驶的最大速度阈值 [m/s]。自车快速行驶时会输出较强加速度。 | 0.5 |
| smooth_stop_min_running_vel | double | 判定自车是否仍在运动的最小速度 [m/s] | 0.01 |
| smooth_stop_min_running_acc | double | 判定自车是否仍在运动的最小加速度 [m/s^2] | 0.01 |
| smooth_stop_weak_stop_time | double | 输出较弱加速度的最长时间 [s]，之后输出较强加速度。 | 0.8 |
| smooth_stop_weak_stop_dist | double | 自车距停车点还有 `smooth_stop_weak_stop_dist` 米时，输出较弱加速度。[m] | -0.3 |
| smooth_stop_strong_stop_dist | double | 自车越过停车点 `smooth_stop_strong_stop_dist` 米时，输出较强加速度。[m] | -0.5 |

<a id="stopped-parameter"></a>

### STOPPED 参数

`STOPPED` 状态假设车辆已完全停止，且制动已完全施加。
因此，`stopped_acc` 应设置为能够使车辆施加最强制动的值。
如果 `stopped_acc` 不够低，车辆可能在陡坡上下滑。

| 名称 | 类型 | 说明 | 默认值 |
| :---------- | :----- | :------------------------------------------- | :------------ |
| stopped_vel | double | STOPPED 状态的目标速度 [m/s] | 0.0 |
| stopped_acc | double | STOPPED 状态的目标加速度 [m/s^2] | -3.4 |

<a id="emergency-parameter"></a>

### EMERGENCY 参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------- | :----- | :------------------------------------------------ | :------------ |
| emergency_vel | double | EMERGENCY 状态的目标速度 [m/s] | 0.0 |
| emergency_acc | double | EMERGENCY 状态的目标加速度 [m/s^2] | -5.0 |
| emergency_jerk | double | EMERGENCY 状态的目标加加速度 [m/s^3] | -3.0 |

<a id="references-external-links"></a>

## 参考资料与外部链接

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分

<a id="related-issues"></a>

## 相关问题
