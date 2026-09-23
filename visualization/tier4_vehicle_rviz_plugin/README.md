# tier4_vehicle_rviz_plugin

This package is including jsk code.  
Note that jsk_overlay_utils.cpp and jsk_overlay_utils.hpp are BSD license.

<a id="purpose"></a>

## 用途

此插件以直观、易懂的方式显示车辆速度、转向灯、转向状态和加速度。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------------------------- | -------------------------------------------------- | ---------------------------------- |
| `/vehicle/status/velocity_status` | `autoware_vehicle_msgs::msg::VelocityReport` | 车辆速度话题 |
| `/control/turn_signal_cmd` | `autoware_vehicle_msgs::msg::TurnIndicatorsReport` | 转向灯状态话题 |
| `/vehicle/status/steering_status` | `autoware_vehicle_msgs::msg::SteeringReport` | 转向状态话题 |
| `/localization/acceleration` | `geometry_msgs::msg::AccelWithCovarianceStamped` | 加速度话题 |

<a id="parameter"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

#### ConsoleMeter

| 名称 | 类型 | 默认值 | 说明 |
| ------------------------------- | ------ | -------------------- | ---------------------------------------- |
| `property_text_color_` | QColor | QColor(25, 255, 240) | 文本颜色 |
| `property_left_` | int | 128 | 绘图窗口左边缘位置 [px] |
| `property_top_` | int | 128 | 绘图窗口上边缘位置 [px] |
| `property_length_` | int | 256 | 绘图窗口高度 [px] |
| `property_value_height_offset_` | int | 0 | 绘图窗口高度偏移量 [px] |
| `property_value_scale_` | float | 1.0 / 6.667 | 数值缩放比例 |

#### SteeringAngle

| 名称 | 类型 | 默认值 | 说明 |
| ------------------------------- | ------ | -------------------- | ---------------------------------------- |
| `property_text_color_` | QColor | QColor(25, 255, 240) | 文本颜色 |
| `property_left_` | int | 128 | 绘图窗口左边缘位置 [px] |
| `property_top_` | int | 128 | 绘图窗口上边缘位置 [px] |
| `property_length_` | int | 256 | 绘图窗口高度 [px] |
| `property_value_height_offset_` | int | 0 | 绘图窗口高度偏移量 [px] |
| `property_value_scale_` | float | 1.0 / 6.667 | 数值缩放比例 |
| `property_handle_angle_scale_` | float | 3.0 | 转向角到方向盘角度的缩放比例 |

#### TurnSignal

| 名称 | 类型 | 默认值 | 说明 |
| ------------------ | ---- | ------------- | -------------------------------- |
| `property_left_` | int | 128 | 绘图窗口左边缘位置 [px] |
| `property_top_` | int | 128 | 绘图窗口上边缘位置 [px] |
| `property_width_` | int | 256 | 绘图窗口左边缘位置 [px] |
| `property_height_` | int | 256 | 绘图窗口宽度 [px] |

#### VelocityHistory

| 名称 | 类型 | 默认值 | 说明 |
| ------------------------------- | ------ | ------------- | -------------------------- |
| `property_velocity_timeout_` | float | 10.0 | 速度超时时间 [s] |
| `property_velocity_alpha_` | float | 1.0 | 速度显示透明度 |
| `property_velocity_scale_` | float | 0.3 | 速度缩放比例 |
| `property_velocity_color_view_` | bool | false | 是否使用固定颜色 |
| `property_velocity_color_` | QColor | Qt::black | 速度历史曲线颜色 |
| `property_vel_max_` | float | 3.0 | 颜色分界的最大速度 [m/s] |

#### AccelerationMeter

| 名称 | 类型 | 默认值 | 说明 |
| ----------------------------------- | ------ | -------------------- | ------------------------------------------------ |
| `property_normal_text_color_` | QColor | QColor(25, 255, 240) | 常规文本颜色 |
| `property_emergency_text_color_` | QColor | QColor(255, 80, 80) | 紧急加速度颜色 |
| `property_left_` | int | 896 | 绘图窗口左边缘位置 [px] |
| `property_top_` | int | 128 | 绘图窗口上边缘位置 [px] |
| `property_length_` | int | 256 | 绘图窗口高度 [px] |
| `property_value_height_offset_` | int | 0 | 绘图窗口高度偏移量 [px] |
| `property_value_scale_` | float | 1 / 6.667 | 数值文本缩放比例 |
| `property_emergency_threshold_max_` | float | 1.0 | 紧急情况的最大加速度阈值 [m/s^2] |
| `property_emergency_threshold_min_` | float | -2.5 | 紧急情况的最小加速度阈值 [m/s^2] |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

待定。

<a id="usage"></a>

## 使用方法

1. 启动 RViz，在 Displays 面板中选择 Add。
   ![选择添加](./images/select_add.png)
2. 选择任一 tier4_vehicle_rviz_plugin 插件，然后点击 OK。
   ![选择车辆插件](./images/select_vehicle_plugin.png)
3. 输入要查看状态的话题名称。
   ![选择话题名称](./images/select_topic_name.png)
