# concatenate_and_time_synchronize_node

<a id="purpose"></a>

## 用途

`concatenate_and_time_synchronize_node` 用于将多个点云同步并合并为一个统一的点云。通过整合多个激光雷达的数据，此节点显著扩大自动驾驶车辆的探测范围和覆盖范围，从而更准确地感知周围环境。同步处理确保点云在时间上对齐，减少时间戳不匹配引起的误差。

例如，一辆车在左侧、右侧和顶部各安装一个激光雷达。每个激光雷达采集各自视场内的数据，如下图所示：

| 左侧 | 顶部 | 右侧 |
| :-----------------------------------------------: | :---------------------------------------------: | :-------------------------------------------------: |
| ![左侧点云](./image/concatenate_left.png) | ![顶部点云](./image/concatenate_top.png) | ![右侧点云](./image/concatenate_right.png) |

经过 `concatenate_and_time_synchronize_node` 处理后，所有激光雷达的输出合并为一个综合点云，提供完整的环境视图：

![完整场景视图](./image/concatenate_all.png)

通过利用多个激光雷达相互补充的视场，生成的点云使自主系统能够更有效地检测障碍物、构建环境地图和导航。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

![点云拼接算法](./image/concatenate_algorithm.drawio.svg)

<a id="step-1-match-and-create-collector"></a>

### 步骤 1：匹配并创建收集器

点云到达时，节点会检查其时间戳，并减去偏移量得到参考时间戳。随后，节点检查是否存在具有相同参考时间戳的收集器。如果存在，就将点云添加到该收集器；否则，使用该参考时间戳创建新的收集器。

<a id="step-2-trigger-the-timer"></a>

### 步骤 2：启动定时器

收集器创建后，对应的定时器开始倒计时（时长由 `timeout_sec` 定义）。当 `input_topics` 中定义的所有点云均已收集齐，或定时器倒计时归零时，收集器开始拼接点云。

<a id="step-3-concatenate-the-point-clouds"></a>

### 步骤 3：拼接点云

拼接过程将多个点云合并为一个点云。拼接后点云的时间戳采用输入点云中最早的时间戳。如果将 `is_motion_compensated` 参数设为 `true`，节点会考虑输入点云的时间戳，并利用 `geometry_msgs::msg::TwistWithCovarianceStamped` 中的 `twist` 信息进行运动补偿，将点云对齐到所选的最早时间戳。

<a id="step-4-publish-the-point-cloud"></a>

### 步骤 4：发布点云

拼接完成后，发布拼接后的点云，并删除收集器以释放资源。

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `~/input/twist` | `geometry_msgs::msg::TwistWithCovarianceStamped` | 根据车辆运动，利用速度信息调整点云扫描，使不同时间戳的激光雷达数据同步后再拼接。 |
| `~/input/odom` | `nav_msgs::msg::Odometry` | 根据车辆运动，利用车辆里程计调整点云扫描，使不同时间戳的激光雷达数据同步后再拼接。 |

将 `input_twist_topic_type` 参数设为 `twist` 或 `odom`，订阅器将分别订阅 `~/input/twist` 或 `~/input/odom`。如果不希望使用速度信息或车辆里程计进行运动补偿，请将 `is_motion_compensated` 设为 `false`。

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `~/output/points` | `sensor_msgs::msg::Pointcloud2` | 拼接后的点云 |
| `~/output/info` | `autoware_sensing_msgs::msg::ConcatenatedPointCloudInfo` | 拼接点云的信息，包括源点云的范围及其状态 |

<a id="core-parameters"></a>

### 核心参数

{{ json_to_markdown("sensing/autoware_pointcloud_preprocessor/schema/concatenate_and_time_sync_node.schema.json") }}

<a id="concatenation-strategies"></a>

## 拼接策略

`concatenate_and_time_synchronize_node` 通过 `matching_strategy.type` 参数支持不同的拼接策略，以处理不同的激光雷达同步场景：

