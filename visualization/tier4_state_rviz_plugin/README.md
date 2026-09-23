# tier4_state_rviz_plugin

<a id="purpose"></a>

## 用途

此插件显示 Autoware 的当前状态。
也可以通过该插件面板启用自动驾驶。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------- |
| `/api/operation_mode/state` | `autoware_adapi_v1_msgs::msg::OperationModeState` | 表示运行模式状态的话题 |
| `/api/routing/state` | `autoware_adapi_v1_msgs::msg::RouteState` | 表示路线状态的话题 |
| `/api/localization/initialization_state` | `autoware_adapi_v1_msgs::msg::LocalizationInitializationState` | 表示定位初始化状态的话题 |
| `/api/motion/state` | `autoware_adapi_v1_msgs::msg::MotionState` | 表示运动状态的话题 |
| `/api/autoware/get/emergency` | `tier4_external_api_msgs::msg::Emergency` | 表示外部紧急状态的话题 |
| `/vehicle/status/gear_status` | `autoware_vehicle_msgs::msg::GearReport` | 表示挡位状态的话题 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `/api/operation_mode/change_to_autonomous` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 将运行模式切换为 autonomous 的服务 |
| `/api/operation_mode/change_to_stop` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 将运行模式切换为 stop 的服务 |
| `/api/operation_mode/change_to_local` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 将运行模式切换为 local 的服务 |
| `/api/operation_mode/change_to_remote` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 将运行模式切换为 remote 的服务 |
| `/api/operation_mode/enable_autoware_control` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 启用 Autoware 车辆控制的服务 |
| `/api/operation_mode/disable_autoware_control` | `autoware_adapi_v1_msgs::srv::ChangeOperationMode` | 禁用 Autoware 车辆控制的服务 |
| `/api/routing/clear_route` | `autoware_adapi_v1_msgs::srv::ClearRoute` | 清除路线状态的服务 |
| `/api/motion/accept_start` | `autoware_adapi_v1_msgs::srv::AcceptStart` | 确认允许车辆起步的服务 |
| `/api/autoware/set/emergency` | `tier4_external_api_msgs::srv::SetEmergency` | 设置外部紧急状态的服务 |
| `/planning/scenario_planning/max_velocity_candidates` | `autoware_internal_planning_msgs::msg::VelocityLimit` | 设置车辆最大速度的话题 |

<a id="howtouse"></a>

## 使用方法

1. 启动 RViz，并选择 panels/Add new panel。

   ![选择面板](./images/select_panels.png)

2. 选择 tier4_state_rviz_plugin/AutowareStatePanel，然后点击 OK。

   ![选择状态插件](./images/select_state_plugin.png)

3. auto 按钮可用时，点击即可启用自动驾驶。

   ![选择自动驾驶](./images/select_auto.png)
