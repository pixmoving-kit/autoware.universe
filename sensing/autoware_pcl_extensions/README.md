# autoware_pcl_extensions

<a id="purpose"></a>

## 用途

`autoware_pcl_extensions` 是 PCL 扩展库。此功能包中的体素网格滤波器采用与原始实现不同的算法。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="original-algorithm-1"></a>

### 原始算法 [1]

1. 在输入点云数据上创建三维体素网格
2. 计算每个体素内的质心
3. 用质心近似替代该体素内的所有点

<a id="extended-algorithm"></a>

### 扩展算法

1. 在输入点云数据上创建三维体素网格
2. 计算每个体素内的质心
3. **用距离质心最近的点近似替代该体素内的所有点**

<a id="inputs-outputs"></a>

## 输入／输出

<a id="parameters"></a>

## 参数

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

[1] <https://pointclouds.org/documentation/tutorials/voxel_grid.html>

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
