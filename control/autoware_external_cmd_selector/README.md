# autoware_external_cmd_selector

<a id="purpose"></a>

## 目的

`autoware_external_cmd_selector` 功能包根据当前模式（`remote` 或 `local`）发布 `external_control_cmd`、`gear_cmd`、`hazard_lights_cmd`、`heartbeat` 和 `turn_indicators_cmd`。

当前模式通过服务设置；`remote` 表示远程操作，`local` 表示使用 Autoware 计算出的值。

<a id="input-output"></a>

## 输入与输出

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| ---------------------------------------------- | ---- | ------------------------------------------------------- |
| `/api/external/set/command/local/control` | TBD | 本地：计算出的控制值。 |
| `/api/external/set/command/local/heartbeat` | TBD | 本地：心跳。 |
| `/api/external/set/command/local/shift` | TBD | 本地：前进挡、倒挡等挡位切换。 |
| `/api/external/set/command/local/turn_signal` | TBD | 本地：左转、右转等转向灯信号。 |
| `/api/external/set/command/remote/control` | TBD | 远程：计算出的控制值。 |
| `/api/external/set/command/remote/heartbeat` | TBD | 远程：心跳。 |
| `/api/external/set/command/remote/shift` | TBD | 远程：前进挡、倒挡等挡位切换。 |
| `/api/external/set/command/remote/turn_signal` | TBD | 远程：左转、右转等转向灯信号。 |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| ------------------------------------------------------ | ------------------------------------------------- | ----------------------------------------------- |
| `/control/external_cmd_selector/current_selector_mode` | TBD | 当前选择的模式：remote 或 local。 |
| `/diagnostics` | diagnostic_msgs::msg::DiagnosticArray | 检查节点是否处于活动状态。 |
| `/external/selected/external_control_cmd` | TBD | 转发当前模式的控制命令。 |
| `/external/selected/gear_cmd` | autoware_vehicle_msgs::msg::GearCommand | 转发当前模式的挡位命令。 |
| `/external/selected/hazard_lights_cmd` | autoware_vehicle_msgs::msg::HazardLightsCommand | 转发当前模式的危险警告灯命令。 |
| `/external/selected/heartbeat` | TBD | 转发当前模式的心跳。 |
| `/external/selected/turn_indicators_cmd` | autoware_vehicle_msgs::msg::TurnIndicatorsCommand | 转发当前模式的转向灯命令。 |

<a id="parameters"></a>

## 参数

{{json_to_markdown("control/autoware_external_cmd_selector/schema/external_cmd_selector.schema.json")}}
