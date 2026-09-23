# pointcloud_accumulator

<a id="purpose"></a>

## 用途

`pointcloud_accumulator` 节点用于累积指定时长内的点云。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---------------- | ------------------------------- | ---------------- |
| `~/input/points` | `sensor_msgs::msg::PointCloud2` | 参考点 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------- | --------------- |
| `~/output/points` | `sensor_msgs::msg::PointCloud2` | 过滤后的点 |

<a id="parameters"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/pointcloud_accumulator_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
