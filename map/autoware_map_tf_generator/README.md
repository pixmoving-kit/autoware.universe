# autoware_map_tf_generator

<a id="purpose"></a>

## 用途

此功能包中的节点广播 `viewer` 坐标系，用于在 RViz 中可视化地图。

请注意，没有任何模块依赖 `viewer` 坐标系；该坐标系仅用于可视化。

支持以下方法计算 `viewer` 坐标系的位置：

- `pcd_map_tf_generator_node` 输出 PCD 中所有点的几何中心。
- `vector_map_tf_generator_node` 输出点图层中所有点的几何中心。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

#### autoware_pcd_map_tf_generator

| 名称                  | 类型                            | 说明                                                       |
| --------------------- | ------------------------------- | ----------------------------------------------------------------- |
| `/map/pointcloud_map` | `sensor_msgs::msg::PointCloud2` | 订阅点云地图，计算 `viewer` 坐标系的位置 |

#### autoware_vector_map_tf_generator

| 名称              | 类型                                    | 说明                                                   |
| ----------------- | --------------------------------------- | ------------------------------------------------------------- |
| `/map/vector_map` | `autoware_map_msgs::msg::LaneletMapBin` | 订阅矢量地图，计算 `viewer` 坐标系的位置 |

<a id="output"></a>

### 输出

| 名称         | 类型                     | 说明               |
| ------------ | ------------------------ | ------------------------- |
| `/tf_static` | `tf2_msgs/msg/TFMessage` | 广播 `viewer` 坐标系 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

无

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("map/autoware_map_tf_generator/schema/map_tf_generator.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

待定。
