# autoware_compare_map_segmentation

<a id="purpose"></a>

## 用途

`autoware_compare_map_segmentation` 功能包利用地图信息（如 pcd、高程地图或 map_loader 接口提供的分块地图点云），从输入点云中过滤地面点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="compare-elevation-map-filter"></a>

### 高程地图比较滤波器

将输入点的 z 值与 elevation_map 中的值比较。通过相邻单元的二元积分计算高度差，并移除高度差低于 `height_diff_thresh` 的点。

<p align="center">
  <img src="./media/compare_elevation_map.png" width="1000">
</p>

<a id="distance-based-compare-map-filter"></a>

### 基于距离的地图比较滤波器

此滤波器使用 `kdtree` 的 `nearestKSearch` 函数比较输入点云与地图点云，并移除靠近地图点云的点。地图点云可在开始时一次性静态加载，也可随车辆移动动态加载。

<a id="voxel-based-approximate-compare-map-filter"></a>

### 基于体素的近似地图比较滤波器

此滤波器加载地图点云（可在开始时静态加载，也可在车辆运动时动态加载），并创建地图点云的体素网格。它结合使用 VoxelGrid 类的 getCentroidIndexAt 与 getGridCoordinates 函数，查找并移除位于体素网格内部的输入点。

<a id="voxel-based-compare-map-filter"></a>

### 基于体素的地图比较滤波器

此滤波器加载地图点云（开始时一次性静态加载整幅地图，或在车辆移动时动态加载），并利用 VoxelGrid 对地图点云进行下采样。

对于输入点云中的每个点，滤波器结合使用 VoxelGrid 类的 `getCentroidIndexAt` 与 `getGridCoordinates` 函数，检查输入点附近是否存在下采样后的地图点。如果包含该点或靠近该点的体素中存在下采样后的地图点，则移除该输入点。

<a id="voxel-distance-based-compare-map-filter"></a>

### 基于体素距离的地图比较滤波器

此滤波器结合了 distance_based_compare_map_filter 和 voxel_based_approximate_compare_map_filter。它加载地图点云（可在开始时静态加载，也可在车辆运动时动态加载），并创建地图点云的体素网格和 k-d 树。滤波器结合使用 VoxelGrid 类的 getCentroidIndexAt 与 getGridCoordinates 函数，查找并移除位于体素网格内部的输入点。对于不属于任何体素网格的点，再使用 k-d 树的 radiusSearch 函数与地图点云比较，若距离地图足够近则移除。

<a id="lanelet-elevation-filter"></a>

### Lanelet 高程滤波器

Lanelet 高程滤波器根据 lanelet 高程信息过滤点云。它从 lanelet 数据创建基于网格的高程地图，并滤除明显偏离预期路面高度的点。此滤波器有助于移除悬浮物体、立交桥结构及其他不应纳入地面导航的非道路元素。

滤波器处理 lanelet 地图，以规则网格间隔提取高程信息，并据此验证输入点云数据。高于或低于预期 lanelet 表面高程过多的点将被滤除。

如果输入点云坐标系与 target_frame 不同，则在高程检查前先将点变换到 target_frame。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="compare-elevation-map-filter_1"></a>

### 高程地图比较滤波器

<a id="input"></a>

#### 输入

| 名称                    | 类型                            | 说明      |
| ----------------------- | ------------------------------- | ---------------- |
| `~/input/points`        | `sensor_msgs::msg::PointCloud2` | 参考点 |
| `~/input/elevation_map` | `grid_map::msg::GridMap`        | 高程地图    |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------- | --------------- |
| `~/output/points` | `sensor_msgs::msg::PointCloud2` | 过滤后的点 |

<a id="parameters"></a>

#### 参数

| 名称                 | 类型   | 说明                                                                     | 默认值 |
| :------------------- | :----- | :------------------------------------------------------------------------------ | :------------ |
| `map_layer_name`     | string | 高程地图图层名称                                                        | elevation     |
| `map_frame`          | float  | 订阅 elevation_map 前临时使用的地图 frame_id | map           |
| `height_diff_thresh` | float  | 移除高度差低于此值的点 [m]                   | 0.15          |

