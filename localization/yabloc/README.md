# YabLoc

**YabLoc** 是基于视觉和矢量地图的定位系统。[https://youtu.be/Eaf6r_BNFfk](https://youtu.be/Eaf6r_BNFfk)

[![缩略图](docs/yabloc_thumbnail.jpg)](https://youtu.be/Eaf6r_BNFfk)

它将图像中提取的路面标线与矢量地图匹配，以估计位置。
无需点云地图或激光雷达。
YabLoc 可为未配备激光雷达的车辆定位，也可用于无法获取点云地图的环境。

<a id="packages"></a>

## 功能包

- [yabloc_common](yabloc_common/README.md)
- [yabloc_image_processing](yabloc_image_processing/README.md)
- [yabloc_particle_filter](yabloc_particle_filter/README.md)
- [yabloc_pose_initializer](yabloc_pose_initializer/README.md)

<a id="how-to-launch-yabloc-instead-of-ndt"></a>

## 使用 YabLoc 替代 NDT 的启动方法

启动 Autoware 时，如果设置参数 `pose_source:=yabloc`，就会启动 YabLoc 来替代 NDT。
默认情况下，`pose_source` 为 `ndt`。

以下是运行 YabLoc 的示例命令。

```shell
ros2 launch autoware_launch logging_simulator.launch.xml \
  map_path:=$HOME/autoware_data/maps/sample-map-rosbag\
  vehicle_model:=sample_vehicle \
  sensor_model:=sample_sensor_kit \
  pose_source:=yabloc
```

<a id="architecture"></a>

## 架构

![节点图](docs/yabloc_architecture.drawio.svg)

<a id="principle"></a>

## 原理

下图展示 YabLoc 的基本原理。
它利用基于图的分割得到道路区域，再提取线段，从而获取路面标线。
图中上方中央的红线表示被识别为路面标线的线段。
YabLoc 针对每个粒子变换这些线段，并将其与由 Lanelet2 生成的代价地图比较，以确定粒子权重。

![原理](docs/yabloc_principle.png)

<a id="visualization"></a>

## 可视化

<a id="core-visualization-topics"></a>

### 核心可视化话题

默认情况下，不会可视化这些话题。

<img src="docs/yabloc_rviz_description.png" width=800>

| 序号 | 话题名称 | 说明 |
| ----- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | `/localization/yabloc/pf/predicted_particle_marker` | 粒子滤波器的粒子分布。红色粒子是可能性较高的候选。 |
| 2 | `/localization/yabloc/pf/scored_cloud` | 投影到三维空间的线段，颜色表示与地图的匹配程度。 |
| 3 | `/localization/yabloc/image_processing/lanelet2_overlay_image` | 根据估计位姿，将 Lanelet2（黄色线条）叠加到图像上。如果与实际路面标线匹配良好，说明定位效果良好。 |

<a id="image-topics-for-debug"></a>

### 用于调试的图像话题

默认情况下，不会可视化这些话题。

<img src="docs/yabloc_image_description.png" width=800>

| 序号 | 话题名称 | 说明 |
| ----- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 1 | `/localization/yabloc/pf/cost_map_image` | 根据 Lanelet2 生成的代价地图 |
| 2 | `/localization/yabloc/pf/match_image` | 投影后的线段 |
| 3 | `/localization/yabloc/image_processing/image_with_colored_line_segment` | 分类后的线段。绿色线段用于粒子校正 |
| 4 | `/localization/yabloc/image_processing/lanelet2_overlay_image` | Lanelet2 叠加图像 |
| 5 | `/localization/yabloc/image_processing/segmented_image` | 基于图的分割结果 |

<a id="limitation"></a>

## 限制

- 不支持同时运行 YabLoc 和 NDT。
  - 原因是同时运行两者的计算成本可能过高。
  - 此外，大多数情况下 NDT 优于 YabLoc，因此同时运行的收益较小。
- 不估计滚转角和俯仰角，因此部分感知节点可能无法正常工作。
- 目前不支持多相机，未来将提供支持。
- 在路口等路面标线较少的区域，估计结果高度依赖 GNSS、IMU 和车辆轮式里程计。
- 如果 Lanelet2 中不包含道路边界或路面标线，估计很可能失败。
- Autoware 教程提供的示例 rosbag 不包含图像，因此无法使用它运行 YabLoc。
  - 如需测试 YabLoc 功能，可以使用此 [PR](https://github.com/autowarefoundation/autoware_universe/pull/3946) 中提供的示例测试数据。
