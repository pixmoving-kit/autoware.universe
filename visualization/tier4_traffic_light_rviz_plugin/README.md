# tier4_traffic_light_rviz_plugin

<a id="purpose"></a>

## 用途

此插件面板用于发布虚拟交通信号灯信号。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ------------------------------------------------------- | ------------------------------------------------------- | ----------------------------- |
| `/perception/traffic_light_recognition/traffic_signals` | `autoware_perception_msgs::msg::TrafficLightGroupArray` | 发布交通信号灯信号 |

<a id="howtouse"></a>

## 使用方法

<div align="center">
  <img src="images/select_panels.png" width=50%>
</div>
<div align="center">
  <img src="images/select_traffic_light_publish_panel.png" width=50%>
</div>
<div align="center">
  <img src="images/select_traffic_light_id.png" width=50%>
</div>

1. 启动 RViz，并选择 panels/Add new panel。
2. 选择 TrafficLightPublishPanel，然后点击 OK。
3. 设置 `Traffic Light ID` 和 `Traffic Light Status`，然后点击 `SET` 按钮。
4. 按下 `PUBLISH` 按钮后，发布交通信号灯信号。

<div align="center">
  <img src="images/traffic_light_publish_panel.gif">
</div>
