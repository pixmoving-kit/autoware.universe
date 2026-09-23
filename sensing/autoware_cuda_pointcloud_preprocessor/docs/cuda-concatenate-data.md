# cuda_concatenate_and_time_synchronize_node

此软件包是 [autoware_cuda_pointcloud_preprocessor](../../autoware_pointcloud_preprocessor/README.md) 中相应功能的 CUDA 加速版本。
由于此节点采用模板化实现，其整体设计、算法、输入和输出均相同。

唯一的变化在于点云话题：它们使用 `cuda_blackboard` 机制，而不是标准的 `sensor_msgs::msg::PointCloud2` 消息类型。
