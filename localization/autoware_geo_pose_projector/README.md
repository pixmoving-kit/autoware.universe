# autoware_geo_pose_projector

<a id="overview"></a>

## 概述

此节点订阅带地理参考的位姿话题，并发布地图坐标系下的位姿。

<a id="subscribed-topics"></a>

## 订阅的话题

| 名称 | 类型 | 说明 |
| ------------------------- | ---------------------------------------------------- | ------------------- |
| `input_geo_pose` | `geographic_msgs::msg::GeoPoseWithCovarianceStamped` | 带地理参考的位姿 |
| `/map/map_projector_info` | `autoware_map_msgs::msg::MapProjectedObjectInfo` | 地图投影信息 |

<a id="published-topics"></a>

## 发布的话题

| 名称 | 类型 | 说明 |
| ------------- | ----------------------------------------------- | ------------------------------------- |
| `output_pose` | `geometry_msgs::msg::PoseWithCovarianceStamped` | 地图坐标系下的位姿 |
| `/tf` | `tf2_msgs::msg::TFMessage` | 从父坐标系到子坐标系的 TF |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("localization/autoware_geo_pose_projector/schema/geo_pose_projector.schema.json") }}

<a id="limitations"></a>

## 限制

根据所用投影类型的不同，协方差转换可能不正确。输入话题的协方差以（纬度、经度、高程）表示，为对角矩阵。
目前假定 X 轴指向东、Y 轴指向北。当这一假设不成立时，转换可能不正确，尤其是在纬度和经度的协方差不同时。
