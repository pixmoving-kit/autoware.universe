<a id="polar-voxel-outlier-filter"></a>

# 极坐标体素离群点滤波器

<a id="overview"></a>

## 概述

极坐标体素离群点滤波器是一种在极坐标空间中运行的点云离群点过滤算法，用于激光雷达数据处理。它既支持基于占据情况的简单过滤，也支持结合回波类型分类的高级双条件过滤，以增强噪声移除效果。

**主要特性**：

- **灵活的过滤模式**，可配置回波类型分类
- 在 PointXYZIRC 与 PointXYZIRCAEDT 之间**自动检测格式**
- **双条件过滤**，通过主要和次要回波分析实现（启用时）
- **考虑范围的能见度估计**，提高诊断精度
- **全面诊断**，包含过滤比例和能见度指标
- **仅估计能见度模式**，只执行诊断而不输出点云
- **可选的调试支持**，通过发布噪声点云辅助分析

<a id="purpose"></a>

## 用途

此滤波器使用针对激光雷达特性优化的极坐标体素网格，移除昆虫、雨等点云噪声。它提供以下可配置的过滤方法：

1. **简单模式**：基础占据过滤（所有回波类型同等计数）
2. **高级模式**：结合回波类型分类进行双条件过滤，提高精度
3. **仅估计能见度模式**：仅执行诊断，用于监测而不产生数据处理开销

高级模式的基本思想是：存在两次回波时，第一次回波更有可能代表噪声区域（雨、雾、烟等）。

<a id="key-differences-from-cartesian-voxel-grid-filter"></a>

## 与笛卡尔坐标体素网格滤波器的主要区别

<a id="coordinate-system"></a>

### 坐标系

- **笛卡尔坐标体素网格**：使用 (x, y, z) 坐标将三维空间划分为规则的立方体体素
- **极坐标体素网格**：使用 (radius, azimuth, elevation) 坐标将三维空间划分为极坐标体素

<a id="advantages-of-polar-voxelization"></a>

### 极坐标体素化的优势

1. **自然表示激光雷达数据**：激光雷达本身按极坐标模式扫描，因此极坐标体素更符合数据结构
2. **自适应分辨率**：在近距离处自动提供更高的角分辨率，在远距离处提供更低的分辨率
3. **考虑距离的过滤**：可根据与传感器的距离采用不同过滤策略
4. **考虑范围的能见度估计**：通过可配置的范围限制准确估计能见度
5. **方位角均匀性**：无论距离如何，都保持一致的方位角覆盖
6. **可配置的回波类型分类**：可选地分析回波类型，以增强过滤效果
7. **仅诊断运行**：估计能见度，而不产生点云处理开销

<a id="point-cloud-format-support"></a>

## 支持的点云格式

此滤波器支持带回波类型信息的点云（高级模式必需），并自动区分以下两种格式：

<a id="pointxyzirc-format"></a>

### PointXYZIRC 格式

- **用途**：包含 (x, y, z, intensity, return_type, channel) 字段的点格式
- **处理**：根据笛卡尔坐标计算极坐标 (radius, azimuth, elevation)
- **回波类型**：使用 return_type 字段进行分类（启用时）
- **性能**：性能良好，但存在坐标转换开销

<a id="pointxyzircaedt-format"></a>

### PointXYZIRCAEDT 格式

- **用途**：包含预计算极坐标字段和回波类型的点云
- **字段**：(x, y, z, intensity, return_type, channel, azimuth, elevation, distance, time_stamp)
- **检测**：自动检测是否存在极坐标字段
- **处理**：直接使用预计算的极坐标，无需转换
- **性能**：避免三角函数计算，因此处理更快

```yaml
# PointXYZIRC: Computes polar coordinates from Cartesian
# - x, y, z       (float32): Cartesian coordinates
# - intensity     (float32): Point intensity
# - return_type   (uint8):   Return type classification
# - channel       (uint16):  Channel information

# PointXYZIRCAEDT: Uses pre-computed polar coordinates
# - x, y, z       (float32): Cartesian coordinates
# - intensity     (float32): Point intensity
# - return_type   (uint8):   Return type classification
# - channel       (uint16):  Channel information
# - azimuth       (float32): Pre-computed azimuth angle
# - elevation     (float32): Pre-computed elevation angle
# - distance      (float32): Pre-computed radius
# - time_stamp    (uint32):  Point timestamp
```

**注意**：滤波器会自动检测格式并采用合适的处理流程。可以根据需要启用或禁用回波类型分类。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="coordinate-conversion"></a>

### 坐标转换

**对于 PointXYZIRC 格式：**
将每个点 (x, y, z) 转换为极坐标：

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

- **半径索引**：`floor(radius / radial_resolution_m)`
- **方位角索引**：`floor(azimuth / azimuth_resolution_rad)`
- **仰角索引**：`floor(elevation / elevation_resolution_rad)`

<a id="return-type-classification"></a>

### 回波类型分类

