<a id="traffic-light-compliance-checker"></a>

# 交通信号灯合规检查器

`traffic_light_compliance_checker` 软件包提供确定性的验证层，将规划的车辆轨迹与实时感知的交通信号灯信号及高精度（HD）矢量地图交叉比对，以确保严格遵守交通规则。

<a id="core-features"></a>

## 核心功能

1. **信号状态跟踪（`TrafficLightStatusTracker`）**
   - 为每个交通信号灯组 ID 维护时间序列状态历史，消除感知抖动和信号频繁切换。
   - 在确认切换到 `RED` 或 `AMBER` 之前，检查状态是否达到稳定持续时间阈值。
   - 使用滞回缓冲，在目标短暂遮挡期间保持已知状态。

2. **轨迹验证（`TrafficLightComplianceChecker`）**
   - 依次扫描前方轨迹段，确定进入交叉路口的位置。
   - 附加前保险杠的实际投影，确保车辆轮廓停留在规定的停止线后方。
   - 基于舒适制动和驶离交叉路口所需时间，为 `AMBER` 信号实现运动学通行或停车可行性判定矩阵。
   - 检测到未经允许的越过停止线行为时，返回按优先级和时间顺序排列的 `Violation` 元数据数组。

<a id="inner-workings"></a>

## 内部机制

<a id="main-processing-pipeline"></a>

### 主要处理流程

处理流程依次执行信号处理与过滤，直至评估轨迹与停止线的关系：

1. **过滤信号并更新状态跟踪器：** 将原始感知数据输入状态跟踪器，消除短暂噪声和跟踪丢失的影响。
2. **生成几何轨迹折线：** 移除自车后方的路径点，将前向路径长度限制为 `max(min_lookahead_distance, comfortable_stop_distance + stop_overshoot_margin)`，或截断到第一个计划停车点；同时延长轨迹末端，以计入自车前部偏移。
3. **提取并分组地图停止线：** 将相交的路口停止线按红灯和黄灯约束分别放入评估队列。
4. **评估停止线违规：** 检查轨迹折线与当前有效停止线的关系，记录检测到的违规，并生成合规检查结果。

```plantuml
@startuml
skinparam defaultTextAlignment center
skinparam backgroundColor #WHITE

start

:Filter signals and update status tracker;<<#LightBlue>>
:Generate Trajectory Linestring\n(Cull backward points & clamp at max(min lookahead, stop distance));<<#LightBlue>>
:Extend trajectory linestring\n(Add physical front bumper footprint offset);<<#LightBlue>>
:Extract and group map stop lines\n(Categorize into RED vs. AMBER targets);<<#LightBlue>>

if (Is check_red_lights enabled?) then (yes)
  group Get red stop line violations #Lavender {
    if (Trajectory Intersects stop line?) then (yes)
      if (Trajectory end exceeds tolerance threshold?) then (yes)
        :Record RED Light Violation; <<#LightPink>>
      else (no)
      endif
    else (no)
    endif
  }
else (no)
endif

if (Is check_amber_lights enabled?) then (yes)
  group Get amber stop line violations #LightYellow {
    if (Trajectory Intersects stop line?) then (yes)
      if (Trajectory end exceeds tolerance threshold?) then (yes)
        :Compute time_to_cross AND ego_stopping_distance; <<#LightBlue>>
        if (ego_stopping_distance< distance to stop line OR time_to_cross > crossing_time_limit?) then (yes)
          :Record Amber Light Violation; <<#LightPink>>
        else (no)
        endif
      else (no)
      endif
    else (no)
    endif
  }
else (no)
endif

:Return Compliance Check Result; <<#LightGreen>>
stop

@enduml
```

<a id="signal-status-tracker-filtering"></a>

### 信号状态跟踪器过滤

为防止原始感知噪声引发突然、剧烈的紧急制动，状态跟踪器先通过以下三种确定性机制过滤原始信号，再将其传给验证层：

- **状态持续缓冲：** 输入状态的变化（例如从绿灯变为黄灯、红灯或未知）必须持续达到最小时间窗口（`stable_duration_threshold_red`、`stable_duration_threshold_amber` 或 `stable_duration_threshold_unknown`），新状态才被视为有效。如果信号在达到该时长阈值之前改变颜色或元素，则清空其有效元素数组，以抑制短暂的传感器噪声。
- **历史记录清除缓冲：** 如果某个交通信号灯组完全从输入消息集合中消失，跟踪器会在一个短暂的清除窗口内保留其记录。未更新信号在内存中的保留时长，由最后记录的颜色状态动态决定：红灯使用 `stable_duration_threshold_red`，黄灯使用 `stable_duration_threshold_amber`，其他非红、非黄状态使用 `stable_duration_threshold_unknown`。如果超过该超时时间仍未检测到信号，则从跟踪记录中彻底移除其过期信息。
- **自车停止时直接放行：** 跟踪器通过 `is_ego_stopped` 持续评估自车运动状态。当车辆速度低于配置的 `ego_stopped_velocity_threshold`、被判定为停止时，完全跳过稳定持续时间过滤逻辑。这样可让静止车辆及时响应输入状态变化，消除路口起步时由过滤引起的延迟。

<a id="safety-compliance-logic"></a>

### 安全与合规逻辑

