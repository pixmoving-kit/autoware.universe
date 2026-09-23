<a id="external-velocity-limit-selector"></a>

# 外部速度限制选择器

<a id="purpose"></a>

## 用途

`external_velocity_limit_selector_node` 是维护外部速度限制一致性的节点。此模块订阅：

1. **API** 发送的速度限制命令，
2. **Autoware 内部模块**发送的速度限制命令。

VelocityLimit.msg 不仅包含**最大速度**，还包含减速时的**加速度/加加速度约束**信息。`external_velocity_limit_selector_node` 综合最低速度限制和最高加加速度约束，计算**最严格的速度限制**，满足 API 与 Autoware 内部模块发送的所有减速点和最大速度要求。

![选择器算法](./image/external_velocity_limit_selector.png)

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

开发中

<!-- Write how this package works. Flowcharts and figures are great. Add sub-sections as you like.

Example:
  ### Flowcharts

  ...(PlantUML or something)

  ### State Transitions

  ...(PlantUML or something)

  ### How to filter target obstacles

  ...

  ### How to optimize trajectory

  ...
-->

## Inputs

| Name                                                | Type                                                            | Description                                   |
| --------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------- |
| `~input/velocity_limit_from_api`                    | autoware_internal_planning_msgs::msg::VelocityLimit             | velocity limit from api                       |
| `~input/velocity_limit_from_internal`               | autoware_internal_planning_msgs::msg::VelocityLimit             | velocity limit from autoware internal modules |
| `~input/velocity_limit_clear_command_from_internal` | autoware_internal_planning_msgs::msg::VelocityLimitClearCommand | velocity limit clear command                  |

## Outputs

| Name                   | Type                                                | Description                                       |
| ---------------------- | --------------------------------------------------- | ------------------------------------------------- |
| `~output/max_velocity` | autoware_internal_planning_msgs::msg::VelocityLimit | current information of the hardest velocity limit |

## Parameters

| Parameter         | Type   | Description                                |
| ----------------- | ------ | ------------------------------------------ |
| `max_velocity`    | double | default max velocity [m/s]                 |
| `normal.min_acc`  | double | minimum acceleration [m/ss]                |
| `normal.max_acc`  | double | maximum acceleration [m/ss]                |
| `normal.min_jerk` | double | minimum jerk [m/sss]                       |
| `normal.max_jerk` | double | maximum jerk [m/sss]                       |
| `limit.min_acc`   | double | minimum acceleration to be observed [m/ss] |
| `limit.max_acc`   | double | maximum acceleration to be observed [m/ss] |
| `limit.min_jerk`  | double | minimum jerk to be observed [m/sss]        |
| `limit.max_jerk`  | double | maximum jerk to be observed [m/sss]        |

## Assumptions / Known limits

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

<!-- Write future extensions of this package.

Example:
  Currently, this package can't handle the chattering obstacles well. We plan to add some probabilistic filters in the perception layer to improve it.
  Also, there are some parameters that should be global(e.g. vehicle size, max steering, etc.). These will be refactored and defined as global parameters so that we can share the same parameters between different nodes.
-->