<a id="naive-strategy-matching_strategytype-naive"></a>

### 简单策略（`matching_strategy.type: "naive"`）

简单策略适用于激光雷达传感器**未同步**，或不要求精确时间戳对齐的场景。该策略具有以下特点：

- **直接拼接**：无需复杂的时间戳匹配，直接拼接点云
- **简单收集**：收集并合并超时窗口内到达的点云
- **无偏移补偿**：不使用 `lidar_timestamp_offsets` 或 `lidar_timestamp_noise_window` 参数
- **处理更快**：通过简化逻辑降低计算开销

**简单策略的适用场景：**

- 激光雷达传感器未进行硬件同步
- 传感器之间的时间戳差异对应用而言可以忽略
- 相比精确的时间对齐，更重视处理速度
- 简单拼接已能满足需求，无需复杂匹配逻辑

<a id="parameter-settings"></a>

#### 参数设置

将 `matching_strategy` 的 `type` 参数设为 `"naive"`，即可直接拼接点云。

<a id="advanced-strategy-matching_strategytype-advanced"></a>

### 高级策略（`matching_strategy.type: "advanced"`）

高级策略适用于激光雷达传感器**已同步**且精确时间戳对齐至关重要的场景。该策略具有以下特点：

- **精确时间戳匹配**：使用经过偏移补偿的参考时间戳
- **噪声容忍**：使用 `lidar_timestamp_noise_window` 处理时间戳抖动
- **偏移校正**：应用 `lidar_timestamp_offsets` 对齐不同激光雷达的时序
- **稳健收集**：处理时间变化，确保点云正确分组

**高级策略的适用场景：**

- 激光雷达传感器已进行硬件同步
- 精确的时间对齐对应用至关重要
- 已测量各激光雷达传感器之间的时间戳偏移
- 希望有效处理时间戳噪声和抖动

**高级策略所需参数：**

- `lidar_timestamp_offsets`：各激光雷达传感器时间偏移量组成的数组
- `lidar_timestamp_noise_window`：处理时间戳抖动的时间窗口
- `timeout_sec`：收集点云的最长等待时间

<a id="parameter-settings_1"></a>

#### 参数设置

将 `matching_strategy` 的 `type` 参数设为 `"advanced"`，并相应配置 `timeout_sec`、`lidar_timestamp_offsets` 和 `lidar_timestamp_noise_window` 参数。

##### timeout_sec

当网络出现问题或点云在前面的处理流水线中发生延迟时，某些点云可能延迟到达或丢失。`timeout_sec` 参数用于处理这种情况。定时器创建后，从 `timeout_sec` 开始倒计时。归零时，收集器不再等待延迟或丢失的点云，而是直接拼接已经收集到的点云。下图展示了 `timeout_sec` 在 `concatenate_and_time_sync_node` 中的工作方式，其中 `timeout_sec` 设为 `0.12`（120 ms）。

![点云拼接边界情况](./image/concatenate_edge_case.drawio.svg)

##### lidar_timestamp_offsets

不同车辆采用的激光雷达扫描设计不同，各激光雷达的时间戳也可能不同。用户需要了解各激光雷达之间的偏移量，并在 `lidar_timestamp_offsets` 中设置对应数值。

要监控各激光雷达的时间戳，请运行以下命令：

```bash
ros2 topic echo "pointcloud_topic" --field header
```

按照 Autoware 的默认设置，时间戳应以约 100 ms 的间隔稳定递增。输出应类似如下内容：

```bash
nanosec: 156260951
nanosec: 257009560
nanosec: 355444581
```

这种模式表示激光雷达时间戳的小数部分为 0.05。

如果有三个激光雷达（左、右、顶），其点云时间戳分别为 `0.01`、`0.05` 和 `0.09` 秒，则应将参数设为 [0.0, 0.04, 0.08]。这些值表示当前点云与时间戳最早点云之间的时间差。注意，`lidar_timestamp_offsets` 的顺序应与 `input_topics` 一致。

