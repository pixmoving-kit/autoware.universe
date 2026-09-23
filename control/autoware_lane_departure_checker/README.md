<a id="lane-departure-checker"></a>

# 车道偏离检查器

**车道偏离检查器**检查车辆是否沿轨迹行驶。如果未沿轨迹行驶，则通过 `diagnostic_updater` 报告状态。

<a id="features"></a>

## 功能

本功能包包含以下功能：

- **车道偏离**：根据控制模块的输出（预测轨迹），检查自车是否即将驶出车道边界。
- **轨迹偏差**：检查自车位姿是否偏离轨迹，包括横向、纵向和偏航角偏差。
- **道路边界越界**：检查根据控制输出生成的自车轮廓是否超出道路边界。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="how-to-extend-footprint-by-covariance"></a>

### 如何根据协方差扩展轮廓

1. 计算车辆坐标系中误差椭圆（协方差）的标准差。

   1.将协方差转换到车辆坐标系。

   $$
   \begin{align}
   \left( \begin{array}{cc} x_{vehicle}\\ y_{vehicle}\\ \end{array} \right) = R_{map2vehicle}  \left( \begin{array}{cc} x_{map}\\ y_{map}\\ \end{array} \right)
   \end{align}
   $$

   计算车辆坐标系中的协方差。

   $$
   \begin{align}
   Cov_{vehicle} &= E \left[
   \left( \begin{array}{cc} x_{vehicle}\\ y_{vehicle}\\ \end{array} \right) (x_{vehicle}, y_{vehicle}) \right] \\
   &= E \left[ R\left( \begin{array}{cc} x_{map}\\ y_{map}\\ \end{array} \right)
   (x_{map}, y_{map})R^t
   \right] \\
   &= R E\left[ \left( \begin{array}{cc} x_{map}\\ y_{map}\\ \end{array} \right)
   (x_{map}, y_{map})
   \right] R^t \\
   &= R Cov_{map} R^t
   \end{align}
   $$

   2.需要扩展的纵向长度对应 $x_{vehicle}$ 的边缘分布，由 $Cov_{vehicle}(0,0)$ 表示。同理，横向长度由 $Cov_{vehicle}(1,1)$ 表示。维基百科参考资料[见此处](https://en.wikipedia.org/wiki/Multivariate_normal_distribution#Marginal_distributions)。

2. 根据标准差乘以 `footprint_margin_scale` 的结果扩展轮廓。

<a id="interface"></a>

## 接口

<a id="input"></a>

### 输入

- /localization/kinematic_state [`nav_msgs::msg::Odometry`]
- /map/vector_map [`autoware_map_msgs::msg::LaneletMapBin`]
- /planning/mission_planning/route [`autoware_planning_msgs::msg::LaneletRoute`]
- /planning/trajectory [`autoware_planning_msgs::msg::Trajectory`]
- /control/trajectory_follower/predicted_trajectory [`autoware_planning_msgs::msg::Trajectory`]

<a id="output"></a>

### 输出

- [`diagnostic_updater`] lane_departure：自车驶出车道时更新诊断级别。

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

<a id="general-parameters"></a>

#### 通用参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------- | :----- | :---------------------------------------------------------------------------------------------------------- | :------------ |
| will_out_of_lane_checker | bool | 启用自车轮廓是否即将驶出车道的检查 | True |
| out_of_lane_checker | bool | 启用自车轮廓是否已驶出车道的检查 | True |
| boundary_departure_checker | bool | 启用自车轮廓是否即将越过 boundary_types_to_detect 指定边界的检查 | False |
| update_rate | double | 发布频率 [Hz] | 10.0 |
| visualize_lanelet | bool | 是否可视化 lanelet | False |

<a id="parameters-for-lane-departure"></a>

#### 车道偏离参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------------------ | :--- | :------------------------------------------------ | :------------ |
| include_right_lanes | bool | 是否将右侧 lanelet 纳入边界范围 | False |
| include_left_lanes | bool | 是否将左侧 lanelet 纳入边界范围 | False |
| include_opposite_lanes | bool | 是否将对向 lanelet 纳入边界范围 | False |
| include_conflicting_lanes | bool | 是否将冲突 lanelet 纳入边界范围 | False |

<a id="parameters-for-road-border-departure"></a>

#### 道路边界越界参数

| 名称 | 类型 | 说明 | 默认值 |
| :----------------------- | :------------------------- | :---------------------------------------------------------- | :------------ |
| boundary_types_to_detect | std::vector\<std::string\> | boundary_departure_checker 需要检测的 line_string 类型 | [road_border] |

<a id="core-parameters"></a>

### 核心参数

| 名称 | 类型 | 说明 | 默认值 |
| :--------------------- | :----- | :--------------------------------------------------------------------------------- | :------------ |
| footprint_margin_scale | double | 扩展轮廓余量的系数，乘以一个标准差。 | 1.0 |
| footprint_extra_margin | double | 检查车道偏离时扩展轮廓余量的系数。 | 0.0 |
| resample_interval | double | 轨迹重采样时点之间的最小欧氏距离。[m] | 0.3 |
| max_deceleration | double | 计算制动距离时采用的最大减速度。 | 2.8 |
| delay_time | double | 计算制动距离时采用的制动执行延迟。[second] | 1.3 |
| min_braking_distance | double | 最小制动距离。[m] | 0.0 |
