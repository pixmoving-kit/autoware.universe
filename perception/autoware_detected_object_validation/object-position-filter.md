# object_position_filter

<a id="purpose"></a>

## 用途

`object_position_filter` 节点根据 x、y 值过滤检测目标。
仅发布位于 x、y 边界内的目标。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称           | 类型                                             | 说明            |
| -------------- | ------------------------------------------------ | ---------------------- |
| `input/object` | `autoware_perception_msgs::msg::DetectedObjects` | 输入检测目标 |

<a id="output"></a>

### 输出

| 名称            | 类型                                             | 说明               |
| --------------- | ------------------------------------------------ | ------------------------- |
| `output/object` | `autoware_perception_msgs::msg::DetectedObjects` | 过滤后的检测目标 |

<a id="parameters"></a>

## 参数

<a id="core-parameters"></a>

### 核心参数

| 名称                             | 类型  | 默认值 | 说明                                                     |
| -------------------------------- | ----- | ------------- | --------------------------------------------------------------- |
| `filter_target_label.UNKNOWN`    | bool  | false         | 若为 true，过滤未知目标。 |
| `filter_target_label.CAR`        | bool  | false         | 若为 true，过滤汽车目标。 |
| `filter_target_label.TRUCK`      | bool  | false         | 若为 true，过滤卡车目标。 |
| `filter_target_label.BUS`        | bool  | false         | 若为 true，过滤巴士目标。 |
| `filter_target_label.TRAILER`    | bool  | false         | 若为 true，过滤拖车目标。 |
| `filter_target_label.MOTORCYCLE` | bool  | false         | 若为 true，过滤摩托车目标。 |
| `filter_target_label.BICYCLE`    | bool  | false         | 若为 true，过滤自行车目标。 |
| `filter_target_label.PEDESTRIAN` | bool  | false         | 若为 true，过滤行人目标。 |
| `upper_bound_x`                  | float | 100.00        | 过滤边界。仅在 filter_by_xy_position 为 true 时使用 |
| `lower_bound_x`                  | float | 0.00          | 过滤边界。仅在 filter_by_xy_position 为 true 时使用 |
| `upper_bound_y`                  | float | 50.00         | 过滤边界。仅在 filter_by_xy_position 为 true 时使用 |
| `lower_bound_y`                  | float | -50.00        | 过滤边界。仅在 filter_by_xy_position 为 true 时使用 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

根据目标中心位置进行过滤。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
