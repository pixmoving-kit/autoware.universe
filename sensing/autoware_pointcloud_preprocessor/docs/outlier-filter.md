# outlier_filter

<a id="purpose"></a>

## 用途

`outlier_filter` 是用于过滤点云离群点的功能包。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

| 滤波器名称 | 说明 | 详情 |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| 二维半径搜索离群点滤波器 | 根据一定半径内的点数移除点云噪声 | [链接](./radius-search-2d-outlier-filter.md) |
| 扫描环离群点滤波器 | 按时间顺序处理扫描，并根据点间距离的变化率移除噪声 | [链接](./ring-outlier-filter.md) |
| 体素网格离群点滤波器 | 根据体素内的点数移除点云噪声 | [链接](./voxel-grid-outlier-filter.md) |
| 极坐标体素离群点滤波器 | 使用极坐标体素移除点云噪声，并针对激光雷达特性进行优化 | [链接](./polar-voxel-outlier-filter.md) |
| 双回波离群点滤波器（开发中） | 根据衰减因子，将物体反射的光按两个阶段处理，以去除雨雾噪声。 | [链接](./dual-return-outlier-filter.md) |

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
