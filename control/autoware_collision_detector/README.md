<a id="collision-detector"></a>

# 碰撞检测器

<a id="purpose"></a>

## 目的

如果检测到当前自车轮廓发生碰撞，本模块会发布 `ERROR` 诊断。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="flow-chart"></a>

### 流程图

1. 检查输入数据。
2. 过滤动态物体。
3. 查找最近物体及其到自车的距离。
4. 根据最近的碰撞检测结果发布 `ERROR` 诊断。

<a id="algorithms"></a>

### 算法

<a id="check-data"></a>

### 数据检查

检查 `collision_detector` 是否收到去地面点云和动态物体数据。

<a id="object-filtering"></a>

### 物体过滤

<a id="recognition-assumptions"></a>

#### 识别假设

1. 如果分类发生变化，但仍被认定为同一个物体，则 uuid 不变。
2. 某个 uuid 丢失几帧后，仍可能被再次识别。
3. 一旦某个物体被判定为排除对象，就会在一定时间内持续排除。

<a id="filtering-process"></a>

#### 过滤过程

1. 首次识别与排除：
   - 系统检查新识别物体的分类是否列在 `nearby_object_type_filters` 中。
   - 支持的标签与 `autoware_perception_msgs/msg/ObjectClassification.msg` 一致：
     `UNKNOWN`、`CAR`、`TRUCK`、`BUS`、`TRAILER`、`MOTORCYCLE`、`BICYCLE`、`PEDESTRIAN`、
     `ANIMAL`、`HAZARD`、`OVER_DRIVABLE` 和 `UNDER_DRIVABLE`。
   - 如果是，且物体位于 `nearby_filter_radius` 范围内，则标记为排除对象。

2. 新物体判定：
   - 根据 UUID 判断物体是否为“新物体”。
   - 如果在近期帧数据中找不到该 UUID，则将其视为新物体。

3. 排除机制：
   - 通过 UUID 记录新排除的物体。
   - 只要这些物体保持 `nearby_object_type_filters` 中指定的分类，并且仍位于 `nearby_filter_radius` 范围内，就会在设定时间（`keep_ignoring_time`）内持续排除。

<a id="get-distance-to-nearest-object"></a>

### 获取到最近物体的距离

计算自车与最近物体之间的距离。
此函数计算自车多边形与点云中所有点及动态物体多边形之间的最小距离。
如果最小距离小于 `collision_distance` 参数，则判定检测到碰撞。

<a id="time-buffer-and-distance-hysteresis"></a>

### 时间缓冲与距离滞回

在发布 `ERROR` 诊断之前，必须持续检测到碰撞至少达到 `time_buffer.on` 规定的时间。
一旦发布 `ERROR` 诊断，就使用 `time_buffer.off_distance_hysteresis` 扩大自车轮廓，
使碰撞更容易被检测到。
要停止发布 `ERROR` 诊断，必须持续未检测到碰撞至少达到 `time_buffer.off` 规定的时间。

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------ |
| `/perception/obstacle_segmentation/pointcloud` | `sensor_msgs::msg::PointCloud2` | 自车应停车或避让的障碍物点云 |
| `/perception/object_recognition/objects` | `autoware_perception_msgs::msg::PredictedObjects` | 动态物体 |
| `/tf` | `tf2_msgs::msg::TFMessage` | TF |
| `/tf_static` | `tf2_msgs::msg::TFMessage` | 静态 TF |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------- | --------------------------------------- | ------------- |
| `/diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 诊断信息 |
| `~/debug_markers` | `visualization_msgs::msg::MarkerArray` | 调试标记 |

<a id="parameters"></a>

## 参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------------------ | :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `use_pointcloud` | `bool` | 使用点云进行障碍物检查 | `true` |
| `use_dynamic_object` | `bool` | 使用动态物体进行障碍物检查 | `true` |
| `collision_distance` | `double` | 将物体判定为碰撞对象的距离阈值。[m] | 0.15 |
| `nearby_filter_radius` | `double` | 物体过滤的距离范围，考虑该半径内的物体。[m] | 5.0 |
| `keep_ignoring_time` | `double` | 持续过滤首次出现在附近的物体的时间。[sec] | 10.0 |
| `nearby_object_type_filters` | `object of bool values` | 指定要过滤的物体类型。支持的键为 `filter_unknown`、`filter_car`、`filter_truck`、`filter_bus`、`filter_trailer`、`filter_motorcycle`、`filter_bicycle`、`filter_pedestrian`、`filter_animal`、`filter_hazard`、`filter_over_drivable` 和 `filter_under_drivable`。仅过滤值为 `true` 的类型。 | `{filter_unknown: true, filter_animal: true, filter_hazard: true, filter_over_drivable: true, filter_under_drivable: true, others: false}` |
| `ignore_behind_rear_axle` | `bool` | 若为 true，则忽略自车后轴后方检测到的碰撞 | `true` |
| `time_buffer.on` | `double` | [s] 触发 ERROR 诊断前所需的最短连续检测时间 | 0.2 |
| `time_buffer.off` | `double` | [s] 解除 ERROR 诊断前所需的最短连续无碰撞时间（包括滞回区域） | 5.0 |
| `time_buffer.off_distance_hysteresis` | `double` | [m] 诊断触发后用于碰撞检测的额外距离 | 1.0 |

<a id="assumptions-known-limits"></a>

## 假设与已知限制

- 本模块基于 `surround_obstacle_checker`。
