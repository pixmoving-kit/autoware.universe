# distortion_corrector

<a id="purpose"></a>

## 用途

`distortion_corrector` 节点用于补偿一次扫描期间自车运动引起的点云畸变。

激光雷达通过旋转内部激光束进行扫描。如果自车在一次扫描期间发生移动，生成的点云就会产生畸变（如下图所示）。此节点利用自车里程计信息对传感器数据进行插值，以校正畸变。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

节点使用 `~/input/twist` 话题中的速度信息（线速度和角速度）校正点云中的每个点。如果将 `use_imu` 设为 true，节点会用 IMU 的角速度替代速度消息中的角速度。

节点支持两种畸变校正模式：二维畸变校正和三维畸变校正。主要区别在于，二维校正仅使用 X 轴线速度和 Z 轴角速度校正点的位置，而三维校正使用全部线速度和角速度分量。

请注意，两种校正方式的处理时间差异明显；三维校正比二维校正耗时多 50%。因此，通常建议将 `use_3d_distortion_correction` 设为 `false`。但在车辆经过减速带等场景下，使用三维校正可能更有帮助。

![畸变校正示意图](./image/distortion_corrector.jpg)

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| -------------------- | ------------------------------------------------ | ---------------------------------- |
| `~/input/pointcloud` | `sensor_msgs::msg::PointCloud2` | 畸变点云话题。 |
| `~/input/twist` | `geometry_msgs::msg::TwistWithCovarianceStamped` | 速度信息话题。 |
| `~/input/imu` | `sensor_msgs::msg::Imu` | IMU 数据话题。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| --------------------- | ------------------------------- | ----------------------------------- |
| `~/output/pointcloud` | `sensor_msgs::msg::PointCloud2` | 去畸变后的点云话题 |

<a id="parameters"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/distortion_corrector_node.schema.json") }}

<a id="launch"></a>

## 启动

```bash
ros2 launch autoware_pointcloud_preprocessor distortion_corrector.launch.xml
```

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 节点要求激光雷达、速度和 IMU 话题之间时间同步。
- 如果希望在不使用 IMU 的情况下进行三维畸变校正，请确认速度消息中的线速度和角速度字段不为空。
- 当输入点云位于传感器坐标系（而非 `base_link`）且 `update_azimuth_and_distance` 参数设为 `true` 时，节点会根据去畸变后的 XYZ 坐标更新各点的方位角和距离。方位角通过 OpenCV 的 `cv::fastAtan2` 函数的修改版本计算。
- 请注意，更新方位角和距离字段会使执行时间增加约 20%。此外，`cv::fastAtan2` 算法的最大误差为 0.3 度，因此**对于方位角分辨率较高的激光雷达，可能会改变光束顺序**。
- 不同厂商的激光雷达采用不同的方位角坐标定义，如下图所示。目前已测试以下坐标系，节点会根据输入坐标系更新方位角。
  - `velodyne`：（x：0 度，y：270 度）
  - `hesai`：（x：90 度，y：0 度）
  - `others`：（x：0 度，y：90 度）和（x：270 度，y：0 度）

| ![Velodyne 方位角坐标](./image/velodyne.drawio.png) | ![Hesai 方位角坐标](./image/hesai.drawio.png) |
| :---------------------------------------------------------: | :---------------------------------------------------: |
| **Velodyne 方位角坐标** | **Hesai 方位角坐标** |

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

<https://docs.opencv.org/3.4/db/de0/group__core__utils.html#ga7b356498dd314380a0c386b059852270>
