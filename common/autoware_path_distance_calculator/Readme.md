# autoware_path_distance_calculator

<a id="purpose"></a>

## 用途

此节点发布从自车当前位置到路线终点的剩余距离。
该距离是沿路线规划的 Lanelet 序列，从当前位置到终点的弧长，而不是两点之间的欧氏距离。

<a id="inner-workings-algorithms"></a>

## 内部机制与算法

每次定时器触发时（1 Hz），执行以下步骤：

1. 如果收到了新的地图或路线消息，则将其交给距离计算器。
   - 收到新地图时，构建或重新构建 Lanelet2 地图。
   - 收到新路线时，根据 `LaneletRoute::segments[].preferred_primitive.id` 直接在地图中查找并缓存 Lanelet 序列。该序列是 mission_planner 实际规划的路线所经过的序列，已考虑车道排除、必要的变道等情况。因此直接使用此序列，而不通过独立的最短路径搜索重新推导路径，以免与实际规划的路线不一致。
2. 将自车位置匹配到缓存序列中的某个 Lanelet，然后计算剩余距离，其值为以下三部分之和：当前 Lanelet 中的剩余长度、当前 Lanelet 与终点 Lanelet 之间所有 Lanelet 的完整长度，以及终点 Lanelet 内到目标位姿的长度。
3. 发布结果。如果地图或路线尚未就绪，或者无法将自车位置匹配到缓存序列，则本次定时器触发不发布结果。

核心计算逻辑（`src/main.cpp` 中的 `RouteDistanceCalculator`）不依赖 ROS 节点；`src/ros_interface.cpp` 仅负责围绕该计算逻辑连接轮询订阅器、定时器和发布器。

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| `~/input/route` | `autoware_planning_msgs::msg::LaneletRoute` | 路线，用于解析目标位置和规划的 Lanelet 序列 |
| `~/input/map` | `autoware_map_msgs::msg::LaneletMapBin` | Lanelet2 地图，用于按 ID 查找 Lanelet |
| `/tf` | `tf2_msgs/TFMessage` | TF（自车位姿） |

默认情况下（见 `launch/path_distance_calculator.launch.xml`），`~/input/route` 重映射到 `/planning/mission_planning/route`，`~/input/map` 重映射到 `/map/vector_map`。

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ------------ | --------------------------------------------------- | --------------------------------------------------------------- |
| `~/distance` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 从自车当前位置到路线终点的剩余距离 [m] |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

无。

<a id="core-parameters"></a>

### 核心参数

无。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

- 收到新路线消息时，根据 `LaneletRoute::segments` 缓存 Lanelet 序列；沿同一路线行驶期间不会重新计算此序列。如果车辆在未重新规划路线的情况下暂时离开缓存的 Lanelet 序列（例如变道绕过障碍物），则无法将自车位置匹配到缓存序列。在车辆返回该序列中的某个 Lanelet 之前，不会发布距离。
- 只有收到路线和地图，且自车位置能够匹配到缓存序列中的某个 Lanelet 后，才会发布距离。
