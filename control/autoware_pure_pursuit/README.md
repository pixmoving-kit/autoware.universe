<a id="pure-pursuit-controller"></a>

# 纯追踪控制器

纯追踪控制器模块使用纯追踪算法计算跟踪期望轨迹所需的转向角。它作为 `autoware_trajectory_follower_node` 中的横向控制器插件使用。

<a id="inputs"></a>

## 输入

由 [controller_node](../autoware_trajectory_follower_node/README.md) 设置以下内容：

- `autoware_planning_msgs/Trajectory`：需要跟踪的参考轨迹。
- `nav_msgs/Odometry`：当前自车位姿与速度信息。

<a id="outputs"></a>

## 输出

向控制器节点返回包含以下内容的 LateralOutput：

- `autoware_control_msgs/Lateral`：目标转向角。
- LateralSyncData
  - 转向角收敛状态。
- `autoware_planning_msgs/Trajectory`：自车预测路径。

<a id="parameters"></a>

## 参数

{{json_to_markdown("control/autoware_pure_pursuit/schema/pure_pursuit.schema.json")}}
