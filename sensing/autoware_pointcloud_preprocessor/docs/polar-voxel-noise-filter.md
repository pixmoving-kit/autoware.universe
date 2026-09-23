<a id="polar-voxel-noise-filter"></a>

# 极坐标体素噪声滤波器

<a id="overview"></a>

## 概述

极坐标体素噪声滤波器是一种在极坐标空间中运行的点云噪声过滤算法，
用于激光雷达数据处理。它既支持基于强度的简单噪声移除，也支持考虑回波类型的过滤，以处理昆虫等小物体以及其他
稀疏噪声。

**主要特性**：

- **灵活的过滤模式**，可选择启用回波类型分类
- 在 PointXYZIRC 与 PointXYZIRCAEDT 之间**自动检测格式**
- 根据半径、方位角和仰角分箱进行**极坐标体素过滤**
- 根据体素平均强度进行**考虑强度的噪声判断**
- 在最终输出中**可选地抑制次要回波**
- **可选的调试支持**，通过发布噪声点云辅助分析

<a id="purpose"></a>

## 用途

此滤波器使用针对激光雷达特性优化的极坐标体素
网格，移除昆虫等点云噪声以及包含大量次要回波的体素。它提供以下可配置的
过滤方法：

1. **简单模式**：根据体素占据情况和平均强度判断噪声
2. **考虑回波类型的模式**：在噪声判断中增加次要回波计数

考虑回波类型的模式适用于这样的传感器：其次要回波更有可能
来自雨等噪声或其他弱反射。

<a id="key-differences-from-cartesian-voxel-grid-filter"></a>

## 与笛卡尔坐标体素网格滤波器的主要区别

<a id="coordinate-system"></a>

### 坐标系

- **笛卡尔坐标体素网格**：使用 `(x, y, z)` 将三维空间划分为规则的立方体体素
- **极坐标体素网格**：使用 `(radius, azimuth, elevation)` 将三维空间划分为极坐标体素

<a id="advantages-of-polar-voxelization"></a>

### 极坐标体素化的优势

1. **自然表示激光雷达数据**：激光雷达按极坐标模式扫描，因此体素结构
   与数据更自然地匹配
2. **考虑距离的空间分箱**：整个扫描过程中的角度分箱保持一致
3. **格式灵活**：既支持笛卡尔坐标输入，也支持预计算的极坐标输入
4. **考虑回波类型的过滤**：可以利用可用的主要／次要回波信息
5. **调试可视化**：可将移除的点发布为独立点云，以辅助调参

<a id="point-cloud-format-support"></a>

## 支持的点云格式

此滤波器支持带强度信息的点云，并自动区分以下两种
格式：

<a id="pointxyzirc-format"></a>

### PointXYZIRC 格式

- **用途**：包含 `(x, y, z, intensity, return_type, channel)` 字段的点格式
- **处理**：根据笛卡尔坐标计算极坐标 `(radius, azimuth, elevation)`
- **回波类型**：启用后，使用 `return_type` 进行分类
- **性能**：性能良好，但存在坐标转换开销

<a id="pointxyzircaedt-format"></a>

### PointXYZIRCAEDT 格式

- **用途**：包含预计算极坐标字段的点云
- **字段**：`(x, y, z, intensity, return_type, channel, azimuth, elevation, distance, time_stamp)`
- **检测**：自动检测是否存在极坐标字段
- **处理**：直接使用预计算的极坐标
- **性能**：避免三角函数转换，因此处理更快

```yaml
# PointXYZIRC: Computes polar coordinates from Cartesian
# - x, y, z       (float32): Cartesian coordinates
# - intensity     (uint8/compatible field): Point intensity
# - return_type   (uint8):   Return type classification
# - channel       (uint16):  Channel information

# PointXYZIRCAEDT: Uses pre-computed polar coordinates
# - x, y, z       (float32): Cartesian coordinates
# - intensity     (uint8/compatible field): Point intensity
# - return_type   (uint8):   Return type classification
# - channel       (uint16):  Channel information
# - azimuth       (float32): Pre-computed azimuth angle
# - elevation     (float32): Pre-computed elevation angle
# - distance      (float32): Pre-computed radius
# - time_stamp    (uint32):  Point timestamp
```

**注意**：此滤波器始终要求存在 `intensity` 字段。只有在
`use_return_type_classification=true` 时才要求存在 `return_type` 字段。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="coordinate-conversion"></a>

### 坐标转换

