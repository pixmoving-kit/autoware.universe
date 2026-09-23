# dual_return_outlier_filter

<a id="purpose"></a>

## 用途

此节点旨在移除雾、雨等产生的点云噪声，并通过诊断话题发布能见度。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此节点根据衰减因子，将物体反射的光按两个阶段处理，以去除雨雾噪声。如下图所示，它使用按衰减因子区分的两类回波数据去除噪声，因此命名为 `dual_return_outlier_filter`。

![离群点过滤的回波类型](./image/outlier_filter-return_type.drawio.svg)

因此，使用此节点时，传感器驱动必须发布包含 `return_type` 的自定义数据。请参阅 [PointXYZIRCAEDT](https://github.com/autowarefoundation/autoware_core/blob/main/common/autoware_point_types/include/autoware/point_types/types.hpp#L95-L116) 数据结构。

此节点还会通过诊断话题发布能见度。例如，在暴雨情况下，传感器模块可借助此功能通知系统其处理能力已达到极限，从而帮助保障车辆安全。

在某些复杂道路场景中，植物、树叶、塑料网等正常物体也会产生两阶段反射，导致晴天条件下能见度估计值下降。为处理这一情况，增加了可选的感兴趣区域（ROI）设置。

1. `Fixed_xyz_ROI` 模式：根据自车周围固定长方体区域内的弱回波点估计能见度，该区域以 base_link 坐标系中的 x、y、z 定义。
2. `Fixed_azimuth_ROI` 模式：根据自车周围固定区域内的弱回波点估计能见度，该区域以激光雷达坐标系中的方位角和距离定义。

选择这两种固定 ROI 模式时，由于弱回波点的统计范围缩小，能见度估计的灵敏度会降低，因此需要在 `weak_first_local_noise_threshold` 和 `visibility_threshold` 之间进行权衡。

![双回波离群点过滤概览](./image/outlier_filter-dual_return_overall.drawio.svg)

下图介绍节点的工作方式。
![双回波离群点过滤细节](./image/outlier_filter-dual_return_detail.drawio.svg)

下图展示 ROI 选项。

![双回波离群点过滤的 ROI 设置选项](./image/outlier_filter-dual_return_ROI_setting_options.png)

<a id="inputs-outputs"></a>

## 输入／输出

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ---------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| `/dual_return_outlier_filter/frequency_image` | `sensor_msgs::msg::Image` | 表示能见度的直方图图像 |
| `/dual_return_outlier_filter/visibility` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 以 0 到 1 的数值表示能见度 |
| `/dual_return_outlier_filter/pointcloud_noise` | `sensor_msgs::msg::Pointcloud2` | 作为噪声被移除的点云 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/dual_return_outlier_filter_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

目前仍在开发中，不建议使用。
输入必须为包含 `return_type` 的 [PointXYZIRCAEDT](https://github.com/autowarefoundation/autoware_core/blob/main/common/autoware_point_types/include/autoware/point_types/types.hpp#L95-L116) 类型数据。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
