<a id="obstacle-proximity-checker"></a>

# 障碍物接近检查器

<a id="purpose"></a>

## 用途

此软件包提供可复用的库，用于检测自车近旁的障碍物。
它计算扩展后的自车轮廓与附近点云点或动态目标多边形之间的最小距离，并报告是否存在位于调用方指定距离阈值内的障碍物。

核心逻辑提取自 [autoware_surround_obstacle_checker](../../planning/autoware_surround_obstacle_checker/README.md)，以便在规划模块之间共享。
当前使用此库的模块包括：

<a id="inner-workings-algorithms"></a>

## 内部机制与算法

<a id="get-distance-to-nearest-obstacle"></a>

### 获取到最近障碍物的距离

计算自车与最近障碍物之间的距离。
计算自车轮廓与以下对象之间的最小距离：

- 输入点云中的所有点
- 已启用类别的动态目标多边形

根据车辆信息和用户为各类障碍物设置的边距生成自车轮廓。对于每种障碍物类型（如汽车、行人、点云等），定义以下边距：

- `surround_check_front_distance`
- `surround_check_side_distance`
- `surround_check_back_distance`

<a id="obstacle-detection"></a>

### 障碍物检测

满足以下条件时，认为障碍物位于近旁：

```text
nearest_distance < contact_distance_threshold
```

每次调用 `check()` 时，由调用方提供 `contact_distance_threshold`。
这使调用方能够实现距离滞回，例如进入停车状态时使用较小阈值，解除停车状态时使用较大阈值。

此库**不负责**以下事项：

- 检查自车是否停止
- 状态机（`PASS` / `STOP`）
- 基于时间的滞回
- ROS 订阅、发布或 TF 变换

这些职责仍由调用此库的节点或插件承担。

## API

<a id="input-inputs"></a>

### 输入（`Inputs`）

| 字段                     | 类型                                                              | 说明                             |
| ------------------------- | ----------------------------------------------------------------- | --------------------------------------- |
| `ego_pose`                | `geometry_msgs::msg::Pose`                                        | 用于检查动态目标的自车位姿 |
| `pointcloud_in_base_link` | `pcl::PointCloud<pcl::PointXYZ>::ConstPtr`                        | `base_link` 坐标系中的障碍物点云     |
| `objects`                 | `autoware_perception_msgs::msg::PredictedObjects::ConstSharedPtr` | 动态目标                         |

调用方必须先将点云变换到 `base_link` 坐标系，再传入此库。

<a id="output-checkresult"></a>

### 输出（`CheckResult`）

| 字段               | 类型                               | 说明                                                           |
| ------------------- | ---------------------------------- | --------------------------------------------------------------------- |
| `is_obstacle_found` | `bool`                             | 最近障碍物位于 `contact_distance_threshold` 范围内时为 `true` |
| `nearest_obstacle`  | `std::optional<ProximityObstacle>` | 最近的障碍物及其距离（如果存在）                             |

`ProximityObstacle` 包含：

| 字段              | 类型                                | 说明                                            |
| ------------------ | ----------------------------------- | ------------------------------------------------------ |
| `is_point_cloud`   | `bool`                              | 最近障碍物来自点云时为 `true` |
| `nearest_distance` | `double`                            | 到自车轮廓的最小距离 [m]              |
| `nearest_point`    | `geometry_msgs::msg::Point`         | 地图坐标系中的最近点                       |
| `uuid`             | `unique_identifier_msgs::msg::UUID` | 目标 UUID；点云对应的值为空                 |

<a id="parameters-parameters"></a>

### 参数（`Parameters`）

| 名称                       | 类型                                            | 说明                                           |
| -------------------------- | ----------------------------------------------- | ----------------------------------------------------- |
| `pointcloud_enable_check`  | `bool`                                          | 启用点云接近检查                   |
| `object_type_enable_check` | `unordered_map<string, bool>`                   | 按目标类别标签启用检查（`car`、`truck` 等） |
| `obstacle_types_map`       | `unordered_map<string, ObstacleTypeParameters>` | 各类别的轮廓边距                            |

`ObstacleTypeParameters` 字段：

| 名称                            | 类型     | 说明                                 | 典型值 |
| ------------------------------- | -------- | ------------------------------------------- | ------------- |
| `surround_check_front_distance` | `double` | 自车轮廓前方增加的边距 [m] | 0.5           |
| `surround_check_side_distance`  | `double` | 自车轮廓侧面增加的边距 [m]  | 0.0–1.0       |
| `surround_check_back_distance`  | `double` | 自车轮廓后方增加的边距 [m]  | 0.0–0.5       |

支持的目标类别标签：

`unknown`, `car`, `truck`, `bus`, `trailer`, `motorcycle`, `bicycle`, `pedestrian`, `animal`, `hazard`, `over_drivable`, `under_drivable`, `pointcloud`

<a id="assumptions-known-limits"></a>

## 假设与已知限制

- 点云必须已经变换到 `base_link` 坐标系。
- 此库仅进行几何接近检查。停车或解除停车决策、速度限制和轨迹修改由调用方处理。
