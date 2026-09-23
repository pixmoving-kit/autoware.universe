<a id="ar-tag-based-localizer"></a>

# 基于 AR 标签的定位器

**ArTagBasedLocalizer** 是基于视觉的定位节点。

<img src="./doc_image/ar_tag_image.png" width="320px" alt="ar_tag_image">

此节点使用 [ArUco 库](https://index.ros.org/p/aruco/)从相机图像中检测 AR 标签，并根据检测结果计算和发布自车位姿。
假定 AR 标签的位置和姿态已按 Lanelet2 格式记录。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="ar_tag_based_localizer-node"></a>

### `ar_tag_based_localizer` 节点

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| :--------------------- | :---------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `~/input/lanelet2_map` | `autoware_map_msgs::msg::LaneletMapBin` | Lanelet2 数据 |
| `~/input/image` | `sensor_msgs::msg::Image` | 相机图像 |
| `~/input/camera_info` | `sensor_msgs::msg::CameraInfo` | 相机信息 |
| `~/input/ekf_pose` | `geometry_msgs::msg::PoseWithCovarianceStamped` | 未经 IMU 校正的 EKF 位姿。用于过滤误检，验证检测到的 AR 标签。只有 EKF 位姿与通过 AR 标签检测得到的位姿在时间和空间上均处于一定范围内时，后者才会被视为有效并发布。 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| :------------------------------ | :---------------------------------------------- | :---------------------------------------------------------------------------------------- |
| `~/output/pose_with_covariance` | `geometry_msgs::msg::PoseWithCovarianceStamped` | 估计位姿 |
| `~/debug/result` | `sensor_msgs::msg::Image` | [调试话题] 在输入图像上叠加标记检测结果的图像 |
| `~/debug/marker` | `visualization_msgs::msg::MarkerArray` | [调试话题] 已加载的地标，在 RViz 中显示为薄板 |
| `/tf` | `geometry_msgs::msg::TransformStamped` | [调试话题] 从相机到检测标签的 TF |
| `/diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 诊断输出 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("localization/autoware_landmark_based_localizer/autoware_ar_tag_based_localizer/schema/ar_tag_based_localizer.schema.json") }}

<a id="how-to-launch"></a>

## 启动方法

启动 Autoware 时，将 `pose_source` 设为 `artag`。

```bash
ros2 launch autoware_launch ... \
    pose_source:=artag \
    ...
```

### Rosbag

<a id="sample-rosbag-and-map-awsim-data"></a>

#### [示例 rosbag 和地图（AWSIM 数据）](https://drive.google.com/file/d/1ZPsfDvOXFrMxtx7fb1W5sOXdAK1e71hY/view)

这些数据由 [AWSIM](https://tier4.github.io/AWSIM/) 仿真生成。
基于 AR 标签的自定位主要面向较小区域内的行驶，并非此类公共道路驾驶，因此最大行驶速度设为 15 km/h。

一个已知问题是，各 AR 标签开始被检测到的时机会导致估计结果发生明显变化。

![AWSIM 中的示例结果](./doc_image/sample_result_in_awsim.png)

<a id="sample-rosbag-and-map-real-world-data"></a>

#### [示例 rosbag 和地图（真实场景数据）](https://drive.google.com/file/d/1VQCQ_qiEZpCMI3-z6SNs__zJ-4HJFQjx/view)

请重映射话题名称后播放。

```bash
ros2 bag play /path/to/ar_tag_based_localizer_sample_bag/ -r 0.5 -s sqlite3 \
     --remap /sensing/camera/front/image:=/sensing/camera/traffic_light/image_raw \
             /sensing/camera/front/image/info:=/sensing/camera/traffic_light/camera_info
```

此数据集存在 IMU 数据缺失等问题，整体精度较低。即使运行基于 AR 标签的自定位，也能观察到与真实轨迹之间的明显差异。

下图展示了运行示例并绘制得到的轨迹。

![示例结果](./doc_image/sample_result.png)

下方拉取请求中的视频也可供参考。

<https://github.com/autowarefoundation/autoware_universe/pull/4347#issuecomment-1663155248>

<a id="principle"></a>

## 原理

![原理](../doc_image/principle.png)