当 `use_return_type_classification=true` 时，根据 `return_type` 字段对点分类：

- **主要回波**：`primary_return_types` 参数指定的回波类型（默认：[1,6,8,10]）
- **次要回波**：未被指定为主要回波的其他所有回波类型
- **分类用途**：用于高级双条件过滤

<a id="range-aware-visibility-estimation"></a>

### 考虑范围的能见度估计

仅对配置范围内的体素计算能见度指标：

- **范围过滤**：仅考虑满足以下条件的体素：最大半径 ≤ `visibility_estimation_max_range_m`、`visibility_estimation_min_azimuth_rad` ≤ 方位角 ≤ `visibility_estimation_max_azimuth_rad`，以及 `visibility_estimation_min_elevation_rad` ≤ 仰角 ≤ `visibility_estimation_max_elevation_rad`
- **能见度估计调节**：使用 `visibility_estimation_max_secondary_voxel_count` 参数调节报告的能见度值
- **可靠性**：从能见度计算中排除可能不可靠的远距离测量
- **可配置性**：可根据传感器特性和需求进行调整

<a id="filtering-methodology"></a>

### 过滤方法

滤波器根据 `use_return_type_classification` 参数使用不同算法，并支持两种输出模式：

<a id="normal-mode-visibility_estimation_onlyfalse"></a>

#### 正常模式（`visibility_estimation_only=false`）

1. **格式检测**：自动区分 PointXYZIRC 与 PointXYZIRCAEDT
2. **坐标处理**：使用合适的坐标来源
3. **体素处理**：将点分组到极坐标体素中
4. **过滤逻辑**：执行简单过滤或高级过滤
5. **生成输出**：创建过滤后的点云
6. **诊断**：发布过滤比例和能见度指标
7. **可选噪声点云**：启用时发布被过滤掉的点

<a id="visibility-estimation-only-mode-visibility_estimation_onlytrue"></a>

#### 仅估计能见度模式（`visibility_estimation_only=true`）

1. **格式检测**：与正常模式相同
2. **坐标处理**：与正常模式相同
3. **体素处理**：与正常模式相同
4. **过滤逻辑**：与正常模式相同（以确保诊断准确）
5. **生成输出**：**跳过**——为保持接口兼容性，创建空输出
6. **诊断**：**始终发布**——完整的能见度和过滤比例指标
7. **调试点云**：由 `publish_noise_cloud` 和 `publish_low_visibility_voxels` 控制；禁用相应标志后，将跳过发布器创建、消息生成和发布

**仅估计能见度模式的适用场景：**

- **环境监测**：跟踪能见度状况，无需处理开销
- **传感器健康监测**：监控激光雷达性能，不影响数据流水线
- **算法验证**：测试过滤参数，无需处理输出
- **仅诊断应用**：仅执行监测，不进行数据变换

<a id="simple-mode-use_return_type_classificationfalse"></a>

#### 简单模式（`use_return_type_classification=false`）

1. **格式检测**：自动区分 PointXYZIRC 与 PointXYZIRCAEDT
2. **坐标处理**：
   - PointXYZIRC：根据笛卡尔坐标计算极坐标
   - PointXYZIRCAEDT：使用预计算的极坐标
3. **体素分箱**：将点分组到极坐标体素中
4. **简单阈值判断**：保留点数 ≥ `voxel_points_threshold` 的体素（任何回波类型均计入）
5. **输出**：经过基础噪声移除的点云（仅估计能见度模式除外）

<a id="advanced-mode-use_return_type_classificationtrue"></a>

#### 高级模式（`use_return_type_classification=true`）

1. **格式检测**：自动区分 PointXYZIRC 与 PointXYZIRCAEDT
2. **回波类型验证**：确保存在 return_type 字段
3. **坐标处理**：
   - PointXYZIRC：根据笛卡尔坐标计算极坐标
   - PointXYZIRCAEDT：使用预计算的极坐标
4. **回波类型分类**：将点分为主要回波或次要回波
5. **双条件过滤**：
   - **条件 1**：主要回波数量 ≥ `voxel_points_threshold`
   - **条件 2**：次要回波数量 ≤ `secondary_noise_threshold`
   - 只有**同时满足两个条件**才保留该体素
6. **考虑范围的能见度估计**：仅对 `visibility_estimation_max_range_m`、`visibility_estimation_(min|max)_azimuth_rad` 和 `visibility_estimation_(min|max)_elevation_rad` 限定范围内的体素计算能见度；低能见度候选体素受到熵、各向异性、平均强度、次要回波比例及 `visibility_estimation_max_secondary_voxel_count` 的约束
7. **次要回波过滤**：可选择从输出中排除次要回波
8. **输出**：经过增强噪声移除的点云（仅估计能见度模式除外）

<a id="advanced-two-criteria-filtering"></a>

### 高级双条件过滤

启用后，每个体素都必须同时满足以下两个条件：

