# blockage_diag

<a id="purpose"></a>

## 用途

为保证激光雷达性能和自动驾驶安全，需要提供
异常状态诊断功能。
激光雷达遮挡是指异物附着在激光雷达上，阻挡发射的光脉冲和接收的回波
信号，从而导致异常状态。
此节点用于检测激光雷达是否存在遮挡，以及遮挡的大小和位置。

<a id="inner-workings-algorithmsblockage-detection"></a>

## 内部机制／算法（遮挡检测）

此节点根据无回波区域及其位置判断是否存在遮挡。

![遮挡情况](./image/blockage_diag.png)

处理逻辑如下图所示。

![遮挡诊断流程图](./image/blockage_diag_flowchart.drawio.svg)

<a id="inner-workings-algorithmsdust-detection"></a>

## 内部机制／算法（扬尘检测）

扬尘检测采用形态学处理。
如果在预期能接收到地面点云回波的激光雷达区域内，
由于扬尘而无法获取激光射线的回波，
则深度图中会出现作为噪声的黑色像素。
通过对这些黑色像素进行腐蚀和膨胀，确定噪声区域。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------------------- | ------------------------------- | --------------------------------------------------------------- |
| `~/input/pointcloud_raw_ex` | `sensor_msgs::msg::PointCloud2` | 用于检测无回波区域的原始点云数据 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| :-------------------------------------------------------- | :-------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `~/output/blockage_diag/debug/blockage_mask_image` | `sensor_msgs::msg::Image` | 检测到的遮挡区域掩码图像 |
| `~/output/blockage_diag/debug/ground_blockage_ratio` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 地面区域中遮挡区域的面积占比 |
| `~/output/blockage_diag/debug/sky_blockage_ratio` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 天空区域中遮挡区域的面积占比 |
| `~/output/blockage_diag/debug/lidar_depth_map` | `sensor_msgs::msg::Image` | 输入点云的深度图像 |
| `~/output/blockage_diag/debug/single_frame_dust_mask` | `sensor_msgs::msg::Image` | 最新单帧中检测到的扬尘区域掩码图像 |
| `~/output/blockage_diag/debug/multi_frame_dust_mask` | `sensor_msgs::msg::Image` | 持续检测到的扬尘区域掩码图像 |
| `~/output/blockage_diag/debug/blockage_dust_merged_image` | `sensor_msgs::msg::Image` | 遮挡检测结果（红色）与多帧扬尘区域检测结果（黄色）的合成图像 |
| `~/output/blockage_diag/debug/ground_dust_ratio` | `autoware_internal_debug_msgs::msg::Float32Stamped` | 扬尘区域面积与通常能接收到地面回波的区域面积之比。 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/blockage_diag_node.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

1. 仅测试过 Hesai Pandar40P 和 Hesai PandarQT。对于新的激光雷达，需要手动检查通道 ID 在
   垂直方向上的排列顺序，并修改代码。
2. 扬尘区域检测在路面因降雨等原因出现积水时会产生误报。
   此外，目前无法检测射向天空的激光射线所覆盖的区域。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
