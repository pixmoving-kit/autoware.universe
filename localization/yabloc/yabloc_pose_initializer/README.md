# yabloc_pose_initializer

此功能包包含与初始位姿估计相关的节点。

- [camera_pose_initializer](#camera_pose_initializer)

此功能包运行时需要预训练的语义分割模型，通常会在安装期间由 [Ansible artifacts 角色](https://github.com/autowarefoundation/autoware/tree/main/ansible/roles/artifacts)下载。
也可以手动下载。即使未下载模型，初始化仍能完成，但精度可能受到影响。

模型托管在 [Hugging Face](https://huggingface.co/AutowareFoundation/yabloc_pose_initializer/tree/v1.0)。如需使用 [hf CLI](https://huggingface.co/docs/huggingface_hub/guides/cli) 手动下载：

```bash
hf download AutowareFoundation/yabloc_pose_initializer --revision v1.0 --local-dir ~/autoware_data/ml_models/yabloc_pose_initializer
```

## Note

This package makes use of external code. The trained files are provided by apollo. The trained files are automatically downloaded during env preparation.

Original model URL

<https://github.com/openvinotoolkit/open_model_zoo/tree/master/models/intel/road-segmentation-adas-0001>

> Open Model Zoo is licensed under Apache License Version 2.0.

Converted model URL

<https://github.com/PINTO0309/PINTO_model_zoo/tree/main/136_road-segmentation-adas-0001>

> model conversion scripts are released under the MIT license

## Special thanks

- [openvinotoolkit/open_model_zoo](https://github.com/openvinotoolkit/open_model_zoo)
- [PINTO0309](https://github.com/PINTO0309)

## camera_pose_initializer

<a id="purpose"></a>

### 用途

- 此节点根据 ADAPI 请求，利用相机估计初始位置。

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------- | --------------------------------------- | ------------------------ |
| `input/camera_info` | `sensor_msgs::msg::CameraInfo` | 去畸变后的相机信息 |
| `input/image_raw` | `sensor_msgs::msg::Image` | 去畸变后的相机图像 |
| `input/vector_map` | `autoware_map_msgs::msg::LaneletMapBin` | 矢量地图 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ----------------------- | -------------------------------------- | ----------------------- |
| `debug/init_candidates` | `visualization_msgs::msg::MarkerArray` | 初始位姿候选 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_pose_initializer/schema/camera_pose_initializer.schema.json") }}

<a id="services"></a>

### 服务

| 名称 | 类型 | 说明 |
| ------------------ | --------------------------------------------------------------------- | ------------------------------- |
| `yabloc_align_srv` | `autoware_internal_localization_msgs::srv::PoseWithCovarianceStamped` | 初始位姿估计请求 |
