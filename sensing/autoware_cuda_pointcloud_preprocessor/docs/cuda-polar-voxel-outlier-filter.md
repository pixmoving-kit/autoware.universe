# cuda_polar_voxel_outlier_filter

<a id="purpose"></a>

## 用途

此节点是 [autoware_cuda_pointcloud_preprocessor](../../autoware_pointcloud_preprocessor/README.md) 中 `PolarVoxelOutlierFilter` 的 CUDA 加速版本。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此节点是 `autoware::pointcloud_preprocessor::PolarVoxelOutlierFilterComponent` 的另一种实现，基于极坐标空间中的体素而非笛卡尔坐标空间中的体素过滤离群点。

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
| -------------------------- | ------------------------------------------------ | ---------------------------------------- |
| `~/output/pointcloud` | `sensor_msgs::msg::PointCloud2` | 处理后的点云话题 |
| `~/output/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 处理后的点云协商话题 |

<a id="additional-debug-topics"></a>

#### 附加调试话题

| 名称 | 类型 | 说明 |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------ |
| `~/debug/filter_ratio` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 输出点数与输入点数之比 |
| `~/debug/visibility` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 通过次要回波阈值测试的体素比例（仅限 PointXYZIRCAEDT） |
| `~/debug/pointcloud_noise` | `sensor_msgs::msg::PointCloud2` | 处理后被归类为离群点的点云话题 |
| `~/debug/pointcloud_noise/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 协商话题 |

<a id="parameters"></a>

## 参数

详情请参阅 [autoware_cuda_pointcloud_preprocessor 中的原始实现](../../autoware_pointcloud_preprocessor/docs/polar-voxel-outlier-filter.md)。

<a id="core-parameters-schema-based"></a>

### 核心参数（基于模式定义）

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/polar_voxel_outlier_filter_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

由于 CPU 与 GPU 的浮点运算存在差异，`autoware::pointcloud_preprocessor::PolarVoxelOutlierFilterComponent` 与此滤波器的输出可能不完全一致。

添加以下编译器选项可以减小数值差异，但也可能略微影响性能，而且仍难以获得完全一致的结果。

```CMake
list(APPEND CUDA_NVCC_FLAGS "--fmad=false")
```

为优先保证性能，未启用这些编译器选项。
