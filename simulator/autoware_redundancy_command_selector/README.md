# autoware_redundancy_command_selector

<a id="purpose"></a>

## 用途

此节点在 `autoware_simple_planning_simulator` 中模拟 ECU 冗余切换行为。
仿真时，此节点在主 ECU 和副 ECU 控制指令之间切换并转发选中的指令流，而非切换输出 ECU 本身。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

- 订阅主 ECU 和副 ECU 的指令话题。
- 订阅 `/system/redundancy/active_control_unit`。
- 如果 `main_ecu_id` 处于活动状态，转发主 ECU 指令。
- 如果 `sub_ecu_id` 处于活动状态，转发副 ECU 指令。
- 如果两者均活动（或均不活动），则保持当前选择，避免不必要的切换。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                                   | 类型                                              | 说明                      |
| -------------------------------------- | ------------------------------------------------- | -------------------------------- |
| `~/input/main/control_command`         | `autoware_control_msgs/msg/Control`               | 主 ECU 控制指令         |
| `~/input/main/gear_command`            | `autoware_vehicle_msgs/msg/GearCommand`           | 主 ECU 挡位指令            |
| `~/input/main/hazard_lights_command`   | `autoware_vehicle_msgs/msg/HazardLightsCommand`   | 主 ECU 危险警示灯指令   |
| `~/input/main/turn_indicators_command` | `autoware_vehicle_msgs/msg/TurnIndicatorsCommand` | 主 ECU 转向灯指令 |
| `~/input/sub/control_command`          | `autoware_control_msgs/msg/Control`               | 副 ECU 控制指令          |
| `~/input/sub/gear_command`             | `autoware_vehicle_msgs/msg/GearCommand`           | 副 ECU 挡位指令             |
| `~/input/sub/hazard_lights_command`    | `autoware_vehicle_msgs/msg/HazardLightsCommand`   | 副 ECU 危险警示灯指令    |
| `~/input/sub/turn_indicators_command`  | `autoware_vehicle_msgs/msg/TurnIndicatorsCommand` | 副 ECU 转向灯指令  |
| `~/input/active_control_unit`          | `tier4_system_msgs/msg/ActiveControlUnit`         | 活动 ECU ID                   |

<a id="output"></a>

### 输出

| 名称                               | 类型                                              | 说明                      |
| ---------------------------------- | ------------------------------------------------- | -------------------------------- |
| `~/output/control_command`         | `autoware_control_msgs/msg/Control`               | 选中的控制指令         |
| `~/output/gear_command`            | `autoware_vehicle_msgs/msg/GearCommand`           | 选中的挡位指令            |
| `~/output/hazard_lights_command`   | `autoware_vehicle_msgs/msg/HazardLightsCommand`   | 选中的危险警示灯指令   |
| `~/output/turn_indicators_command` | `autoware_vehicle_msgs/msg/TurnIndicatorsCommand` | 选中的转向灯指令 |

<a id="parameters"></a>

## 参数

| 名称          | 类型    | 说明            |
| ------------- | ------- | ---------------------- |
| `main_ecu_id` | `uint8` | 视为主 ECU 的 ECU ID |
| `sub_ecu_id`  | `uint8` | 视为副 ECU 的 ECU ID  |

<a id="launch"></a>

## 启动

- XML 启动文件：`launch/redundancy_command_selector.launch.xml`

在仿真器端的启动文件中包含此启动文件，即可通过切换主/副指令流复现冗余 ECU 切换行为。
