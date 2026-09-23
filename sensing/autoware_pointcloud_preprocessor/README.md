# autoware_pointcloud_preprocessor

<a id="purpose"></a>

## 用途

`autoware_pointcloud_preprocessor` 功能包包含以下过滤功能：

- 移除离群点
- 裁剪
- 拼接点云
- 校正畸变
- 降采样
- 增密点云

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

各滤波器的算法详情可通过以下链接查看。

| 滤波器名称 | 说明 | 详情 |
| ----------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------- |
| concatenate_data | 订阅多个点云并将其拼接为一个点云 | [链接](docs/concatenate-data.md) |
| crop_box_filter | 移除给定包围盒内的点 | [链接](docs/crop-box-filter.md) |
| distortion_corrector | 补偿一次扫描期间自车运动引起的点云畸变 | [链接](docs/distortion-corrector.md) |
| downsample_filter | 对输入点云进行降采样 | [链接](docs/downsample-filter.md) |
| outlier_filter | 将硬件故障、雨滴和小昆虫引起的点视为噪声并移除 | [链接](docs/outlier-filter.md) |
| polar_voxel_noise_filter | 使用极坐标体素和考虑回波类型的规则移除点云噪声 | [链接](docs/polar-voxel-noise-filter.md) |
| passthrough_filter | 移除指定字段（例如 x、y、z、intensity）超出给定范围的点 | [链接](docs/passthrough-filter.md) |
| pointcloud_accumulator | 累积指定时长内的点云 | [链接](docs/pointcloud-accumulator.md) |
| pointcloud_densifier | 利用前序帧的信息增强稀疏点云 | [链接](docs/pointcloud-densifier.md) |
| vector_map_filter | 使用矢量地图移除车道外的点 | [链接](docs/vector-map-filter.md) |
| vector_map_inside_area_filter | 移除矢量地图中属于参数指定类型的区域内的点 | [链接](docs/vector-map-inside-area-filter.md) |

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------- | ----------------- |
| `~/input/points` | `sensor_msgs::msg::PointCloud2` | 参考点 |
| `~/input/indices` | `pcl_msgs::msg::Indices` | 参考索引 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------- | --------------- |
| `~/output/points` | `sensor_msgs::msg::PointCloud2` | 过滤后的点 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

| 名称 | 类型 | 默认值 | 说明 |
| ------------------ | ------ | ------------- | ------------------------------------- |
| `input_frame` | string | " " | 输入坐标系 ID |
| `output_frame` | string | " " | 输出坐标系 ID |
| `max_queue_size` | int | 5 | 输入／输出话题的最大队列长度 |
| `use_indices` | bool | false | 是否使用点云索引 |
| `latched_indices` | bool | false | 是否锁存点云索引 |
| `approximate_sync` | bool | false | 是否使用近似同步选项 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

由于[此问题](https://github.com/ros-perception/perception_pcl/issues/9)，
`autoware::pointcloud_preprocessor::Filter` 基于 pcl_perception [1] 实现。

<a id="measuring-the-performance"></a>

## 测量性能

在 Autoware 中，每个激光雷达传感器的点云数据都会先在传感器处理流水线中进行预处理，
再输入感知流水线。预处理阶段如下图所示：

![点云预处理流水线](docs/image/pointcloud_preprocess_pipeline.drawio.png)

流水线中的每个阶段都会产生处理延迟。此前通常使用 `ros2 topic delay /topic_name` 来测量
消息头时间戳与当前时间之间的差值。这种方法适用于小型消息。然而，
对于大型点云消息，该方法会引入额外延迟，主要原因是
从外部访问这些大型点云消息会影响流水线的性能。

传感器处理／感知节点设计为在可组合节点容器中运行，并使用进程内
通信。从外部订阅这些消息（例如使用 ros2 topic delay 或 rviz2）会增加延迟，
甚至使流水线变慢。因此，这样测得的结果不准确。

为缓解这一问题，我们采用由流水线中各节点自行报告流水线延迟的方法。
该方法保持了进程内通信的完整性，并能更准确地测量
流水线中的延迟。

<a id="benchmarking-the-pipeline"></a>

### 流水线性能基准测试

流水线中的节点会报告流水线延迟，即从传感器驱动输出点云
到该节点输出数据所经过的时间。该数据对于评估流水线的运行状况和效率非常关键。

运行 Autoware 时，可以订阅以下 ROS 2 话题，监控流水线中
各节点的流水线延迟：

- `/sensing/lidar/LidarX/crop_box_filter_self/debug/pipeline_latency_ms`
- `/sensing/lidar/LidarX/crop_box_filter_mirror/debug/pipeline_latency_ms`
- `/sensing/lidar/LidarX/distortion_corrector/debug/pipeline_latency_ms`
- `/sensing/lidar/LidarX/ring_outlier_filter/debug/pipeline_latency_ms`
- `/sensing/lidar/concatenate_data_synchronizer/debug/sensing/lidar/LidarX/pointcloud/pipeline_latency_ms`

这些话题提供流水线延迟信息，便于了解从 LidarX 传感器输出
到各后续节点之间不同阶段的延迟。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

[1] <https://github.com/ros-perception/perception_pcl/blob/ros2/pcl_ros/src/pcl_ros/filters/filter.cpp>

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
