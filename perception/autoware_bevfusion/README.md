# autoware_bevfusion

<a id="purpose"></a>

## 用途

`autoware_bevfusion` 功能包用于基于激光雷达或相机与激光雷达融合的 3D 目标检测。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此功能包实现了由 TensorRT 驱动的 BEVFusion [1] 推理节点。
稀疏卷积后端对应 [spconv](https://github.com/traveller59/spconv)。
Autoware 会在设置脚本中自动安装它。如有需要，用户也可按照[以下说明](https://github.com/autowarefoundation/spconv_cpp) 构建并安装。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                   | 类型                            | 说明                                                   |
| ---------------------- | ------------------------------- | ------------------------------------------------------------- |
| `~/input/pointcloud`   | `sensor_msgs::msg::PointCloud2` | 输入点云话题。                                      |
| `~/input/image*`       | `sensor_msgs::msg::Image`       | 输入图像话题。假定图像使用 RGB8 编码。 |
| `~/input/camera_info*` | `sensor_msgs::msg::CameraInfo`  | 输入相机信息话题。                                     |

<a id="output"></a>

### 输出

| 名称                                   | 类型                                             | 说明                 |
| -------------------------------------- | ------------------------------------------------ | --------------------------- |
| `~/output/objects`                     | `autoware_perception_msgs::msg::DetectedObjects` | 检测到的目标。           |
| `debug/cyclic_time_ms`                 | `tier4_debug_msgs::msg::Float64Stamped`          | 循环时间（ms）。           |
| `debug/pipeline_latency_ms`            | `tier4_debug_msgs::msg::Float64Stamped`          | 流水线延迟（ms）。 |
| `debug/processing_time/preprocess_ms`  | `tier4_debug_msgs::msg::Float64Stamped`          | 预处理时间（ms）。            |
| `debug/processing_time/inference_ms`   | `tier4_debug_msgs::msg::Float64Stamped`          | 推理时间（ms）。        |
| `debug/processing_time/postprocess_ms` | `tier4_debug_msgs::msg::Float64Stamped`          | 后处理时间（ms）。      |
| `debug/processing_time/total_ms`       | `tier4_debug_msgs::msg::Float64Stamped`          | 总处理时间（ms）。 |

<a id="parameters"></a>

## 参数

<a id="bevfusion-node"></a>

### BEVFusion 节点

{{ json_to_markdown("perception/autoware_bevfusion/schema/bevfusion.schema.json") }}

<a id="bevfusion-model"></a>

### BEVFusion 模型

{{ json_to_markdown("perception/autoware_bevfusion/schema/ml_package_bevfusion.schema.json") }}

<a id="detection-class-remapper"></a>

### 检测类别重映射器

{{ json_to_markdown("perception/autoware_bevfusion/schema/detection_class_remapper.schema.json") }}

<a id="the-build_only-option"></a>

### `build_only` 选项

`autoware_bevfusion` 节点提供 `build_only` 选项，可从指定 ONNX 文件构建 TensorRT 引擎文件，之后程序退出。

```bash
ros2 launch autoware_bevfusion bevfusion.launch.xml build_only:=true
```

<a id="the-log_level-option"></a>

### `log_level` 选项

`autoware_bevfusion` 的默认日志级别为 `info`。开发人员可使用 `log_level` 参数降低严重性级别以便调试：

```bash
ros2 launch autoware_bevfusion bevfusion.launch.xml log_level:=debug
```

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

此节点假定输入点云遵循 `autoware_point_types` 中定义的 `PointXYZIRC` 布局。

<a id="trained-models"></a>

## 已训练模型

模型包托管于 [Hugging Face](https://huggingface.co/AutowareFoundation/bevfusion/tree/v2.0)。
[ansible artifacts role](https://github.com/autowarefoundation/autoware/tree/main/ansible/roles/artifacts) 会将其下载到 `$(env HOME)/autoware_data/ml_models/bevfusion`。
其中包含仅激光雷达模型和相机与激光雷达融合模型的 ONNX 文件、机器学习包配置及类别重映射器。

模型使用 TIER IV 内部数据库（约 35k 帧激光雷达数据）训练了 30 个 epoch。

<a id="changelog"></a>

### 变更日志

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

[1] Zhijian Liu, Haotian Tang, Alexander Amini, Xinyu Yang, Huizi Mao, Daniela Rus, and Song Han. "BEVFusion: Multi-Task Multi-Sensor Fusion with Unified Bird's-Eye View Representation." 2023 International Conference on Robotics and Automation. <!-- cspell:disable-line -->

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分

虽然此节点能够执行相机与激光雷达融合，但作为 autoware 中首个实际同时使用图像和激光雷达进行推理的方法，其功能包结构及与 autoware 流程的完整集成仍留待后续实现。按当前结构，可不作任何更改直接将其用作基于激光雷达的检测器。
