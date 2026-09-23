# autoware_carla_interface

<a id="ros-2-autoware-universe-bridge-for-carla-simulator"></a>

## CARLA 仿真器的 ROS 2 / Autoware Universe 桥接器

感谢 <https://github.com/gezp> 为 CARLA 通信提供 ROS 2 Humble 支持。
此 ROS 功能包实现 Autoware 与 CARLA 之间的通信，用于自动驾驶仿真。

<a id="supported-environment"></a>

## 支持的环境

| ubuntu |  ros   | carla  | autoware |
| :----: | :----: | :----: | :------: |
| 22.04  | humble | 0.9.15 |   Main   |

<a id="setup"></a>

## 配置

<a id="install"></a>

### 安装

<a id="prerequisites"></a>

#### 前置条件

1. **安装 CARLA 0.9.15**：请按照 [CARLA 安装指南](https://carla.readthedocs.io/en/latest/start_quickstart/)操作。

2. **安装 CARLA Python 包**：安装 [CARLA 0.9.15 ROS 2 Humble 通信包](https://github.com/gezp/carla_ros/releases/tag/carla-0.9.15-ubuntu-22.04)。

   - 方式 A：使用 pip 安装 wheel 包。
   - 方式 B：将 egg 文件添加到 `PYTHONPATH`。

3. **下载 CARLA Lanelet2 地图**：从 [CARLA Autoware 资源](https://bitbucket.org/carla-simulator/autoware-contents/src/master/maps/)获取 y 轴反转版本的地图。

<a id="map-setup"></a>

#### 地图配置

1. 将地图（y 轴反转版本）下载到任意位置。
2. 在 `$HOME/autoware_data/maps` 中创建地图文件夹结构：
   - 重命名 `point_cloud/Town01.pcd` → `$HOME/autoware_data/maps/Town01/pointcloud_map.pcd`
   - 重命名 `vector_maps/lanelet2/Town01.osm` → `$HOME/autoware_data/maps/Town01/lanelet2_map.osm`
3. 创建 `$HOME/autoware_data/maps/Town01/map_projector_info.yaml`，内容如下：

   ```yaml
   projector_type: Local
   ```

<a id="build"></a>

### 构建

```bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

<a id="run"></a>

### 运行

1. 运行 CARLA、切换地图，并按需生成目标。
   <!--- cspell:ignore prefernvidia -->

   ```bash
   cd CARLA
   ./CarlaUE4.sh -prefernvidia -quality-level=Low -RenderOffScreen
   ```

2. 运行 Autoware 与 CARLA。

   ```bash
   ros2 launch autoware_launch e2e_simulator.launch.xml \
       map_path:=$HOME/autoware_data/maps/Town01 \
       vehicle_model:=sample_vehicle \
       sensor_model:=carla_sensor_kit \
       simulator_type:=carla
   ```

   使用 VAD 进行端到端规划时：

   ```bash
   ros2 launch autoware_launch e2e_simulator.launch.xml \
       map_path:=$HOME/autoware_data/maps/Town01 \
       vehicle_model:=sample_vehicle \
       sensor_model:=carla_sensor_kit \
       simulator_type:=carla \
       use_e2e_planning:=true
   ```

3. 设置初始位姿（通过 GNSS 初始化）。
4. 设置目标位置。
5. 等待规划完成。
6. 启用自动驾驶。

<a id="viewing-multi-camera-view-in-rviz"></a>

### 在 RViz 中查看多摄像头画面

`carla_sensor_kit` 包含 6 个摄像头，覆盖 360 度视野（前、左前、右前、后、左后、右后）。多摄像头合并节点自动将所有摄像头画面合成为一个 2x3 网格视图。

在 RViz 中查看合成画面：

1. 在左侧 **Displays** 面板中点击 **"Add"** 按钮。
2. 选择 **"By topic"** 选项卡。
3. 找到 `/sensing/camera/all_cameras/image_raw`。
4. 选择 **"Image"** 显示类型。
5. 点击 **OK**。

![RViz 中的多摄像头视图](docs/images/rviz_multi_camera_view.png)

合成视图显示全部 6 个摄像头，并标注 FL（左前）、F（前）、FR（右前）、BL（左后）、B（后）、BR（右后）。

**注意：** 如果不需要多摄像头合并功能（以节省 CPU 资源），可将 `launch/autoware_carla_interface.launch.xml` 中的以下行注释掉：

```xml
<!-- Multi-camera combiner for RViz visualization -->
<!-- <node pkg="autoware_carla_interface" exec="multi_camera_combiner" output="screen"/> -->
```

<a id="following-the-ego-vehicle-with-the-carla-spectator-camera"></a>

### 使用 CARLA 观察者摄像头跟随自车

`spectator_follow` 脚本将 CARLA 观察者（自由）摄像头锁定到自车，使仿真器中的视角自动跟随车辆，无需手动平移。它连接正在运行的 CARLA 服务器，通过 `role_name` 查找自车 actor，并以固定频率更新观察者变换。

在 CARLA 与 Autoware 桥接器运行期间，于单独的终端中运行：

```bash
ros2 run autoware_carla_interface spectator_follow
```

常用选项（均为可选）：

| 参数         | 默认值       | 说明                                                 |
| ------------ | ------------- | ----------------------------------------------------------- |
| `--host`     | `localhost`   | CARLA 服务器主机                                           |
| `--port`     | `2000`        | CARLA 服务器 RPC 端口                                       |
| `--role`     | `ego_vehicle` | 要跟随的自车 actor 的 `role_name` 属性            |
| `--distance` | `8.0`         | 位于自车后方的距离，单位为米（设为 `0` 可获得俯视视角） |
| `--height`   | `4.0`         | 位于自车上方的高度，单位为米                                |
| `--pitch`    | `-15.0`       | 摄像头俯仰角，单位为度（负值表示向下看）               |
| `--rate`     | `30.0`        | 更新频率，单位为 Hz                                           |

如需位于自车正上方的俯视视角：

```bash
ros2 run autoware_carla_interface spectator_follow --distance 0 --height 30 --pitch -90
```

按 `Ctrl+C` 停止脚本。如果自车 actor 尚未生成，脚本会持续重试；如果自车被移除后重新生成，脚本会重新获取它。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

`InitializeInterface` 类是配置 CARLA 世界与自车的核心，通过 `autoware_carla_interface.launch.xml` 获取配置参数。

主仿真循环在 `carla_ros2_interface` 类中运行，按 `fixed_delta_seconds` 的时间步长推进 CARLA 仿真时间，并以 `self.sensor_frequencies` 定义的频率接收数据并发布为 ROS 2 消息。

来自 Autoware 的自车控制指令经过 `autoware_raw_vehicle_cmd_converter` 处理，校准为适用于 CARLA 的指令。校准后的指令通过 `CarlaDataProvider` 直接传入 CARLA 控制接口。

<a id="configurable-parameters-for-world-loading"></a>

### 世界加载的可配置参数

所有关键参数均可在 `autoware_carla_interface.launch.xml` 中配置。

| 名称                              | 类型   | 默认值                                                                     | 说明 |
| --------------------------------- | ------ | --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host`                            | string | "localhost"                                                                       | CARLA 服务器主机名 |
| `port`                            | int    | "2000"                                                                            | CARLA 服务器端口号 |
| `timeout`                         | int    | 20                                                                                | CARLA 客户端超时时间 |
| `ego_vehicle_role_name`           | string | "ego_vehicle"                                                                     | 自车角色名称 |
| `vehicle_type`                    | string | "vehicle.toyota.prius"                                                            | 待生成车辆的蓝图 ID。车辆蓝图 ID 可在 [CARLA 蓝图 ID](https://carla.readthedocs.io/en/latest/catalogue_vehicles/)中查找。 |
| `spawn_point`                     | string | None                                                                              | 自车生成坐标（None 表示随机）。格式为 [x, y, z, roll, pitch, yaw]。 |
| `sync_mode`                       | bool   | True                                                                              | 设置 CARLA 同步模式的布尔标志 |
| `fixed_delta_seconds`             | double | 0.05                                                                              | 仿真时间步长（与客户端 FPS 有关） |
| `use_traffic_manager`             | bool   | False                                                                             | 设置 CARLA 交通管理器的布尔标志 |
| `max_real_delta_seconds`          | double | 0.05                                                                              | 用于将仿真速度限制在 `fixed_delta_seconds` 以下的参数 |
| `tick_follower`                   | bool   | False                                                                             | 为 True 时，桥接器不主动推进 CARLA 世界，而是跟随另一客户端推进的帧。参见[多客户端联合仿真](#multi-client-co-simulation)。 |
| `carla_map`                       | string | ""                                                                                | 显式指定 CARLA 关卡名称。非空时会覆盖从 `map_path` 推导出的名称，适用于名称与 Autoware 地图目录不同的 CARLA 0.10 关卡。留空则保持当前行为。 |
| `no_rendering_mode`               | bool   | False                                                                             | 通过世界设置禁用 CARLA 场景渲染，以进行无界面或更快速的仿真。加载世界时无条件应用，因此即使服务器以无界面方式启动，默认值 `False` 也会（重新）启用渲染；设为 `True` 可保持关闭。 |
| `force_load_world`                | bool   | False                                                                             | 始终使用 `client.load_world()` 重新加载世界，而不使用 `load_world_if_different()`。默认 False 保持当前调用行为（含兼容不同版本的回退方式）。 |
| `map_origin_x`                    | double | 0.0                                                                               | 从 CARLA 世界原点到 Autoware 地图坐标系原点的 X 偏移，适用于采用自身局部原点制作的关卡。默认 0.0 表示恒等变换（无变化）。 |
| `map_origin_y`                    | double | 0.0                                                                               | 从 CARLA 世界原点到 Autoware 地图坐标系原点的 Y 偏移。默认 0.0 表示恒等变换（无变化）。 |
| `spawn_point_ground_snap`         | bool   | False                                                                             | 通过 `ground_projection` 将自车生成点与 RViz 初始位姿吸附到 CARLA 地图几何表面（参见[地面吸附](#ground-snapping)）。默认 False 保持生成点与固定 z 偏移不变。 |
| `spawn_point_ground_offset_z`     | double | 0.5                                                                               | 对生成点进行地面吸附时，在投影地面上方增加的 Z 偏移（仅在 `spawn_point_ground_snap` 为 True 时使用）。 |
| `initial_pose_ground_offset_z`    | double | 1.0                                                                               | 对 RViz 初始位姿进行地面吸附时，在投影地面上方增加的 Z 偏移（仅在 `spawn_point_ground_snap` 为 True 时使用）。 |
| `sensor_kit_name`                 | string | "carla_sensor_kit_description"                                                    | 用于传感器配置的传感器套件包名称。应为包含 config/sensor_kit_calibration.yaml 的 \*\_description 包。 |
| `use_light_weight_sensor_mapping` | bool   | False                                                                             | 为 True 时，使用 `sensor_mapping_light_weight.yaml` 替代默认的 `sensor_mapping.yaml`，以减轻仿真器负载。详见[传感器映射（CARLA 专用）](#2-sensor-mapping-carla-specific)。 |
| `sensor_mapping_file`             | string | "$(find-pkg-share autoware_carla_interface)/config/sensor_mapping.yaml"           | 传感器映射 YAML 配置文件的路径。当 `use_light_weight_sensor_mapping` 为 True 时，默认使用 `config/sensor_mapping_light_weight.yaml`。 |
| `publish_ground_truth_objects`    | bool   | False                                                                             | 为 True 时，将除自车外的每一辆 CARLA 车辆作为真值检测结果发布到 `/perception/object_recognition/detection/objects`，使跟踪与预测基于仿真器真值运行，而非传感器检测结果。不包含行人。 |
| `config_file`                     | string | "$(find-pkg-share autoware_carla_interface)/raw_vehicle_cmd_converter.param.yaml" | `autoware_raw_vehicle_cmd_converter` 使用的控制映射文件。当前控制基于 CARLA 中的 `vehicle.toyota.prius` 蓝图 ID 校准。更改车辆类型可能需要重新校准。 |
| `traffic_light.publish`           | bool   | False                                                                             | 将 CARLA 交通灯状态以 `autoware_perception_msgs/TrafficLightGroupArray` 发布到 `/perception/traffic_light_recognition/traffic_signals`。参见[发布 CARLA 交通灯状态](#publishing-carla-traffic-light-states)。 |
| `traffic_light.force_green`       | bool   | False                                                                             | 启动时将所有 CARLA 交通灯设为绿灯并冻结在该状态。适用于无摄像头、无交通灯识别的闭环运行，否则车辆会在每条受信号灯控制的停车线处等待。 |
| `traffic_light.map_path`          | string | ""                                                                                | Lanelet2 地图（`.osm`）的路径。设置后，按**位置**将 CARLA 交通灯与地图中的交通灯灯头匹配，并使用匹配的监管元素 ID 发布。留空时回退为直接使用 CARLA OpenDRIVE 信号 ID 作为组 ID。 |
| `traffic_light.match_distance`    | double | 5.0                                                                               | 将 CARLA 交通灯匹配到 Lanelet2 灯头时允许的最大灯头间距离（m）。 |
| `traffic_light.match_ratio`       | double | 0.6                                                                               | 歧义阈值：若最近的、对应于_不同_信号的灯头与最佳候选几乎同样近（`nearest > ratio * second`），则拒绝匹配。值越小越严格。 |
| `traffic_light.id_map`            | string | ""                                                                                | 可选覆盖配置，格式为 `opendrive_id:group_id,...`，将 CARLA 信号 ID 固定映射到一个或多个 Autoware 组 ID，优先于位置匹配。单个条目可列出多个组 ID（以竖线分隔），将共用灯头映射到其全部监管元素；可用来修复匹配器报告为有歧义或未匹配的少量交通灯。 |
| `wake_sleeping_physics`           | bool   | False                                                                             | 从静止状态起步时，通过很小的 `set_target_velocity` 唤醒自车物理刚体。仅 CARLA 0.10（UE5/Chaos）需要：其中静止刚体会进入休眠，而 `VehicleControl` 的油门无法将其唤醒。在支持的 0.9.15 环境中，刚体不会休眠，应保持 `false`，以保持原有起步动力学行为。 |

> 这些 `traffic_light.*` 启动参数也是同名节点参数，在 `ros2 param list` 中统一归入 `traffic_light.` 命名空间。

<a id="ground-snapping"></a>

#### 地面吸附

启用 `spawn_point_ground_snap` 后，自车生成点以及 RViz 中的 "2D
Pose Estimate" 初始位姿会吸附到 CARLA 地图几何表面，
而非使用固定 z 偏移。这有助于处理部分关卡（例如某些 CARLA 0.10 地图）中
地图坐标系 z 值与地形不一致的问题；固定偏移可能使
车辆落在远高于或低于道路的位置。

地面高度通过 `world.ground_projection` 获取：从
`z = 1000 m` 向下投射射线。系统不只探测目标 `(x, y)`，而是采样一个较小的
十字形邻域，并采用命中的**最高**地面：

```text
sample offsets (dx, dy in meters)
              (0, +1.5)
              (0, +0.75)
  (-1.5, 0) (-0.75, 0) (0, 0) (+0.75, 0) (+1.5, 0)
              (0, -0.75)
              (0, -1.5)
```

- 采样邻域（9 个点）可以提高稳健性：单条射线可能
  穿过网格间隙，或落在排水沟/路缘接缝中，返回
  低于道路表面的高度。
- 取最大值可以选中路面，而不是较低的接缝或间隙，
  从而使车辆位于路面上方，避免陷入路面。

如果 CARLA API 不提供 `ground_projection`，则跳过吸附并使用原有
固定 z 偏移，因此启用该标志不会导致异常。生成点处理流程
在回退时记录警告；RViz 初始位姿的回退不会输出提示（并且
使用默认随机生成方式时，完全不会执行生成点处理流程）。

<a id="multi-client-co-simulation"></a>

### 多客户端联合仿真

默认情况下，此桥接器控制 CARLA 仿真时钟：主循环在每次
循环中调用 `world.tick()`。同步模式的服务器每收到一次 `tick()` 调用就推进一帧，因此若第二个客户端
也推进时钟，例如向同一服务器注入背景车辆的外部交通仿真器，
就会使仿真在每个预期步长内推进多次。

将 `tick_follower` 设为 `True` 会使桥接器进入被动模式。其主循环不再推进世界，
而是针对外部客户端推进的帧发布传感器数据、
时钟和自车控制。整个系统中必须恰好只有一个客户端控制时钟。

使用此模式时需注意以下事项：

- **先启动桥接器，再启动时钟控制端。** 加载世界时仍会推进几次时钟，以启动
  自车及其传感器；这些推进操作不能与外部控制端竞争。
- **运行节奏由时钟控制端决定**，因此 `max_real_delta_seconds` 不再控制循环节奏。
  将 `fixed_delta_seconds` 设置为时钟控制端使用的步长。
- **`/clock` 在桥接器处理第一帧时从零开始。** 此模式下，它与
  CARLA 已用时间之间保持固定偏移，偏移量等于时钟控制端启动前的空闲时间。

如果桥接器跟不上输入帧的节奏，会丢弃已经落后的帧，
并通过限频警告报告跳过的帧数。

<a id="sensor-configuration"></a>

### 传感器配置

此接口使用 **`carla_sensor_kit`**，提供覆盖 360 度视野的 6 个摄像头，以及激光雷达、IMU 和 GNSS 传感器。传感器配置通过以下两个配置文件，从 Autoware 传感器套件标定文件动态加载：

<a id="1-sensor-kit-calibration-from-autoware-sensor-kit"></a>

#### 1. 传感器套件标定（来自 Autoware 传感器套件）

位于 `<sensor_kit_name>_description/config/sensor_kit_calibration.yaml`。

定义传感器相对于 `base_link`（后轴中心）的位置与姿态。例如：

```yaml
sensor_kit_base_link:
  CAM_FRONT/camera_link:
    x: 2.225
    y: 0.000
    z: 1.600
    roll: 0.000
    pitch: 0.000
    yaw: 0.000 # Angles in radians
```

<a id="2-sensor-mapping-carla-specific"></a>

#### 2. 传感器映射（CARLA 专用）

位于 `config/sensor_mapping.yaml`。

将 Autoware 传感器映射到 CARLA 传感器类型和参数。主要部分：

- `default_sensor_kit_name`：使用的默认传感器套件（例如 `carla_sensor_kit_description`）。
- `sensor_mappings`：将每个传感器映射到 CARLA 类型和 ROS 话题。
- `enabled_sensors`：要在 CARLA 中生成的传感器列表。
- `vehicle_config`（可选）：轴距等车辆参数。

传感器映射示例：

```yaml
sensor_mappings:
  CAM_FRONT/camera_link:
    carla_type: sensor.camera.rgb
    id: CAM_FRONT
    ros_config:
      frame_id: CAM_FRONT/camera_optical_link
      topic_image: /sensing/camera/CAM_FRONT/image_raw
      topic_info: /sensing/camera/CAM_FRONT/camera_info
      frequency_hz: 11
      qos_profile: reliable
      image_encoding: bgra8
    parameters:
      image_size_x: 1600
      image_size_y: 900
      fov: 70.0
```

`image_encoding` 适用于摄像头，可设为 `bgra8`（默认值，也是 CARLA
渲染的格式）或 `mono8`。发布 `mono8` 时在桥接器中转换一次，发送的
字节数为原来的四分之一；当摄像头的所有下游使用方都仅处理亮度时，
例如特征跟踪或视觉里程计，这种方式很有价值。
1600x900 图像帧使用 `bgra8` 时为 5,760,000 字节，使用 `mono8` 时为 1,440,000 字节。

CARLA 传感器参数请参见 [CARLA 传感器参考](https://carla.readthedocs.io/en/latest/ref_sensors/)。

<a id="capture-rate"></a>

##### 采集频率

`frequency_hz` 限制桥接器的发布频率，并不改变
CARLA 的采集频率。保持 CARLA 默认设置的传感器在每个仿真
步都会采集，因此在 1/600 s 步长下，摄像头每秒渲染 600 帧，而桥接器
仅保留少量帧，其余全部丢弃。限频只能丢弃完整帧，因此
在该步长下，要求 60 Hz 的映射会以 85.7 Hz 发布，而 200 Hz 的 IMU 会以
300 Hz 发布。

在传感器的 `parameters` 中设置 `sensor_tick`（两次采集之间的秒数），
让 CARLA 按映射所需频率生成数据：

```yaml
parameters:
  image_size_x: 1600
  image_size_y: 900
  fov: 70.0
  sensor_tick: 0.0166667
```

未设置 `sensor_tick` 的传感器仍与之前一样，每个仿真步都进行采集。避免将
`sensor_tick` 设为与 `fixed_delta_seconds` 完全相等：CARLA 使用浮点数比较采集
间隔和已用时间，采集周期与仿真步长相等的传感器
可能漏帧
（[carla#3653](https://github.com/carla-simulator/carla/issues/3653)）。

CARLA 无法在两个仿真步之间采集，因此通过
交替使用较短和较长的间隔保持所需平均频率——在 1/60 s 步长下，0.04 s 的采集周期会
交替在两个和三个仿真步后到达。发布限频允许此类传感器的帧
最多提前半个自身采集周期到达，这样这些帧就会被发布，
而非被丢弃。

<a id="sensor-noise"></a>

##### 传感器噪声

除非映射另有设置，IMU 和 GNSS 默认生成无噪声数据，
这有助于复现运行结果。但也意味着这些传感器的使用方
看到的是任何实际硬件都无法产生的测量值：使用完美陀螺仪进行评估的
定位或里程计系统，会报告实车无法达到的精度。

在传感器的 `parameters` 中设置相应 CARLA 噪声属性，
可使传感器表现更接近硬件：

```yaml
sensor_mappings:
  imu_link:
    carla_type: sensor.other.imu
    id: imu
    ros_config:
      frame_id: tamagawa/imu_link
      topic: /sensing/imu/imu_data
      frequency_hz: 50
    parameters:
      noise_gyro_stddev_x: 0.001
      noise_gyro_stddev_y: 0.001
      noise_gyro_stddev_z: 0.001
      noise_gyro_bias_x: 0.0005
      noise_accel_stddev_x: 0.01
```

IMU 支持的名称为 `noise_accel_stddev_{x,y,z}`、`noise_gyro_stddev_{x,y,z}`
和 `noise_gyro_bias_{x,y,z}`；GNSS 支持 `noise_{alt,lat,lon}_stddev` 和
`noise_{alt,lat,lon}_bias`。未设置的属性保持为零。

<a id="light-weight-sensor-mapping"></a>

##### 轻量级传感器映射

针对 GPU/CPU 资源有限的机器，另提供 `config/sensor_mapping_light_weight.yaml`，用于降低仿真器负载。与默认映射相比，它：

- 使用**单个前置摄像头**（`CAM_FRONT`）替代 6 摄像头 360 度配置，并采用更宽的视场角（120°）和更低的分辨率（1080x720），大致覆盖车辆前方区域。
- 降低传感器频率（例如激光雷达/摄像头从 11 Hz 降为 10 Hz）。
- 保留与默认映射相同的激光雷达、IMU 和 GNSS 配置。

通过 `use_light_weight_sensor_mapping:=True` 启用后，启动文件还会自动跳过已禁用摄像头的 `image_transport` 重发布节点，以及 `multi_camera_combiner` 节点。

使用示例：

```bash
ros2 launch autoware_launch e2e_simulator.launch.xml \
    map_path:=$HOME/autoware_map/Town01 \
    vehicle_model:=sample_vehicle \
    sensor_model:=carla_sensor_kit \
    simulator_type:=carla \
    use_light_weight_sensor_mapping:=True
```

请注意，使用此配置时，依赖环视摄像头的功能（例如 RViz 多摄像头视图）不可用。

<a id="world-loading"></a>

### 世界加载

`carla_ros.py` 负责配置 CARLA 世界：

1. **客户端连接**：

   ```python
   client = carla.Client(self.local_host, self.port)
   client.set_timeout(self.timeout)
   ```

2. **加载地图**：

   根据 `carla_map` 参数，在 CARLA 世界中加载相应地图。

   ```python
   client.load_world(self.map_name)
   self.world = client.get_world()
   ```

   加载后，接口会检查 CARLA 实际运行的地图是否
   与请求一致。若不一致——例如关卡不存在、
   服务器切换失败或连接中断——会将不匹配信息记录到
   `rosout`，并以 `CarlaWorldLoadError` 中止节点。如果 Autoware 使用的
   地图与仿真器实际仿真的地图不同，整个运行结果就会在没有明显提示的情况下失效，
   因此该问题被视为致命错误，不予容忍。

3. **生成自车**：

   根据 `vehicle_type`、`spawn_point` 和 `agent_role_name` 参数生成车辆。

   ```python
   spawn_point = carla.Transform()
   point_items = self.spawn_point.split(",")
   if len(point_items) == 6:
      spawn_point.location.x = float(point_items[0])
      spawn_point.location.y = float(point_items[1])
      spawn_point.location.z = float(point_items[2]) + 2
      spawn_point.rotation.roll = float(point_items[3])
      spawn_point.rotation.pitch = float(point_items[4])
      spawn_point.rotation.yaw = float(point_items[5])
   CarlaDataProvider.request_new_actor(self.vehicle_type, spawn_point, self.agent_role_name)
   ```

<a id="traffic-light-recognition"></a>

## 交通灯识别

Carla 仿真器提供的地图（[Carla Lanelet2 地图](https://bitbucket.org/carla-simulator/autoware-contents/src/master/maps/)）目前缺少适用于 Autoware 的完整交通灯组件，且经纬度坐标与点云地图不同。要启用交通灯识别，请按以下步骤修改地图。

- 修改地图的方式

  - A. 从零创建新地图
  - 使用 [TIER IV Vector Map Builder](https://tools.tier4.jp/feature/vector_map_builder_ll2/) 创建新地图。

  - B. 修改现有 Carla Lanelet2 地图
  - 调整 [Carla Lanelet2 地图](https://bitbucket.org/carla-simulator/autoware-contents/src/master/maps/)的经纬度，使其与 PCD（原点）对齐。
    - 使用此[工具](https://github.com/mraditya01/offset_lanelet2/tree/main)修改坐标。
    - 使用 [TIER IV Vector Map Builder](https://tools.tier4.jp/feature/vector_map_builder_ll2/) 将 Lanelet 与 PCD 对齐，并添加交通灯。

- 使用 TIER IV Vector Map Builder 时，必须将 PCD 格式从 `binary_compressed` 转换为 `ascii`。可使用 `pcl_tools` 完成转换。
- 作为参考，可在[此处](https://drive.google.com/drive/folders/1QFU0p3C8NW71sT5wwdnCKXoZFQJzXfTG?usp=sharing)下载一个 Town01 示例，其中一个交叉口已添加交通灯。

<a id="publishing-carla-traffic-light-states"></a>

### 发布 CARLA 交通灯状态

桥接器可直接发布 CARLA 服务器的交通灯状态，
无需运行基于摄像头的识别。设置 `traffic_light.publish:=true` 后，每个仿真步都会将
`autoware_perception_msgs/TrafficLightGroupArray` 发布到
`/perception/traffic_light_recognition/traffic_signals`。每个 CARLA 交通灯
均报告为圆形信号，其颜色和状态跟随 CARLA 状态：`Red`/`Yellow`/`Green`
对应 `RED`/`AMBER`/`GREEN`，状态为 `SOLID_ON`；已知熄灭的 `Off` 状态对应
`SOLID_OFF`；仅当桥接器无法解释状态时才发布为 `UNKNOWN`/`UNKNOWN`。

Autoware 使用 `traffic_light_group_id` 标识交通信号，该值是 Lanelet2 地图中 `traffic_light`
监管元素的 ID。桥接器按以下优先级解析每个 CARLA 交通灯
所属的一个或多个组：

1. **`traffic_light.id_map` 覆盖配置。** 如果交通灯的 OpenDRIVE 信号 ID 出现在
   `opendrive_id:group_id[|group_id...],...` 映射中，则直接使用这些组 ID。一个条目
   可指定多个组 ID（用 `|` 分隔），从而将共用的物理灯头映射到管理它的全部
   监管元素。格式错误的条目（缺少 `:`、ID 非整数或 `:` 后无组
   ID）会被跳过，并输出指明该条目的警告。因此，拼写错误既不会
   使桥接器停止，也不会悄然用空组列表覆盖某个交通灯；其余
   条目仍然生效。
2. **位置匹配（`traffic_light.map_path`）。** 提供 Lanelet2 地图后，每个 CARLA
   灯头会与最近的地图交通灯灯头匹配，其状态会通过引用该灯头的
   **每个**监管元素发布（一个物理交通灯通常由多个监管元素共用，
   每条驶入车道各有一个）。此方式不要求 CARLA 与地图采用一致的 ID 约定，
   因此也适用于手工制作或 Vector Map Builder 创建的地图，即使其中的监管元素
   ID 与 OpenDRIVE 信号 ID 不对应。
3. **OpenDRIVE ID 回退。** 未提供地图路径且未设置覆盖配置时，直接将 OpenDRIVE 信号 ID
   用作组 ID（仅适用于生成时将监管元素 ID 保持为
   OpenDRIVE 信号 ID 的地图）。

位置匹配采用保守策略：只有单个地图灯头明显最近时，才将 CARLA 交通灯与其绑定。
如果属于_不同_信号的灯头距离几乎同样近（典型的
“交叉口对面交通灯”情况，由 `traffic_light.match_ratio` 控制），或者距离完全
相等（平局时没有最佳候选，因此不能由 .osm 中的顺序决定），或没有任何灯头位于
`traffic_light.match_distance` 范围内，则不发布该交通灯，并记录为有歧义/未匹配，
而不是猜测。请查看节点启动日志中的匹配报告（`N matched, M ambiguous,
K too far`），必要时通过 `traffic_light.id_map` 为所报告的交通灯指定映射。

> 位置匹配读取 Lanelet2 节点的 `local_x`/`local_y` 标签，即 Autoware 地图坐标系，
> 并通过 `map_origin_x`/`map_origin_y` 将每个 CARLA 灯头表示在该坐标系中（与定位使用
> 相同的偏移）。定位对齐后，匹配也会对齐。

如需在没有任何识别配置的情况下让自车通过所有交叉口——例如
无摄像头的闭环运行——请设置 `traffic_light.force_green:=true`。启动时会将所有
CARLA 交通灯设为绿灯并冻结；结合 `traffic_light.publish:=true`，
冻结后的绿灯状态也会发布到上述话题。

<a id="tips"></a>

## 提示

- 初始化时可能出现对齐偏差，按下 `init by gnss` 按钮应可解决。
- 更改 `fixed_delta_seconds` 可以提高仿真更新频率（默认步长为 0.05 s），修改时还需调整 `sensor_mapping.yaml` 中的部分传感器参数（例如激光雷达旋转频率应与 FPS 匹配）。

<a id="known-issues-and-future-works"></a>

## 已知问题与后续工作

- **在程序化生成地图（Adv Digital Twin）上测试**：由于 Adv Digital Twin 地图创建失败，目前无法测试。
- **交通灯识别**：默认 CARLA Lanelet2 地图缺少适当的交通灯监管元素。解决方法见上文“交通灯识别”一节，也可通过 `traffic_light.publish` 完全绕过摄像头识别（参见“发布 CARLA 交通灯状态”）。
