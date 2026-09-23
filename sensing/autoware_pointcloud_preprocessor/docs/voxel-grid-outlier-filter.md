# voxel_grid_outlier_filter

<a id="purpose"></a>

## 用途

此节点旨在移除昆虫、雨等点云噪声。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

根据体素内的点数移除点云噪声。
[radius_search_2d_outlier_filter](./radius-search-2d-outlier-filter.md) 的精度更高，而此方法的优势在于计算成本较低。

![体素网格离群点过滤示意图](./image/outlier_filter-voxel_grid.drawio.svg)

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

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/voxel_grid_outlier_filter_node.schema.json") }}

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
