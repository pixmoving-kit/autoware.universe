# autoware_camera_streampetr

<a id="purpose"></a>

## 用途

`autoware_camera_streampetr` 功能包用于仅基于图像的 3D 目标检测。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

此功能包实现了由 TensorRT 驱动的 StreamPETR [1] 推理节点。这是 autoware 中首个仅使用相机的 3D 目标检测节点。

此节点针对多相机系统进行了优化：各相机话题依次发布，而非同时发布。节点利用这一特点，
对图像进行预处理（缩放、裁剪、归一化）并妥善存储到 GPU，以尽量降低预处理引起的延迟。

```pgsql

Topic for image_i arrived                                     -------------------------
  |                                                                                   |
  |                                                                                   |
  |                                                                                   |
  v                                                                                   |
Is image distorted?                                                                   |
  |              \                                                                    |
  |               \                                                                   |
Yes               No                                                                  |Image Updates
  |                |                                                                  |done in parallel, if multitheading is on
  v                |                                                                  |otherwise done sequentially in FIFO order
Undistort          |                                                                  |
  |                |                                                                  |
  v                v                                                                  |
Load image into GPU memory                                                            |
  |                                                                                   |
  v                                                                                   |
Preprocess image (scale & crop ROI & normalize)                                       |
  |                                                                                   |
  v                                                                                   |
Store in GPU memory binding location for model input                                  |
  |                                                          -------------------------|
  v                                                                                   |
Is image the `anchor_image`?                                                          |
  |                \                                                                  |
  |                 \                                                                 |
No                  Yes                                                               |
  |                  |                                                                |
  v                  v                                                                | If multithreading is on
(Wait)     Are all images synced within `max_time_difference`?                        | image Updates are temporarily frozen
                      |                           \                                   | until this part completes.
                      |                            \                                  |
                    Yes                             No                                |
                      |                             |                                 |
                      v                             v                                 |
         Perform model forward pass            (Sync failed! Skip prediction)         |
                      |                                                               |
                      v                                                               |
         Postprocess (NMS + ROS2 format)                                              |
                      |                                                               |
                      v                                                               |
             Publish predictions                             -------------------------|

```

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称                          | 类型                           | 说明                                                                                                                |
| ----------------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `~/input/camera*/image`       | `sensor_msgs::msg::Image`      | 输入图像话题，通过 `image_transport` 订阅（`raw` 或 `compressed` 传输，由 `is_compressed_image` 选择）。 |
| `~/input/camera*/camera_info` | `sensor_msgs::msg::CameraInfo` | 输入相机信息话题，提供相机参数。                                                                           |

<a id="output"></a>

### 输出

| 名称                          | 类型                                                | 说明                                                                                                    | RTX 3090 延迟（ms） |
| ----------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------- |
| `~/output/objects`            | `autoware_perception_msgs::msg::DetectedObjects`    | 检测到的目标。                                                                                              | —                     |
| `latency/preprocess`          | `autoware_internal_debug_msgs::msg::Float64Stamped` | 每张图像的预处理时间（ms）。                                                                              | 3.25                  |
| `latency/total`               | `autoware_internal_debug_msgs::msg::Float64Stamped` | 总处理时间（ms）：预处理 + 推理 + 后处理。                                        | 26.04                 |
| `latency/inference`           | `autoware_internal_debug_msgs::msg::Float64Stamped` | 模型前向计算时间（ms），返回前执行流同步。                                                 | 22.13                 |
| `latency/inference/backbone`  | `autoware_internal_debug_msgs::msg::Float64Stamped` | 骨干网络推理时间（ms）。                                                                                  | 16.21                 |
| `latency/inference/ptshead`   | `autoware_internal_debug_msgs::msg::Float64Stamped` | 点检测头推理时间（ms）。                                                                               | 5.45                  |
| `latency/inference/pos_embed` | `autoware_internal_debug_msgs::msg::Float64Stamped` | 位置嵌入推理时间（ms）。                                                                        | 0.40                  |
| `latency/postprocess`         | `autoware_internal_debug_msgs::msg::Float64Stamped` | 包围框解码 + NMS + 将网络预测转换为 autoware 格式（ms）；与 `latency/inference` 不重叠。 | 0.40                  |
| `latency/cycle_time_ms`       | `autoware_internal_debug_msgs::msg::Float64Stamped` | 两次连续预测之间的时间（ms）。                                                                 | 110.65                |

