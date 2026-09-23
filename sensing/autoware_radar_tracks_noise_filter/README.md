# autoware_radar_tracks_noise_filter

此功能包包含用于 `radar_msgs/msg/RadarTrack` 的雷达目标过滤模块。
此功能包可以过滤 RadarTracks 中的噪声目标。

<a id="algorithm"></a>

## 算法

此功能包的核心算法为 `RadarTrackCrossingNoiseFilterNode::isNoise()` 函数。
详情请参阅该函数及其参数。

- Y 轴阈值

雷达可以通过多普勒速度检测 X 轴速度，但无法直接检测 Y 轴速度。
某些雷达可在设备内部估计 Y 轴速度，但有时精度不足。
在 Y 轴阈值过滤中，如果 RadarTrack 的 Y 轴速度大于 `velocity_y_threshold`，则将其视为噪声目标。

<a id="input"></a>

## 输入

| 名称 | 类型 | 说明 |
| ---------------- | ------------------------------ | ------------------- |
| `~/input/tracks` | radar_msgs/msg/RadarTracks.msg | 检测到的三维轨迹。 |

<a id="output"></a>

## 输出

| 名称 | 类型 | 说明 |
| -------------------------- | ------------------------------ | ---------------- |
| `~/output/noise_tracks` | radar_msgs/msg/RadarTracks.msg | 噪声目标 |
| `~/output/filtered_tracks` | radar_msgs/msg/RadarTracks.msg | 过滤后的目标 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("sensing/autoware_radar_tracks_noise_filter/schema/radar_tracks_noise_filter.schema.json") }}
