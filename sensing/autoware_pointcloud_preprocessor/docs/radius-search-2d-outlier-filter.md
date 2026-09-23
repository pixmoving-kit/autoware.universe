# radius_search_2d_outlier_filter

<a id="purpose"></a>

## 用途

此节点旨在移除昆虫、雨等点云噪声。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

> RadiusOutlierRemoval 滤波器会移除输入点云中在一定范围内邻居数量不足的所有点对应的索引。

上述说明引自 [1]。此功能包使用 `pcl::search::KdTree` [2] 实现。

![二维半径搜索离群点过滤示意图](./image/outlier_filter-radius_search_2d.drawio.svg)

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

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/radius_search_2d_outlier_filter_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

此方法统计以重力方向为轴向的圆柱体内的点数，因此前提是已经移除了地面点。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

[1] <https://pcl.readthedocs.io/projects/tutorials/en/latest/remove_outliers.html>

[2] <https://pcl.readthedocs.io/projects/tutorials/en/latest/kdtree_search.html#kdtree-search>

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
