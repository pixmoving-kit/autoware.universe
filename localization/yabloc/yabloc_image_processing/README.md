# yabloc_image_processing

此功能包包含一些与图像处理相关的可执行节点。

- [line_segment_detector](#line_segment_detector)
- [graph_segmentation](#graph_segmentation)
- [segment_filter](#segment_filter)
- [undistort](#undistort)
- [lanelet2_overlay](#lanelet2_overlay)
- [line_segments_overlay](#line_segments_overlay)

## line_segment_detector

<a id="purpose"></a>

### 用途

此节点从灰度图像中提取所有线段。

<a id="inputs-outputs"></a>

### 输入／输出

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------- | ----------------- |
| `input/image_raw` | `sensor_msgs::msg::Image` | 去畸变后的图像 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| --------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `output/image_with_line_segments` | `sensor_msgs::msg::Image` | 突出显示线段的图像 |
| `output/line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 以点云形式表示检测到的线段。每个点包含 x、y、z、normal_x、normal_y、normal_z，其中 z 和 normal_z 始终为空。 |

## graph_segmentation

<a id="purpose_1"></a>

### 用途

此节点通过[基于图的分割](https://docs.opencv.org/4.5.4/dd/d19/classcv_1_1ximgproc_1_1segmentation_1_1GraphSegmentation.html)提取路面区域。

<a id="inputs-outputs_1"></a>

### 输入／输出

<a id="input_1"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------- | ----------------- |
| `input/image_raw` | `sensor_msgs::msg::Image` | 去畸变后的图像 |

<a id="output_1"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ------------------------ | ------------------------- | ---------------------------------------------------------- |
| `output/mask_image` | `sensor_msgs::msg::Image` | 以掩码标示路面区域分割结果的图像 |
| `output/segmented_image` | `sensor_msgs::msg::Image` | 用于可视化的分割图像 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_image_processing/schema/graph_segment.schema.json") }}

## segment_filter

<a id="purpose_2"></a>

### 用途

此节点整合 graph_segment 和 lsd 的结果，以提取路面标线。

<a id="inputs-outputs_2"></a>

### 输入／输出

<a id="input_2"></a>

#### 输入

| 名称 | 类型 | 说明 |
| --------------------------- | ------------------------------- | ---------------------------------------------------------- |
| `input/line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 检测到的线段 |
| `input/mask_image` | `sensor_msgs::msg::Image` | 以掩码标示路面区域分割结果的图像 |
| `input/camera_info` | `sensor_msgs::msg::CameraInfo` | 去畸变后的相机信息 |

<a id="output_2"></a>

#### 输出

| 名称 | 类型 | 说明 |
| -------------------------------------- | ------------------------------- | -------------------------------------------------- |
| `output/line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 用于可视化的过滤后线段 |
| `output/projected_image` | `sensor_msgs::msg::Image` | 用于可视化的过滤后线段投影 |
| `output/projected_line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 过滤后线段的投影 |

<a id="parameters_1"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_image_processing/schema/segment_filter.schema.json") }}

## undistort

<a id="purpose_3"></a>

### 用途

此节点同时进行图像缩放和去畸变。

<a id="inputs-outputs_3"></a>

### 输入／输出

<a id="input_3"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ---------------------------- | ----------------------------------- | ----------------------- |
| `input/camera_info` | `sensor_msgs::msg::CameraInfo` | 相机信息 |
| `input/image_raw` | `sensor_msgs::msg::Image` | 原始相机图像 |
| `input/image_raw/compressed` | `sensor_msgs::msg::CompressedImage` | 压缩相机图像 |

此节点同时订阅压缩图像和原始图像话题。
只要接收过一次原始图像，就不再订阅压缩图像。
这样可避免在 Autoware 内部重复解压。

<a id="output_3"></a>

#### 输出

| 名称 | 类型 | 说明 |
| -------------------- | ----------------------------------- | ----------------------------- |
| `output/camera_info` | `sensor_msgs::msg::CameraInfo` | 缩放后的相机信息 |
| `output/image_raw` | `sensor_msgs::msg::CompressedImage` | 去畸变并缩放后的图像 |

<a id="parameters_2"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_image_processing/schema/undistort.schema.json") }}

<a id="about-tf_static-overriding"></a>

#### 关于覆盖 tf_static

<details><summary>点击展开</summary><div>

某些节点需要从 `/base_link` 到 `/sensing/camera/traffic_light/image_raw/compressed` 的 frame_id（例如 `/traffic_light_left_camera/camera_optical_link`）的 `/tf_static`。
可以使用以下命令验证 tf_static 是否正确。

```shell
ros2 run tf2_ros tf2_echo base_link traffic_light_left_camera/camera_optical_link
```

如果因使用原型车辆、缺少准确标定数据或其他不可避免的原因，广播了错误的 `/tf_static`，可以在 `override_camera_frame_id` 中指定 frame_id。
如果提供非空字符串，`/image_processing/undistort_node` 会重写 camera_info 中的 frame_id。
例如，可以按以下方式提供不同的 tf_static。

```shell
ros2 launch yabloc_launch sample_launch.xml override_camera_frame_id:=fake_camera_optical_link
ros2 run tf2_ros static_transform_publisher \
  --frame-id base_link \
  --child-frame-id fake_camera_optical_link \
  --roll -1.57 \
  --yaw -1.570
```

</div></details>

## lanelet2_overlay

<a id="purpose_4"></a>

### 用途

此节点根据估计的自车位置，将 Lanelet2 叠加到相机图像上。

<a id="inputs-outputs_4"></a>

### 输入／输出

<a id="input_4"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------------------------- | ---------------------------------- | --------------------------------------------------- |
| `input/pose` | `geometry_msgs::msg::PoseStamped` | 估计的自车位姿 |
| `input/projected_line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 投影后的线段，包含非道路标线 |
| `input/camera_info` | `sensor_msgs::msg::CameraInfo` | 去畸变后的相机信息 |
| `input/image_raw` | `sensor_msgs::msg::Image` | 去畸变后的相机图像 |
| `input/ground` | `std_msgs::msg::Float32MultiArray` | 地面倾斜信息 |
| `input/ll2_road_marking` | `sensor_msgs::msg::PointCloud2` | 与路面标线相关的 Lanelet2 元素 |
| `input/ll2_sign_board` | `sensor_msgs::msg::PointCloud2` | 与交通标志牌相关的 Lanelet2 元素 |

<a id="output_4"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ------------------------------- | --------------------------------- | ------------------------------------------------------ |
| `output/lanelet2_overlay_image` | `sensor_msgs::msg::Image` | 叠加 Lanelet2 的图像 |
| `output/projected_marker` | `visualization_msgs::msg::Marker` | 投影到三维空间的线段，包含非道路标线 |

## line_segments_overlay

<a id="purpose_5"></a>

### 用途

此节点在相机图像上可视化分类后的线段。

<a id="inputs-outputs_5"></a>

### 输入／输出

<a id="input_5"></a>

#### 输入

| 名称 | 类型 | 说明 |
| --------------------------- | ------------------------------- | ------------------------ |
| `input/line_segments_cloud` | `sensor_msgs::msg::PointCloud2` | 分类后的线段 |
| `input/image_raw` | `sensor_msgs::msg::Image` | 去畸变后的相机图像 |

<a id="output_5"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ----------------------------------------- | ------------------------- | ------------------------------------ |
| `output/image_with_colored_line_segments` | `sensor_msgs::msg::Image` | 突出显示线段的图像 |
