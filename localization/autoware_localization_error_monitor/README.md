# autoware_localization_error_monitor

<a id="purpose"></a>

## 用途

<p align="center">
<img src="./media/diagnostics.png" width="400">
</p>

autoware_localization_error_monitor 通过监控定位结果的不确定性来诊断定位误差。
此功能包监控以下两个值：

- 置信椭圆的长半轴长度
- 置信椭圆沿车体坐标系横向的尺寸

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------ | ------------------------- | ------------------- |
| `input/odom` | `nav_msgs::msg::Odometry` | 定位结果 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ---------------------- | --------------------------------------- | ------------------- |
| `debug/ellipse_marker` | `visualization_msgs::msg::Marker` | 椭圆标记 |
| `diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 诊断输出 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("localization/autoware_localization_error_monitor/schema/localization_error_monitor.schema.json") }}