- **主要回波阈值**：`primary_count >= voxel_points_threshold`
- **次要回波阈值**：`secondary_count <= secondary_noise_threshold`
- **最终判定**：`valid_voxel = (primary_threshold_met AND secondary_threshold_met)`

<a id="key-features"></a>

### 主要特性

- **灵活的架构**：可在简单过滤和高级过滤之间配置切换
- **针对格式优化的处理**：自动选择最优坐标来源
- **考虑范围的诊断**：将能见度估计限制在传感器的可靠范围内
- **次要体素数量限制**：可配置能见度估计中次要体素的数量上限
- **仅估计能见度模式**：执行诊断而不输出点云
- **全面诊断**：提供与模式相对应的过滤比例和能见度指标
- **调试支持**：可选地发布噪声点云，用于分析和调参

<a id="geometric-entropy-based-visibility-estimation-advanced-mode-only"></a>

### 基于几何熵的能见度估计（仅限高级模式）

滤波器通过几何熵和各向异性分析，检测恶劣天气造成的低能见度状况。根据体素内点的空间分布计算熵和各向异性，以识别不规则点簇（雨、雾、烟），同时避免对窗户等平面表面误报。较高的熵值与低于特定阈值的各向异性值同时出现时，表示存在天气引起的噪声；平面表面的熵和各向异性值则不同，因此可避免在使用次要回波类型时将结构表面误判为噪声。

<a id="return-type-management-advanced-mode-only"></a>

### 回波类型管理（仅限高级模式）

- **主要回波**：可配置的回波类型列表（默认：[1,6,8,10]）
- **次要回波**：未被指定为主要回波的所有回波类型
- **动态分类**：可通过参数更新在运行时配置
- **输出过滤**：可选择从最终输出中排除次要回波

<a id="inputs-outputs"></a>

## 输入／输出

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

<a id="input-requirements"></a>

### 输入要求

- **支持的格式**：PointXYZIRC 或 PointXYZIRCAEDT
- **回波类型字段**：仅在 `use_return_type_classification=true` 时必需
- **无效输入**：高级模式下会拒绝不含 return_type 字段的点云

<a id="additional-debug-topics"></a>

### 附加调试话题

| 名称 | 类型 | 说明 |
| ---------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `~/polar_voxel_outlier_filter/debug/filter_ratio` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 输出点数与输入点数之比（始终发布） |
| `~/polar_voxel_outlier_filter/debug/visibility` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 通过次要回波阈值测试的体素比例（仅限高级模式，受范围限制） |
| `~/polar_voxel_outlier_filter/debug/pointcloud_noise` | `sensor_msgs::msg::PointCloud2` | 被过滤掉的点，附加逐体素信息字段；在 `publish_noise_cloud=true` 时用于调试 |
| `~/polar_voxel_outlier_filter/debug/low_visibility_voxels` | `sensor_msgs::msg::PointCloud2` | 选定低能见度体素中的点，附加逐体素信息字段；在 `publish_low_visibility_voxels=true` 时用于调试 |

噪声点云和低能见度体素调试点云保留输入点字段，并为每个点附加以下 `FLOAT32` 字段：`voxel_radius_idx`、`voxel_azimuth_idx`、`voxel_elevation_idx`、`voxel_entropy`、`voxel_anisotropy`、`voxel_point_count`、`voxel_average_intensity`、`voxel_secondary_return_count` 和 `voxel_secondary_return_ratio`。

仅对配置的能见度估计范围内的体素计算 `voxel_entropy` 和 `voxel_anisotropy`。

如有需要，可以在运行时修改 `publish_noise_cloud` 和 `publish_low_visibility_voxels`。

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

此实现继承 `autoware::pointcloud_preprocessor::Filter` 类，请参阅 [README](../README.md)。

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/polar_voxel_outlier_filter_node.schema.json") }}

<a id="parameter-interactions"></a>

### 参数之间的作用关系

- **use_return_type_classification**：必须为 `true` 才能启用高级双条件过滤
- **filter_secondary_returns**：为 `true` 时，输出中仅包含主要回波（仅限高级模式）
- **secondary_noise_threshold**：仅在 `use_return_type_classification=true` 时使用
- **low_visibility_entropy_threshold**：当 `use_return_type_classification=true` 时，筛选低能见度候选体素所需的最小归一化几何熵（默认：0.3）
- **low_visibility_anisotropy_threshold**：当 `use_return_type_classification=true` 时，筛选低能见度候选体素允许的最大几何各向异性（默认：0.95）
- **low_visibility_average_intensity_threshold**：当 `use_return_type_classification=true` 时，筛选能见度候选体素允许的最大体素平均强度
- **低能见度体素**：范围内的次要回波噪声候选体素受熵、各向异性和平均强度阈值约束。范围内的稀疏体素如果点数小于 `low_visibility_sparse_voxel_point_count_threshold`，满足 `voxel_secondary_return_ratio > low_visibility_sparse_voxel_secondary_return_ratio_threshold`，且平均强度低于 `low_visibility_average_intensity_threshold`，也会被选为低能见度体素
- **visibility_estimation_max_secondary_voxel_count**：仅在 `use_return_type_classification=true` 时使用，限制能见度计算中的次要体素计数
- **primary_return_types**：仅在 `use_return_type_classification=true` 时使用
- **visibility_estimation_max_range_m**：将能见度计算限制在传感器的可靠范围内（仅限高级模式）
- **publish_low_visibility_voxels**：为 `true` 时，发布低能见度体素中的点，以监控基于熵的检测结果（仅限高级模式）
- **visibility_estimation_only**：为 `true` 时，跳过点云输出生成，但仍计算并发布诊断信息
- **publish_noise_cloud**：为 `false` 时，通过跳过噪声点云发布器创建、消息生成和发布来提高性能
- **诊断**：仅在启用回波类型分类时发布能见度

