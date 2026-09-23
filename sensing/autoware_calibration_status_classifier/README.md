# autoware_calibration_status_classifier

<a id="purpose"></a>

## 用途

`autoware_calibration_status_classifier` 功能包利用深度学习推理，实时验证激光雷达与相机的标定状态。它通过神经网络分析叠加在相机图像上的投影点云，检测激光雷达与相机传感器之间的标定偏差。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

标定状态检测系统按照以下流程运行：

<a id="1-data-preprocessing-cuda-accelerated"></a>

### 1. 数据预处理（CUDA 加速）

- **图像去畸变**：校正相机畸变
- **点云投影**：将三维激光雷达点投影到去畸变后的二维图像平面，添加深度和强度信息
- **形态学膨胀**：增强点的可见性，作为神经网络输入

<a id="2-neural-network-inference-tensorrt"></a>

### 2. 神经网络推理（TensorRT）

- **输入格式**：5 通道归一化数据（RGB + 深度 + 强度）
- **架构**：使用标定正确／存在标定偏差的数据训练的深度神经网络
- **输出**：带置信度分数的标定状态二分类结果

<a id="3-runtime-modes"></a>

### 3. 运行模式

- **MANUAL**：通过服务调用按需验证
- **PERIODIC**：按可配置的时间间隔定期验证
- **ACTIVE**：使用同步的传感器数据持续监测

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| -------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------- |
| `~/input/linear_velocity` | `prerequisite.linear_velocity_check.source` 参数 | 用于线速度检查的车辆速度（支持多种消息类型） |
| `~/input/angular_velocity` | `prerequisite.angular_velocity_check.source` 参数 | 用于角速度检查的车辆速度（支持多种消息类型） |
| `~/input/objects` | `prerequisite.objects_check.source` 参数 | 用于目标检查的检测目标（支持多种消息类型） |
| `input.cloud_topics` | `sensor_msgs::msg::PointCloud2` | 激光雷达点云数据 |
| `input.image_topics` | `sensor_msgs::msg::Image` | 相机图像数据（BGR8 格式） |
| 相机信息话题 | `sensor_msgs::msg::CameraInfo` | 相机内参和畸变系数 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ---------------------------- | --------------------------------------- | ------------------------------------------ |
| `/diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 包含标定状态的 ROS 诊断信息 |
| `~/validate_calibration_srv` | `std_srvs::srv::Trigger` | 手动验证服务（MANUAL 模式） |
| 预览图像话题 | `sensor_msgs::msg::Image` | 叠加投影点的可视化图像 |

<a id="services"></a>

### 服务

| 名称 | 类型 | 说明 |
| ---------------------------------- | ------------------------ | ------------------------------------- |
| `~/input/validate_calibration_srv` | `std_srvs::srv::Trigger` | 手动标定验证请求 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

{{ json_to_markdown("sensing/autoware_calibration_status_classifier/schema/calibration_status_classifier.schema.json") }}

<a id="network-parameters"></a>

### 网络参数

{{ json_to_markdown("sensing/autoware_calibration_status_classifier/schema/ml_package_calibration_status_classifier.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 输入图像必须采用 BGR8 格式（每通道 8 位）
- 输入点云应包含强度信息（XYZIRC 格式）

<a id="usage-example"></a>

## 使用示例

```bash
ros2 launch autoware_calibration_status_classifier calibration_status_classifier.launch.xml
```

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展／尚未实现的部分

- 提供包含详细响应的手动运行模式（自定义 srv）
- 将场景目标计数筛选器替换为相机视场内的目标计数筛选器（光线追踪）
- 为多组相机与激光雷达配对提供多线程处理
- 增加更多筛选条件（例如横摆角速度）
- 支持 cuda_blackboard
- 在适用场景下用 NPP 函数替换自定义内核

<a id="references"></a>

## 参考资料

- [AWML——标定状态分类](https://github.com/tier4/AWML/tree/main/projects/CalibrationStatusClassification)
