# autoware_detected_object_feature_remover

<a id="purpose"></a>

## 用途

`autoware_detected_object_feature_remover` 功能包用于将话题类型从 `DetectedObjectWithFeatureArray` 转换为 `DetectedObjects`。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称      | 类型                                                         | 说明                         |
| --------- | ------------------------------------------------------------ | ----------------------------------- |
| `~/input` | `tier4_perception_msgs::msg::DetectedObjectWithFeatureArray` | 带特征字段的检测目标 |

<a id="output"></a>

### 输出

| 名称       | 类型                                             | 说明      |
| ---------- | ------------------------------------------------ | ---------------- |
| `~/output` | `autoware_perception_msgs::msg::DetectedObjects` | 检测目标 |

<a id="parameters"></a>

## 参数

无

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制
