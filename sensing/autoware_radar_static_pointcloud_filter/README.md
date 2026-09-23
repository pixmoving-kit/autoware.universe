# radar_static_pointcloud_filter

## radar_static_pointcloud_filter_node

利用多普勒速度和自车运动提取静态／动态雷达点云。
计算复杂度为 O(n)，`n` 为雷达点云中的点数。

<a id="input-topics"></a>

### 输入话题

| 名称 | 类型 | 说明 |
| -------------- | -------------------------- | -------------------------- |
| input/radar | radar_msgs::msg::RadarScan | RadarScan |
| input/odometry | nav_msgs::msg::Odometry | 自车里程计话题 |

<a id="output-topics"></a>

### 输出话题

| 名称 | 类型 | 说明 |
| ------------------------- | -------------------------- | ------------------------ |
| output/static_radar_scan | radar_msgs::msg::RadarScan | 静态雷达点云 |
| output/dynamic_radar_scan | radar_msgs::msg::RadarScan | 动态雷达点云 |

<a id="parameters"></a>

### 参数

| 名称 | 类型 | 说明 |
| ------------------- | ------ | ---------------------------------------------------- |
| doppler_velocity_sd | double | 雷达多普勒速度的标准差。[m/s] |

<a id="how-to-launch"></a>

### 启动方法

```sh
ros2 launch autoware_radar_static_pointcloud_filter radar_static_pointcloud_filter.launch.xml
```

<a id="algorithm"></a>

### 算法

![算法](docs/radar_static_pointcloud_filter.drawio.svg)
