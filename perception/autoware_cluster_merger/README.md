# autoware cluster merger

<a id="purpose"></a>

## 用途

autoware_cluster_merger 功能包用于将点云聚类合并为带特征类型的检测目标。

<a id="inner-working-algorithms"></a>

## 内部原理 / 算法

合并话题中的聚类由输入话题中的聚类直接拼接而成。

<a id="input-output"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称             | 类型                                                     | 说明         |
| ---------------- | -------------------------------------------------------- | ------------------- |
| `input/cluster0` | `tier4_perception_msgs::msg::DetectedObjectsWithFeature` | 点云聚类 |
| `input/cluster1` | `tier4_perception_msgs::msg::DetectedObjectsWithFeature` | 点云聚类 |

<a id="output"></a>

### 输出

| 名称              | 类型                                                     | 说明     |
| ----------------- | -------------------------------------------------------- | --------------- |
| `output/clusters` | `tier4_perception_msgs::msg::DetectedObjectsWithFeature` | 合并后的聚类 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("perception/autoware_cluster_merger/schema/cluster_merger.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

<!-- Write assumptions and limitations of your implementation.

Example:
  This algorithm assumes obstacles are not moving, so if they rapidly move after the vehicle started to avoid them, it might collide with them.
  Also, this algorithm doesn't care about blind spots. In general, since too close obstacles aren't visible due to the sensing performance limit, please take enough margin to obstacles.
-->

## (Optional) Error detection and handling

<!-- Write how to detect errors and how to recover from them.

Example:
  This package can handle up to 20 obstacles. If more obstacles found, this node will give up and raise diagnostic errors.
-->

## (Optional) Performance characterization

<!-- Write performance information like complexity. If it wouldn't be the bottleneck, not necessary.

Example:
  ### Complexity

  This algorithm is O(N).

  ### Processing time

  ...
-->

## (Optional) References/External links

<!-- Write links you referred to when you implemented.

Example:
  [1] {link_to_a_thesis}
  [2] {link_to_an_issue}
-->

## (Optional) Future extensions / Unimplemented parts