**对于 PointXYZIRC 格式：**
将每个点 `(x, y, z)` 转换为极坐标：

- **半径**：`r = sqrt(x² + y² + z²)`
- **方位角**：`θ = atan2(y, x)`
- **仰角**：`φ = atan2(z, sqrt(x² + y²))`

**对于 PointXYZIRCAEDT 格式：**
直接使用点字段中预计算的极坐标：

- **半径**：`r = point.distance`
- **方位角**：`θ = point.azimuth`
- **仰角**：`φ = point.elevation`

<a id="voxel-index-calculation"></a>

### 体素索引计算

根据以下索引将各点分配到对应体素：

- **半径索引**：`floor(radius / radial_resolution)`
- **方位角索引**：`floor(azimuth / azimuth_resolution)`
- **仰角索引**：`floor(elevation / elevation_resolution)`

<a id="return-type-classification"></a>

### 回波类型分类

当 `use_return_type_classification=true` 时，根据 `return_type` 字段对点分类：

- **主要回波**：`primary_return_types` 中指定的回波类型
- **次要回波**：未被指定为主要回波的其他所有回波类型
- **分类用途**：仅用于考虑回波类型的噪声判断流程

<a id="filtering-methodology"></a>

### 过滤方法

滤波器根据 `use_return_type_classification` 参数使用不同算法。

<a id="simple-mode-use_return_type_classificationfalse"></a>

#### 简单模式（`use_return_type_classification=false`）

1. **格式检测**：自动区分 PointXYZIRC 与 PointXYZIRCAEDT
2. **坐标处理**：
   - PointXYZIRC：根据笛卡尔坐标计算极坐标
   - PointXYZIRCAEDT：使用预计算的极坐标
3. **体素分箱**：将点分组到极坐标体素中
4. **体素统计**：对每个体素计算：
   - 总点数
   - 平均强度
5. **噪声判断**：
   - 满足以下条件时，将体素视为噪声：
     `point_count <= voxel_points_threshold AND intensity_avg <= avg_intensity_threshold`
6. **输出**：保留非噪声体素中的点

<a id="return-type-aware-mode-use_return_type_classificationtrue"></a>

#### 考虑回波类型的模式（`use_return_type_classification=true`）

1. **格式检测**：自动区分 PointXYZIRC 与 PointXYZIRCAEDT
2. **回波类型验证**：确保存在 `return_type` 字段
3. **坐标处理**：
   - PointXYZIRC：根据笛卡尔坐标计算极坐标
   - PointXYZIRCAEDT：使用预计算的极坐标
4. **回波类型分类**：将点分为主要回波或次要回波
5. **体素统计**：对每个体素计算：
   - 总点数
   - 平均强度
   - 次要回波数量
6. **噪声判断**：
   - 满足以下任一条件时，将体素视为噪声：
     - `point_count <= voxel_points_threshold AND intensity_avg <= avg_intensity_threshold`
     - `secondary_return_count >= secondary_noise_threshold AND intensity_avg <= avg_intensity_threshold`
7. **可选的次要回波过滤**：
   - 当 `filter_secondary_returns=true` 时，即使体素被保留，也只发布其中的主要回波
8. **输出**：过滤后的点云，以及可选的调试噪声点云

<a id="noise-judgement"></a>

### 噪声判断

<a id="simple-mode"></a>

#### 简单模式

- **噪声体素**：
  `point_count <= voxel_points_threshold AND intensity_avg <= avg_intensity_threshold`

<a id="return-type-aware-mode"></a>

#### 考虑回波类型的模式

- **噪声体素**：
  `((point_count <= voxel_points_threshold) OR (secondary_return_count >= secondary_noise_threshold)) AND intensity_avg <= avg_intensity_threshold`

<a id="key-features"></a>

### 主要特性

- **灵活的架构**：可在简单过滤和考虑回波类型的过滤之间配置切换
- **针对格式优化的处理**：自动选择最优坐标来源
- **考虑强度的过滤**：使用体素平均强度，避免移除密集的有效回波
- **次要回波处理**：根据配置利用或抑制次要回波
- **调试支持**：可选地发布噪声点云，用于分析和调参

<a id="inputs-outputs"></a>

## 输入／输出

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅
[README](../README.md)。

<a id="input-requirements"></a>

### 输入要求

- **支持的格式**：PointXYZIRC 或 PointXYZIRCAEDT
- **强度字段**：始终必需
- **回波类型字段**：仅在 `use_return_type_classification=true` 时必需
- **无效输入**：拒绝不含 `intensity` 的点云；在考虑回波类型的模式下，
  还会拒绝不含 `return_type` 的点云

