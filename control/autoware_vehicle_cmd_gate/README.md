# vehicle_cmd_gate

<a id="purpose"></a>

## 目的

`vehicle_cmd_gate` 功能包从紧急处理器、规划模块和外部控制器获取信息，并向车辆发送消息。

<a id="role"></a>

## 作用

接收多组控制命令，选择其中一组转发给车辆。

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| `~/input/steering` | `autoware_vehicle_msgs::msg::SteeringReport` | 转向状态 |
| `~/input/auto/control_cmd` | `autoware_control_msgs::msg::Control` | 规划模块提供的横向和纵向速度控制命令 |
| `~/input/auto/turn_indicators_cmd` | `autoware_vehicle_msgs::msg::TurnIndicatorsCommand` | 规划模块提供的转向灯命令 |
| `~/input/auto/hazard_lights_cmd` | `autoware_vehicle_msgs::msg::HazardLightsCommand` | 规划模块提供的危险警告灯命令 |
| `~/input/auto/gear_cmd` | `autoware_vehicle_msgs::msg::GearCommand` | 规划模块提供的挡位命令 |
| `~/input/external/control_cmd` | `autoware_control_msgs::msg::Control` | 外部提供的横向和纵向速度控制命令 |
| `~/input/external/turn_indicators_cmd` | `autoware_vehicle_msgs::msg::TurnIndicatorsCommand` | 外部提供的转向灯命令 |
| `~/input/external/hazard_lights_cmd` | `autoware_vehicle_msgs::msg::HazardLightsCommand` | 外部提供的危险警告灯命令 |
| `~/input/external/gear_cmd` | `autoware_vehicle_msgs::msg::GearCommand` | 外部提供的挡位命令 |
| `~/input/external_emergency_stop_heartbeat` | `tier4_external_api_msgs::msg::Heartbeat` | 心跳 |
| `~/input/gate_mode` | `tier4_control_msgs::msg::GateMode` | 命令门控模式（AUTO 或 EXTERNAL） |
| `~/input/emergency/control_cmd` | `autoware_control_msgs::msg::Control` | 紧急处理器提供的横向和纵向速度控制命令 |
| `~/input/emergency/turn_indicators_cmd` | `autoware_vehicle_msgs::msg::TurnIndicatorsCommand` | 紧急处理器提供的转向灯命令 |
| `~/input/emergency/hazard_lights_cmd` | `autoware_vehicle_msgs::msg::HazardLightsCommand` | 紧急处理器提供的危险警告灯命令 |
| `~/input/emergency/gear_cmd` | `autoware_vehicle_msgs::msg::GearCommand` | 紧急处理器提供的挡位命令 |
| `~/input/engage` | `autoware_vehicle_msgs::msg::Engage` | 接管使能信号 |
| `~/input/operation_mode` | `autoware_adapi_v1_msgs::msg::OperationModeState` | Autoware 运行模式 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------------------------------- | --------------------------------------------------- | -------------------------------------------------------- |
| `~/output/vehicle_cmd_emergency` | `tier4_vehicle_msgs::msg::VehicleEmergencyStamped` | 原先包含在车辆命令中的紧急状态 |
| `~/output/command/control_cmd` | `autoware_control_msgs::msg::Control` | 发给车辆的横向和纵向速度控制命令 |
| `~/output/command/turn_indicators_cmd` | `autoware_vehicle_msgs::msg::TurnIndicatorsCommand` | 发给车辆的转向灯命令 |
| `~/output/command/hazard_lights_cmd` | `autoware_vehicle_msgs::msg::HazardLightsCommand` | 发给车辆的危险警告灯命令 |
| `~/output/command/gear_cmd` | `autoware_vehicle_msgs::msg::GearCommand` | 发给车辆的挡位命令 |
| `~/output/gate_mode` | `tier4_control_msgs::msg::GateMode` | 命令门控模式（AUTO 或 EXTERNAL） |
| `~/output/engage` | `autoware_vehicle_msgs::msg::Engage` | 接管使能信号 |
| `~/output/external_emergency` | `tier4_external_api_msgs::msg::Emergency` | 外部紧急信号 |
| `~/output/operation_mode` | `tier4_system_msgs::msg::OperationMode` | vehicle_cmd_gate 的当前运行模式 |

<a id="parameters"></a>

## 参数

