# radar_threshold_filter

## radar_threshold_filter_node

通过阈值过滤去除雷达回波中的噪声。

- 幅度过滤：将低幅度回波视为噪声
- 视场过滤：雷达视场边缘的点云容易出现扰动
- 距离过滤：过近的点云往往存在噪声

计算复杂度为 O(n)，`n` 为雷达回波数量。

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| ----------- | ---------------------------- | --------------------- |
| input/radar | radar_msgs/msg/RadarScan.msg | 雷达点云数据 |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| ------------ | ---------------------------- | ------------------------- |
| output/radar | radar_msgs/msg/RadarScan.msg | 过滤后的雷达点云 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("sensing/autoware_radar_threshold_filter/schema/radar_threshold_filter.schema.json") }} |

<a id="how-to-launch"></a>

### 启动方法

```sh
ros2 launch autoware_radar_threshold_filter radar_threshold_filter.launch.xml
```
