# detected_object_validation

<a id="purpose"></a>

## 用途

此功能包旨在消除 DetectedObjects 中明显的误检。

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

- [基于障碍物点云的验证器](obstacle-pointcloud-based-validator.md)
- [基于占据栅格的验证器](occupancy-grid-based-validator.md)
- [目标 lanelet 过滤器](object-lanelet-filter.md)
- [目标位置过滤器](object-position-filter.md)

<a id="node-parameters"></a>

### 节点参数

#### object_lanelet_filter

<a id="detected-objects"></a>

##### 检测目标

{{ json_to_markdown("perception/autoware_detected_object_validation/schema/detected_object_lanelet_filter.schema.json") }}

<a id="tracked-objects"></a>

##### 跟踪目标

{{ json_to_markdown("perception/autoware_detected_object_validation/schema/tracked_object_lanelet_filter.schema.json") }}

#### object_position_filter

{{ json_to_markdown("perception/autoware_detected_object_validation/schema/object_position_filter.schema.json") }}

#### obstacle_pointcloud_based_validator

{{ json_to_markdown("perception/autoware_detected_object_validation/schema/obstacle_pointcloud_based_validator.schema.json") }}

#### occupancy_grid_based_validator

{{ json_to_markdown("perception/autoware_detected_object_validation/schema/occupancy_grid_based_validator.schema.json") }}
