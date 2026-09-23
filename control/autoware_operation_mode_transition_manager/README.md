# autoware_operation_mode_transition_manager

<a id="note"></a>

## 说明

本功能包的状态管理功能目前正在迁移到 command_mode_decider 功能包。
与 command_mode_decider 配合使用时，本功能包仅负责判断切换条件。
此外，调试话题中的 `status`、`in_autoware_control` 和 `in_transition` 数据不再可用，请改用 `/system/command_mode_decider/debug` 话题。

- in_autoware_control：当前命令模式为手动模式（默认值 1000）时为 True。
- in_transition：参见 command_mode_decider 功能包中的 is_in_transition 函数。
- status：
  - 如果 in_autoware_control，则为 `DISENGAGE (autoware mode = curr_mode)`。
  - 否则，如果 in_transition，则为 `curr_mode (in transition from prev_mode)`。
  - 否则为 `curr_mode`。
  - 注意：`curr_mode = current operation mode`，`prev_mode = last operation mode`，即当前运行模式和上一次运行模式。

<a id="purpose-use-cases"></a>

## 目的与使用场景

本模块负责管理 Autoware 系统的不同运行模式。可用模式包括：

- `Autonomous`：车辆完全由自动驾驶系统控制。
- `Local`：车辆由物理连接的控制系统控制，例如手柄。
- `Remote`：车辆由远程控制器控制。
- `Stop`：车辆已停止，没有活动的控制系统。

每次模式切换时还会进入 `In Transition` 状态。在该状态下，向新操作者的切换尚未完成，原操作者仍负责控制系统，直到切换完成。`In Transition` 期间可能限制某些动作，例如急制动或急转向（由 `vehicle_cmd_gate` 限制）。

<a id="features"></a>

### 功能

- 根据指示命令，在 `Autonomous`、`Local`、`Remote` 和 `Stop` 之间切换模式。
- 检查各项切换是否可用（是否安全）。
- 在 `In Transition` 模式中限制某些突变运动控制（通过 `vehicle_cmd_gate` 实现）。
- 检查切换是否完成。

- 根据指示命令，在 `Autonomous`、`Local`、`Remote` 和 `Stop` 模式之间切换。
- 判断各项切换是否可以安全执行。
- 在 `In Transition` 模式中限制某些突变运动控制（使用 `vehicle_cmd_gate` 功能）。
- 确认切换已完成。

<a id="design"></a>

## 设计

