<a id="localization-api"></a>

# 定位 API

<a id="overview"></a>

## 概述

将定位初始化方式统一为服务。来自 rviz 的 `/initialpose` 话题现在仅由适配节点订阅，并转换为 API 调用。该调用转发给位姿初始化节点，以集中管理位姿初始化状态。对于需要初始位姿的其他节点，位姿初始化节点通过 `/initialpose3d` 发布。 AD API 规范请参阅 [Autoware 文档](https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture-v1/interfaces/ad-api/features/localization/)。

![localization-architecture](images/localization.drawio.svg)
