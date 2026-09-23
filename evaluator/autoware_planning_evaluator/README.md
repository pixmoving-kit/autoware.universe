<a id="planning-evaluator"></a>

# 规划评估器

<a id="purpose"></a>

## 用途

本功能包提供用于生成各类指标、评估规划质量的节点。

指标可以实时发布，并在节点关闭时保存到 JSON 文件：

- `metrics_for_publish`：
  - 计算 `metrics_for_publish` 中列出的指标，并发布到话题。

- `metrics_for_output`：
  - 节点关闭时，将 `metrics_for_output` 中列出的指标保存到 JSON 文件（要求 `output_metrics` 为 `true`）。
  - 这些指标包括从 `metrics_for_publish` 衍生的统计信息，以及参数和说明等附加信息。

<a id="metrics"></a>

## 指标

所有可用指标均定义于以下文件的 `Metric` 枚举中：

- `include/autoware/planning_evaluator/metrics/metric.hpp`
- `include/autoware/planning_evaluator/metrics/output_metric.hpp`

这些文件还提供字符串转换和便于阅读的说明，供输出文件使用。

<a id="metric-classifications"></a>

### 指标分类

<a id="by-data-type"></a>

#### 按数据类型分类

1. **统计型指标**：
   - 使用 `autoware_utils::Accumulator` 计算，记录最小值、最大值、平均值和计数。
   - 子指标：`/mean`、`/min`、`/max` 和 `/count`。

2. **数值型指标**：
   - 仅包含一个数值的指标。
   - 子指标：`/value`。
   - 某些较早实现的指标采用统计型格式 `/mean`、`/min`、`/max`，但所有值均相同。

3. **计数型指标**：
   - 统计一段时间内特定事件的发生次数。
   - 子指标：`/count` 或 `/count_in_duration`。

<a id="by-purpose"></a>

#### 按用途分类

