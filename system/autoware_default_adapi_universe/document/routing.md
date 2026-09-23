<a id="routing-api"></a>

# 路线规划 API

<a id="overview"></a>

## 概述

将路线设置方式统一为服务。此 API 支持两种途经点格式：位姿和 Lanelet 路段。来自 rviz 的目标点及检查点话题仅由适配节点订阅，并转换为 API 调用。该调用转发给任务规划节点，以集中管理路线状态。对于需要路线的其他节点，任务规划节点通过 `/planning/mission_planning/route` 发布。 AD API 规范请参阅 [Autoware 文档](https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture-v1/interfaces/ad-api/features/routing/)。

![routing-architecture](images/routing.drawio.svg)
