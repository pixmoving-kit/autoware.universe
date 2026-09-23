# autoware_cuda_pointcloud_preprocessor

<a id="purpose"></a>

## 用途

`autoware_pointcloud_preprocessor` 中实现的点云预处理已在 Autoware 中经过充分测试。然而，现代激光雷达产生的点数很多，现有实现引入的延迟难以随数据规模良好扩展。

为缓解这一问题，此功能包利用通用 GPU（GPGPU）重新实现了 `autoware_pointcloud_preprocessor` 中的大部分处理流水线。具体而言，它使用 CUDA 为已有实现提供加速版本，同时保持与普通 ROS 节点和话题的兼容性。 <!-- cSpell: ignore GPGPUs -->

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

各滤波器的算法详情可通过以下链接查看。

| 滤波器名称 | 说明 | 详情 |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| cuda_pointcloud_preprocessor | 实现 `autoware_pointcloud_preprocessor` CPU 版本中的裁剪、畸变校正和基于扫描环的离群点过滤。 | [链接](docs/cuda-pointcloud-preprocessor.md) |
| cuda_concatenate_and_time_sync_node | 按照 `autoware_pointcloud_preprocessor` 的 CPU 实现，进行点云拼接与同步。 | [链接](docs/cuda-concatenate-data.md) |
| cuda_voxel_grid_downsample_filter | 实现 `autoware_pointcloud_preprocessor` CPU 版本中的体素降采样过滤。 | [链接](docs/cuda-voxel-grid-downsample.md) |
| cuda_polar_voxel_outlier_filter | 实现 `autoware_pointcloud_preprocessor` CPU 版本中的极坐标体素离群点过滤。 | [链接](docs/cuda-polar-voxel-outlier-filter.md) |

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分

此功能包将为 `autoware_pointcloud_preprocessor` 中的子采样滤波器提供类似的对应实现。
