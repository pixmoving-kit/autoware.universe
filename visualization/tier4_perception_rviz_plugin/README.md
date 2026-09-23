# tier4_perception_rviz_plugin

<a id="purpose"></a>

## 用途

这是用于可视化 tier4 感知模块结果的 RViz 插件。此功能包基于 Autoware.Auto 开发的 RViz 插件实现。

原始设计理念请参阅 Autoware.Auto 设计文档。[[1]](https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/blob/master/src/tools/visualization/autoware_rviz_plugins)

<!-- Write the purpose of this package and briefly describe the features.

Example:
  {package_name} is a package for planning trajectories that can avoid obstacles.
  This feature consists of two steps: obstacle filtering and optimizing trajectory.
-->

## Input Types / Visualization Results

### DetectedObjectsWithFeature

#### Input Types

| Name | Type                                                     | Description            |
| ---- | -------------------------------------------------------- | ---------------------- |
|      | `tier4_perception_msgs::msg::DetectedObjectsWithFeature` | detection result array |

#### Visualization Result

![detected-object-with_feature-visualization-description](./images/detected-object-with-feature-visualization-description.jpg)

## References/External links

[1] <https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/tree/master/src/tools/visualization/autoware_rviz_plugins>

## Future extensions / Unimplemented parts