仅在启用 `debug_mode` 时发布所有 `latency/*` 话题。

<a id="diagnostics"></a>

### 诊断

无论是否启用 `debug_mode`，均发布到 `/diagnostics`。两种状态均为同一个
定时器驱动更新器的任务（周期为 `diagnostics.validation_callback_interval_ms`，hardware_id 为
节点名称），因此相机或推理停止时仍会持续报告——这正是它们
要检测的情况——并且始终出现在同一条 `/diagnostics` 消息中。每种故障模式对应一个状态，
使诊断图能够将输入故障和处理时间故障路由至
不同路径。

| 名称                     | 说明                                           |
| ------------------------ | ----------------------------------------------------- |
| `camera_status`          | 每个相机的输入可用性、时效性和有效性 |
| `processing_time_status` | 每个周期的处理时间看门狗                    |

`camera_status` 为每个相机报告三个键值：解析后的输入 `cameraN/topic`（
每次部署中，模型输入索引与物理相机的对应关系都可能不同）、`cameraN/image_age_ms`
（节点时钟减去最新消息头时间戳；首帧前为 `n/a`）以及 `cameraN/state`，后者
将相机归纳为一种状态，并优先采用最具体的原因：

| `cameraN/state`       | 含义                                                     | 级别   |
| --------------------- | ----------------------------------------------------------- | ------- |
| `rejected`            | 最新帧被输入验证丢弃                | `ERROR` |
| `waiting_camera_info` | 尚无 `camera_info`（启动期间正常）               | `WARN`  |
| `waiting_image`       | 尚未收到首帧（启动期间正常）                 | `WARN`  |
| `stale`               | 曾发布过图像；最新帧已超过 `max_image_age_ms` | `ERROR` |
| `active`              | 健康                                                     | `OK`    |

另外还报告汇总键值：

| 键                                              | 含义                                                                        |
| ------------------------------------------------ | ------------------------------------------------------------------------------ |
| `num_cameras`                                    | 预期相机数量（`rois_number`）                                          |
| `num_waiting` / `num_stale` / `num_rejected`     | 各状态计数；汇总级别由这些计数决定                             |
| `oldest_image_camera_id` / `oldest_image_age_ms` | 最久没有收到新帧的相机                           |
| `max_inter_camera_time_diff_ms`                  | 订阅侧各相机之间的时间戳跨度（有相机缺失时为 `n/a`） |

时间戳和时长均格式化为定点字符串（从不使用科学计数法），与
点云拼接节点的诊断保持一致。

汇总级别取最差的相机状态；此外，若相机间时间戳跨度超过
`max_camera_time_diff`，则报告 `WARN`（同步检查正在跳过预测周期）。
`rejected` 和 `stale` 为 `ERROR` 而非 `WARN`，因为两者都要在
修复发布者后才能恢复：被拒绝的相机发布了非 `rgb8`/`bgr8` 编码，或缓冲区
并非紧密排列，这些帧永远不会到达模型。

当一个周期超过
`diagnostics.max_allowed_processing_time_ms` 时，`processing_time_status` 报告 `WARN`；若超预算状态持续
超过 `diagnostics.max_acceptable_consecutive_delay_ms`，则升级为 `ERROR`。在
首次推理完成前报告“waiting”，并携带所测周期各阶段的详细时间——
`preprocess_time_ms`、`inference_time_ms` 和 `postprocess_time_ms`，与
`latency/preprocess`、`latency/inference` 和 `latency/postprocess` 调试话题对应——从而
无需启用 `debug_mode` 即可定位超预算的 `processing_time_ms`。

