<a id="obstacle-pointcloud-based-validator"></a>

# 基于障碍物点云的验证器

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

如果 DetectedObjects 内的障碍物点数量较少，则将其视为误检并移除。
障碍物点云可以是经过地图比较过滤或地面过滤后的点云。

![调试示例图像](image/obstacle_pointcloud_based_validator/debug_image.gif)

上方调试图像中，红色 DetectedObject 为通过验证的目标，蓝色目标为被删除的目标。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                          | 类型                                             | 说明                             |
| ----------------------------- | ------------------------------------------------ | --------------------------------------- |
| `~/input/detected_objects`    | `autoware_perception_msgs::msg::DetectedObjects` | DetectedObjects |
| `~/input/obstacle_pointcloud` | `sensor_msgs::msg::PointCloud2`                  | 动态目标的障碍物点云 |

<a id="output"></a>

### 输出

| 名称               | 类型                                             | 说明               |
| ------------------ | ------------------------------------------------ | ------------------------- |
| `~/output/objects` | `autoware_perception_msgs::msg::DetectedObjects` | 通过验证的 DetectedObjects |

<a id="parameters"></a>

## 参数

| 名称                            | 类型  | 说明                                                                                                                                                                |
| ------------------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `using_2d_validator`            | bool  | 使用投影至 xy 平面的（2D）障碍物点云进行验证 |
| `min_points_num`                | int   | DetectedObjects 中障碍物点云的最小点数 |
| `max_points_num`                | int   | DetectedObjects 中障碍物点云的最大点数 |
| `min_points_and_distance_ratio` | float | 距 baselink 为 1m 时每个目标的点云点数阈值，因为点数会随到 baselink 的距离而变化。 |
| `enable_debugger`               | bool  | 是否创建调试话题 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

当前仅支持以 BoundingBox 或 Cylinder 表示的目标。