<a id="configuration-examples"></a>

## 配置示例

<a id="visibility-only-configuration"></a>

### 仅估计能见度的配置

```yaml
# Run filter for diagnostics only - no point cloud processing
visibility_estimation_only: true
use_return_type_classification: true
voxel_points_threshold: 2
secondary_noise_threshold: 4
low_visibility_entropy_threshold: 0.3
low_visibility_anisotropy_threshold: 0.95
low_visibility_average_intensity_threshold: 2.0
low_visibility_sparse_voxel_point_count_threshold: 10
low_visibility_sparse_voxel_secondary_return_ratio_threshold: 0.2
visibility_estimation_max_secondary_voxel_count: 500
visibility_estimation_max_range_m: 20.0
primary_return_types: [1, 6, 8, 10]
radial_resolution_m: 0.5
azimuth_resolution_rad: 0.0175
elevation_resolution_rad: 0.0175
publish_noise_cloud: false
publish_low_visibility_voxels: false
```

<a id="simple-mode-configuration"></a>

### 简单模式配置

```yaml
# Basic occupancy filtering - any return type counts equally
use_return_type_classification: false
voxel_points_threshold: 2 # Total points threshold
radial_resolution_m: 0.5
azimuth_resolution_rad: 0.0175 # ~1 degree
elevation_resolution_rad: 0.0175 # ~1 degree
visibility_estimation_only: false # Normal filtering mode
publish_noise_cloud: true # Enable for debugging
```

<a id="advanced-mode-configuration"></a>

### 高级模式配置

```yaml
# Two-criteria filtering with return type classification
use_return_type_classification: true
voxel_points_threshold: 2 # Primary return threshold
secondary_noise_threshold: 4 # Secondary return threshold
low_visibility_entropy_threshold: 0.3 # Min normalized geometric entropy for visibility candidates
low_visibility_anisotropy_threshold: 0.95 # Max geometric anisotropy for visibility candidates
low_visibility_average_intensity_threshold: 255.0 # Max voxel average intensity for visibility candidates
low_visibility_sparse_voxel_point_count_threshold: 10 # Sparse voxel point count threshold
low_visibility_sparse_voxel_secondary_return_ratio_threshold: 0.2 # Sparse voxel secondary return ratio threshold
visibility_estimation_max_secondary_voxel_count: 500 # Max secondary voxels for visibility
primary_return_types: [1, 6, 8, 10] # Primary return types
filter_secondary_returns: false # Include secondary returns in output
radial_resolution_m: 0.5
azimuth_resolution_rad: 0.0175 # ~1 degree
elevation_resolution_rad: 0.0175 # ~1 degree
visibility_estimation_max_range_m: 20.0 # Range limit for visibility calculation
visibility_estimation_min_azimuth_rad: 0.78
visibility_estimation_max_azimuth_rad: 2.35
visibility_estimation_min_elevation_rad: -0.26
visibility_estimation_max_elevation_rad: 1.04
visibility_estimation_only: false # Normal filtering mode
publish_noise_cloud: true
```

<a id="debug-configuration"></a>

### 调试配置

```yaml
# Enable debugging and monitoring
use_return_type_classification: true
visibility_estimation_max_range_m: 20.0 # Configured range for urban environments
visibility_estimation_min_azimuth_rad: 0.78
visibility_estimation_max_azimuth_rad: 2.35
visibility_estimation_min_elevation_rad: -0.26
visibility_estimation_max_elevation_rad: 1.04
visibility_estimation_max_secondary_voxel_count: 500 # Allow secondary voxels in visibility calculation
low_visibility_entropy_threshold: 0.3 # Entropy threshold for low-visibility detection
low_visibility_anisotropy_threshold: 0.95 # Anisotropy threshold for low-visibility detection
low_visibility_average_intensity_threshold: 255.0 # Average intensity threshold for low-visibility detection
low_visibility_sparse_voxel_point_count_threshold: 10 # Sparse voxel point count threshold
low_visibility_sparse_voxel_secondary_return_ratio_threshold: 0.2 # Sparse voxel secondary return ratio threshold
publish_low_visibility_voxels: true # Monitor voxels used for visibility estimation
visibility_estimation_only: false # Normal filtering with debug output
publish_noise_cloud: true # Enable noise cloud for analysis
filter_ratio_error_threshold: 0.5
filter_ratio_warn_threshold: 0.7
visibility_error_threshold: 0.8
visibility_warn_threshold: 0.9
```