`inference_time_ms`（模型前向计算）和 `postprocess_time_ms`（包围框解码及 NMS）
是互不重叠的阶段。但 `preprocess_time_ms` 是单张图像的耗时，而非
所有相机耗时之和，且总时间从锚定相机回调开始计算，因此
其他相机回调中的预处理不包含在内——各阶段之和并不等于
`processing_time_ms`；应将 `preprocess_time_ms` 与自身历史值比较。

最新发布的检测结果同时报告两种时钟下的时间，因为 `processing_time_ms` 仅
覆盖节点内部的工作：

| 键                          | 说明                                                                                               |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| `last_frame_timestamp`       | 用于计算检测结果的消息头时间戳（感知时刻）                                       |
| `last_published_timestamp`   | 检测结果离开节点时的节点时钟时刻                                                            |
| `output_latency_ms`          | 两者之差：端到端延迟，包含相机传输及节点之前的解码队列 |
| `time_since_last_publish_ms` | 距离 `last_published_timestamp` 的时长，即节点距上次生成目标的时间                     |

这些值仅用于观测——级别仍仅由处理时间阈值
决定。节点时钟与消息头时间戳比较时，两者必须具有相同时间基准，因此
rosbag 回放需要发布 `/clock`，节点必须使用 `use_sim_time` 运行，
`output_latency_ms` 和 `time_since_last_publish_ms` 才有意义。

<a id="parameters"></a>

## 参数

<a id="streampetr-node"></a>

### StreamPETR 节点

`autoware_camera_streampetr` 节点提供以下配置参数：

<a id="model-parameters"></a>

#### 模型参数

- `model_params.backbone_path`：骨干网络 ONNX 模型路径
- `model_params.head_path`：头部网络 ONNX 模型路径
- `model_params.position_embedding_path`：位置嵌入 ONNX 模型路径
- `model_params.fp16_mode`：启用 FP16 推理模式
- `model_params.use_temporal`：启用时序建模
- `model_params.input_image_height`：预处理输入图像高度
- `model_params.input_image_width`：预处理输入图像宽度
- `model_params.class_names`：检测类别名称列表
- `model_params.num_proposals`：目标候选数量
- `model_params.detection_range`：用于筛选目标的检测范围

<a id="post-processing-parameters"></a>

#### 后处理参数

- `post_process_params.iou_nms_search_distance_2d`：IoU NMS 的 2D 搜索距离
- `post_process_params.circle_nms_dist_threshold`：圆形 NMS 的距离阈值
- `post_process_params.iou_nms_threshold`：NMS 的 IoU 阈值
- `post_process_params.confidence_threshold`：检测置信度阈值
- `post_process_params.yaw_norm_thresholds`：偏航归一化阈值

<a id="ego-vehicle-mask"></a>

#### 自车遮罩

遮蔽自车区域以减少反射引起的误检。通过 **launch** 或 `camera_streampetr.param.yaml` 配置，而不是 `tensorrt_stream_petr.param.yaml`（后者仅用于模型/后处理）。

- `ego_mask.enabled`：启用遮罩（默认：`false`）
- `ego_mask.fill_value_rgb`：多边形内部 RGB 填充值，范围 0–255（默认：`[0, 0, 0]`）
- `ego_mask.roi_polygons_yaml`：每个模型 ROI 索引对应一个 YAML 路径；空字符串表示禁用该 ROI。

多边形文件示例：`config/camera9_polygons.yaml`、`config/camera10_polygons.yaml`。

**X2 五相机布局**（`tensorrt_stream_petr.x2.launch.xml`）：ROI 2 → camera10（左侧条带），ROI 4 → camera9（右侧条带）。自车遮罩参数在该 launch 文件中设置。

