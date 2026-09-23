<a id="planning-components"></a>

# 规划组件

<a id="getting-started"></a>

## 入门

Autoware Universe 规划模块是整个开源自动驾驶软件栈中的先进组件。这些模块在自动驾驶车辆导航中发挥关键作用，负责路线规划、动态障碍物避让以及对不同交通状况的实时适应。

- 有关规划组件的总体概念，请参阅[规划组件设计文档](https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture-v1/components/planning/)
- 有关规划组件与其他组件的交互方式，请参阅[规划组件接口文档](https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture-v1/interfaces/components/planning/)
- [节点图](https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture-v1/node-diagram/)展示了 Autoware Universe 中所有模块（包括规划模块）的交互、输入和输出。

<a id="planning-module"></a>

## 规划模块

规划组件中的**模块**是共同组成软件规划系统的各个组件。这些模块涵盖自动驾驶车辆规划所需的多种功能。Autoware 的规划功能采用模块化设计，用户可以通过修改配置来自定义启用的功能。这种设计可以灵活适应自动驾驶车辆运行中的不同场景和需求。

<a id="how-to-enable-or-disable-planning-module"></a>

### 如何启用或禁用规划模块

启用和禁用模块需要管理关键配置文件和启动文件中的设置。

<a id="key-files-for-configuration"></a>

### 关键配置文件

`default_preset.yaml` 是主要配置文件，可用于启用或禁用规划模块。此外，用户还可以从多种运动规划器中选择使用的类型。例如：

- `launch_avoidance_module`：设为 `true` 可启用避让模块，设为 `false` 可禁用该模块。