下图展示了 `lidar_timestamp_offsets` 在 `concatenate_and_time_sync_node` 中的工作方式。

![理想时间戳偏移](./image/ideal_timestamp_offset.drawio.svg)

##### lidar_timestamp_noise_window

此外，由于激光雷达的机械设计，每次扫描的时间戳可能存在抖动，如下图所示。例如，扫描频率设为 10 Hz（每 100 ms 扫描一次）时，相邻扫描的时间戳间隔可能并非恰好为 100 ms。`lidar_timestamp_noise_window` 参数用于处理这种噪声。

用户可以使用[此工具](https://github.com/tier4/timestamp_analyzer)将各次扫描之间的噪声可视化。

![时间戳抖动](./image/jitter.png)

上述示例中的噪声范围为 0 到 8 ms，因此应将 `lidar_timestamp_noise_window` 设为 `0.008`。

下图展示了 `lidar_timestamp_noise_window` 在 `concatenate_and_time_sync_node` 中的工作方式。如果绿色 `X` 位于红色三角形限定的范围内，则表示该点云与收集器的参考时间戳匹配。

![带噪声的时间戳偏移](./image/noise_timestamp_offset.drawio.svg)

<a id="meta-information-topic"></a>

## 元信息话题

拼接节点通过 `~/output/info` 话题发布拼接过程的详细元信息。有关 `ConcatenatedPointCloudInfo` 消息结构的详细信息，请参阅 [autoware_msgs 仓库文档](https://github.com/autowarefoundation/autoware_msgs/tree/main/autoware_sensing_msgs#concatenated-point-cloud-messages)。

<a id="handling-serialized-configuration"></a>

### 处理序列化配置

`matching_strategy_config` 字段包含匹配策略的序列化配置数据。
如果某种策略有自己的配置，就需要基于 [concatenation_info_manager.hpp](https://github.com/autowarefoundation/autoware_universe/blob/main/sensing/autoware_pointcloud_preprocessor/include/autoware/pointcloud_preprocessor/concatenate_data/concatenation_info_manager.hpp) 中定义的 `StrategyConfig` 类实现序列化和反序列化。

以下示例展示如何处理高级策略的序列化配置：

<a id="serialization-example"></a>

#### 序列化示例

```cpp
auto cfg = StrategyAdvancedConfig(reference_timestamp_min, reference_timestamp_max);
ConcatenationInfoManager::set_config(cfg.serialize(), concatenation_info_msg);
```

<a id="deserialization-example"></a>

#### 反序列化示例

```cpp
std::vector<uint8_t> raw_cfg = concat_info_msg->matching_strategy_config;
auto cfg = StrategyAdvancedConfig(raw_cfg);
```

<a id="launch"></a>

## 启动

```bash
# The launch file will read the parameters from the concatenate_and_time_sync_node.param.yaml
ros2 launch autoware_pointcloud_preprocessor concatenate_and_time_sync_node.launch.xml
```

<a id="test"></a>

## 测试

```bash
# build autoware_pointcloud_preprocessor
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release --packages-up-to autoware_pointcloud_preprocessor

# test autoware_pointcloud_preprocessor
colcon test --packages-select autoware_pointcloud_preprocessor --event-handlers console_cohesion+
```

<a id="debug-and-diagnostics"></a>

## 调试与诊断

要验证节点是否成功拼接了点云，可以查看 rqt，或使用以下命令检查 `/diagnostics` 话题：

```bash
ros2 topic echo /diagnostics
```

以下是点云拼接成功时的输出示例：

- 每个点云对应的值均为 `True`。
- `Pointcloud concatenation succeeded` 为 `True`。
- `level` 值为 `\0`。（diagnostic_msgs::msg::DiagnosticStatus::OK）

```bash
header:
  stamp:
    sec: 1722492015
    nanosec: 848508777
  frame_id: ''
status:
- level: "\0"
  name: 'concatenate_and_time_sync_node: concat_status'
  message: Concatenated pointcloud is published and include all topics
  hardware_id: concatenate_data_checker
  values:
  - key: Concatenated pointcloud timestamp
    value: '1718260240.159229994'
  - key: Minimum reference timestamp
    value: '1718260240.149230003'
  - key: Maximum reference timestamp
    value: '1718260240.169229984'
  - key: Timestamp: /sensing/lidar/left/pointcloud_before_sync
    value: '1718260240.159229994'
  - key: Concatenated: /sensing/lidar/left/pointcloud_before_sync
    value: 'True'
  - key: Timestamp: /sensing/lidar/right/pointcloud_before_sync
    value: '1718260240.194104910'
  - key: Concatenated: /sensing/lidar/right/pointcloud_before_sync
    value: 'True'
  - key: Timestamp: /sensing/lidar/top/pointcloud_before_sync
    value: '1718260240.234578133'
  - key: Concatenated: /sensing/lidar/top/pointcloud_before_sync
    value: 'True'
  - key: Pointcloud concatenation succeeded
    value: 'True'
```

以下是点云未能成功拼接时的示例。

- 某些点云对应的值可能为 `False`。
- `Pointcloud concatenation succeeded` 为 `False`。
- `level` 值为 `\x02`。（diagnostic_msgs::msg::DiagnosticStatus::ERROR）

```bash
header:
  stamp:
    sec: 1722492663
    nanosec: 344942959
  frame_id: ''
status:
- level: "\x02"
  name: 'concatenate_and_time_sync_node: concat_status'
  message: Concatenated pointcloud is published but miss some topics
  hardware_id: concatenate_data_checker
  values:
  - key: Concatenated pointcloud timestamp
    value: '1718260240.859827995'
  - key: Minimum reference timestamp
    value: '1718260240.849828005'
  - key: Maximum reference timestamp
    value: '1718260240.869827986'
  - key: Timestamp: /sensing/lidar/left/pointcloud_before_sync/timestamp
    value: '1718260240.859827995'
  - key: Concatenated: /sensing/lidar/left/pointcloud_before_sync
    value: 'True'
  - key: Timestamp: /sensing/lidar/right/pointcloud_before_sync/timestamp
    value: '1718260240.895193815'
  - key: Concatenated: /sensing/lidar/right/pointcloud_before_sync
    value: 'True'
  - key: Concatenated: /sensing/lidar/top/pointcloud_before_sync
    value: 'False'
  - key: Pointcloud concatenation succeeded
    value: 'False'
```

<a id="node-separation-options"></a>

## 节点拆分选项

也可以将 concatenate_and_time_sync_node 拆分为两个节点：一个负责 `time synchronization`（时间同步），另一个负责 `concatenate pointclouds`（点云拼接）（[参见此 PR](https://github.com/autowarefoundation/autoware_universe/pull/3312)）。

注意，`concatenate_pointclouds` 和 `time_synchronizer_nodelet` 使用的是拼接节点的[旧版设计](https://github.com/autowarefoundation/autoware_universe/blob/9bb228fe5b7fa4c6edb47e4713c73489a02366e1/sensing/autoware_pointcloud_preprocessor/docs/concatenate-data.md)。

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 如果将 `is_motion_compensated` 设为 `false`，`concatenate_and_time_sync_node` 将直接拼接点云而不进行运动补偿。根据参与拼接的激光雷达数量，这可以节省数毫秒。因此，如果点云之间的时间戳差异可以忽略，就可以将 `is_motion_compensated` 设为 `false`，节点也就无需输入速度信息或里程计数据。
- 如上所述，用户应清楚了解激光雷达点云时间戳的管理方式，以便正确设置参数。如果点云未同步，请将 `matching_strategy.type` 设为 `naive`。
