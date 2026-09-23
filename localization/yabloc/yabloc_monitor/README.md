# yabloc_monitor

YabLoc monitor 是监控 YabLoc 定位系统状态的节点。它封装了状态监控功能，并将监控结果作为诊断信息发布。

<a id="feature"></a>

## 功能

<a id="availability"></a>

### 可用性

此节点通过监控 YabLoc 最终输出的位姿来验证其可用性。

<a id="others"></a>

### 其他

待补充。

<a id="interfaces"></a>

## 接口

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------------- | --------------------------- | ------------------------------- |
| `~/input/yabloc_pose` | `geometry_msgs/PoseStamped` | YabLoc 的最终输出位姿 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------- | --------------------------------- | ------------------- |
| `/diagnostics` | `diagnostic_msgs/DiagnosticArray` | 诊断输出 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_monitor/schema/yabloc_monitor.schema.json") }}
