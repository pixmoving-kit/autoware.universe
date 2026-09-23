<a id="autoware-diffusion-planner"></a>

# Autoware 扩散规划器

<a id="overview"></a>

## 概述

**Autoware 扩散规划器**是面向自动驾驶车辆的轨迹生成模块，设计用于 [Autoware](https://autoware.org/) 生态系统。它使用 [Diffusion Planner](https://github.com/ZhengYinan-AIR/Diffusion-Planner) 模型，该模型由 Zheng 等人在论文 ["Diffusion-Based Planning for Autonomous Driving with Flexible Guidance"](https://arxiv.org/abs/2501.15564) 中介绍。<!-- cSpell:ignore Zheng -->

此规划器通过考虑以下因素来生成平滑、可行且安全的轨迹：

- 动态和静态障碍物
- 车辆运动学
- 用户定义的约束
- Lanelet2 地图上下文
- 交通信号灯和速度限制

它以 ROS 2 组件节点实现，便于集成到基于 Autoware 的软件栈。该节点面向拟议的 [Autoware 新规划框架](https://github.com/tier4/new_planning_framework)。

---

<a id="how-to-use"></a>

## 使用方法

<a id="1-prerequisites"></a>

### (1) 前提条件

确保 `planning/autoware_diffusion_planner/config/diffusion_planner.param.yaml` 中指定的目录指向正确的模型版本，并包含所需的模型权重文件和参数文件。

```bash
$ ls ~/autoware_data/ml_models/diffusion_planner/v4.0/
diffusion_planner.onnx diffusion_planner.param.json
```

可按照[下载制品](https://github.com/autowarefoundation/autoware/blob/main/ansible/roles/artifacts/README.md#download-artifacts)中的说明下载。

<a id="2-launch-the-planning-simulator"></a>

### (2) 启动规划仿真器

传入 `planning_setting:=diffusion_planner`，将规划栈从基于规则的场景规划器切换为扩散规划器。此参数会自动替换轨迹生成器、规划验证器的输入话题以及诊断图，无需额外修改启动文件。

```bash
ros2 launch autoware_launch planning_simulator.launch.xml \
  map_path:=/path/to/your/map \
  vehicle_model:=sample_vehicle \
  sensor_model:=sample_sensor_kit \
  planning_setting:=diffusion_planner
```

<a id="features"></a>

## 功能

- **基于扩散的轨迹生成**，实现灵活且稳健的规划

  [![基于扩散的轨迹生成](media/diffusion_planner.gif)](media/diffusion_planner.gif)

- **集成 Lanelet2 地图**，提供车道级上下文

  [![Lanelet 地图集成](media/lanelet_map_integration.png)](media/lanelet_map_integration.png)

- 利用感知输入**处理动态和静态障碍物**

  [![对静态参与者的响应](media/diffusion_planner_reacts_to_bus.gif)](media/diffusion_planner_reacts_to_bus.gif)

  [![扩散规划器](media/reaction_to_other_agents.gif)](media/reaction_to_other_agents.gif)

- **感知交通信号灯和速度限制**

  [![交通信号灯支持](media/traffic_light_support.gif)](media/traffic_light_support.gif)

- 使用 **ONNX Runtime** 推理，快速执行神经网络
- 通过 **ROS 2 发布器**发布规划轨迹、预测对象和调试标记

---

<a id="parameters"></a>

## 参数

{{ json_to_markdown("planning/autoware_diffusion_planner/schema/diffusion_planner.schema.json") }}

参数可通过 YAML 设置（参见 `config/diffusion_planner.param.yaml`）。

---

<a id="inputs"></a>

## 输入

| 话题 | 消息类型 | 说明 |
| ------------------------- | --------------------------------------------------- | -------------------------- |
| `~/input/odometry` | nav_msgs/msg/Odometry | 自车里程计信息 |
| `~/input/acceleration` | geometry_msgs/msg/AccelWithCovarianceStamped | 自车加速度 |
| `~/input/tracked_objects` | autoware_perception_msgs/msg/TrackedObjects | 检测到的动态对象 |
| `~/input/traffic_signals` | autoware_perception_msgs/msg/TrafficLightGroupArray | 交通信号灯状态 |
| `~/input/vector_map` | autoware_map_msgs/msg/LaneletMapBin | Lanelet2 地图 |
| `~/input/route` | autoware_planning_msgs/msg/LaneletRoute | 路线信息 |
| `~/input/turn_indicators` | autoware_vehicle_msgs/msg/TurnIndicatorsReport | 转向指示灯信息 |

<a id="outputs"></a>

## 输出

| 话题 | 消息类型 | 说明 |
| ------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------- |
| `~/output/trajectory` | autoware_planning_msgs/msg/Trajectory | 为自车规划的轨迹 |
| `~/output/trajectories` | autoware_internal_planning_msgs/msg/CandidateTrajectories | 多条候选轨迹 |
| `~/output/predicted_objects` | autoware_perception_msgs/msg/PredictedObjects | 动态对象的未来状态预测 |
| `~/output/turn_indicators` | autoware_vehicle_msgs/msg/TurnIndicatorsCommand | 规划的转向指示灯命令 |
| `~/output/debug/traffic_signal` | autoware_perception_msgs/msg/TrafficLightGroup | 路线上自车前方的第一个交通信号灯，用于 RViz/ad_api |
| `~/debug/lane_marker` | visualization_msgs/msg/MarkerArray | 车道调试标记 |
| `~/debug/route_marker` | visualization_msgs/msg/MarkerArray | 路线调试标记 |

---

<a id="testing"></a>

## 测试

本模块提供单元测试，可使用以下命令运行：

```bash
colcon test --packages-select autoware_diffusion_planner
colcon test-result --all
```

---

<a id="onnx-model-and-versioning"></a>

## ONNX 模型与版本管理

扩散规划器依赖 ONNX 模型进行推理。
为确保模型与 ROS 2 节点实现兼容，模型版本采用**主版本号**和**次版本号**：
模型版本由传给节点的目录名称定义，或在 `diffusion_planner.param.json` 配置文件中定义。

- **主版本号**
  当模型的**输入/输出或架构**发生变化时递增。

  > :warning: 主版本号不同的模型与当前 ROS 节点**不兼容**。

- **次版本号**
  当**仅更新权重文件**时递增。
  只要主版本号一致，节点就保持兼容，可直接使用新模型。

如需下载最新模型，请按照[下载制品](https://github.com/autowarefoundation/autoware/blob/main/ansible/roles/artifacts/README.md#download-artifacts)中的说明操作。

<a id="model-version-history"></a>

### 模型版本历史

| 版本 | 发布日期 | 说明 | ROS 节点兼容性 |
| ------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| **0.1** | 2025/07/05 | - 首个公开版本<br>- 基于 TIER IV 真实数据进行路线规划 | NG |
| **1.0** | 2025/09/12 | - 路线终点学习<br>- 输出转向信号（指示灯）<br>- 在高精地图中集成车道类型以提高精度<br>- 新增数据集：<br>&nbsp;&nbsp;- 合成数据：**4.0M 点**<br>&nbsp;&nbsp;- 真实数据：**1.5M 点** | NG |
| **2.0** | 2025/11/26 | - 增加左右边界可接受的车道类型（"crosswalk"、"pedestrian_lane" 和 "walkway"）。<br>- 新增 `Polygon` 和 `LineString` 作为可接受的输入类型。<br>- 将各历史记录的最大时长增加到 3 秒。<br>- 新增 turn_indicator 输入支持（仅提供接口，v2.0 权重未使用）。<br>- 将 `NUM_SEGMENTS_IN_LANE` 从 70 增加到 140。 | NG |
| **3.0** | 2026/01/09 | - 新增 `TURN_INDICATOR_OUTPUT_KEEP`，使模型关注状态变化的时机。<br>- 使用经过严格筛选的数据进行监督微调（SFT）。<br>- 将编码器层数从 3 增加到 6。 | OK |
| **3.1** | 2026/03/05 | - 简化 ONNX 模型，以加快 TRT 引擎构建并减少 GPU 内存占用。<br>- 使用与 v3.0 相同的权重（未重新训练）。 | OK |
| **4.0** | 2026/03/23 | - 为实时分块（RTC）新增 `delay` 输入：复用上一次预测的前 N 个时间步，保持轨迹连续。<br>- 为多边形（`intersection_area`）和线串（`stop_line`、`road_border`）新增独热类型编码。<br>- 将 `NUM_LINE_STRINGS` 从 10 增加到 60。<br>- 新增线串重采样（`line_string_max_step_m`）。<br>- 新增线串调试可视化。 | OK |

---

<a id="development-contribution"></a>

## 开发与贡献

- 遵循 [Autoware 编码规范](https://autowarefoundation.github.io/autoware-documentation/main/contributing/)。
- 欢迎通过 GitHub issue 和拉取请求提交贡献、缺陷报告及功能请求。

---

<a id="references"></a>

## 参考资料

- [Diffusion Planner（原始仓库）](https://github.com/ZhengYinan-AIR/Diffusion-Planner)
- [Diffusion planner（我们基于上述仓库创建的分支，用于训练模型）](https://github.com/tier4/Diffusion-Planner)
- ["Diffusion-Based Planning for Autonomous Driving with Flexible Guidance"](https://arxiv.org/abs/2501.15564)

---

## License

This package is released under the Apache 2.0 License.