| 参数 | 类型 | 说明 |
| ----------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `update_period` | double | 更新周期 |
| `use_emergency_handling` | bool | 使用紧急处理器时为 true |
| `check_external_emergency_heartbeat` | bool | 检查紧急停车心跳时为 true |
| `system_emergency_heartbeat_timeout` | double | 系统紧急心跳超时时间 |
| `external_emergency_stop_heartbeat_timeout` | double | 外部紧急心跳超时时间 |
| `filter_activated_count_threshold` | int | 过滤器激活次数阈值 |
| `filter_activated_velocity_threshold` | double | 过滤器激活速度阈值 |
| `stop_hold_acceleration` | double | 车辆应停止时的纵向加速度命令 |
| `emergency_acceleration` | double | 车辆紧急停车时的纵向加速度命令 |
| `moderate_stop_service_acceleration` | double | 车辆通过平缓停车服务停车时的纵向加速度命令 |
| `nominal.vel_lim` | double | 纵向速度限制（在 AUTONOMOUS 运行模式下启用） |
| `nominal.reference_speed_points` | <double> | 计算控制命令限制时使用的参考速度点（在 AUTONOMOUS 运行模式下启用）。此数组长度必须与限制数组相同。 |
| `nominal.lon_acc_lim_for_lon_vel` | <double> | 纵向加速度限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.lon_jerk_lim_for_lon_acc` | <double> | 纵向加加速度限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.lat_acc_lim_for_steer_cmd` | <double> | 横向加速度限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.lat_jerk_lim_for_steer_cmd` | <double> | 横向加加速度限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.steer_cmd_lim` | <double> | 转向角限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.steer_rate_lim_for_steer_cmd` | <double> | 命令转向角速度限制数组（在 AUTONOMOUS 运行模式下启用） |
| `nominal.lat_jerk_lim_for_steer_rate` | double | 用横向加加速度约束转向角速度的限制（在 AUTONOMOUS 运行模式下启用） |
| `nominal.steer_cmd_diff_lim_from_current_steer` | <double> | 当前转向角与命令转向角之差的限制数组（在 AUTONOMOUS 运行模式下启用） |
| `on_transition.vel_lim` | double | 纵向速度限制（在 TRANSITION 运行模式下启用） |
| `on_transition.reference_speed_points` | <double> | 计算控制命令限制时使用的参考速度点（在 TRANSITION 运行模式下启用）。此数组长度必须与限制数组相同。 |
| `on_transition.lon_acc_lim_for_lon_vel` | <double> | 纵向加速度限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.lon_jerk_lim_for_lon_acc` | <double> | 纵向加加速度限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.lat_acc_lim_for_steer_cmd` | <double> | 横向加速度限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.lat_jerk_lim_for_steer_cmd` | <double> | 横向加加速度限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.steer_cmd_lim` | <double> | 转向角限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.steer_rate_lim_for_steer_cmd` | <double> | 命令转向角速度限制数组（在 TRANSITION 运行模式下启用） |
| `on_transition.lat_jerk_lim_for_steer_rate` | double | 用横向加加速度约束转向角速度的限制（在 TRANSITION 运行模式下启用） |
| `on_transition.steer_cmd_diff_lim_from_current_steer` | <double> | 当前转向角与命令转向角之差的限制数组（在 TRANSITION 运行模式下启用） |

<a id="parameter-naming-convention"></a>

### 参数命名约定

参数遵循特定命名模式，以清楚区分不同类型的约束及其关系：

<a id="pattern-1-constraint_lim_for_target"></a>

#### 模式 1：`[constraint]_lim_for_[target]`

- **格式**：`[physical_constraint]_lim_for_[controlled_variable]`
- **说明**：定义应用于控制变量的物理约束限制（加速度、加加速度等）。

<a id="pattern-2-target_constraint_lim_from_reference"></a>

#### 模式 2：`[target]_[constraint]_lim_from_[reference]`

- **格式**：`[controlled_variable]_[constraint_type]_lim_from_[reference_variable]`
- **说明**：定义控制变量相对于参考值的差值或偏差限制。

<a id="pattern-3-target_lim"></a>

#### 模式 3：`[target]_lim`

- **格式**：`[controlled_variable]_lim`
- **说明**：定义控制变量的绝对限制。

<a id="functionality"></a>

## 功能

<a id="main-functionality"></a>

### 主要功能

- 接收多组控制命令（来自 Autoware 规划、紧急处理器、远程控制等），选择一组转发给车辆。
- 对所选命令施加最终保护，强制满足绝对安全限制（例如最大转向角速度）。这不是舒适性过滤器。
- 切换至自动驾驶模式时（例如远程→自动、手动→自动），施加切换保护以限制突变。建议与运行模式切换管理器集成，但由于其复杂性，代码之间应保持清晰边界。

<a id="sub-functionality"></a>

### 辅助功能

- 检查心跳信号，验证各输入的连接状态（例如外部紧急心跳）。
- 发布最终保护是否激活的状态。自动驾驶模式下保护激活，意味着命令生成中出现了非预期约束，需要关注。
- 在模式切换期间利用保护状态，通知操作人员或驾驶员当前存在较强约束（重点提示“切换进行中”，而非仅提示过滤器激活）。

<a id="filter-function"></a>

### 过滤功能

本模块在控制命令发布前应用限幅过滤器。该过滤器主要用于安全保护，限制 Autoware 发布的所有控制命令的输出范围。

限制值通过对限制数组参数进行一维插值计算。以下是纵向加加速度限制的示例。

![过滤器示例](./image/filter.png)

说明：此过滤器并非用于提高乘坐舒适性。其主要目的是在 Autoware 输出末端检测并移除异常控制值。如果过滤器频繁激活，说明控制模块可能需要调参。如果希望通过低通滤波等方法平滑信号，应在控制模块中处理。过滤器激活时，会发布 `~/is_filter_activated` 话题。

说明 2：如果车辆通过油门或制动踏板控制驱动力，则低速时必须充分放宽加加速度限制，也就是踏板变化率限制。
否则，起步和停车时无法快速改变踏板输入，会导致起步缓慢以及坡道溜车。
此起停功能曾内置于源代码中，但由于逻辑复杂且可通过参数实现，后来被移除。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

<a id="external-emergency-heartbeat"></a>

### 外部紧急心跳

`check_external_emergency_heartbeat` 参数（默认 true）启用来自外部模块的紧急停车请求。
此功能要求存在 `~/input/external_emergency_stop_heartbeat` 话题，用于监控外部模块的健康状态；缺少该话题时，vehicle_cmd_gate 模块不会启动。
不使用“外部紧急停车”功能时，必须将 `check_external_emergency_heartbeat` 设为 false。

<a id="commands-on-mode-changes"></a>

### 模式切换时的命令

输出命令话题 `turn_indicators_cmd`、`hazard_light` 和 `gear_cmd` 根据 `gate_mode` 选择。
但为保证命令连续性，即使发生模式切换，也会等待新输入命令话题到达后才更改这些命令。

<a id="caution"></a>

## 注意事项

- 在设计层面，本节点依赖运行模式切换管理器完成接管状态切换。
- 测试必不可少，必须保留。