<a id="sensor-specific-configuration-examples"></a>

### 针对特定传感器的配置示例

```yaml
# Long-range highway LiDAR
visibility_estimation_max_range_m: 200.0
visibility_estimation_max_secondary_voxel_count: 500
radial_resolution_m: 0.5
voxel_points_threshold: 2
visibility_estimation_only: false

# Urban short-range LiDAR
visibility_estimation_max_range_m: 20.0
visibility_estimation_max_secondary_voxel_count: 500
radial_resolution_m: 0.5
voxel_points_threshold: 2
visibility_estimation_only: false

# High-resolution near-field processing
visibility_estimation_max_range_m: 20.0
visibility_estimation_max_secondary_voxel_count: 500
radial_resolution_m: 0.5
azimuth_resolution_rad: 0.0175 # ~1 degree
elevation_resolution_rad: 0.0175 # ~1 degree
visibility_estimation_only: false

# Environmental monitoring only
visibility_estimation_max_range_m: 50.0
visibility_estimation_max_secondary_voxel_count: 500
radial_resolution_m: 1.0 # Coarser resolution for performance
visibility_estimation_only: true # Diagnostics only
```

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- **简单模式**：适用于任何点云格式，仅执行基础占据过滤
- **高级模式**：要求存在 return_type 字段，以进行增强过滤
- **仅估计能见度模式**：运行完整过滤算法，但不生成点云输出
- **支持的格式**：仅支持 PointXYZIRC 和 PointXYZIRCAEDT
- **要求有限坐标值**：自动过滤掉 NaN／Inf 点
- **回波类型依赖**：高级过滤效果取决于回波类型分类的准确性
- **能见度范围依赖**：能见度精度取决于 `visibility_estimation_max_range_m`、`visibility_estimation_(min|max)_azimuth_rad` 和 `visibility_estimation_(min|max)_elevation_rad` 的合理设置。此外，方位角和仰角范围采用以下定义
  - 方位角：以 Y 轴为起点，绕 Z 轴按逆螺旋规则递增。取值域为 $`[0, 2\pi]`$
  - 仰角：以 X 轴为起点，绕 Y 轴按逆螺旋规则递增。取值域为 $`[-\frac{\pi}{2}, \frac{pi}{2}]`$
- **次要体素数量限制**：可通过 `visibility_estimation_max_secondary_voxel_count` 调节能见度估计

<a id="error-detection-and-handling"></a>

## 错误检测与处理

滤波器包含稳健的错误处理机制：

- **针对模式的验证**：高级模式下检查是否存在 return_type 字段
- **输入验证**：检查空点云
- **坐标验证**：自动过滤无效点（NaN、Inf 值）
- **范围验证**：排除配置半径范围之外的点
- **参数验证**：确保 `visibility_estimation_max_range_m` > 0 且 `visibility_estimation_max_secondary_voxel_count` ≥ 0
- **动态参数验证**：验证运行时参数更新
- **模式兼容性**：验证不同运行模式下的参数组合

<a id="usage"></a>

## 使用方法

<a id="launch-the-filter"></a>

### 启动滤波器

```bash
# Example launch file integration for simple mode:
# <node pkg="autoware_pointcloud_preprocessor" exec="polar_voxel_outlier_filter_node" name="polar_voxel_filter">
#   <param name="use_return_type_classification" value="false"/>
#   <param name="radial_resolution_m" value="1.0"/>
#   <param name="azimuth_resolution_rad" value="0.0349"/>
#   <param name="voxel_points_threshold" value="3"/>
#   <param name="visibility_estimation_only" value="false"/>
# </node>

# Example launch file integration for advanced mode:
# <node pkg="autoware_pointcloud_preprocessor" exec="polar_voxel_outlier_filter_node" name="polar_voxel_filter">
#   <param name="use_return_type_classification" value="true"/>
#   <param name="radial_resolution_m" value="0.5"/>
#   <param name="azimuth_resolution_rad" value="0.0175"/>
#   <param name="voxel_points_threshold" value="2"/>
#   <param name="secondary_noise_threshold" value="4"/>
#   <param name="visibility_estimation_max_secondary_voxel_count" value="500"/>
#   <param name="primary_return_types" value="[1,6,8,10]"/>
#   <param name="visibility_estimation_max_range_m" value="20.0"/>
#   <param name="visibility_estimation_min_azimuth_rad" value="0.78"/>
#   <param name="visibility_estimation_max_azimuth_rad" value="2.35"/>
#   <param name="visibility_estimation_min_elevation_rad" value="-0.26"/>
#   <param name="visibility_estimation_max_elevation_rad" value="1.04"/>
#   <param name="visibility_estimation_only" value="false"/>
# </node>

# Example launch file integration for visibility-only mode:
# <node pkg="autoware_pointcloud_preprocessor" exec="polar_voxel_outlier_filter_node" name="polar_voxel_filter">
#   <param name="use_return_type_classification" value="true"/>
#   <param name="visibility_estimation_only" value="true"/>
#   <param name="visibility_estimation_max_range_m" value="20.0"/>
#   <param name="visibility_estimation_min_azimuth_rad" value="0.78"/>
#   <param name="visibility_estimation_max_azimuth_rad" value="2.35"/>
#   <param name="visibility_estimation_min_elevation_rad" value="-0.26"/>
#   <param name="visibility_estimation_max_elevation_rad" value="1.04"/>
#   <param name="visibility_estimation_max_secondary_voxel_count" value="500"/>
# </node>
```