1. [轨迹指标](#trajectory-metrics)
2. [轨迹偏差指标](#trajectory-deviation-metrics)
3. [轨迹稳定性指标](#trajectory-stability-metrics)
4. [轨迹障碍物指标](#trajectory-obstacle-metrics)
5. [修正目标指标](#modified-goal-metrics)
6. [规划因素指标](#planning-factor-metrics)
7. [转向指标](#steering-metrics)
8. [转向灯指标](#blinker-metrics)
9. [其他信息](#other-information)

<a id="detailed-metrics"></a>

## 指标详情

<a id="trajectory-metrics"></a>

### 轨迹指标

评估当前轨迹 `T(0)` 本身。
收到轨迹时计算并发布指标。

<a id="implemented-metrics"></a>

#### 已实现的指标

- **`curvature`**：各轨迹点曲率的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`point_interval`**：相邻轨迹点之间距离的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`relative_angle`**：相邻轨迹点之间夹角的统计值。
  - 参数：`trajectory.min_point_dist_m`（点之间的最小距离）。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`resampled_relative_angle`**：与 `relative_angle` 类似，但计算相对角度时，将与当前点相距固定距离（例如车长的一半）的点作为下一个点，而非使用紧邻的点。

- **`length`**：轨迹总长度。
  - 子指标：数值型指标，但采用统计型格式 `/mean`、`/min`、`/max`，且三者取值相同。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`duration`**：沿轨迹行驶的预计时间。
  - 子指标：数值型指标，但采用统计型格式 `/mean`、`/min`、`/max`，且三者取值相同。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`velocity`**：各轨迹点速度的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`acceleration`**：各轨迹点加速度的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`jerk`**：各轨迹点加加速度的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

<a id="trajectory-deviation-metrics"></a>

### 轨迹偏差指标

通过比较轨迹 `T(0)` 与参考轨迹来评估轨迹偏差。

仅在节点收到轨迹时计算并发布指标。

使用以下信息计算指标：

- 轨迹 `T(0)`。
- 假定用于规划 `T(0)` 的_参考_轨迹。

<a id="implemented-metrics_1"></a>

#### 已实现的指标

- **`lateral_deviation`**：轨迹点与最近参考轨迹点之间横向偏差的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`yaw_deviation`**：轨迹点与最近参考轨迹点之间偏航角偏差的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`velocity_deviation`**：轨迹点与最近参考轨迹点之间速度偏差的统计值。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

<a id="trajectory-stability-metrics"></a>

### 轨迹稳定性指标

通过比较轨迹 `T(0)` 与上一条轨迹 `T(-1)` 来评估轨迹稳定性。

仅在节点收到轨迹时计算并发布指标。

使用以下信息计算指标：

- 轨迹 `T(0)` 本身。
- 上一条轨迹 `T(-1)`。
- 自车当前里程计信息。

<a id="implemented-metrics_2"></a>

#### 已实现的指标

**`stability`**：前视时间与距离范围内，`T(0)` 与 `T(-1)` 之间横向偏差的统计值。

- 参数：`trajectory.lookahead.max_time_s`、`trajectory.lookahead.max_dist_m`。
- 发布的子指标：`/mean`、`/min`、`/max`。
- 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`stability_frechet`**：前视时间与距离范围内，`T(0)` 与 `T(-1)` 之间的 Fréchet 距离。
  - 参数：与 `stability` 相同。
  - 发布的子指标：`/mean`、`/min`、`/max`。
  - 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

- **`lateral_trajectory_displacement_local`**：自车位置处 `T(0)` 与 `T(-1)` 之间的横向位移绝对值。
  - 发布的子指标：数值型指标，但采用统计型格式。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`lateral_trajectory_displacement_lookahead`**：前视时间范围内，`T(0)` 与 `T(-1)` 之间横向位移绝对值的统计值。
- 参数：`trajectory.evaluation_time_s`。
- 发布的子指标：`/mean`、`/min`、`/max`。
- 输出的子指标：与上述相同，但以已发布的数据作为数据点，而非各个轨迹点。

<a id="trajectory-obstacle-metrics"></a>

### 轨迹障碍物指标

评估 `T(0)` 相对于障碍物的安全性。

仅在节点收到轨迹与感知结果时计算并发布指标。

使用以下信息计算指标：

- 轨迹 `T(0)`。
- 环境中的目标集合。

默认情况下，以目标 UUID 为标识分别发布每个目标的指标，
同时发布所有目标中的最差值。可将 `obstacle.worst_only` 设为 `true`，仅发布最差值。

<a id="implemented-metrics_3"></a>

#### 已实现的指标

- **`obstacle_distance`**：目标中心点到自车未来轨迹（最近点）的距离。
  - 参数：无。
  - 发布的子指标（每个目标）：`/{object_uuid}` 或 `/worst`（所有目标中的最差值）。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`obstacle_ttc`**：与目标的碰撞时间（TTC），考虑目标预测路径及自车轨迹。
  - 仅对实际会与自车碰撞（同一时刻发生重叠）的目标计算 TTC。
  - 参数：
    - `obstacle.collision_thr_m`：判断目标轮廓与自车轨迹轮廓发生碰撞时使用的距离余量。
    - `obstacle.use_ego_traj_vel`：为 true 时使用规划轨迹速度，否则使用自车当前速度。
    - `obstacle.stop_velocity_mps`：将目标或自车视为静止的速度阈值。
    - `obstacle.min_time_interval_s`：自车轨迹重采样的最小时间间隔。
    - `obstacle.min_spatial_interval_m`：自车轨迹重采样的最小空间间隔。
  - 发布的子指标（每个目标）：`/{object_uuid}` 或 `/worst`（所有目标中的最差值）。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`obstacle_pet`**：与目标的后侵入时间（PET）。
  - PET 是目标离开重叠区域与自车进入该区域之间的时间差。
  - 仅对未来轨迹与自车轨迹重叠、但不会在同一时刻碰撞的目标计算。
  - 同一目标不能同时具有 TTC 和 PET（存在 TTC 时，PET 为零）。
  - 参数：与 `obstacle_ttc` 相同。
  - 发布的子指标（每个目标）：`/{object_uuid}` 或 `/worst`（所有目标中的最差值）。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`obstacle_drac`**：与目标的避碰减速度（DRAC）。
  - 避免与目标碰撞所需的最小减速度。
  - 针对可能发生碰撞的目标，与 TTC 一同发布。
  - 参数：与 `obstacle_ttc` 相同。
  - 发布的子指标（每个目标）：`/{object_uuid}` 或 `/worst`（所有目标中的最差值）。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

<a id="modified-goal-metrics"></a>

### 修正目标指标

评估修正目标与自车位置之间的偏差。

仅在节点收到修正目标消息时计算并发布指标。

<a id="implemented-metrics_4"></a>

#### 已实现的指标

- **`modified_goal_longitudinal_deviation`**：修正目标与自车当前位置之间纵向偏差的统计值。
  - 发布的子指标：数值型指标，但采用统计型格式。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`modified_goal_lateral_deviation`**：修正目标与自车当前位置之间横向偏差的统计值。
  - 发布的子指标：数值型指标，但采用统计型格式。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

- **`modified_goal_yaw_deviation`**：修正目标与自车当前位置之间偏航角偏差的统计值。
  - 发布的子指标：数值型指标，但采用统计型格式。
  - 输出的子指标：已发布数据的 `/mean`、`/min`、`/max`。

<a id="planning-factor-metrics"></a>

### 规划因素指标

通过检查规划因素，评估各规划模块的行为。

仅在节点收到各模块发布的规划因素时计算并发布指标。

评估范围为参数文件中 `module_list` 所列的模块。

<a id="implemented-metrics_5"></a>

#### 已实现的指标

- **`stop_decision`**：评估各模块的停车决策。
  - 参数：
    - `stop_decision.time_count_threshold_s`：将停车决策计为新决策的时间阈值。
    - `stop_decision.dist_count_threshold_m`：将停车决策计为新决策的距离阈值。
    - `stop_decision.topic_prefix`：规划因素的话题前缀。
    - `stop_decision.module_list`：待检查模块的名称列表，`{topic_prefix}/{module_name}` 应为有效话题。
  - 发布的子指标：
    - `/{module_name}/keep_duration`（数值型）：当前停车决策持续的时间。
    - `/{module_name}/distance_to_stop`（数值型）：到停车线的距离。
    - `/{module_name}/count`（计数型）：停车决策的序号，即模块作出的停车决策次数。
  - 输出的子指标：
    - `/{module_name}/keep_duration/mean`、`/{module_name}/keep_duration/min`、`/{module_name}/keep_duration/max`：已发布 keep_duration 的统计值。
    - `/{module_name}/count`：停车决策总次数。

- **`abnormal_stop_decision`**：评估各模块的异常停车决策。
  - 如果自车在当前速度和最大减速度限制下无法停下，则该停车决策被视为异常。
  - 参数：
    - `stop_decision.abnormal_deceleration_threshold_mps2`：判断停车决策为异常时使用的最大减速度限制。
    - 其他参数与 `stop_decision` 共用。
  - 发布的子指标：与 `stop_decision` 相同。
  - 输出的子指标：与 `stop_decision` 相同。

<a id="blinker-metrics"></a>

### 转向灯指标

评估车辆转向灯状态。

仅在节点收到转向灯报告消息时计算并发布指标。

<a id="implemented-metrics_6"></a>

#### 已实现的指标

- **`blinker_change_count`**：统计转向灯状态变化次数。
  - 当转向灯状态从关闭/左转变为右转，或从关闭/右转变为左转时，计为一次变化。
- 参数：
  - `blinker_change_count.window_duration_s`：发布时统计变化次数的时间窗口长度。
- 发布的子指标：`/count_in_duration`。
- 输出的子指标：
  - `/count_in_duration/min`、`/count_in_duration/max`、`/count_in_duration/mean`：已发布 keep_duration 的统计值。
  - `/count`：变化总次数。

<a id="steering-metrics"></a>

### 转向指标

评估车辆转向状态。

仅在节点收到转向报告消息时计算并发布指标。

<a id="implemented-metrics_7"></a>

### 已实现的指标

- **`steer_change_count`**：统计转向速率变化次数。
  - 当转向速率从正值/0 变为负值，或从负值/0 变为正值时，计为一次变化。
  - 参数：
    - `steer_change_count.window_duration_s`：发布时统计变化次数的时间窗口长度。
    - `steer_change_count.steer_rate_margin`：将转向速率视为 0 的容差。
  - 发布的子指标：`/count_in_duration`。
  - 输出的子指标：
    - `/count_in_duration/min`、`/count_in_duration/max`、`/count_in_duration/mean`：已发布 keep_duration 的统计值。
    - `/count`：变化总次数。

<a id="other-information"></a>

### 其他信息

与规划相关的其他有用信息：

<a id="implemented-metrics_8"></a>

#### 已实现的指标

- **`kinematic_state`**：车辆当前运动状态。
  - 发布的子指标：
    - `/velocity`：自车当前速度。
    - `/acceleration`：自车当前加速度。
    - `/jerk`：自车当前加加速度。

- **`ego_lane_info`**：Lanelet 信息。
  - 发布的子指标：
    - `/lanelet_id`：自车所在 lanelet 的 ID。
    - `/s`：自车位置在 lanelet 中的弧长。
    - `/t`：自车位置在 lanelet 中的横向偏移。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="inputs"></a>

### 输入

| 名称                             | 类型                                                        | 说明                                       |
| -------------------------------- | ----------------------------------------------------------- | ------------------------------------------------- |
| `~/input/trajectory`             | `autoware_planning_msgs::msg::Trajectory`                   | 待评估的主轨迹                       |
| `~/input/reference_trajectory`   | `autoware_planning_msgs::msg::Trajectory`                   | 用于偏差指标的参考轨迹 |
| `~/input/objects`                | `autoware_perception_msgs::msg::PredictedObjects`           | 障碍物                                         |
| `~/input/modified_goal`          | `autoware_planning_msgs::msg::PoseWithUuidStamped`          | 修正目标                                     |
| `~/input/odometry`               | `nav_msgs::msg::Odometry`                                   | 车辆当前里程计信息                   |
| `~/input/route`                  | `autoware_planning_msgs::msg::LaneletRoute`                 | 路线信息                                 |
| `~/input/vector_map`             | `autoware_map_msgs::msg::LaneletMapBin`                     | 矢量地图信息                            |
| `~/input/acceleration`           | `geometry_msgs::msg::AccelWithCovarianceStamped`            | 车辆当前加速度               |
| `~/input/steering_status`        | `autoware_vehicle_msgs::msg::SteeringReport`                | 车辆当前转向状态                   |
| `~/input/turn_indicators_status` | `autoware_vehicle_msgs::msg::TurnIndicatorsReport`          | 车辆当前转向灯状态             |
| `{topic_prefix}/{module_name}`   | `autoware_internal_planning_msgs::msg::PlanningFactorArray` | 待评估各模块的规划因素       |

<a id="outputs"></a>

### 输出

所有需要发布的指标均通过同一话题发布。

| 名称                         | 类型                                                | 说明                            |
| ---------------------------- | --------------------------------------------------- | -------------------------------------- |
| `~/metrics`                  | `tier4_metric_msgs::msg::MetricArray`               | 包含全部已发布指标的 MetricArray |
| `~/debug/processing_time_ms` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 节点处理耗时，单位为毫秒   |

- 若 `output_metrics = true`，评估节点在关闭时，会将其运行期间测得的输出型指标写入
  `<ros2_logging_directory>/autoware_metrics/<node_name>-<time_stamp>.json`。

<a id="parameters"></a>

## 参数

{{ json_to_markdown("evaluator/autoware_planning_evaluator/schema/autoware_planning_evaluator.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

这里有一个很强的假设：收到轨迹 `T(0)` 时，
该轨迹是使用最近收到的参考轨迹和目标生成的。
如果在计算 `T(0)` 时发布了新的参考轨迹或目标，该假设可能不成立。

当前精度受轨迹分辨率限制。
可以通过对轨迹和参考轨迹进行插值来提高精度，但这会显著增加计算开销。

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分

- 使用 `Route` 或 `Path` 消息作为参考轨迹。
- 碰撞评估指标（在另一个节点中实现，参见 <https://tier4.atlassian.net/browse/AJD-263>）。
- `motion_evaluator_node`。
  - 根据自车实际运动随时间构建轨迹的节点。
  - 目前仅实现了概念验证。
- 障碍物指标应考虑目标形状，而不仅是其质心。
