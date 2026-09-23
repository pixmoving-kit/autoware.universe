# autoware_tensorrt_vad

<a id="overview"></a>

## 概述

`autoware_tensorrt_vad` 是一个 ROS 2 组件，使用经过 TensorRT 优化的矢量化自动驾驶（VAD）模型实现端到端自动驾驶。它采用 [VAD 模型](https://github.com/hustvl/VAD)（Jiang 等，2023），并使用 NVIDIA 的 [DL4AGX](https://github.com/NVIDIA/DL4AGX) TensorRT 框架进行部署优化。 <!-- cSpell:ignore Jiang Shaoyu Bencheng Liao Jiajie Helong Wenyu Xinggang -->

此模块使用单个神经网络替代传统的定位、感知和规划模块，利用 CARLA 仿真数据在 [Bench2Drive](https://github.com/Thinklab-SJTU/Bench2Drive) 基准（Jia 等，2024）上训练。它与 [Autoware](https://autowarefoundation.github.io/autoware-documentation/main/) 无缝集成，专为 Autoware 框架设计。

---

<a id="features"></a>

## 功能

- **一体式端到端架构**：单个神经网络直接将相机输入映射为轨迹，以统一模型替代整个传统感知与规划流程，无需独立的检测、跟踪、预测或规划模块
- **多相机感知**：同时处理 6 路环视相机，实现 360° 环境感知
- **矢量化场景表示**：使用矢量地图高效编码场景，降低计算开销
- **实时 TensorRT 推理**：针对嵌入式部署进行优化，推理耗时约为 20ms
- **集成感知输出**：生成目标预测（含未来轨迹）和地图元素作为辅助输出
- **时序建模**：利用历史特征改善时间一致性和预测精度

---

<a id="visualization"></a>

## 可视化

<a id="lane-following-demo"></a>

### 车道跟随演示

![车道跟随](media/lane_follow_demo.jpg)

<a id="turn-right-demo"></a>

### 右转演示

![右转](media/turn_right_demo.jpg)

---

<a id="parameters"></a>

## 参数

可通过配置文件设置参数：

- 部署配置（节点和接口参数）：`config/vad_carla_tiny.param.yaml`
- 模型架构参数：`vad-carla-tiny.param.json`（随模型下载至 `~/autoware_data/ml_models/vad/v0.1/`）

---

<a id="inputs"></a>

## 输入

| 话题                   | 消息类型                                 | 说明                                                                    |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------------------------ |
| ~/input/image\*         | sensor_msgs/msg/Image\*                      | 相机图像 0-5：FRONT、BACK、FRONT_LEFT、BACK_LEFT、FRONT_RIGHT、BACK_RIGHT |
| ~/input/camera_info\*   | sensor_msgs/msg/CameraInfo                   | 相机 0-5 的标定信息                                             |
| ~/input/kinematic_state | nav_msgs/msg/Odometry                        | 车辆里程计                                                               |
| ~/input/acceleration    | geometry_msgs/msg/AccelWithCovarianceStamped | 车辆加速度                                                           |

\*图像传输支持原始和压缩格式。可通过 `use_raw` 参数逐相机配置（默认：压缩格式）。

---

<a id="outputs"></a>

## 输出

| 话题                 | 消息类型                                              | 说明                         |
| --------------------- | --------------------------------------------------------- | ----------------------------------- |
| ~/output/trajectory   | autoware_planning_msgs/msg/Trajectory                     | 选定的自车轨迹             |
| ~/output/trajectories | autoware_internal_planning_msgs/msg/CandidateTrajectories | 全部 6 条候选轨迹        |
| ~/output/objects      | autoware_perception_msgs/msg/PredictedObjects             | 带轨迹的预测目标 |
| ~/output/map          | visualization_msgs/msg/MarkerArray                        | 预测的地图元素              |

---

<a id="building"></a>

## 构建

使用 colcon 构建此功能包：

```bash
colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DCMAKE_BUILD_TYPE=Release --packages-up-to autoware_tensorrt_vad
```

---

<a id="testing"></a>

## 测试

<a id="unit-tests"></a>

### 单元测试

已提供单元测试，可通过以下命令运行：

```bash
colcon test --packages-select autoware_tensorrt_vad
colcon test-result --all
```

如需详细输出：

```bash
colcon test --packages-select autoware_tensorrt_vad --event-handlers console_cohesion+
```

<a id="carla-simulator-testing"></a>

### CARLA 仿真器测试

首先按照 [autoware_carla_interface](https://github.com/autowarefoundation/autoware.universe/tree/main/simulator/autoware_carla_interface) 的说明设置 CARLA。

然后启动 E2E VAD 系统：

```bash
ros2 launch autoware_launch e2e_simulator.launch.xml \
  map_path:=$HOME/autoware_data/maps/Town01 \
  vehicle_model:=sample_vehicle \
  sensor_model:=carla_sensor_kit \
  simulator_type:=carla \
  use_e2e_planning:=true
```

---

<a id="model-setup-and-versioning"></a>

## 模型设置与版本管理

<a id="model-download"></a>

### 模型下载

设置 Autoware 开发环境时会自动下载 VAD 模型文件。

要下载最新模型，只需运行提供的设置脚本：
[如何设置开发环境](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/source-installation/#how-to-set-up-a-development-environment)

默认情况下，模型将下载至 `~/autoware_data/ml_models/vad/`。

**手动下载**（如有需要）：
模型托管地址：<https://huggingface.co/AutowareFoundation/tensorrt_vad/tree/v0.1>

<a id="model-preparation"></a>

### 模型准备

> :warning: **注意**：节点首次运行时会自动从 ONNX 模型构建 TensorRT 引擎。构建好的引擎将被缓存供后续运行使用，并与硬件相关。

**模型组件**（使用 Bench2Drive CARLA 数据集训练）：

- `vad-carla-tiny_backbone.onnx` - 图像特征提取骨干网络
- `vad-carla-tiny_head_no_prev.onnx` - 规划头（首帧）
- `vad-carla-tiny_head.onnx` - 时序规划头

如需手动配置模型路径，请在 `config/vad_carla_tiny.param.yaml` 中更新：

```yaml
model_params:
  nets:
    backbone:
      onnx_path: "$(var model_path)/v0.1/vad-carla-tiny_backbone.onnx"
      engine_path: "$(var model_path)/v0.1/vad-carla-tiny_backbone.engine"
    head:
      onnx_path: "$(var model_path)/v0.1/vad-carla-tiny_head.onnx"
      engine_path: "$(var model_path)/v0.1/vad-carla-tiny_head.engine"
    head_no_prev:
      onnx_path: "$(var model_path)/v0.1/vad-carla-tiny_head_no_prev.onnx"
      engine_path: "$(var model_path)/v0.1/vad-carla-tiny_head_no_prev.engine"
```

1. **启动节点**：首次运行时，节点将自动：
   - 从 ONNX 模型构建 TensorRT 引擎
   - 针对当前 GPU 进行优化
   - 将引擎缓存到指定的 `engine_path` 位置
   - 骨干网络使用 FP16 精度，头部网络使用 FP32 精度（可配置）

<a id="model-version-history"></a>

### 模型版本历史

| 版本 | 训练数据集  | 发布日期 | 备注                                                                                                    | 节点兼容版本 |
| ------- | ----------------- | ------------ | -------------------------------------------------------------------------------------------------------- | ------------------ |
| **0.1** | Bench2Drive CARLA | 2025-11-04   | - 首次发布<br>- 6 相机环视<br>- 使用 CARLA Towns 训练<br>- FP16/FP32 混合精度 | >= 0.1.0           |

---

<a id="limitations"></a>

## ❗ 局限性

虽然 VAD 展现出了有前景的端到端驾驶能力，但用户应了解以下局限：

<a id="training-data-constraints"></a>

### 训练数据限制

- **仅使用仿真数据训练**：模型完全使用 CARLA 仿真器数据训练，可能无法涵盖真实驾驶场景的全部复杂性和变化

<a id="lack-of-high-level-command-interface"></a>

### 缺少高层指令接口

- **不支持动态任务控制**：当前实现缺少高层指令接口，因此模型无法在运行时动态切换驾驶行为（例如从“沿车道行驶”切换到“在下一个路口右转”）

---

<a id="development-contribution"></a>

## 开发与贡献

- 遵循 [Autoware 编码规范](https://autowarefoundation.github.io/autoware-documentation/main/contributing/)。
- 欢迎通过 GitHub issue 和 pull request 提交贡献、缺陷报告和功能需求。

---

<a id="references"></a>

## 参考资料

<a id="core-model"></a>

### 核心模型

1. VAD: Vectorized Scene Representation for Efficient Autonomous Driving (2023)
   - 论文：[arXiv:2303.12077](https://arxiv.org/abs/2303.12077)
   - 代码：[github.com/hustvl/VAD](https://github.com/hustvl/VAD)

<a id="training-and-datasets"></a>

### 训练与数据集

1. Bench2Drive: Towards Multi-Ability Benchmarking of Closed-Loop End-To-End Autonomous Driving (2024)
   - 论文：[arXiv:2406.03877](https://arxiv.org/abs/2406.03877)
   - 代码：[github.com/Thinklab-SJTU/Bench2Drive](https://github.com/Thinklab-SJTU/Bench2Drive)
   - 说明：基于 CARLA 的端到端自动驾驶评估基准

<a id="deployment-and-optimization"></a>

### 部署与优化

1. DL4AGX (2024)
   - 资源：[github.com/NVIDIA/DL4AGX](https://github.com/NVIDIA/DL4AGX)
   - 说明：面向自动驾驶工作负载的 TensorRT 优化及嵌入式 GPU 部署策略

<a id="related-work"></a>

### 相关工作

1. nuScenes: A Multimodal Dataset for Autonomous Driving (2020)
   - 论文：[arXiv:1903.11027](https://arxiv.org/abs/1903.11027)
   - 数据集：[nuscenes.org](https://www.nuscenes.org)

2. BEVFormer: Learning Bird's-Eye-View Representation from Multi-Camera Images via Spatiotemporal Transformers (2022)
   - 论文：[arXiv:2203.17270](https://arxiv.org/abs/2203.17270)

---

## License

This package is released under the Apache 2.0 License.