<a id="ros-2-topics"></a>

### ROS 2 话题

<a id="inputoutput"></a>

#### 输入／输出

- **输入**：`/input`（sensor_msgs/PointCloud2）——高级模式要求存在 return_type 字段
- **输出**：`/output`（sensor_msgs/PointCloud2）——仅估计能见度模式下为空

<a id="debug-topics"></a>

#### 调试话题

- **过滤比例**：`~/polar_voxel_outlier_filter/debug/filter_ratio`（autoware_internal_debug_msgs/Float32Stamped）——始终发布
- **能见度**：`~/polar_voxel_outlier_filter/debug/visibility`（autoware_internal_debug_msgs/Float32Stamped）——仅限高级模式，受范围限制
- **噪声点云**：`~/polar_voxel_outlier_filter/debug/pointcloud_noise`（sensor_msgs/PointCloud2）——仅在 `publish_noise_cloud=true` 时发布
- **低能见度体素**：`~/polar_voxel_outlier_filter/debug/low_visibility_voxels`（sensor_msgs/PointCloud2）——仅在 `publish_low_visibility_voxels=true` 时发布

<a id="programmatic-usage"></a>

### 编程使用方式

```cpp
#include "autoware/pointcloud_preprocessor/outlier_filter/polar_voxel_outlier_filter_node.hpp"

// Create node
auto node = std::make_shared<autoware::pointcloud_preprocessor::PolarVoxelOutlierFilterComponent>(options);

// The filter automatically detects point cloud format and applies filtering based on configuration:
// - Simple mode (use_return_type_classification=false): Basic occupancy filtering
// - Advanced mode (use_return_type_classification=true): Two-criteria filtering with return type analysis
// - Visibility-only mode (visibility_estimation_only=true): Diagnostics without point cloud output
// - Both modes support PointXYZIRC and PointXYZIRCAEDT formats
// - Advanced mode uses range-limited visibility estimation with configurable secondary voxel limiting
```

<a id="performance-characterization"></a>

## 性能特征

<a id="computational-complexity"></a>

### 计算复杂度

- **时间复杂度**：O(n)，其中 n 为输入点数
- **空间复杂度**：O(v)，其中 v 为已占据体素数

<a id="performance-impact-by-mode"></a>

### 各模式的性能影响

<a id="visibility-estimation-only-mode"></a>

#### **仅估计能见度模式**

- **计算**：运行完整过滤算法，以确保诊断准确
- **内存**：内存使用量极低，不分配输出点云内存
- **I/O**：仅发布诊断信息，不输出点云
- **适用场景**：最适合无需数据处理的监测应用

<a id="normal-mode"></a>

#### **正常模式**

- **PointXYZIRCAEDT**：使用预计算坐标，性能最佳
- **PointXYZIRC**：性能良好，但存在坐标转换开销
- **输出处理**：生成完整点云和可选的噪声点云

<a id="simple-mode"></a>

#### **简单模式**

- **PointXYZIRCAEDT**：使用预计算坐标，处理快速
- **PointXYZIRC**：性能良好，但存在坐标转换开销
- **无回波类型分析**：降低计算开销

<a id="advanced-mode"></a>

#### **高级模式**

- **PointXYZIRCAEDT**：使用预计算坐标并分析回波类型，性能最佳
- **PointXYZIRC**：进行坐标转换和回波类型分析，性能良好
- **增强过滤**：额外进行回波类型分类和考虑范围的能见度处理

<a id="memory-usage"></a>

### 内存使用

- **仅估计能见度模式**：显著减少内存占用
- **基于哈希的体素存储**：高效处理稀疏体素占据情况
- **单次遍历处理**：无论哪种模式，内存开销都很低
- **针对模式的输出**：按模式优化内存分配
- **范围过滤**：使用额外的哈希表计算能见度（仅限高级模式）

<a id="optimization-tips"></a>

### 优化建议

1. 纯监测应用应**使用仅估计能见度模式**
2. 根据需求**选择合适模式**：
   - 有数据处理需求时使用正常模式
   - 环境／传感器监测使用仅估计能见度模式
