# downsample_filter

<a id="purpose"></a>

## 用途

`downsample_filter` 是用于减少点数的节点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="approximate-downsample-filter"></a>

### 近似降采样滤波器

使用 `pcl::VoxelGridNearestCentroid`。算法说明见 [autoware_pcl_extensions](../../autoware_pcl_extensions/README.md)。

<a id="random-downsample-filter"></a>

### 随机降采样滤波器

使用 `pcl::RandomSample`，以均匀概率对点进行采样。

<a id="voxel-grid-downsample-filter"></a>

### 体素网格降采样滤波器

使用 `pcl::VoxelGrid`，以各体素的质心近似替代该体素内的点。

<a id="pickup-based-voxel-grid-downsample-filter"></a>

### 基于选点的体素网格降采样滤波器

此算法从体素中选取一个实际存在的点，而非质心。与基于质心的体素网格滤波器相比，计算成本更低。

<a id="inputs-outputs"></a>

## 输入／输出

这些实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="parameters"></a>

## 参数

<a id="note-parameters"></a>

### 参数说明

这些实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="core-parameters"></a>

### 核心参数

<a id="approximate-downsample-filter_1"></a>

#### 近似降采样滤波器

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/approximate_downsample_filter_node.schema.json") }}

<a id="random-downsample-filter_1"></a>

### 随机降采样滤波器

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/random_downsample_filter_node.schema.json") }}

<a id="voxel-grid-downsample-filter_1"></a>

### 体素网格降采样滤波器

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/voxel_grid_downsample_filter_node.schema.json") }}

<a id="pickup-based-voxel-grid-downsample-filter_1"></a>

### 基于选点的体素网格降采样滤波器

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/pickup_based_voxel_grid_downsample_filter_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

<!-- cspell: ignore martinus -->

此实现使用 martinus 编写的 `robin_hood.h` 哈希库，该库以 MIT 许可证发布于 GitHub 的 [martinus/robin-hood-hashing](https://github.com/martinus/robin-hood-hashing)。特别感谢 martinus 的贡献。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