`autoware_operation_mode_transition_manager`` 与其他节点之间关系的概要设计如下。

![切换模块概要结构](image/transition_rough_structure.drawio.svg)

更详细的结构如下。

![切换模块详细结构](image/transition_detailed_structure.drawio.svg)

可以看到，`autoware_operation_mode_transition_manager` 具有以下多种状态切换：

- **AUTOWARE ENABLED <---> DISABLED**
  - **ENABLED**：车辆由 Autoware 控制。
  - **DISABLED**：车辆不受 Autoware 控制，预期由人工驾驶等方式控制。
- **AUTOWARE ENABLED <---> AUTO/LOCAL/REMOTE/NONE**
  - **AUTO**：车辆由 Autoware 控制，使用规划和控制组件计算出的自动驾驶控制命令。
  - **LOCAL**：车辆由 Autoware 控制，使用本地连接的操作者输入，例如手柄控制器。
  - **REMOTE**：车辆由 Autoware 控制，使用远程连接的操作者输入。
  - **NONE**：车辆不受任何操作者控制。
- **IN TRANSITION <---> COMPLETED**
  - **IN TRANSITION**：上述模式处于切换过程中，原操作者有责任确认切换完成。
  - **COMPLETED**：模式切换已完成。

<a id="inputs-outputs-api"></a>

## 输入、输出与 API

<a id="inputs"></a>

### 输入

用于模式切换：

- /system/operation_mode/change_autoware_control [`autoware_system_msgs/srv/ChangeAutowareControl`]：将运行模式切换为 Autonomous。
- /system/operation_mode/change_operation_mode [`autoware_system_msgs/srv/ChangeOperationMode`]：切换运行模式。

用于检查切换可用性及完成情况：

- /control/command/control_cmd [`autoware_control_msgs/msg/Control`]：车辆控制信号。
- /localization/kinematic_state [`nav_msgs/msg/Odometry`]：自车状态。
- /planning/trajectory [`autoware_planning_msgs/msg/Trajectory`]：规划轨迹。
- /vehicle/status/control_mode [`autoware_vehicle_msgs/msg/ControlModeReport`]：车辆控制模式（自动或手动）。
- /control/vehicle_cmd_gate/operation_mode [`autoware_adapi_v1_msgs/msg/OperationModeState`]：`vehicle_cmd_gate` 中的运行模式。（将移除）

用于向后兼容（将移除）：

- /api/autoware/get/engage [`autoware_vehicle_msgs/msg/Engage`]
- /control/current_gate_mode [`tier4_control_msgs/msg/GateMode`]
- /control/external_cmd_selector/current_selector_mode [`tier4_control_msgs/msg/ExternalCommandSelectorMode`]

<a id="outputs"></a>

### 输出

- /system/operation_mode/state [`autoware_adapi_v1_msgs/msg/OperationModeState`]：通知当前运行模式。
- /control/autoware_operation_mode_transition_manager/debug_info [`autoware_operation_mode_transition_manager/msg/OperationModeTransitionManagerDebug`]：运行模式切换的详细信息。

- /control/gate_mode_cmd [`tier4_control_msgs/msg/GateMode`]：更改 `vehicle_cmd_gate` 状态以使用其功能（将移除）。
- /autoware/engage [`autoware_vehicle_msgs/msg/Engage`]：

- /control/control_mode_request [`autoware_vehicle_msgs/srv/ControlModeCommand`]：切换车辆控制模式（自动或手动）。
- /control/external_cmd_selector/select_external_command [`tier4_control_msgs/srv/ExternalCommandSelect`]：

<a id="parameters"></a>

## 参数

{{ json_to_markdown("control/autoware_operation_mode_transition_manager/schema/operation_mode_transition_manager.schema.json") }}

| 名称 | 类型 | 说明 | 默认值 |
| :--------------------------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `transition_timeout` | `double` | 如果状态切换未在此时间内完成，则认为切换失败。 | 10.0 |
| `frequency_hz` | `double` | 运行频率，单位 Hz | 10.0 |
| `enable_engage_on_driving` | `bool` | 若希望车辆行驶时也能启用自动驾驶模式，请设为 true。若为 false，则车速非零时一律拒绝接管。请注意，未经调参就启用此功能可能导致突然减速等问题。使用前，请确保已适当调整接管条件和 vehicle_cmd_gate 切换过滤器。 | 0.1 |
| `check_engage_condition` | `bool` | 若为 false，则始终允许切换到自动驾驶 | 0.1 |
| `nearest_dist_deviation_threshold` | `double` | 查找最近轨迹点所用的距离阈值 | 3.0 |
| `nearest_yaw_deviation_threshold` | `double` | 查找最近轨迹点所用的角度阈值 | 1.57 |

`engage_acceptable_limits` 相关参数：

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `allow_autonomous_in_stopped` | `bool` | 若为 true，车辆停止时即使其他检查失败，也允许切换到自动驾驶。 | true |
| `dist_threshold` | `double` | 切换到 `Autonomous` 时，轨迹与自车之间的距离必须在此范围内。 | 1.5 |
| `yaw_threshold` | `double` | 切换到 `Autonomous` 时，轨迹与自车之间的偏航角差必须在此阈值内。 | 0.524 |
| `speed_upper_threshold` | `double` | 切换到 `Autonomous` 时，控制命令与自车之间的速度偏差必须在此阈值内。 | 10.0 |
| `speed_lower_threshold` | `double` | 切换到 `Autonomous` 时，控制命令与自车之间的速度偏差必须在此阈值内。 | -10.0 |
| `acc_threshold` | `double` | 切换到 `Autonomous` 时，控制命令加速度必须小于此阈值。 | 1.5 |
| `lateral_acc_threshold` | `double` | 切换到 `Autonomous` 时，控制命令横向加速度必须小于此阈值。 | 1.0 |
| `lateral_acc_diff_threshold` | `double` | 切换到 `Autonomous` 时，控制命令的横向加速度偏差必须小于此阈值。 | 0.5 |

`stable_check` 相关参数：

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------- | :------- | :-------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `duration` | `double` | 稳定条件必须持续满足此时长，才能完成切换。 | 0.1 |
| `dist_threshold` | `double` | 完成 `Autonomous` 切换时，轨迹与自车之间的距离必须在此范围内。 | 1.5 |
| `yaw_threshold` | `double` | 完成 `Autonomous` 切换时，轨迹与自车之间的偏航角差必须在此阈值内。 | 0.262 |
| `speed_upper_threshold` | `double` | 完成 `Autonomous` 切换时，控制命令与自车之间的速度偏差必须在此阈值内。 | 2.0 |
| `speed_lower_threshold` | `double` | 完成 `Autonomous` 切换时，控制命令与自车之间的速度偏差必须在此阈值内。 | 2.0 |

<a id="engage-check-behavior-on-each-parameter-setting"></a>

## 各参数设置下的接管检查行为

下表说明不同参数组合下允许车辆接管的场景：

| `enable_engage_on_driving` | `check_engage_condition` | `allow_autonomous_in_stopped` | 允许接管的场景 |
| :------------------------: | :----------------------: | :---------------------------: | :---------------------------------------------------------------- |
| x | x | x | 仅车辆静止时。 |
| x | x | o | 仅车辆静止时。 |
| x | o | x | 车辆静止且满足所有接管条件时。 |
| x | o | o | 仅车辆静止时。 |
| o | x | x | 任何时候（注意：不推荐）。 |
| o | x | o | 任何时候（注意：不推荐）。 |
| o | o | x | 满足所有接管条件时，与车辆状态无关。 |
| o | o | o | 满足所有接管条件，或车辆静止时。 |

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分

- 需要移除向后兼容接口。
- 由于与 `vehicle_cmd_gate` 联系紧密，本节点应合并到其中。
