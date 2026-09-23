<a id="occupancy-grid-based-validator"></a>

# 基于占据栅格的验证器

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

将占据栅格地图与 DetectedObject 比较，如果障碍物位于自由空间的比例较高，则删除它们。

![调试示例图像](image/occupancy_grid_based_validator/debug_image.png)

基本流程是输入占据栅格地图，生成区分自由空间和其他区域的二值图像。

为每个 DetectedObject 生成掩码图像，并计算掩码内的平均值（比例）。
如果比例较低，则删除该目标。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                         | 类型                                             | 说明                                                 |
| ---------------------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| `~/input/detected_objects`   | `autoware_perception_msgs::msg::DetectedObjects` | DetectedObjects |
| `~/input/occupancy_grid_map` | `nav_msgs::msg::OccupancyGrid`                   | 建议使用未进行时间序列计算的 OccupancyGrid。 |

<a id="output"></a>

### 输出

| 名称               | 类型                                             | 说明               |
| ------------------ | ------------------------------------------------ | ------------------------- |
| `~/output/objects` | `autoware_perception_msgs::msg::DetectedObjects` | 通过验证的 DetectedObjects |

<a id="parameters"></a>

## 参数

| 名称             | 类型  | 说明                                        |
| ---------------- | ----- | -------------------------------------------------- |
| `mean_threshold` | float | 允许的非自由空间比例阈值。 |
| `enable_debug`   | bool  | 是否显示调试图像。 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

当前仅支持以 BoundingBox 表示的车辆。
