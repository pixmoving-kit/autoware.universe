<a id="autoware-traffic-light-rviz-plugin"></a>

# Autoware 交通信号灯 RViz 插件

此功能包提供 RViz2 插件，用于可视化 Autoware 的交通信号灯识别结果。

![可视化示例](visualization.png)

<a id="property-description"></a>

## 属性说明

<a id="topic-settings"></a>

### 话题设置

- **Lanelet Map Topic**：接收 Lanelet2 地图数据的话题
  - 类型：`autoware_map_msgs/msg/LaneletMapBin`
- **Traffic Light Topic**：接收交通信号灯识别结果的话题
  - 类型：`autoware_perception_msgs/msg/TrafficLightGroupArray`

<a id="display-settings"></a>

### 显示设置

- **Timeout**：清除交通信号灯状态显示前的等待时间，单位为秒
- **Show Text**：切换是否显示交通信号灯状态文本
- **Show Bulb**：切换是否显示交通信号灯图形
- **Text Prefix**：交通信号灯状态文本的前缀
- **Font Size**：交通信号灯状态文本的字号
- **Text Color**：交通信号灯状态文本的颜色

<a id="text-position-adjustment"></a>

### 文本位置调整

- **Text X Offset**：文本显示沿 X 轴的偏移量
- **Text Y Offset**：文本显示沿 Y 轴的偏移量
- **Text Z Offset**：文本显示沿 Z 轴的偏移量
