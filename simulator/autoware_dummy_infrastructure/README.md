# autoware_dummy_infrastructure

这是用于基础设施通信的调试节点。

<a id="usage"></a>

## 使用方法

```sh
ros2 launch autoware_dummy_infrastructure dummy_infrastructure.launch.xml
ros2 run rqt_reconfigure rqt_reconfigure
```

<a id="inputs-outputs"></a>

## 输入／输出

<a id="inputs"></a>

### 输入

| 名称                    | 类型                                              | 说明            |
| ----------------------- | ------------------------------------------------- | ---------------------- |
| `~/input/command_array` | `tier4_v2x_msgs::msg::InfrastructureCommandArray` | 基础设施指令 |

<a id="outputs"></a>

### 输出

| 名称                   | 类型                                                 | 说明                 |
| ---------------------- | ---------------------------------------------------- | --------------------------- |
| `~/output/state_array` | `tier4_v2x_msgs::msg::VirtualTrafficLightStateArray` | 虚拟交通灯数组 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

| 名称                | 类型   | 默认值 | 说明                                       |
| ------------------- | ------ | ------------- | ------------------------------------------------- |
| `update_rate`       | double | `10.0`        | 定时器回调频率 [Hz]                        |
| `use_first_command` | bool   | `true`        | 是否考虑设备 ID                     |
| `use_command_state` | bool   | `false`       | 是否考虑指令状态                     |
| `instrument_id`     | string | ``            | 用作指令 ID                                |
| `approval`          | bool   | `false`       | 将 approval 字段设为 ROS 参数值                   |
| `is_finalized`      | bool   | `false`       | 若尚未完成最终确认，则在 stop_line 停车 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

待定。
