# cuda_polar_voxel_noise_filter

<a id="purpose"></a>

## 用途

此节点是 [autoware_pointcloud_preprocessor](../../autoware_pointcloud_preprocessor) 中 `PolarVoxelNoiseFilter` 的 CUDA 加速版本。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此节点是 `autoware::pointcloud_preprocessor::PolarVoxelNoiseFilterComponent` 的另一种实现，用于移除低密度极坐标体素，并可根据回波类型分类选择性地抑制次要回波。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------------- | ------------------------------------------------ | ---------------------------------------- |
| `~/input/pointcloud` | `sensor_msgs::msg::PointCloud2` | 输入点云话题。 |
| `~/input/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 输入点云的类型协商话题。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------------------- | ------------------------------------------------ | ------------------------------------------- |
| `~/output/pointcloud` | `sensor_msgs::msg::PointCloud2` | 过滤后的点云话题。 |
| `~/output/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 过滤后点云的类型协商话题。 |

<a id="additional-debug-topics"></a>

#### 附加调试话题

| 名称 | 类型 | 说明 |
| ------------------------------- | ------------------------------------------------ | ----------------------------------- |
| `~/debug/pointcloud_noise` | `sensor_msgs::msg::PointCloud2` | 被归类为噪声的点。 |
| `~/debug/pointcloud_noise/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 噪声点云的协商话题。 |

<a id="parameters"></a>

## 参数

参数的详细定义请参阅 [autoware_pointcloud_preprocessor 中的原始实现](../../autoware_pointcloud_preprocessor/docs/polar-voxel-noise-filter.md)。

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 输入点云必须包含 `intensity` 字段。
- 如果启用 `use_return_type_classification`，输入点云还必须包含 `return_type` 字段。
- CUDA 实现支持 `PointXYZIRC` 输入，以及带预计算极坐标的 `PointXYZIRCAEDT` 输入。
- 由于 CPU 和 GPU 执行浮点运算时存在差异，结果可能无法与 CPU 实现逐位一致。
