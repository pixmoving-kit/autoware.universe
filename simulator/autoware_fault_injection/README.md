# fault_injection

<a id="purpose"></a>

## 用途

此功能包将 PSim 中的模拟系统故障转换为诊断信息，并通知 Autoware。
组件图如下：

![fault_injection 功能包组件图](img/component.drawio.svg)

<a id="test"></a>

## 测试

```bash
source install/setup.bash
cd fault_injection
launch_test test/test_fault_injection_node.test.py
```

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                        | 类型                                           | 说明            |
| --------------------------- | ---------------------------------------------- | ---------------------- |
| `~/input/simulation_events` | `tier4_simulation_msgs::msg::SimulationEvents` | 仿真事件      |
| `~/input/diagnostics`       | `diagnostic_msgs::msg::DiagnosticArray`        | 各节点的诊断信息 |

<a id="output"></a>

### 输出

| 名称                   | 类型                                    | 说明                              |
| ---------------------- | --------------------------------------- | ---------------------------------------- |
| `~/output/diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 应用故障注入后的诊断信息 |

<a id="notes"></a>

### 注意事项

- 使用独立的输入/输出话题（默认重映射为 `/diagnostics` -> `~/input/diagnostics` 和 `/diagnostics/fault_injection` -> `~/output/diagnostics`），以避免混淆原始诊断与修改后的诊断。

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

## 前提假设／已知限制

待定。
