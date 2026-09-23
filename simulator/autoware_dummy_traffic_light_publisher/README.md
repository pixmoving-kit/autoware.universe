# autoware_dummy_traffic_light_publisher

<a id="purpose"></a>

## 用途

为无法使用交通灯识别模块的仿真环境发布虚拟交通信号。

<a id="modes"></a>

## 模式

| 模式         | 说明                                                                                    |
| ------------ | ---------------------------------------------------------------------------------------------- |
| `standalone` | 对矢量地图中的所有交通灯循环执行绿灯 → 黄灯 → 红灯。          |
| `empty`      | 发布空的 `TrafficLightGroupArray`（不包含交通灯组）。                         |
| `fixed`      | 对矢量地图中的所有交通灯发布同一种固定颜色（`fixed_color`）。 |

如需按交通灯 ID 分配不同信号，请使用下述透传功能；`fixed` 模式特意为所有交通灯应用同一种颜色。

<a id="pass-through"></a>

## 透传

当输入话题（`~/input/traffic_signals`）收到消息时，节点将其原样转发，不自行生成信号。如果在 `passthrough_timeout` 秒内未收到新输入，节点会回退到配置的模式。

<a id="interface"></a>

## 接口

<a id="subscriptions"></a>

### 订阅

| 话题                     | 类型                                                  | 说明                                                   |
| ------------------------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| `~/input/vector_map`      | `autoware_map_msgs/msg/LaneletMapBin`                 | 用于提取交通灯监管元素 ID 的 Lanelet2 地图。 |
| `~/input/traffic_signals` | `autoware_perception_msgs/msg/TrafficLightGroupArray` | 用于透传的外部交通信号。              |

<a id="publications"></a>

### 发布

| 话题                      | 类型                                                  | 说明                                 |
| -------------------------- | ----------------------------------------------------- | ------------------------------------------- |
| `~/output/traffic_signals` | `autoware_perception_msgs/msg/TrafficLightGroupArray` | 生成或转发的交通信号。 |

<a id="parameters"></a>

## 参数

| 参数             | 类型   | 默认值   | 说明                                                           |
| --------------------- | ------ | --------- | --------------------------------------------------------------------- |
| `mode`                | string | `"empty"` | 工作模式：`"standalone"`、`"empty"` 或 `"fixed"`。               |
| `publish_rate`        | double | `10.0`    | 发布频率 [Hz]。                                            |
| `green_duration`      | double | `30.0`    | 绿灯阶段持续时间 [s]（`standalone` 模式）。                  |
| `yellow_duration`     | double | `3.0`     | 黄灯阶段持续时间 [s]（`standalone` 模式）。                 |
| `red_duration`        | double | `30.0`    | 红灯阶段持续时间 [s]（`standalone` 模式）。                    |
| `passthrough_timeout` | double | `1.0`     | 最后一次输入后，回退到配置模式前等待的时间 [s]。 |
| `fixed_color`         | string | `"red"`   | `fixed` 模式发布的颜色：`"green"`、`"yellow"` 或 `"red"`。    |

<a id="usage"></a>

## 使用方法

<a id="launch"></a>

### 启动

```bash
ros2 launch autoware_dummy_traffic_light_publisher dummy_traffic_light_publisher.launch.xml
```

覆盖模式设置：

```bash
ros2 launch autoware_dummy_traffic_light_publisher dummy_traffic_light_publisher.launch.xml mode:=standalone
```

为所有交通灯发布固定颜色：

```bash
ros2 launch autoware_dummy_traffic_light_publisher dummy_traffic_light_publisher.launch.xml mode:=fixed fixed_color:=green
```

<a id="design-and-extension-tactics"></a>

## 设计与扩展方法

此功能包将职责划分为三层：

| 层             | 类                            | 职责                                                                                       |
| ----------------- | -------------------------------- | ------------------------------------------------------------------------------------------ |
| ROS 输入/输出           | `DummyTrafficLightPublisherNode` | 订阅（`take()`）、定时器、发布器、矢量地图解析。                            |
| 消息组装  | `DummyTrafficLight`              | 透传判断、交通灯 ID 管理、构建 `TrafficLightGroupArray`。 |
| 信号生成 | `TrafficLightCycle`              | 根据已用时间计算阶段（绿灯/黄灯/红灯），并输出 `TrafficLightElement`。   |

扩展时，仅修改承担相应职责的层：

- **添加箭头形状、闪烁模式或新信号类型**——修改 `TrafficLightCycle`，由它负责构建 `TrafficLightElement`。节点与 `DummyTrafficLight` 不受影响。
- **按交叉口或 ID 控制信号**——修改 `DummyTrafficLight`，由它将 ID 映射到元素。`TrafficLightCycle` 与节点不受影响。
- **添加新输入源或输出话题**——修改 `DummyTrafficLightPublisherNode`。逻辑层不受影响。

<a id="run-node-directly"></a>

### 直接运行节点

参数在代码中没有默认值，因此必须提供参数文件（例如功能包内的 `config/dummy_traffic_light_publisher.param.yaml`）：

```bash
ros2 run autoware_dummy_traffic_light_publisher autoware_dummy_traffic_light_publisher_node --ros-args \
  --params-file $(ros2 pkg prefix --share autoware_dummy_traffic_light_publisher)/config/dummy_traffic_light_publisher.param.yaml \
  -p mode:=standalone \
  -r ~/input/vector_map:=/map/vector_map \
  -r ~/input/traffic_signals:=/simulator/input/traffic_signals \
  -r ~/output/traffic_signals:=/perception/traffic_light_recognition/traffic_signals
```
