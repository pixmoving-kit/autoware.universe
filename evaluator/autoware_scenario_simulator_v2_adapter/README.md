<a id="scenario_simulator_v2-adapter"></a>

# scenario_simulator_v2 适配器

<a id="purpose"></a>

## 用途

此功能包提供一个节点，将 Autoware 的各种消息转换为供 scenario_simulator_v2 使用的 `tier4_simulation_msgs::msg::UserDefinedValue` 消息。
目前，此节点支持转换：

- 指标话题中的 `tier4_metric_msgs::msg::MetricArray`
- 从 `.webauto-ci.yml` 通过 `autoware.diagnostic_config` 传入的诊断话题

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

- 对于 `tier4_metric_msgs::msg::MetricArray`，
  节点订阅参数 `metric_topic_list` 中列出的全部话题。
  每次收到此类消息时，会将其转换为与 `Metric` 对象数量相同的 `UserDefinedValue` 消息。
  输出话题格式详见_输出_一节。

- 对于来自 `autoware.diagnostic_config` 的诊断话题，
  节点订阅 `/diagnostics`。
  每次收到此类消息时，会将其转换为与 `DiagnosticArray` 中 `DiagnosticStatus` 对象数量相同的 `UserDefinedValue` 消息。
  输入 `autoware.diagnostic_config` 的格式详见_输入_一节。
  输出话题格式详见_输出_一节。

<a id="metric-array-inputs-outputs"></a>

## 指标数组输入/输出

<a id="metric-array-inputs"></a>

### 指标数组输入

节点在 `metric_topic_list` 指定的话题上监听 `MetricArray` 消息。

<a id="metric-array-outputs"></a>

### 指标数组输出

节点输出由接收消息转换得到的 `UserDefinedValue` 消息。

输出话题名称根据对应的输入话题和指标名称生成。

- 例如，可以监听话题 `/planning/planning_evaluator/metrics`，并收到包含 2 个指标的 `MetricArray`：
  - `name: "metricA/x"` 的指标
  - `name: "metricA/y"` 的指标
- 用于发布 `UserDefinedValue` 的话题如下：
  - `/planning/planning_evaluator/metrics/metricA/x`
  - `/planning/planning_evaluator/metrics/metricA/y`

<a id="diagnostics-inputs-outputs"></a>

## 诊断输入/输出

诊断功能从 `DiagnosticStatus` 提取 `level` 字段，并将其作为单个值输出，以适配 `tier4_simulation_msgs::msg::UserDefinedValue` 消息。

<a id="diagnostics-inputs"></a>

### 诊断输入

- 节点监听 `/diagnostics`。
  从 `/diagnostics` 的 `DiagnosticArray` 中提取多个 `DiagnosticStatus` 对象。

- 关于 `autoware.diagnostic_config`：
  - 配置文件由 `.webauto-ci.yml` 指定。
  - 在 `diagnostic_groups` 下创建一个组名，并在该组下定义 `output_topic_name` 和 `aggregation_list`。
    - `output_topic_name`
      - 指定要发布到的话题名称。
      - 推荐使用 `/diagnostics/scenario_simulator_v2_adapter/***` 格式。
    - `aggregation_list`
      - 列出需要分组的诊断话题。
      - 可包含以相同格式声明的 `output_topic_name`（递归展开）。
  - 配置示例见 `autoware_scenario_simulator_v2_adapter/config/diagnostic_config.param.yaml`。

```yml
/**:
  ros__parameters:
    diagnostic_groups:
      overall_diagnostics:
        output_topic_name: /diagnostics/scenario_simulator_v2_adapter/overall_diagnostics
        aggregation_list:
          - /diagnostics/scenario_simulator_v2_adapter/topic_state_monitor_diagnostics

      topic_state_monitor_diagnostics:
        output_topic_name: /diagnostics/scenario_simulator_v2_adapter/topic_state_monitor_diagnostics
        aggregation_list:
          - /diagnostics/topic_state_monitor_mission_planning_route/planning_topic_status
          - /diagnostics/topic_state_monitor_scenario_planning_trajectory/planning_topic_status
```

<a id="diagnostics-outputs"></a>

### 诊断输出

节点输出由接收消息转换得到的 `UserDefinedValue` 消息。
诊断输出有两种方式：

1. 与诊断状态名称一一对应。
2. 根据 `autoware.diagnostic_config` 聚合诊断状态名称，进行分组输出。

- 单独发布时：
  输出话题名称由 `/diagnostics/` 前缀和各个 `DiagnosticStatus` 的 `status.name` 字段生成。
  状态名称中的 `": "` 会被替换为 `/`，以构成有效的话题名称。
  - 状态名称：`planning_validator: intersection_validation_collision_check`
  - 发布话题名称：`/diagnostics/planning_validator/intersection_validation_collision_check`

- 按组发布话题时：
  输出话题名称使用 `autoware.diagnostic_config` 中的 `output_topic_name`。
  每次收到与 `aggregation_list` 中任意话题匹配的 `DiagnosticStatus` 时，都会将其 level 值发布到 `output_topic_name` 话题。
  如果级别为 `ERROR`，则记录包含诊断话题名称和组名的警告日志。

<a id="parameters"></a>

## 参数

{{ json_to_markdown("evaluator/autoware_scenario_simulator_v2_adapter/schema/scenario_simulator_v2_adapter.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

假定 `MetricArray` 中 `Metric` 对象的数值类型为 `double`。

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分
