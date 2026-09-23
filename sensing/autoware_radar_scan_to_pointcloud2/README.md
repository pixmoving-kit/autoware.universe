# radar_scan_to_pointcloud2

## radar_scan_to_pointcloud2_node

- 将 `radar_msgs::msg::RadarScan` 转换为 `sensor_msgs::msg::PointCloud2`
- 计算复杂度为 O(n)
  - n：雷达回波数量

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| ----------- | -------------------------- | ----------- |
| input/radar | radar_msgs::msg::RadarScan | RadarScan |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| --------------------------- | ----------------------------- | ----------------------------------------------------------------- |
| output/amplitude_pointcloud | sensor_msgs::msg::PointCloud2 | 以幅度作为强度值的 PointCloud2 雷达点云。 |
| output/doppler_pointcloud | sensor_msgs::msg::PointCloud2 | 以多普勒速度作为强度值的 PointCloud2 雷达点云。 |

<a id="parameters"></a>

### 参数

| 名称 | 类型 | 说明 |
| ---------------------------- | ---- | ----------------------------------------------------------------------------------------- |
| publish_amplitude_pointcloud | bool | 是否发布以幅度作为强度值的雷达点云。默认值为 `true`。 |
| publish_doppler_pointcloud | bool | 是否发布以多普勒速度作为强度值的雷达点云。默认值为 `false`。 |

<a id="how-to-launch"></a>

### 启动方法

```sh
ros2 launch autoware_radar_scan_to_pointcloud2 radar_scan_to_pointcloud2.launch.xml
```
