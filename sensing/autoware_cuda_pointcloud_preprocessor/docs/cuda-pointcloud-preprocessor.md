# cuda_pointcloud_preprocessor

<a id="purpose"></a>

## 用途

此节点使用 CUDA 实现应用于单个激光雷达点云的所有标准点云预处理算法。
具体实现包括：

- 包围盒裁剪（自车及自车后视镜）
- 畸变校正
- 基于扫描环的离群点过滤

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此节点重新实现了 CPU 版本算法的功能。有关这些算法的更多信息，请参阅[包围盒裁剪](../../autoware_pointcloud_preprocessor/docs/crop-box-filter.md)、[畸变校正](../../autoware_pointcloud_preprocessor/docs/distortion-corrector.md)和[基于扫描环的离群点过滤](../../autoware_pointcloud_preprocessor/docs/ring-outlier-filter.md)文档。

除上述各项算法外，此节点还使用 `cuda_blackboard`。这是一个 CUDA 传输层，可在输入和输出中实现 GPU 显存之间的零拷贝。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------------- | ------------------------------------------------ | ----------------------------------------- |
| `~/input/pointcloud` | `sensor_msgs::msg::PointCloud2` | 输入点云话题。 |
| `~/input/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 输入点云的类型协商话题。 |
| `~/input/twist` | `geometry_msgs::msg::TwistWithCovarianceStamped` | 速度信息话题。 |
| `~/input/imu` | `sensor_msgs::msg::Imu` | IMU 数据话题。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------------------- | ------------------------------------------------ | ---------------------------------------- |
| `~/output/pointcloud` | `sensor_msgs::msg::PointCloud2` | 处理后的点云话题 |
| `~/output/pointcloud/cuda` | `negotiated_interfaces/msg/NegotiatedTopicsInfo` | 处理后的点云协商话题 |

<a id="parameters"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_cuda_pointcloud_preprocessor/schema/cuda_pointcloud_preprocessor.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- CUDA 实现遵循原始 CPU 实现，但不会得到完全相同的数值结果；为充分利用 GPU，需要采用少量近似处理。
- 此节点要求输入点云符合 `autoware::point_types::PointXYZIRCAEDT` 布局，输出点云则使用 `autoware_point_types` 功能包中定义的 `autoware::point_types::PointXYZIRC` 布局。
