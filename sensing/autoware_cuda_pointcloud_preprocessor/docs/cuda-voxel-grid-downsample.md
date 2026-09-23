# cuda_voxel_grid_downsample_filter

<a id="purpose"></a>

## 用途

此节点是 [autoware_cuda_pointcloud_preprocessor](../../autoware_pointcloud_preprocessor/README.md) 中 `FasterVoxelGridDownsampleFilter` 的 CUDA 加速版本。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此节点重新实现了 `autoware::pointcloud_preprocessor::FasterVoxelGridDownsampleFilter` 的功能，计算各体素的质心作为代表点。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------------- | ------------------------------------------------ | ----------------------------------------- |
| `~/input/pointcloud` | `sensor_msgs::msg::PointCloud2` | 输入点云话题。 |
| `~/input/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 输入点云的类型协商话题。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| `~/output/pointcloud` | `sensor_msgs::msg::PointCloud2` | 处理后的点云话题（采用 `PointXYZIRC` 格式） |
| `~/output/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 处理后的点云协商话题 |

<a id="parameters"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_cuda_pointcloud_preprocessor/schema/cuda_voxel_grid_downsample_filter.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 此节点要求输入点云与 `autoware::point_types::PointXYZI` 兼容（[参考](https://github.com/autowarefoundation/autoware_core/tree/main/common/autoware_point_types)）。这里的“兼容”指点云字段以 XYZI 的顺序开头。`intensity` 字段支持 `uint8` 和 `float` 等多种数据类型。
