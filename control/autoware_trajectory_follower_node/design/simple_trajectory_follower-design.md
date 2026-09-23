<a id="simple-trajectory-follower"></a>

# 简单轨迹跟踪器

<a id="purpose"></a>

## 目的

提供简单、灵活的基础轨迹跟踪器代码。本节点根据参考轨迹和自车运动学状态计算控制命令。

<a id="design"></a>

## 设计

<a id="inputs-outputs"></a>

### 输入与输出

输入

- `input/reference_trajectory` [autoware_planning_msgs::msg::Trajectory]：待跟踪的参考轨迹。
- `input/current_kinematic_state` [nav_msgs::msg::Odometry]：车辆当前状态（位置、速度等）。
- 输出
- `output/control_cmd` [autoware_control_msgs::msg::Control]：生成的控制命令。

<a id="parameters"></a>

### 参数

| 名称 | 类型 | 说明 | 默认值 |
| :---------------------- | :---- | :----------------------------------------------------------------------------------------------------------------- | :------------ |
| use_external_target_vel | bool | 为 true 时使用参数定义的外部目标速度，否则跟踪目标轨迹点上的速度。 | false |
| external_target_vel | float | `use_external_target_vel` 为 true 时使用的目标速度。 | 0.0 |
| lateral_deviation | float | 跟踪时的目标横向偏差。 | 0.0 |
