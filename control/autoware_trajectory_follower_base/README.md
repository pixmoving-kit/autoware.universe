<a id="trajectory-follower"></a>

# 轨迹跟踪器

本文是 `trajectory_follower` 功能包的设计文档。

<a id="purpose-use-cases"></a>

## 目的与使用场景

<!-- Required -->
<!-- Things to consider:
    - Why did we implement this feature? -->

本功能包提供纵向和横向控制器接口，供 `autoware_trajectory_follower_node` 功能包中的节点使用。
可以通过继承纵向和横向基础接口实现具体控制器。

<a id="design"></a>

## 设计

系统提供横向和纵向基础接口类，各算法通过继承相应类完成实现。
接口类包含以下基本函数。

- `isReady()`：检查控制器是否已准备好进行计算。
- `run()`：计算控制命令，并返回给[轨迹跟踪节点](../autoware_trajectory_follower_node/README.md)。继承接口的算法必须实现此函数。
- `sync()`：输入另一控制器的运行结果。
  - 转向角收敛状态。
    - 支持在转向收敛前保持停车。
  - 速度收敛状态（目前未使用）。

这些函数如何在节点中工作，请参阅[轨迹跟踪节点的设计](../autoware_trajectory_follower_node/README.md#Design)。

<a id="separated-lateral-steering-and-longitudinal-velocity-controls"></a>

## 分离的横向（转向）与纵向（速度）控制

本纵向控制器假设横向和纵向控制的职责按以下方式分离。

- 横向控制在速度跟踪完全准确的假设下，计算目标转向，使车辆沿轨迹行驶。
- 纵向控制在轨迹跟踪完全准确的假设下，计算目标速度或加速度，使车速符合轨迹速度。

理想情况下，将横向和纵向控制作为一个联合问题处理可以获得较高性能。但将速度控制器作为独立功能提供，也有以下两个原因。

<a id="complex-requirements-for-longitudinal-motion"></a>

### 纵向运动的复杂需求

人们期望的车辆纵向行为很难用单一逻辑表达。例如，为实现类似人类驾驶的动作，即将停车前的期望行为会随自车位于停止线前方还是后方、当前速度高于还是低于目标速度而变化。

此外，部分车辆很难在极低速时测量自身速度。在这些情况下，能够在不影响横向控制的前提下改进纵向控制功能的结构十分重要。

纵向控制有许多独特的特性和需求。将其与横向控制分开设计，可以降低模块耦合、提高可维护性。

<a id="nonlinear-coupling-of-lateral-and-longitudinal-motion"></a>

### 横向与纵向运动的非线性耦合

横纵向联合控制问题非常复杂，需要非线性优化来获得较高性能。由于难以保证非线性优化收敛，开发中也需要简单的控制逻辑。

此外，如果车辆不高速行驶，同时进行横纵向控制的收益也较小。

<a id="related-issues"></a>

## 相关问题

<!-- Required -->