<a id="node-parameters"></a>

#### 节点参数

- `max_camera_time_diff`：相机之间允许的最大时间差（秒）
- `rois_number`：相机 ROI/相机数量（默认：6）
- `is_compressed_image`：是否使用 `compressed` 图像传输订阅，而非 `raw`
- `is_distorted_image`：输入图像是否带有畸变
- `multithreading`：是否使用多线程处理图像回调
- `anchor_camera_id`：用于同步的锚定相机 ID（默认：0）
- `debug_mode`：启用调试模式以测量时间
- `build_only`：构建 TensorRT 引擎后退出，不运行推理
- `diagnostics.max_allowed_processing_time_ms`：每周期处理时间预算；超过时报告 `WARN`（默认：200.0）
- `diagnostics.max_acceptable_consecutive_delay_ms`：处理时间超预算持续多久后，将 `WARN` 升级为 `ERROR`（默认：1000.0）
- `diagnostics.max_image_age_ms`：若相机最新帧年龄超过此值，则以 `ERROR` 级别报告停滞（默认：300.0）
- `diagnostics.validation_callback_interval_ms`：诊断回调间隔（默认：100.0）

<a id="the-build_only-option"></a>

### `build_only` 选项

`autoware_camera_streampetr` 节点提供 `build_only` 选项，可从指定 ONNX 文件构建 TensorRT 引擎文件，之后程序退出。

```bash
ros2 launch autoware_camera_streampetr tensorrt_stream_petr.launch.xml build_only:=true
```

<a id="the-log_level-option"></a>

### `log_level` 选项

`autoware_camera_streampetr` 默认日志级别为 `info`。开发人员可使用 `log_level` 参数降低严重性级别以便调试：

```bash
ros2 launch autoware_camera_streampetr tensorrt_stream_petr.launch.xml log_level:=debug
```

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

此节点仅使用相机，不需要点云输入。它假定：

- 所有相机的同步时间差均在指定的 `max_camera_time_diff` 内
- 相机标定信息可用且准确
- 锚定相机（由 `anchor_camera_id` 指定）触发推理周期
- 可通过 tf 获取相机坐标系与 base_link 之间的变换信息
- 可通过 tf 获取 map 与 base_link 之间的变换信息，用于自车运动补偿
- **输入图像已经去畸变**

<a id="trained-models"></a>

## 已训练模型

模型包托管于 [Hugging Face](https://huggingface.co/AutowareFoundation/camera_streampetr/tree/v1.0)。[ansible artifacts role](https://github.com/autowarefoundation/autoware/tree/main/ansible/roles/artifacts) 会将其下载到 `~/autoware_data/ml_models/camera_streampetr`。

所需模型文件：

- `simplify_extract_img_feat.onnx`（骨干网络）
- `simplify_pts_head_memory.onnx`（头部网络）
- `simplify_position_embedding.onnx`（位置嵌入）
- `ml_package_camera_streampetr.param.yaml`（机器学习包配置）

如需训练并部署自己的模型，可在 [AWML](https://github.com/tier4/AWML/tree/main/projects/StreamPETR) 中找到相关源代码。

<a id="changelog"></a>

## 变更日志

<a id="referencesexternal-links"></a>

## 参考资料／外部链接

[1] Wang, Shihao and Liu, Yingfei and Wang, Tiancai and Li, Ying and Zhang, Xiangyu. "Exploring Object-Centric Temporal Modeling for Efficient Multi-View 3D Object Detection." 2023 <!-- cspell:disable-line -->

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分

- 启用 2D 目标检测。由于训练期间使用 2D 目标检测作为辅助损失，只需少量更新，同一节点即可轻松支持 2D 目标检测。
- 为骨干网络实现 int8 量化，进一步降低推理延迟
- 每张图像到达时立即执行图像骨干网络，进一步降低延迟。
- 为预测结果添加速度。
