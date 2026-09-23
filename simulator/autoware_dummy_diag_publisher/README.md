# dummy_diag_publisher

<a id="purpose"></a>

## 用途

此功能包输出虚拟诊断数据，用于调试和开发。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="outputs"></a>

### 输出

| 名称           | 类型                                     | 说明         |
| -------------- | ---------------------------------------- | ------------------- |
| `/diagnostics` | `diagnostic_msgs::msgs::DiagnosticArray` | 诊断输出 |

<a id="parameters"></a>

## 参数

<a id="node-parameters"></a>

### 节点参数

参数 `DIAGNOSTIC_NAME` 必须是参数 YAML 文件中存在的名称。如果通过命令行指定 `status` 参数，`is_active` 参数会自动设为 `true`。

| 名称                        | 类型   | 默认值 | 说明                             | 可重新配置 |
| --------------------------- | ------ | ------------- | --------------------------------------- | -------------- |
| `update_rate`               | int    | `10`          | 定时器回调频率 [Hz]              | false          |
| `DIAGNOSTIC_NAME.is_active` | bool   | `true`        | 是否强制更新                     | true           |
| `DIAGNOSTIC_NAME.status`    | string | `"OK"`        | 由虚拟诊断发布器设置的诊断状态 | true           |

<a id="yaml-format-for-dummy_diag_publisher"></a>

### dummy_diag_publisher 的 YAML 格式

如果值为 `default`，则使用默认值。

| 键                                        | 类型   | 默认值 | 说明                             |
| ------------------------------------------ | ------ | ------------- | --------------------------------------- |
| `required_diags.DIAGNOSTIC_NAME.is_active` | bool   | `true`        | 是否强制更新                     |
| `required_diags.DIAGNOSTIC_NAME.status`    | string | `"OK"`        | 由虚拟诊断发布器设置的诊断状态 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

待定。

<a id="usage"></a>

## 使用方法

<a id="launch"></a>

### 启动

```sh
ros2 launch autoware_dummy_diag_publisher dummy_diag_publisher.launch.xml
```

<a id="reconfigure"></a>

### 重新配置

```sh
ros2 param set /dummy_diag_publisher velodyne_connection.status "Warn"
ros2 param set /dummy_diag_publisher velodyne_connection.is_active true
```
