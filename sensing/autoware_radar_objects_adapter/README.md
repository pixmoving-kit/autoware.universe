# autoware_radar_objects_adapter

<a id="purpose"></a>

## 用途

此功能包将 `autoware_sensing_msgs::msg::RadarObjects` 转换为 `autoware_perception_msgs::msg::DetectedObjects`，以简单方式将雷达接入感知流水线。

## RadarObjectsAdapter

该节点将传感器定义的雷达目标转换为便于感知模块使用的格式，不进行任何过滤。

<a id="parameter-classification_remap"></a>

### 参数：classification_remap

此参数可将分类标签从 `autoware_sensing_msgs::msg::RadarClassification` 重映射到 `autoware_perception_msgs::msg::ObjectClassification`。它应以一维字符串列表形式提供，每两个字符串构成一组，分别表示输入标签及其对应的输出标签。

例如，当前默认配置将雷达分类中的 `MOTORCYCLE` 和 `BICYCLE` 重映射为感知分类中的 `CAR`，其他标签保持不变。

**注意**：如果多个雷达标签映射到同一个感知标签，该标签可能对应多个概率值。
这不违反 Autoware 中的任何逻辑，但建议关注这一情况。

<a id="inputs-outputs"></a>

### 输入／输出

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------ | ---------------------------------------- | ------------------------------------------ |
| ~/input/objects | autoware_sensing_msgs::msg::RadarObjects | 按传感器定义输入的雷达目标。 |
| ~/input/radar_info | autoware_sensing_msgs::msg::RadarInfo | 输入雷达信息。 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ---------------- | ---------------------------------------------- | ---------------------------------------------- |
| ~/output/objects | autoware_perception_msgs::msg::DetectedObjects | 以感知格式输出的雷达目标。 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("sensing/autoware_radar_objects_adapter/schema/radar_objects_adapter.schema.json") }}
