# crop_box_filter

<a id="purpose"></a>

## 用途

`crop_box_filter` 节点用于移除给定包围盒区域内的点。此滤波器用于移除击中车辆自身形成的点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

使用 `pcl::CropBox` 过滤给定包围盒内的所有点。

<a id="inputs-outputs"></a>

## 输入／输出

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/crop_box_filter_node.schema.json") }}

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
