# autoware_joy_controller

<a id="role"></a>

## 作用

`autoware_joy_controller` 功能包将手柄消息转换为车辆的 Autoware 命令（例如方向盘、挡位、转向灯、接管使能）。

<a id="usage"></a>

## 使用方法

<a id="ros-2-launch"></a>

### ROS 2 启动

```bash
# With default config (ds4)
ros2 launch autoware_joy_controller joy_controller.launch.xml

# Default config but select from the existing parameter files
ros2 launch autoware_joy_controller joy_controller_param_selection.launch.xml joy_type:=ds4 # or g29, p65, xbox

# Override the param file
ros2 launch autoware_joy_controller joy_controller.launch.xml config_file:=/path/to/your/param.yaml
```

<a id="input-output"></a>

## 输入与输出

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| ------------------ | ----------------------- | --------------------------------- |
| `~/input/joy` | sensor_msgs::msg::Joy | 手柄控制器命令 |
| `~/input/odometry` | nav_msgs::msg::Odometry | 用于获取 twist 的自车里程计 |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| ----------------------------------- | --------------------------------------------------- | ---------------------------------------- |
| `~/output/control_command` | autoware_control_msgs::msg::Control | 横向和纵向控制命令 |
| `~/output/external_control_command` | tier4_external_api_msgs::msg::ControlCommandStamped | 横向和纵向控制命令 |
| `~/output/shift` | tier4_external_api_msgs::msg::GearShiftStamped | 挡位命令 |
| `~/output/turn_signal` | tier4_external_api_msgs::msg::TurnSignalStamped | 转向灯命令 |
| `~/output/gate_mode` | tier4_control_msgs::msg::GateMode | 命令门控模式（Auto 或 External） |
| `~/output/heartbeat` | tier4_external_api_msgs::msg::Heartbeat | 心跳 |
| `~/output/vehicle_engage` | autoware_vehicle_msgs::msg::Engage | 车辆接管使能 |

<a id="parameters"></a>

## 参数

| 参数 | 类型 | 说明 |
| ------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------ |
| `joy_type` | string | 手柄控制器类型（默认：DS4） |
| `update_rate` | double | 控制命令的发布频率 |
| `accel_ratio` | double | 计算加速度的比例（命令加速度为 ratio \* operation） |
| `brake_ratio` | double | 计算减速度的比例（命令加速度为 -ratio \* operation） |
| `steer_ratio` | double | 计算减速度的比例（命令转向为 ratio \* operation） |
| `steering_angle_velocity` | double | 操作使用的转向角速度 |
| `accel_sensitivity` | double | 为外部 API 计算加速度的灵敏度（命令加速度为 pow(operation, 1 / sensitivity)） |
| `brake_sensitivity` | double | 为外部 API 计算减速度的灵敏度（命令加速度为 pow(operation, 1 / sensitivity)） |
| `raw_control` | bool | 若为 true，则跳过输入里程计 |
| `velocity_gain` | double | 根据加速度计算速度的比例 |
| `max_forward_velocity` | double | 前进的最大速度绝对值 |
| `max_backward_velocity` | double | 后退的最大速度绝对值 |
| `backward_accel_ratio` | double | 计算减速度的比例（命令加速度为 -ratio \* operation） |

<a id="p65-joystick-key-map"></a>

## P65 手柄按键映射

| 操作 | 按键 |
| -------------------- | --------------------- |
| 加速 | R2 |
| 制动 | L2 |
| 转向 | 左摇杆左右移动 |
| 升挡 | 方向键上 |
| 降挡 | 方向键下 |
| 切换前进挡 | 方向键左 |
| 切换倒挡 | 方向键右 |
| 左转向灯 | L1 |
| 右转向灯 | R1 |
| 关闭转向灯 | A |
| 命令门控模式 | B |
| 紧急停车 | Select |
| 解除紧急停车 | Start |
| 启用 Autoware 接管 | X |
| 解除 Autoware 接管 | Y |
| 启用车辆接管 | PS |
| 解除车辆接管 | 右扳机键 |

<a id="ds4-joystick-key-map"></a>

## DS4 手柄按键映射

| 操作 | 按键 |
| -------------------- | -------------------------- |
| 加速 | R2、× 或右摇杆向上 |
| 制动 | L2、□ 或右摇杆向下 |
| 转向 | 左摇杆左右移动 |
| 升挡 | 方向键上 |
| 降挡 | 方向键下 |
| 切换前进挡 | 方向键左 |
| 切换倒挡 | 方向键右 |
| 左转向灯 | L1 |
| 右转向灯 | R1 |
| 关闭转向灯 | SHARE |
| 命令门控模式 | OPTIONS |
| 紧急停车 | PS |
| 解除紧急停车 | PS |
| 启用 Autoware 接管 | ○ |
| 解除 Autoware 接管 | ○ |
| 启用车辆接管 | △ |
| 解除车辆接管 | △ |

<a id="xbox-joystick-key-map"></a>

## XBOX 手柄按键映射

| 操作 | 按键 |
| -------------------- | --------------------- |
| 加速 | RT |
| 制动 | LT |
| 转向 | 左摇杆左右移动 |
| 升挡 | 方向键上 |
| 降挡 | 方向键下 |
| 切换前进挡 | 方向键左 |
| 切换倒挡 | 方向键右 |
| 左转向灯 | LB |
| 右转向灯 | RB |
| 关闭转向灯 | A |
| 命令门控模式 | B |
| 紧急停车 | View |
| 解除紧急停车 | Menu |
| 启用 Autoware 接管 | X |
| 解除 Autoware 接管 | Y |
| 启用车辆接管 | 左摇杆按键 |
| 解除车辆接管 | 右摇杆按键 |