- **逐段几何扫描：** 检查器通过局部 `boost::geometry::intersection` 检查，依次评估轨迹段与地图停止线的关系。按时间顺序找到第一个交点后立即终止循环，计算到停止线的动态距离，并使用 `autoware::interpolation::lerp` 插值求得准确的越线时间戳（用于黄灯评估）。
- **红灯评估：** 如果轨迹与停止线相交，则判定为违规；但若轨迹使自车在指定的 `stop_overshoot_margin` 范围内完全停止，则不判为违规。
- **黄灯评估：** 根据当前速度、加速度、采用的减速度和系统响应延迟计算动态停车距离。如果车辆能够在停止线前安全停车，或者无法在 `crossing_time_limit` 内驶离交叉路口，则记录黄灯违规，要求停车。
- **考虑箭头灯的黄灯通行：** 对于地图中设有独立方向箭头灯的受保护转向车道，圆形信号灯通常会先经历 `GREEN → AMBER → RED`，随后才出现 `GREEN *_ARROW`。圆形灯从绿灯变为黄灯期间，要求停车可能过于严格。当 `enable_arrow_aware_amber_passing` 为 true，且以下条件全部满足时，检查器跳过停止线收集，不判定黄灯或红灯违规：
  - 自车路线所在车道为左转或右转车道（`turn_direction`）
  - 地图中该交通信号灯具有静态箭头信息（`subtype` 包含 `arrow`，或灯泡具有 `arrow` 属性）
  - 黄灯阶段由圆形绿灯转换而来（`AmberState::kFromGreen`）
  - 当前信号仍报告圆形黄灯

  从红灯或未知状态变为黄灯时仍要求停车。在报告绿色箭头灯之前短暂出现的圆形红灯，也仍按停车处理，处理范围与行为速度交通信号灯模块一致。

<a id="structs-and-interface-definitions"></a>

## 结构体与接口定义

接口通过以下标准数据类型传递输入和输出结果：

<a id="inputs"></a>

### 输入

| 字段名称             | 数据类型                      |
| :--------------------- | :----------------------------- |
| `trajectory`           | `std::vector<TrajectoryPoint>` |
| `map`                  | `lanelet::LaneletMapPtr`       |
| `route`                | `LaneletRoute`                 |
| `signals`              | `TrafficLightGroupArray`       |
| `current_time`         | `rclcpp::Time`                 |
| `current_velocity`     | `double`                       |
| `current_acceleration` | `double`                       |

<a id="violation"></a>

### 违规信息

| 结构体字段                | 数据类型                    |
| :-------------------------- | :--------------------------- |
| `type`                      | `ViolationType`              |
| `stop_line`                 | `lanelet::BasicLineString2d` |
| `traffic_light_id`          | `int64_t`                    |
| `cross_point`               | `lanelet::BasicPoint2d`      |
| `arc_length_to_cross_point` | `double`                     |

<a id="parameters"></a>

## 参数

| 参数名称                                 | 类型     | 说明                                                                                                |
| :--------------------------------------------- | :------- | :--------------------------------------------------------------------------------------------------------- |
| `deceleration_limit`                           | `double` | 制动时的最大减速度限制（$m/s^2$），用于评估停车可行性。                        |
| `jerk_limit`                                   | `double` | 制动时的最大加加速度限制（$m/s^3$），用于评估停车可行性。                                |
| `delay_response_time`                          | `double` | 计算周期延迟和制动执行延迟的合并缓冲时间（秒）。                               |
| `crossing_time_limit`                          | `double` | 允许车辆在黄灯状态下驶离交叉路口的最长时间（秒）。                   |
| `stop_overshoot_margin`                        | `double` | 允许已停止车辆越过停止线的实际距离余量（米）。                        |
| `allow_if_cannot_stop_distance`                | `double` | 自车无法在停止线前停车时，允许采用越线轨迹的距离范围（米）。 |
| `min_lookahead_distance`                       | `double` | 即使自车速度较低，也用于扫描停止线的最小前向轨迹长度（米）。                  |
| `stable_duration_threshold_red`                | `double` | 确认 `RED` 状态有效所需的连续持续时间（秒）。                         |
| `stable_duration_threshold_amber`              | `double` | 确认 `AMBER` 状态有效所需的连续持续时间（秒）。                      |
| `stable_duration_threshold_unknown`            | `double` | 确认 `UNKNOWN` 状态有效所需的连续持续时间（秒）。                    |
| `amber_rejection_hysteresis_duration`          | `double` | 感知更新中断时，保留有效黄灯状态的时长（秒）。                   |
| `ego_stopped_velocity_threshold`               | `double` | 自车速度低于此阈值时，视为完全停止（$m/s$）。                 |
| `treat_amber_light_as_red_light`               | `bool`   | 为 true 时，禁用黄灯通行逻辑，并严格将所有黄灯状态按红灯处理。                   |
| `treat_unknown_light_as_red_light`             | `bool`   | 为 true 时，严格将未分类或空白信号状态按红灯处理。                              |
| `enable_arrow_aware_amber_passing`             | `bool`   | 为 true 时，在地图标有静态箭头的转向车道上，允许在圆形绿灯变为黄灯时通行。                         |
| `checked_trajectory_length.deceleration_limit` | `double` | 用于计算轨迹检查长度的舒适停车减速度限制（$m/s^2$）。                    |
| `checked_trajectory_length.jerk_limit`         | `double` | 用于计算轨迹检查长度的舒适停车加加速度限制（$m/s^3$）。                            |