3. 条件允许时，**使用 PointXYZIRCAEDT 格式**以获得最佳性能
4. **动态组合模式**——根据运行需求在运行时切换
5. 根据使用场景**调节体素分辨率**
6. **配置回波类型映射**，使其与传感器匹配（高级模式）
7. 为传感器和环境**设置合适的能见度范围**（`visibility_estimation_max_range_m`）
8. **调节次要体素数量限制**（`visibility_estimation_max_secondary_voxel_count`），提高能见度估计精度
9. **监控诊断信息**，实时评估两种模式的性能

<a id="diagnostics-and-monitoring"></a>

## 诊断与监测

<a id="filter-ratio-diagnostics"></a>

### 过滤比例诊断

- **所有模式均发布**：整体过滤效果（输出／输入点数比）
- **可配置阈值**：为自动监测配置错误／警告级别
- **实时反馈**：即时评估过滤性能
- **仅估计能见度模式**：显示理论过滤效果，而不实际输出过滤结果

<a id="visibility-diagnostics"></a>

### 能见度诊断

- **仅限高级模式**：使用回波类型分类数据
- **受范围限制的指标**：仅考虑 `visibility_estimation_max_range_m`、`visibility_estimation_(min|max)_azimuth_rad` 和 `visibility_estimation_(min|max)_elevation_rad` 范围内的体素
- **次要体素数量限制**：由 `visibility_estimation_max_secondary_voxel_count` 参数控制
- **基于体素的指标**：限定范围内通过次要回波阈值测试的体素百分比
- **环境指标**：有助于在可靠范围内检测传感器工作状况
- **诊断上下文**：状态消息包含配置的能见度估计范围及次要体素数量限制
- **所有输出模式均可用**：即使在仅估计能见度模式下，也会发布以供监测

<a id="debug-features"></a>

### 调试功能

- **噪声点云**：提供所有被过滤掉的点以供分析（启用且不处于仅估计能见度模式时）
- **运行时参数更新**：动态调整阈值和范围
- **针对模式的日志**：按过滤模式输出相应的调试消息
- **考虑范围的诊断**：能见度计算明确标明估计范围

<a id="use-cases-and-configuration-guidelines"></a>

## 使用场景与配置指南

<a id="visibility-estimation-only-mode-use-cases"></a>

### 仅估计能见度模式的使用场景

1. **环境监测**：跟踪大气状况，无需处理开销
2. **传感器健康监测**：监控激光雷达性能和能见度状况
3. **算法验证**：测试和调节过滤参数，无需生成输出
4. **纯诊断**：仅需要能见度和过滤比例指标的应用
5. **资源受限系统**：在保持监测能力的同时尽量降低计算负载
6. **气象站集成**：为气象系统自动报告能见度

<a id="simple-mode-use-cases"></a>

### 简单模式的使用场景

1. **旧系统集成**：无需回波类型信息的基础过滤
2. **性能关键应用**：计算资源受限的场景
3. **回波类型可靠性未知**：传感器回波类型信息存疑的场景
4. **基础噪声移除**：仅需要基于占据情况的简单过滤

<a id="advanced-mode-use-cases"></a>

### 高级模式的使用场景

1. **现代激光雷达处理**：利用可靠的回波类型信息进行增强过滤
2. **环境监测**：考虑范围的能见度估计和天气状况检测
3. **高质量过滤**：通过双条件方法获得更好的噪声移除效果
4. **自动驾驶车辆应用**：用于安全关键过滤，并提供全面诊断
5. **特定范围分析**：近场与远场有不同的能见度要求

<a id="parameter-tuning-guidelines"></a>

### 参数调节指南

<a id="visibility-estimation-only-mode-configuration"></a>

#### 仅估计能见度模式配置

- **用于环境监测**：启用高级模式，并设置合适的能见度范围
- **用于传感器监测**：使用回波类型分类进行详细分析
- **注重性能**：使用较大的体素尺寸以减少计算
- **注重精度**：使用较小的体素尺寸以精确估计能见度

<a id="simple-mode-configuration_1"></a>

#### 简单模式配置

- **密集环境**：使用较小的体素尺寸和较高的点数阈值
- **稀疏数据**：使用较大的体素尺寸和较低的点数阈值
- **注重性能**：增大体素尺寸，禁用噪声点云发布

<a id="advanced-mode-configuration_1"></a>

#### 高级模式配置

- **积极去噪**：使用较低的次要回波噪声阈值（0-2）
- **保守过滤**：使用较高的次要回波噪声阈值（3-5）
- **严格估计能见度**：将 `visibility_estimation_max_secondary_voxel_count` 设为较小值
- **宽松估计能见度**：允许更多次要体素（数百个）
- **仅输出主要回波**：启用 `filter_secondary_returns`
- **针对传感器优化**：根据传感器特性调整 `primary_return_types`
- **针对特定范围的能见度估计**：根据传感器有效范围和应用需求，设置 `visibility_estimation_max_range_m`、`visibility_estimation_(min|max)_azimuth_rad` 和 `visibility_estimation_(min|max)_elevation_rad`

