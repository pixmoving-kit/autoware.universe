# object_lanelet_filter

<a id="purpose"></a>

## 用途

`object_lanelet_filter` 节点使用矢量地图过滤检测目标。
仅发布位于矢量地图内部的目标。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称               | 类型                                             | 说明            |
| ------------------ | ------------------------------------------------ | ---------------------- |
| `input/vector_map` | `autoware_map_msgs::msg::LaneletMapBin`          | 矢量地图 |
| `input/object`     | `autoware_perception_msgs::msg::DetectedObjects` | 输入检测目标 |

<a id="output"></a>

### 输出

| 名称            | 类型                                             | 说明               |
| --------------- | ------------------------------------------------ | ------------------------- |
| `output/object` | `autoware_perception_msgs::msg::DetectedObjects` | 过滤后的检测目标 |

<a id="parameters"></a>

## 参数

`object_lanelet_filter` 节点参数中 `filter_settings` 的说明。

| 名称                                              | 类型     | 说明                                                                                                                           |
| ------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `debug`                                           | `bool`   | 若为 `true`，发布额外调试信息，包括 `~/debug/marker` 话题上的可视化标记，供 RViz 等工具使用。 |
| `lanelet_extra_margin`                            | `double` | 若 `> 0`，以此裕量（米）扩张用于重叠检查的 lanelet 多边形；若 `<= 0`，则禁用多边形扩张。 |
| `lanelet_xy_overlap_filter.enabled`               | `bool`   | 若为 `true`，启用基于目标与 lanelet 多边形重叠情况的过滤。 |
| `lanelet_direction_filter.enabled`                | `bool`   | 若为 `true`，启用基于目标速度方向相对于 lanelet 方向的过滤。 |
| `lanelet_direction_filter.velocity_yaw_threshold` | `double` | 目标速度向量与 lanelet 方向之间的偏航角差阈值（弧度）。 |
| `lanelet_direction_filter.object_speed_threshold` | `double` | 应用方向过滤器所需的目标最低速度（m/s）。 |
| `lanelet_object_elevation_filter.enabled`         | `bool`   | 若为 `true`，启用基于目标相对于最近 lanelet 表面高程的过滤。 |
| `max_elevation_threshold`                         | `double` | 目标相对于最近 lanelet 表面的最大允许高程（米）。 |
| `min_elevation_threshold`                         | `double` | 目标相对于最近 lanelet 表面的最小允许高程（米）。 |
| `lanelet_extra_margin`                            | `double` | 添加至 lanelet 边界的裕量值。 |

<a id="core-parameters"></a>

### 核心参数

| 名称                             | 类型 | 默认值 | 说明                               |
| -------------------------------- | ---- | ------------- | ----------------------------------------- |
| `filter_target_label.UNKNOWN`    | bool | false         | 若为 true，过滤未知目标。 |
| `filter_target_label.CAR`        | bool | false         | 若为 true，过滤汽车目标。 |
| `filter_target_label.TRUCK`      | bool | false         | 若为 true，过滤卡车目标。 |
| `filter_target_label.BUS`        | bool | false         | 若为 true，过滤巴士目标。 |
| `filter_target_label.TRAILER`    | bool | false         | 若为 true，过滤拖车目标。 |
| `filter_target_label.MOTORCYCLE` | bool | false         | 若为 true，过滤摩托车目标。 |
| `filter_target_label.BICYCLE`    | bool | false         | 若为 true，过滤自行车目标。 |
| `filter_target_label.PEDESTRIAN` | bool | false         | 若为 true，过滤行人目标。 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

lanelet 过滤根据目标的形状多边形和包围框进行。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
