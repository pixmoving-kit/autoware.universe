<a id="predicted-path-checker"></a>

# 预测路径检查器

<a id="purpose"></a>

## 目的

预测路径检查器功能包用于自动驾驶车辆，检查控制模块生成的
预测路径。它处理规划模块可能无法应对的潜在碰撞，以及制动
距离内的碰撞。如果碰撞位于制动距离内，功能包发送标记为“ERROR”的诊断消息，提醒
系统触发紧急处理；如果碰撞位于参考轨迹之外，则向暂停接口发送暂停请求，
使车辆停止。

![总体结构](images/general-structure.png)

<a id="algorithm"></a>

## 算法

本功能包将预测轨迹与参考轨迹以及环境中的预测物体进行
比较，检查潜在碰撞，并在必要时生成适当响应来避免碰撞
（紧急处理或暂停请求）。

<a id="inner-algorithm"></a>

### 内部算法

![流程图](images/FlowChart.png)

**cutTrajectory() ->** 按输入长度截取预测轨迹。长度根据自车
速度
与 "trajectory_check_time" 参数的乘积，以及 "min_trajectory_length" 计算。

**filterObstacles() ->** 过滤环境中的预测物体，排除不在
车辆前方以及远离预测轨迹的物体。

**checkTrajectoryForCollision() ->** 检查预测轨迹与预测物体是否碰撞。
计算轨迹点和预测物体的多边形，检查两者是否相交。如果
相交，则计算最近碰撞点，并返回多边形与
预测物体的最近碰撞点。为避免非预期行为，还会检查此前与车辆轮廓相交的
预测物体历史。历史记录保存最近 "chattering_threshold"
秒内检测到的物体。

如果 "enable_z_axis_obstacle_filtering" 为 true，则使用 "z_axis_filtering_buffer"
在 Z 轴方向过滤预测物体。如果物体在 Z 轴方向不相交，则将其过滤掉。

![Z 轴过滤](images/Z_axis_filtering.png)

**calculateProjectedVelAndAcc() ->** 计算预测物体的速度和加速度在
预测轨迹碰撞点坐标轴上的投影。

**isInBrakeDistance() ->** 检查停车点是否在制动距离内。获取自车相对于
预测物体的速度和加速度，并计算制动距离。如果该点位于
制动距离内，则返回 true。

**isItDiscretePoint() ->** 检查预测轨迹上的停车点是否为离散点。如果不是
离散点，则应由规划模块处理停车。

**isThereStopPointOnRefTrajectory() ->** 检查参考轨迹上是否有停车点。如果在停车索引之前
存在停车点，则返回 true；否则返回 false，节点会调用暂停接口
使车辆停止。

<a id="inputs"></a>

## 输入

| 名称 | 类型 | 说明 |
| ------------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| `~/input/reference_trajectory` | `autoware_planning_msgs::msg::Trajectory` | 参考轨迹 |
| `~/input/predicted_trajectory` | `autoware_planning_msgs::msg::Trajectory` | 预测轨迹 |
| `~/input/objects` | `autoware_perception_msgs::msg::PredictedObject` | 环境中的动态物体 |
| `~/input/odometry` | `nav_msgs::msg::Odometry` | 用于获取当前速度的车辆里程计消息 |
| `~/input/current_accel` | `geometry_msgs::msg::AccelWithCovarianceStamped` | 当前加速度 |
| `/control/vehicle_cmd_gate/is_paused` | `tier4_control_msgs::msg::IsPaused` | 车辆当前暂停状态 |

<a id="outputs"></a>

## 输出

| 名称 | 类型 | 说明 |
| ------------------------------------- | ---------------------------------------- | -------------------------------------- |
| `~/debug/marker` | `visualization_msgs::msg::MarkerArray` | 可视化标记 |
| `~/debug/virtual_wall` | `visualization_msgs::msg::MarkerArray` | 用于可视化的虚拟墙标记 |
| `/control/vehicle_cmd_gate/set_pause` | `tier4_control_msgs::srv::SetPause` | 使车辆停止的暂停服务 |
| `/diagnostics` | `diagnostic_msgs::msg::DiagnosticStatus` | 车辆诊断状态 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------------------- | :------- | :-------------------------------------------------------------------- | :------------ |
| `update_rate` | `double` | 更新频率 [Hz] | 10.0 |
| `delay_time` | `double` | 紧急响应考虑的时间延迟 [s] | 0.17 |
| `max_deceleration` | `double` | 自车停车的最大减速度 [m/s^2] | 1.5 |
| `resample_interval` | `double` | 轨迹重采样间隔 [m] | 0.5 |
| `stop_margin` | `double` | 停车余量 [m] | 0.5 |
| `ego_nearest_dist_threshold` | `double` | 自车最近点搜索的距离阈值 [m] | 3.0 |
| `ego_nearest_yaw_threshold` | `double` | 自车最近点搜索的偏航角阈值 [rad] | 1.046 |
| `min_trajectory_check_length` | `double` | 最小轨迹检查长度，单位米 [m] | 1.5 |
| `trajectory_check_time` | `double` | 轨迹检查时间，单位秒 [s] | 3.0 |
| `distinct_point_distance_threshold` | `double` | 离散点的距离阈值 [m] | 0.3 |
| `distinct_point_yaw_threshold` | `double` | 离散点的偏航角阈值 [deg] | 5.0 |
| `filtering_distance_threshold` | `double` | 忽略距离大于此值的物体 [m] | 1.5 |
| `use_object_prediction` | `bool` | 若为 true，节点根据时间差预测物体当前位姿 [-] | true |

<a id="collision-checker-parameters"></a>

### 碰撞检查器参数

| 名称 | 类型 | 说明 | 默认值 |
| :--------------------------------- | :------- | :---------------------------------------------------------------- | :------------ |
| `width_margin` | `double` | 碰撞检查的宽度余量 [Hz] | 0.2 |
| `chattering_threshold` | `double` | 碰撞检测的抖动阈值 [s] | 0.2 |
| `z_axis_filtering_buffer` | `double` | Z 轴过滤缓冲距离 [m] | 0.3 |
| `enable_z_axis_obstacle_filtering` | `bool` | 是否启用 Z 轴障碍物过滤 | false |
