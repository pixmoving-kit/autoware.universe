# tier4_control_mode_rviz_plugin

<a id="purpose"></a>

## 用途

此插件显示 Autoware 当前的控制模式状态。
背景颜色会随模式变化，便于直观识别状态。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------------------ | ----------------------------------------------- | ------------------------------------------- |
| `/vehicle/status/control_mode` | `autoware_vehicle_msgs::msg::ControlModeReport` | 表示当前控制模式的话题 |

<a id="control-mode-types"></a>

## 控制模式类型

| 模式 | 值 | 颜色 | 说明 |
| ------------------------ | ----- | --------- | -------------------------------- |
| NO_COMMAND | 0 | 深灰色 | 无命令状态 |
| AUTONOMOUS | 1 | 绿色 | 自动驾驶模式 |
| AUTONOMOUS_STEER_ONLY | 2 | 深灰色 | 仅自动控制转向 |
| AUTONOMOUS_VELOCITY_ONLY | 3 | 深灰色 | 仅自动控制速度 |
| MANUAL | 4 | 红色 | 人工驾驶模式 |
| DISENGAGED | 5 | 橙色 | 控制已脱离状态 |
| NOT_READY | 6 | 深灰色 | 系统尚未就绪 |

<a id="how-to-use"></a>

## 使用方法

1. 启动 RViz
2. 从菜单中选择 `Panels` → `Add New Panel`
3. 选择 `rviz_plugins/ControlModeDisplay`
4. 面板将显示当前控制模式

<a id="rviz-configuration-example"></a>

## RViz 配置示例

```yaml
Panels:
  - Class: rviz_plugins/ControlModeDisplay
    Name: ControlModeDisplay
```