<a id="mode-selection-guidelines"></a>

#### 模式选择指南

- **以下情况选择仅估计能见度模式**：

  - 只需要诊断信息
  - 计算资源有限
  - 在主处理流程旁并行运行监测
  - 测试过滤参数，但不希望影响下游系统

- **以下情况选择正常模式**：
  - 需要输出过滤后的点云
  - 需要集成到数据处理流水线
  - 为感知系统进行实时过滤
  - 需要包括噪声点云调试在内的完整功能

<a id="visibility-range-guidelines"></a>

#### 能见度范围设置指南

- **城市环境**：30-80m（使用较短范围进行可靠的近场分析）
- **高速公路应用**：100-200m（高速场景使用较长范围）
- **泊车／装卸**：10-30m（使用很短的范围进行精确近场监测）
- **传感器规格**：与传感器的可靠探测范围相匹配

<a id="comparison-table"></a>

## 对比表

| 项目 | 简单模式 | 高级模式 | 仅估计能见度模式 |
| ------------------------ | --------------- | ------------------------------- | ------------------------------- |
| **过滤方法** | 基础占据过滤 | 结合回波类型的双条件过滤 | 与所选过滤模式相同 |
| **是否需要回波类型** | 否 | 是 | 取决于所选模式 |
| **计算成本** | 低 | 中等 | 中等（无输出） |
| **过滤质量** | 良好 | 优秀 | 不适用（无输出） |
| **能见度指标** | 无 | 考虑范围并限制体素数量 | 考虑范围并限制体素数量 |
| **配置** | 简单 | 高级 | 高级（侧重诊断） |
| **使用场景** | 基础过滤 | 增强噪声移除 | 监测／诊断 |
| **环境适应性** | 有限 | 全面 | 全面 |
| **范围感知能力** | 基础 | 可配置 | 可配置 |
| **点云输出** | 有 | 有 | 无（空点云） |

<a id="migration-guide"></a>

## 迁移指南

<a id="enabling-visibility-estimation-only-mode"></a>

### 启用仅估计能见度模式

要启用仅诊断运行：

1. **设置参数**：`visibility_estimation_only: true`
2. **配置诊断**：确保能见度和过滤比例阈值合适
3. **验证模式**：检查日志中的 "visibility estimation only" 确认信息
4. **监控诊断**：使用发布的指标进行监测
5. **预期无输出**：下游节点应能处理空点云

<a id="dynamic-mode-switching"></a>

### 动态切换模式

可在运行时更改模式：

```bash
# Switch to visibility-only mode
ros2 param set /polar_voxel_filter visibility_estimation_only true

# Switch back to normal mode
ros2 param set /polar_voxel_filter visibility_estimation_only false
```

<a id="enabling-advanced-mode"></a>

### 启用高级模式

要在现有系统上启用高级过滤：

1. **确认回波类型字段**：验证输入点云包含 return_type 字段
2. **设置参数**：`use_return_type_classification: true`
3. **配置回波类型**：为传感器设置 `primary_return_types`
4. **设置能见度范围**：根据应用配置 `visibility_estimation_max_range_m`、`visibility_estimation_(min|max)_azimuth_rad` 和 `visibility_estimation_(min|max)_elevation_rad`
5. **配置次要体素数量限制**：根据需求设置 `visibility_estimation_max_secondary_voxel_count`
6. **调节阈值**：根据需求调整 `secondary_noise_threshold`
7. **监控诊断**：使用考虑范围的能见度指标评估性能

<a id="disabling-advanced-mode"></a>

### 禁用高级模式

要使用简单模式进行基础过滤：

1. **设置参数**：`use_return_type_classification: false`
2. **配置阈值**：为总点数设置 `voxel_points_threshold`
3. **移除高级参数**：回波类型和能见度范围参数将被忽略
4. **简化监测**：仅提供过滤比例诊断

<a id="compatibility-considerations"></a>

### 兼容性注意事项

- **输出接口**：仅估计能见度模式使用空点云保持话题兼容性
- **诊断话题**：无论输出模式如何，均提供相同的诊断信息
- **参数兼容性**：所有过滤参数均适用于正常模式和仅估计能见度模式
- **性能影响**：模式变更在下一次调用滤波器时生效

<a id="updating-existing-configurations"></a>

### 更新现有配置

对于已经使用高级模式的系统，添加以下新参数：

```yaml
# Add to existing configuration:
visibility_estimation_only: false # Default for normal operation
visibility_estimation_max_secondary_voxel_count: 500 # Updated default
primary_return_types: [1, 6, 8, 10] # Updated to include return type 8
```

此方法通过**考虑范围的能见度估计**、**可配置的次要体素数量限制**以及**仅诊断运行**提供**最大的灵活性**，同时为各种使用场景保持最佳性能！🎯✨