!!! note

    点击[此处](https://github.com/autowarefoundation/autoware_launch/blob/main/autoware_launch/config/planning/preset/default_preset.yaml)查看 `default_preset.yaml`。

[启动文件](https://github.com/autowarefoundation/autoware_universe/tree/main/launch/tier4_planning_launch/launch/scenario_planning/lane_driving)引用 `default_preset.yaml` 中定义的设置，在行为路径规划器节点运行时应用配置。例如，以下文件中的参数 `avoidance.enable_module`

```xml
<param name="avoidance.enable_module" value="$(var launch_avoidance_module)"/>
```

对应于 `default_preset.yaml` 中的 launch_avoidance_module。

<a id="parameter-configuration"></a>

### 参数配置

有多个参数可供配置，用户可以在[此处](https://github.com/autowarefoundation/autoware_launch/tree/main/autoware_launch/config/planning)修改它们。并非所有参数都能通过 `rqt_reconfigure` 调整。为确保修改生效，请修改参数后重新启动 Autoware。此外，规划选项卡下的相应文档提供了各参数的详细信息。

<a id="integrating-a-custom-module-into-autoware-a-step-by-step-guide"></a>

### 将自定义模块集成到 Autoware：分步指南

本指南介绍将自定义模块集成到 Autoware 的步骤：

- 将模块添加到 `default_preset.yaml` 文件。例如：

```yaml
- arg:
  name: launch_intersection_module
  default: "true"
```

- 将模块加入[启动器](https://github.com/autowarefoundation/autoware_universe/tree/main/launch/tier4_planning_launch/launch/scenario_planning)。例如，在 [behavior_planning.launch.xml](https://github.com/autowarefoundation/autoware_universe/blob/main/launch/tier4_planning_launch/launch/scenario_planning/lane_driving/behavior_planning/behavior_planning.launch.xml) 中：

```xml
<arg name="launch_intersection_module" default="true"/>

<let
  name="behavior_velocity_planner_launch_modules"
  value="$(eval &quot;'$(var behavior_velocity_planner_launch_modules)' + 'behavior_velocity_planner::IntersectionModulePlugin, '&quot;)"
  if="$(var launch_intersection_module)"
/>
```

- 如适用，将参数文件夹放入相应的现有参数文件夹。例如，[intersection_module 的参数](https://github.com/autowarefoundation/autoware_launch/blob/main/autoware_launch/config/planning/scenario_planning/lane_driving/behavior_planning/behavior_velocity_planner/intersection.param.yaml)位于 [behavior_velocity_planner](https://github.com/autowarefoundation/autoware_launch/tree/main/autoware_launch/config/planning/scenario_planning/lane_driving/behavior_planning/behavior_velocity_planner) 中。
- 在 [tier4_planning_component.launch.xml](https://github.com/autowarefoundation/autoware_launch/blob/main/autoware_launch/launch/components/tier4_planning_component.launch.xml) 中插入参数路径。例如，使用 `behavior_velocity_planner_intersection_module_param_path`。

```xml
<arg name="behavior_velocity_planner_intersection_module_param_path" value="$(var behavior_velocity_config_path)/intersection.param.yaml"/>
```

- 在相应启动器中定义参数路径变量。例如，在 [behavior_planning.launch.xml](https://github.com/autowarefoundation/autoware_universe/blob/04aa54bf5fb0c88e70198ca74b9ac343cc3457bf/launch/tier4_planning_launch/launch/scenario_planning/lane_driving/behavior_planning/behavior_planning.launch.xml#L191) 中

```xml
<param from="$(var behavior_velocity_planner_intersection_module_param_path)"/>
```

!!! note

    具体涉及的文件和步骤可能因待添加模块而异。本指南提供总体概述和起点，请根据模块的具体情况调整这些说明。

<a id="join-our-community-driven-effort"></a>

## 加入社区协作

Autoware 的发展依靠社区协作。无论贡献大小，都非常宝贵。无论是报告缺陷、提出改进建议、分享新想法，还是其他贡献，我们都欢迎。

<a id="how-to-contribute"></a>

### 如何贡献？

准备好参与贡献了吗？请先阅读[贡献指南](https://autowarefoundation.github.io/autoware-documentation/main/contributing/)，其中包含入门所需的所有信息，包括提交缺陷报告、提出功能增强建议以及贡献代码的说明。

<a id="join-our-planning-control-working-group-meetings"></a>

### 参加规划与控制工作组会议

规划与控制工作组是社区的重要组成部分。我们每两周举行一次会议，讨论当前进展、即将面临的挑战，并交流新想法。这些会议是直接参与讨论和决策的良好机会。

会议详情：

- **频率：**每两周一次
- **日期：**星期四
- **时间：** UTC 08:00 AM（JST 05:00 PM）
- **议程：**讨论当前进展，规划后续开发。您可以在[此处](https://github.com/orgs/autowarefoundation/discussions?discussions_q=is%3Aopen+label%3Ameeting%3Aplanning-control-wg+)查看并评论以往会议纪要。

欢迎参加会议！有关参与方式，请访问以下链接：[如何参与工作组](https://github.com/autowarefoundation/autoware-projects/wiki/Autoware-Planning-Control-Working-Group#how-to-participate-in-the-working-group)。

<a id="citations"></a>

### 引用

我们会不定期发表有关 Autoware 规划组件的论文。欢迎阅读这些论文，为您的工作获取参考。如果这些论文对您有帮助，且您在项目中采用了我们的方法或算法，请引用相关论文。这有助于让更多人了解我们的工作，并支持我们继续为该领域作出贡献。

如果您使用规划组件中[运动速度平滑器](https://autowarefoundation.github.io/autoware_core/main/planning/autoware_velocity_smoother/)模块的加加速度约束速度规划算法，请引用相关论文。

<!-- cspell:ignore Shimizu, Horibe, Watanabe, Kato -->

Y. Shimizu, T. Horibe, F. Watanabe and S. Kato, "[Jerk Constrained Velocity Planning for an Autonomous Vehicle: Linear Programming Approach](https://arxiv.org/abs/2202.10029)," 2022 International Conference on Robotics and Automation (ICRA)

```tex
@inproceedings{shimizu2022,
  author={Shimizu, Yutaka and Horibe, Takamasa and Watanabe, Fumiya and Kato, Shinpei},
  booktitle={2022 International Conference on Robotics and Automation (ICRA)},
  title={Jerk Constrained Velocity Planning for an Autonomous Vehicle: Linear Programming Approach},
  year={2022},
  pages={5814-5820},
  doi={10.1109/ICRA46639.2022.9812155}}
```
