# autoware_goal_distance_calculator

<a id="purpose"></a>

## 用途

此节点发布自车位姿相对于目标位姿的偏差。

<a id="inner-workings-algorithms"></a>

## 内部机制与算法

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---------------------------------- | ------------------------------------ | --------------------- |
| `/planning/mission_planning/route` | `autoware_planning_msgs::msg::Route` | 用于获取目标位姿 |
| `/tf` | `tf2_msgs/TFMessage` | TF（自车位姿） |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ------------------------ | --------------------------------------------------- | ------------------------------------------------------------- |
| `deviation/lateral` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 发布自车位姿相对于目标位姿的横向偏差 [m] |
| `deviation/longitudinal` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 发布自车位姿相对于目标位姿的纵向偏差 [m] |
| `deviation/yaw` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 发布自车位姿相对于目标位姿的偏航角偏差 [rad] |
| `deviation/yaw_deg` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 发布自车位姿相对于目标位姿的偏航角偏差 [deg] |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

| 名称 | 类型 | 默认值 | 说明 |
| ------------- | ------ | ------------- | --------------------------- |
| `update_rate` | double | 10.0 | 定时器回调周期。[Hz] |

<a id="core-parameters"></a>

### 核心参数

| 名称 | 类型 | 默认值 | 说明 |
| --------- | ---- | ------------- | ------------------------------------------ |
| `oneshot` | bool | true | 仅发布一次偏差，还是重复发布 |

<a id="assumptions-known-limits"></a>

## 假设与已知限制

待补充。
