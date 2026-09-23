# vector_map_inside_area_filter

<a id="purpose"></a>

## 用途

`vector_map_inside_area_filter` 节点用于移除矢量地图中属于参数指定类型的区域内的点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

- 获取矢量地图中类型与 `polygon_type` 参数指定值一致的区域
- 提取与输入点包围盒相交的矢量地图区域，以降低计算成本
- 根据提取的矢量地图区域创建二维多边形
- 移除位于该多边形内的输入点
- 如果使用 z 值过滤，则移除低于 z 阈值的点

![矢量地图区域内过滤示意图](./image/vector_map_inside_area_filter_overview.svg)

<a id="inputs-outputs"></a>

## 输入／输出

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，另请参阅 [README](../README.md)。

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| -------------------- | --------------------------------------- | ------------------------------------ |
| `~/input` | `sensor_msgs::msg::PointCloud2` | 输入点 |
| `~/input/vector_map` | `autoware_map_msgs::msg::LaneletMapBin` | 用于过滤点的矢量地图 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ---------- | ------------------------------- | --------------- |
| `~/output` | `sensor_msgs::msg::PointCloud2` | 过滤后的点 |

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/vector_map_inside_area_filter_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制