<a id="lanelet-elevation-filter_1"></a>

### Lanelet 高程滤波器

<a id="input_1"></a>

#### 输入

| 名称                  | 类型                                    | 说明       |
| --------------------- | --------------------------------------- | ----------------- |
| `~/input/pointcloud`  | `sensor_msgs::msg::PointCloud2`         | 输入点云 |
| `~/input/lanelet_map` | `autoware_map_msgs::msg::LaneletMapBin` | lanelet 地图       |

<a id="output_1"></a>

#### 输出

| 名称                        | 类型                                   | 说明                  |
| --------------------------- | -------------------------------------- | ---------------------------- |
| `~/output/pointcloud`       | `sensor_msgs::msg::PointCloud2`        | 过滤后的点云         |
| `~/debug/elevation_markers` | `visualization_msgs::msg::MarkerArray` | 高程网格可视化 |

<a id="parameters_1"></a>

#### 参数

| 名称                   | 类型   | 说明                                                                               | 默认值                                                               |
| :--------------------- | :----- | :---------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| `grid_resolution`      | double | 高程处理的网格单元尺寸，单位为米                                         | 1.0                                                                         |
| `height_threshold`     | double | 相对于 lanelet 高程的最大高度差（米）                                 | 2.0                                                                         |
| `sampling_distance`    | double | 沿 lanelet 边界采样点之间的距离（米）                         | 0.5                                                                         |
| `extension_count`      | int    | 原始 lanelet 点周围扩展的单元数量                                  | 5                                                                           |
| `target_frame`         | string | 处理所用的目标坐标系                                                    | map                                                                         |
| `cache_directory`      | string | 缓存网格文件的目录 | $(find-pkg-share autoware_compare_map_segmentation)/data/lanelet_grid_cache |
| `require_map_coverage` | bool   | 若为 true，仅保留地图直接覆盖的点；拒绝需要插值的点 | true                                                                        |
| `enable_debug`         | bool   | 启用调试模式（包含高程标记和处理时间发布者） | false                                                                       |

<a id="other-filters"></a>

### 其他滤波器

<a id="input_2"></a>

#### 输入

| 名称                            | 类型                            | 说明                                            |
| ------------------------------- | ------------------------------- | ------------------------------------------------------ |
| `~/input/points`                | `sensor_msgs::msg::PointCloud2` | 参考点 |
| `~/input/map`                   | `sensor_msgs::msg::PointCloud2` | 地图（静态地图加载时） |
| `/localization/kinematic_state` | `nav_msgs::msg::Odometry`       | 当前自车位姿（动态地图加载时） |

<a id="output_2"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------- | --------------- |
| `~/output/points` | `sensor_msgs::msg::PointCloud2` | 过滤后的点 |

<a id="parameters_2"></a>

#### 参数

| 名称                            | 类型   | 说明                                                                                                                             | 默认值 |
| :------------------------------ | :----- | :-------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `use_dynamic_map_loading`       | bool   | 地图加载模式选择：`true` 为动态地图加载，`false` 为静态地图加载，建议用于未分块的地图点云 | true          |
| `distance_threshold`            | float  | 比较输入点与地图点的距离阈值 [m] | 0.5           |
| `map_update_distance_threshold` | float  | 需要更新地图时的车辆移动距离阈值（动态地图加载时）[m] | 10.0          |
| `map_loader_radius`             | float  | 需要加载的地图半径（动态地图加载时）[m] | 150.0         |
| `timer_interval_ms`             | int    | 检查是否需要更新地图的定时器间隔（动态地图加载时）[ms] | 100           |
| `publish_debug_pcd`             | bool   | 启用后在 `debug/downsampled_map/pointcloud` 发布体素化的更新地图用于调试，可能增加计算开销 | false         |
| `downsize_ratio_z_axis`         | double | 用于减小 z 轴方向 voxel_leaf_size 和邻近点距离阈值的正比例系数 | 0.5           |

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
