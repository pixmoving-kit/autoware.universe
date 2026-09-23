# autoware_pose2twist

<a id="purpose"></a>

## 用途

`autoware_pose2twist` 根据输入位姿的历史数据计算速度。除计算出的速度消息外，此节点还以浮点消息输出 linear-x 和 angular-z 分量，便于调试。

`twist.linear.x` 按 `sqrt(dx * dx + dy * dy + dz * dz) / dt` 计算，`y` 和 `z` 字段的值为零。
`twist.angular` 的各字段均按 `relative_rotation_vector / dt` 计算。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---- | ------------------------------- | ------------------------------------------------- |
| pose | geometry_msgs::msg::PoseStamped | 用于计算速度的位姿来源。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| --------- | ------------------------------------------------- | --------------------------------------------- |
| twist | geometry_msgs::msg::TwistStamped | 根据输入位姿历史计算得到的速度。 |
| linear_x | autoware_internal_debug_msgs::msg::Float32Stamped | 输出速度的 linear-x 字段。 |
| angular_z | autoware_internal_debug_msgs::msg::Float32Stamped | 输出速度的 angular-z 字段。 |

<a id="parameters"></a>

## 参数

无。

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

无。