<a id="additional-debug-topics"></a>

### 附加调试话题

| 名称 | 类型 | 说明 |
| --------------------------------------------------- | ------------------------------- | --------------------------------- |
| `~/polar_voxel_noise_filter/debug/pointcloud_noise` | `sensor_msgs::msg::PointCloud2` | 用于调试的被过滤掉的点 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅
[README](../README.md)。

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/polar_voxel_noise_filter_node.schema.json") }}

<a id="parameter-interactions"></a>

### 参数之间的作用关系

- **use_return_type_classification**：启用考虑回波类型的噪声过滤
- **primary_return_types**：仅在 `use_return_type_classification=true` 时使用
- **secondary_noise_threshold**：仅在 `use_return_type_classification=true` 时使用
- **filter_secondary_returns**：为 `true` 时，输出中仅保留主要回波
- **avg_intensity_threshold**：在两种模式中均参与噪声判断
- **publish_noise_cloud**：为 `true` 时，发布移除的点以供调试

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- **简单模式**：仅使用点数和平均强度
- **考虑回波类型的模式**：要求存在 `return_type` 字段
- **支持的格式**：仅支持 PointXYZIRC 和 PointXYZIRCAEDT
- **要求有限坐标值**：忽略 NaN 和 Inf 值
- **半径范围过滤**：排除 `[min_radius, max_radius]` 之外的点
- **不支持索引**：忽略输入索引
- **角度取值域假设**：
  - **PointXYZIRC 方位角**：由 `atan2(y, x)` 计算，因此取值域为 `[-π, π]`；此滤波器
    在体素化之前不会将其归一化到 `[0, 2π]`
  - **PointXYZIRCAEDT 方位角／仰角**：原样使用输入
    消息提供的极坐标
  - **仰角取值域**：预期为 `[-π/2, π/2]`

<a id="error-detection-and-handling"></a>

## 错误检测与处理

滤波器包含输入和参数验证：

- **输入验证**：检查是否缺少必需字段
- **回波类型验证**：仅在考虑回波类型的模式下强制执行
- **坐标验证**：忽略无效或超出范围的点
- **动态参数验证**：拒绝无效的运行时更新，例如负半径或
  超出范围的回波类型

<a id="usage"></a>

## 使用方法

<a id="launch-the-filter"></a>

### 启动滤波器

```xml
<node pkg="autoware_pointcloud_preprocessor"
      exec="polar_voxel_noise_filter_node"
      name="polar_voxel_noise_filter">
  <param from="$(find-pkg-share autoware_pointcloud_preprocessor)/config/polar_voxel_noise_filter_node.param.yaml"/>
</node>
```

<a id="ros-2-topics"></a>

### ROS 2 话题

- **输入**：继承自 `autoware::pointcloud_preprocessor::Filter`
- **输出**：继承自 `autoware::pointcloud_preprocessor::Filter`
- **调试噪声点云**：`~/polar_voxel_noise_filter/debug/pointcloud_noise`

<a id="performance-characterization"></a>

## 性能特征

<a id="computational-complexity"></a>

### 计算复杂度

- **时间复杂度**：约为 `O(n)`，其中 `n` 为输入点数
- **空间复杂度**：约为 `O(v)`，其中 `v` 为已占据体素数

<a id="performance-impact-by-mode"></a>

### 各模式的性能影响

<a id="simple-mode_1"></a>

#### 简单模式

- 不进行回波类型分类，因此开销较低
- 当 `return_type` 不可用或不需要时，这是最佳选择

<a id="return-type-aware-mode_1"></a>

#### 考虑回波类型的模式

- 增加回波类型查找和次要回波计数
- 对次要回波较多的稀疏噪声具有更强的抑制能力

<a id="point-format-impact"></a>

#### 点格式的影响

- **PointXYZIRCAEDT**：由于使用预计算的极坐标，速度更快
- **PointXYZIRC**：由于需要逐点转换坐标，速度略慢

<a id="optimization-tips"></a>

### 优化建议

1. 条件允许时，使用 `PointXYZIRCAEDT` 输入以获得更好的性能
2. 仅在调试时启用 `publish_noise_cloud`
3. 将 `avg_intensity_threshold` 与 `voxel_points_threshold` 配合调节
4. 仅在传感器提供有意义的 `return_type` 值时，使用考虑回波类型的模式
