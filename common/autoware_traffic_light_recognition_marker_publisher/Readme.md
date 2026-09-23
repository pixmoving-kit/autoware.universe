<a id="traffic-light-recognition-marker-publisher"></a>

# 交通信号灯识别标记发布器

<a id="purpose"></a>

## 用途

此节点发布标记数组，用于在 Rviz 中显示交通信号灯识别结果。

![sample_img](./images/traffic_light_recognition_visualization_sample.png)

<a id="inner-workings-algorithms"></a>

## 内部机制与算法

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称                                                    | 类型                                                    | 说明                                       |
| ------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------- |
| `/map/vector_map`                                       | `autoware_map_msgs::msg::LaneletMapBin`                 | 用于获取交通信号灯信息的矢量地图 |
| `/perception/traffic_light_recognition/traffic_signals` | `autoware_perception_msgs::msg::TrafficLightGroupArray` | 交通信号灯识别结果          |

<a id="output"></a>

### 输出

| 名称                                                           | 类型                                   | 说明                                                                    |
| -------------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------ |
| `/perception/traffic_light_recognition/traffic_signals_marker` | `visualization_msgs::msg::MarkerArray` | 发布用于显示交通信号灯识别结果的标记数组 |

<a id="parameters"></a>

## 参数

无。

<a id="node-parameters"></a>

### 节点参数

无。

<a id="core-parameters"></a>

### 核心参数

无。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

待补充。
