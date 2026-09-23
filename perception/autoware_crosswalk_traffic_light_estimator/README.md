# autoware_crosswalk_traffic_light_estimator

<a id="purpose"></a>

## 用途

`autoware_crosswalk_traffic_light_estimator` 估计行人交通信号，可归纳为以下两项任务：

- 估计不属于感知流程检测对象的行人交通信号。
- 估计行人交通信号是否闪烁，并修改结果。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                                 | 类型                                                  | 说明        |
| ------------------------------------ | ----------------------------------------------------- | ------------------ |
| `~/input/vector_map`                 | autoware_map_msgs::msg::LaneletMapBin                 | 矢量地图 |
| `~/input/classified/traffic_signals` | autoware_perception_msgs::msg::TrafficLightGroupArray | 已分类的信号 |

<a id="output"></a>

### 输出

| 名称                         | 类型                                                  | 说明                                               |
| ---------------------------- | ----------------------------------------------------- | --------------------------------------------------------- |
| `~/output/traffic_signals`   | autoware_perception_msgs::msg::TrafficLightGroupArray | 包含估计行人交通信号的输出 |
| `~/debug/processing_time_ms` | autoware_internal_debug_msgs::msg::Float64Stamped     | 流水线延迟（ms） |

<a id="parameters"></a>

## 参数

| 名称                           | 类型   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 默认值 |
| :----------------------------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `use_last_detect_color`        | bool   | 若此参数为 `true`，不仅在车辆交通信号检测为 GREEN/AMBER 时，而且在检测结果从 GREEN/AMBER 变为 UNKNOWN 时，此模块也会将行人交通信号估计为 RED。（若检测结果从 RED 或 AMBER 变为 UNKNOWN，则此模块将行人交通信号估计为 UNKNOWN。）若此参数为 `false`，则仅使用最新检测结果进行估计。（仅在检测结果为 GREEN/AMBER 时，将行人交通信号估计为 RED。） | true          |
| `use_pedestrian_signal_detect` | bool   | 若此参数为 `true`，使用感知流程估计的行人交通信号；若为 `false`，则使用车辆交通信号和高精地图估计的行人信号覆盖它。 | true          |
| `last_detect_color_hold_time`  | double | 保持上次检测颜色的时间阈值，单位为秒。 | 2.0           |
| `last_colors_hold_time`        | double | 保持历史检测行人交通灯颜色的时间阈值，单位为秒。 | 1.0           |

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

当感知流程**检测到**行人交通信号时

- 如果估计行人交通信号正在闪烁，则覆盖结果
- 优先使用感知流程的输出，但若行人交通信号无效（`no detection`、`backlight` 或 `occlusion`），则覆盖它

当感知流程**未检测到**行人交通信号时

- 根据检测到的车辆交通信号和高精地图估计行人交通信号颜色

也可在 lanelet 地图中定义针对某些交通灯的覆盖规则。
此时，人行横道交通灯估计结果将被这些规则覆盖。

<a id="estimate-whether-pedestrian-traffic-signals-are-flashing"></a>

### 估计行人交通信号是否闪烁

```plantumul
start
if (the pedestrian traffic light classification result exists)then
    : update the flashing flag according to the classification result(in_signal) and last_signals
    if (the traffic light is flashing?)then(yes)
      : update the traffic light state
    else(no)
      : the traffic light state is the same with the classification result
if (the classification result not exists)
    : the traffic light state is the same with the estimation
 : output the current traffic light state
end
```

<a id="update-flashing-flag"></a>

#### 更新闪烁标志

<div align="center">
  <img src="images/flashing_state.png" width=50%>
</div>

<a id="update-traffic-light-status"></a>

#### 更新交通灯状态

<div align="center">
  <img src="images/traffic_light.png" width=50%>
</div>

<a id="estimate-the-color-of-pedestrian-traffic-signals"></a>

### 估计行人交通信号颜色

```plantuml

start
:subscribe detected traffic signals and HDMap;
:extract crosswalk lanelets from HDMap;
:extract road lanelets that conflicts with crosswalk;
:initialize non_red_lanelets(lanelet::ConstLanelets);
if (Latest detection result is **GREEN** or **AMBER**?) then (yes)
  :push back non_red_lanelets;
else (no)
  if (use_last_detect_color is **true**?) then (yes)
    if (Latest detection result is **UNKNOWN** and last detection result is **GREEN** or **AMBER**?) then (yes)
     :push back non_red_lanelets;
    endif
  endif
endif
if (Is there **STRAIGHT-NON-RED** road lanelet in non_red_lanelets?) then (yes)
  :estimate related pedestrian's traffic signal as **RED**;
else if (Is there both **LEFT-NON-RED** and **RIGHT-NON-RED** road lanelet in non_red_lanelets?) then (yes)
  :estimate related pedestrian's traffic signal as **RED**;
else (no)
  :estimate related pedestrian's traffic signal as **UNKNOWN**;
endif
end

```

如果行人和车辆之间的通行由交通信号控制，那么在满足以下条件时，人行横道交通信号可能为**红色**，以阻止行人横穿。

<a id="situation1"></a>

#### 场景 1

- 人行横道与**直行** lanelet 冲突
- 该 lanelet 引用**绿色**或**黄色**交通信号（下图仅显示**绿色**情况）

<div align="center">
  <img src="images/straight.drawio.svg" width=80%>
</div>
<div align="center">
  <img src="images/intersection1.svg" width=80%>
</div>

<a id="situation2"></a>

#### 场景 2

- 人行横道与不同转向方向的 lanelet 冲突（直行与左转、左转与右转、右转与直行）
- 这些 lanelet 引用**绿色**或**黄色**交通信号（下图仅显示**绿色**情况）

<div align="center">
  <img src="images/intersection2.svg" width=80%>
</div>

<a id="map-based-estimation-rules"></a>

### 基于地图的估计规则

可在 lanelet 地图中定义规则，覆盖常规估计结果。
这些规则根据车辆交通灯的值定义人行横道交通灯的值。

例如，当 ID 为 Y 的车辆交通灯为 `red` 时，将 ID 为 X 的人行横道交通灯视为 `green`，可在 lanelet 地图中按如下方式表示：

```XML
  <relation id="Y">
    ...
    <tag k="signal_color_relation:red:green" v="X"/>
```

当前支持颜色 `green`、`amber`、`red` 和 `white`。
可列出多个以逗号分隔且不含任何空白的人行横道 ID（例如 `v="1,2,3"`）。

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分
