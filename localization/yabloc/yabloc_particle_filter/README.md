# yabLoc_particle_filter

此功能包包含一些与粒子滤波相关的可执行节点。

- [particle_predictor](#particle_predictor)
- [gnss_particle_corrector](#gnss_particle_corrector)
- [camera_particle_corrector](#camera_particle_corrector)

## particle_predictor

<a id="purpose"></a>

### 用途

- 此节点执行粒子的预测更新和重采样。
- 它会将校正节点确定的粒子权重追溯应用到粒子状态中。

<a id="inputs-outputs"></a>

### 输入／输出

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ----------------------------- | ------------------------------------------------ | --------------------------------------------------------- |
| `input/initialpose` | `geometry_msgs::msg::PoseWithCovarianceStamped` | 指定粒子的初始位置 |
| `input/twist_with_covariance` | `geometry_msgs::msg::TwistWithCovarianceStamped` | 预测更新所用的线速度和角速度 |
| `input/height` | `std_msgs::msg::Float32` | 地面高度 |
| `input/weighted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 经校正节点赋权的粒子 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ------------------------------ | ----------------------------------------------- | --------------------------------------------------------- |
| `output/pose_with_covariance` | `geometry_msgs::msg::PoseWithCovarianceStamped` | 带协方差的粒子质心 |
| `output/pose` | `geometry_msgs::msg::PoseStamped` | 带协方差的粒子质心 |
| `output/predicted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 经预测节点赋权的粒子 |
| `debug/init_marker` | `visualization_msgs::msg::Marker` | 初始位置的调试可视化 |
| `debug/particles_marker_array` | `visualization_msgs::msg::MarkerArray` | 粒子可视化，在 `visualize` 为 true 时发布 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_particle_filter/schema/predictor.schema.json") }}

<a id="services"></a>

### 服务

| 名称 | 类型 | 说明 |
| -------------------- | ------------------------ | ------------------------------------------------ |
| `yabloc_trigger_srv` | `std_srvs::srv::SetBool` | 启用或禁用 YabLoc 估计 |

## gnss_particle_corrector

<a id="purpose_1"></a>

### 用途

- 此节点利用 GNSS 估计粒子权重。
- 支持两种输入类型：`ublox_msgs::msg::NavPVT` 和 `geometry_msgs::msg::PoseWithCovarianceStamped`。

<a id="inputs-outputs_1"></a>

### 输入／输出

<a id="input_1"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ---------------------------- | ----------------------------------------------- | -------------------------------------------------- |
| `input/height` | `std_msgs::msg::Float32` | 地面高度 |
| `input/predicted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 预测粒子 |
| `input/pose_with_covariance` | `geometry_msgs::msg::PoseWithCovarianceStamped` | GNSS 测量值，在 `use_ublox_msg` 为 false 时使用 |
| `input/navpvt` | `ublox_msgs::msg::NavPVT` | GNSS 测量值，在 `use_ublox_msg` 为 true 时使用 |

<a id="output_1"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ------------------------------ | -------------------------------------------- | --------------------------------------------------------- |
| `output/weighted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 加权粒子 |
| `debug/gnss_range_marker` | `visualization_msgs::msg::MarkerArray` | GNSS 权重分布 |
| `debug/particles_marker_array` | `visualization_msgs::msg::MarkerArray` | 粒子可视化，在 `visualize` 为 true 时发布 |

<a id="parameters_1"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_particle_filter/schema/gnss_particle_corrector.schema.json") }}

## camera_particle_corrector

<a id="purpose_2"></a>

### 用途

- 此节点利用 GNSS 估计粒子权重。

<a id="inputs-outputs_2"></a>

### 输入／输出

<a id="input_2"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------------------------- | -------------------------------------------- | ----------------------------------------------------------- |
| `input/predicted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 预测粒子 |
| `input/ll2_bounding_box` | `sensor_msgs::msg::PointCloud2` | 转换为线段的路面标线 |
| `input/ll2_road_marking` | `sensor_msgs::msg::PointCloud2` | 转换为线段的路面标线 |
| `input/projected_line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 投影后的线段 |
| `input/pose` | `geometry_msgs::msg::PoseStamped` | 用于检索自车位置周围区域地图的参考位姿 |

<a id="output_2"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ------------------------------ | -------------------------------------------- | --------------------------------------------------------- |
| `output/weighted_particles` | `yabloc_particle_filter::msg::ParticleArray` | 加权粒子 |
| `debug/cost_map_image` | `sensor_msgs::msg::Image` | 根据 Lanelet2 创建的代价地图 |
| `debug/cost_map_range` | `visualization_msgs::msg::MarkerArray` | 代价地图边界 |
| `debug/match_image` | `sensor_msgs::msg::Image` | 投影线段图像 |
| `debug/scored_cloud` | `sensor_msgs::msg::PointCloud2` | 加权三维线段 |
| `debug/scored_post_cloud` | `sensor_msgs::msg::PointCloud2` | 可信度存疑的加权三维线段 |
| `debug/state_string` | `std_msgs::msg::String` | 描述节点状态的字符串 |
| `debug/particles_marker_array` | `visualization_msgs::msg::MarkerArray` | 粒子可视化，在 `visualize` 为 true 时发布 |

<a id="parameters_2"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_particle_filter/schema/camera_particle_corrector.schema.json") }}

<a id="services_1"></a>

### 服务

| 名称 | 类型 | 说明 |
| ------------ | ------------------------ | ----------------------------------------- |
| `switch_srv` | `std_srvs::srv::SetBool` | 启用或禁用校正 |
