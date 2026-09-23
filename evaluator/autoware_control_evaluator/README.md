<a id="control-evaluator"></a>

# 控制评估器

<a id="purpose"></a>

## 用途

本功能包提供用于生成指标、评估控制质量的节点。

它发布控制模块输出的指标信息，以及自车当前的运动学状态和位置。

<a id="evaluated-metrics"></a>

## 评估指标

控制评估器使用 `include/autoware/control_evaluator/metrics/metric.hpp` 中定义的指标，计算自车相对于设定点的偏航角和横向距离偏差。也可以定制 control_evaluator，为其他控制模块提供指标和评估。目前，control_evaluator 根据 autonomous_emergency_braking 节点的输出提供简单指标，但此功能可扩展为评估其他控制模块的性能。

<a id="kinematics-output"></a>

## 运动学输出

控制评估器持续发布自车运动学和位置信息，包括当前所在车道 ID，以及纵向 `s` 和横向 `t` 弧线坐标。指标消息中还包含当前自车速度、加速度和加加速度。

其他节点可以利用这些信息，通过 rosbag 建立自动评估：将自车位置、运动学状态与被评估控制模块的输出交叉比较，即可判断在 rosbag 回放的关键时刻，控制模块是否做出了令人满意的响应。
