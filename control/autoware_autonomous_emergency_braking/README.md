<a id="autonomous-emergency-braking-aeb"></a>

# 自动紧急制动（AEB）

<a id="purpose-role"></a>

## 目的与作用

`autonomous_emergency_braking` 模块用于防止车辆与预测路径上的障碍物碰撞，该路径由控制模块生成，或根据控制模块估计所用的传感器数值生成。

<a id="assumptions"></a>

### 前提假设

本模块基于以下假设。

- 自车预测路径可以使用基于传感器生成的路径、控制模块生成的路径，或同时使用两者。

- 可以从自车传感器获取当前速度和角速度，并使用点来表示障碍物。

- AEB 的目标障碍物是二维点，可从输入点云获得，也可通过计算自车预测轮廓路径与预测物体形状的交点获得。

<a id="imu-path-generation-steering-angle-vs-imus-angular-velocity"></a>

### IMU 路径生成：转向角与 IMU 角速度

目前，基于 IMU 的路径使用 IMU 自身测得的角速度生成。有人建议用转向角替代角速度。

两种方法的优缺点如下：

IMU 角速度：

- （优点）通常精度较高。
- （缺点）车辆振动可能引入噪声。

转向角：

- （优点）噪声较小。
- （缺点）可能存在转向偏置或传动比错误，而且 Autoware 中的转向角可能与实际转向角不一致。

目前暂不计划在 AEB 模块的路径生成过程中引入转向角。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

AEB 在输出紧急停车信号之前执行以下步骤。

1. 必要时启用 AEB。

2. 生成自车预测路径。

3. 从输入点云和/或预测物体数据中获取目标障碍物。

4. 估计最近障碍物的速度。

5. 与目标障碍物进行碰撞检查。

6. 向 `/diagnostics` 发送紧急停车信号。

下面详细介绍各个步骤。

<a id="1-activate-aeb-if-necessary"></a>

### 1. 必要时启用 AEB

满足以下情况时，不启用 AEB 模块。

- 自车未处于自动驾驶状态。

- 自车未运动（当前速度低于 0.1 m/s 阈值）。

<a id="2-generate-a-predicted-path-of-the-ego-vehicle"></a>

### 2. 生成自车预测路径

<a id="21-overview-of-imu-path-generation"></a>

#### 2.1 IMU 路径生成概述

AEB 根据车载传感器获取的当前速度和角速度，生成预测轮廓路径。请注意，如果 `use_imu_path` 为 `false`，则跳过此步骤。预测路径按以下公式生成：

$$
x_{k+1} = x_k + v cos(\theta_k) dt \\
y_{k+1} = y_k + v sin(\theta_k) dt \\
\theta_{k+1} = \theta_k + \omega dt
$$

其中，$v$ 和 $\omega$ 分别为当前纵向速度和角速度。$dt$ 是时间间隔，用户可通过 `imu_prediction_time_interval` 参数预先定义。IMU 路径在 `imu_prediction_time_horizon` 参数定义的预测时域内生成。

<a id="22-constraints-and-countermeasures-in-imu-path-generation"></a>

#### 2.2 IMU 路径生成的约束与应对措施

由于 IMU 路径生成仅使用自车当前角速度，不考虑 MPC 规划的转向，IMU 路径形状容易发生畸变并伸出当前车道，可能导致不必要的紧急停车。对此有两种应对措施：

1. 通过 `max_generated_imu_path_length` 参数控制。
   - 路径长度超过设定值时停止生成。
   - 避免使用过大的 `imu_prediction_time_horizon`。

2. 根据横向偏差控制。
   - 将 `limit_imu_path_lat_dev` 参数设为 "true"。
   - 使用 `imu_path_lat_dev_threshold` 设置偏差阈值。
   - 横向偏差超过阈值时停止生成路径。

<a id="23-advantages-and-limitations-of-lateral-deviation-control"></a>

#### 2.3 横向偏差控制的优点与局限

通过 `limit_imu_path_lat_dev` 设置横向偏差限制的优点是，可以增大 `imu_prediction_time_horizon` 和 `max_generated_imu_path_length`，而不必担心 IMU 预测路径的畸变超过一定阈值。缺点是自车角速度较高时，IMU 路径会被提前截断；此时 AEB 模块主要依靠 MPC 路径来防止或减轻碰撞。

如果假设自车大多沿所在 lanelet 的中心线行驶，可以将横向偏差阈值 `imu_path_lat_dev_threshold` 设为不大于平均 lanelet 宽度的一半。这样 IMU 预测路径驶出当前 lanelet 的可能性更小，并且可以增大 `imu_prediction_time_horizon`，以在车辆主要直线行驶时预防前方碰撞。

横向偏差以自车当前位置为参考，测量自车预测轮廓中最远顶点到预测路径的距离。下图说明如何测量给定自车位姿的横向偏差。

![横向偏差测量](./image/measuring-lat-dev-on-imu-path.drawio.svg)

<a id="24-imu-path-generation-algorithm"></a>

#### 2.4 IMU 路径生成算法

<a id="241-selection-of-lateral-deviation-check-points"></a>

##### 2.4.1 选择横向偏差检查点

根据以下条件选择用于横向偏差检查的车辆顶点：

- 前进（$v > 0$）
  - 右转（$\omega > 0$）：右前顶点。
  - 左转（$\omega < 0$）：左前顶点。
- 后退（$v < 0$）
  - 右转（$\omega > 0$）：右后顶点。
  - 左转（$\omega < 0$）：左后顶点。
- 直行（$\omega = 0$）：根据前进或后退，检查两个前顶点或后顶点。

<a id="242-path-generation-process"></a>

##### 2.4.2 路径生成过程

在每个时间步执行以下步骤：

1. 状态更新
   - 根据当前速度 $v$ 和角速度 $\omega$，计算下一个位置 $(x_{k+1}, y_{k+1})$ 和偏航角 $\theta_{k+1}$。
   - 时间间隔 $dt$ 由 `imu_prediction_time_interval` 参数决定。

2. 生成车辆轮廓
   - 在计算出的位置放置车辆轮廓。
   - 计算检查点坐标。

3. 计算横向偏差
   - 计算所选顶点到路径的横向偏差。
   - 更新路径长度和已用时间。

4. 判断终止条件

<a id="243-termination-conditions"></a>

##### 2.4.3 终止条件

满足以下任一条件时，终止路径生成：

1. 基本终止条件（两项必须同时满足）
   - 预测时间超过 `imu_prediction_time_horizon`。
   - 并且路径长度超过 `min_generated_imu_path_length`。

2. 路径长度终止条件
   - 路径长度超过 `max_generated_imu_path_length`。

3. 横向偏差终止条件（当 `limit_imu_path_lat_dev = true` 时）
   - 所选顶点的横向偏差超过 `imu_path_lat_dev_threshold`。

<a id="mpc-path-generation"></a>

#### MPC 路径生成

如果 `use_predicted_trajectory` 参数设为 true，AEB 模块会直接以 MPC 的预测路径为基础生成轮廓路径。它会复制 MPC 在给定预测时域内生成的自车位姿。`mpc_prediction_time_horizon` 决定 MPC 路径向未来预测自车运动的时长。IMU 轮廓路径和 MPC 轮廓路径可以同时使用。

<a id="3-get-target-obstacles"></a>

### 3. 获取目标障碍物

生成自车轮廓路径后，需要识别目标障碍物。有两种获取方法：使用输入点云，或使用感知模块提供的预测物体信息。

<a id="pointcloud-obstacle-filtering"></a>

#### 点云障碍物过滤

AEB 模块可以过滤输入点云，找出自车可能与之碰撞的目标障碍物。将 `use_pointcloud_data` 设为 true 即可启用此方法。点云障碍物过滤包括三个主要步骤：粗过滤、通过聚类去除噪声、精过滤。

<a id="rough-filtering"></a>

##### 粗过滤

粗过滤阶段使用简单过滤器选择目标障碍物。在距自车预测路径一定距离内建立搜索区域（默认距离为自车宽度的一半加上 `path_footprint_extra_margin` 和 `expand_width` 参数），忽略该区域之外的点云。粗过滤过程如下图所示。

![粗过滤](./image/obstacle_filtering_1.drawio.svg)

<a id="noise-filtering-with-clustering-and-convex-hulls"></a>

##### 使用聚类与凸包过滤噪声

为防止 AEB 将噪声点纳入考虑，模块对过滤后的点云执行欧氏聚类。无法与其他点靠近到足以形成簇的点会被丢弃。此外，会将簇中各点的高度与 `cluster_minimum_height` 比较；如果簇内没有任何点的高度或 z 值大于 `cluster_minimum_height`，则丢弃整个点簇。可以通过 `cluster_tolerance`、`minimum_cluster_size` 和 `maximum_cluster_size` 调整聚类及需要忽略的物体大小。有关 AEB 使用的聚类方法，请参阅 PCL 库的欧氏聚类官方文档：<https://pcl.readthedocs.io/projects/tutorials/en/master/cluster_extraction.html>。

此外，会为每个检测到的簇构建二维凸包，其顶点表示该簇最外侧的极值点。这些顶点将在下一步中检查。

<a id="rigorous-filtering"></a>

##### 精过滤

去除噪声后，模块执行几何碰撞检查，判断过滤后的障碍物或凸包顶点是否确实可能与自车碰撞。检查时，自车表示为矩形，点云障碍物表示为点。只有可能发生碰撞的顶点才会被标记为目标障碍物。

![精过滤](./image/obstacle_filtering_2.drawio.svg)

<a id="obstacle-labeling"></a>

##### 障碍物标记

精过滤后，对剩余障碍物进行标记。只有位于所定义自车轮廓内的障碍物才会获得用于碰撞检查的“目标”标签；该轮廓由自车宽度与 `expand_width` 参数构建。至少有一个障碍物被标记为目标，才可能触发紧急停车。

![障碍物标记](./image/labeling.drawio.svg)

<a id="using-predicted-objects-to-get-target-obstacles"></a>

#### 使用预测物体获取目标障碍物

如果 `use_predicted_object_data` 参数设为 true，AEB 可使用感知模块提供的预测物体数据获取目标障碍点。具体方式是计算自车预测轮廓路径（由自车宽度与 `expand_width` 参数构建）与每个预测物体的包络多边形或包围框之间的二维交点。如果没有交点，则丢弃所有点。

![预测物体与路径的交点](./image/using-predicted-objects.drawio.svg)

<a id="finding-the-closest-target-obstacle"></a>

### 查找最近的目标障碍物

使用点云数据和/或预测物体数据识别所有可能的障碍物后，AEB 模块选择距自车最近的点作为碰撞检查候选。“最近物体”是指位于自车轮廓内（由车宽与 `expand_width` 确定），以 IMU 或 MPC 路径为参考，在纵向上距自车最近的障碍物。目标障碍物的优先级高于自车路径外的障碍物，即使后者在纵向上更近。这确保碰撞检查聚焦于对车辆轨迹威胁最大的物体。

如果没有找到目标障碍物，AEB 会考虑路径外的其他邻近障碍物。这时会跳过碰撞检查，但记录最近障碍物的位置，用于计算其速度（第 4 步）。请注意，通过预测物体数据获得的障碍物均位于自车轮廓路径内，因此都是目标障碍物，并且无须计算速度（感知模块已完成计算）。这类障碍物不参与第 4 步。

![最近物体](./image/closest-point.drawio.svg)

<a id="4-obstacle-velocity-estimation"></a>

### 4. 障碍物速度估计

开始计算目标点速度之前，该点必须进入速度计算区域，
此区域由 `speed_calculation_expansion_margin` 参数、自车宽度和 `expand_width` 参数共同定义。
根据运行环境，
该余量可以减少不必要的自动紧急制动，
这些误制动可能由初始计算阶段的速度误算引起。

![速度计算区域扩展](./image/speed_calculation_expansion.drawio.svg)

确定最近障碍物或点的位置后，AEB 模块利用此前检测物体的历史数据，按以下公式估计最近物体的相对速度：

$$
d_{t} = t_{1} - t_{0}
$$

$$
d_{x} = norm(o_{x} - prev_{x})
$$

$$
v_{norm} = d_{x} / d_{t}
$$

其中，$t_{1}$ 和 $t_{0}$ 分别是用于检测当前最近物体和上一帧最近物体的点云时间戳，$o_{x}$ 和 $prev_{x}$ 分别是这两个物体的位置。

![相对速度](./image/object_relative_speed.drawio.svg)

请注意，如果最近障碍物或点来自预测物体数据，则直接对预测物体在 x、y 轴上的速度求范数，得到 $v_{norm}$。

随后，将速度向量与自车预测路径比较，得到纵向速度 $v_{obj}$：

$$
v_{obj} = v_{norm} * Cos(yaw_{diff}) + v_{ego}
$$

其中，$yaw_{diff}$ 是自车路径与位移向量 $$v_{pos} = o_{pos} - prev_{pos} $$ 之间的偏航角差，$v_{ego}$ 是自车当前速度，用于补偿由自车运动而非物体运动引起的点位移。所有这些计算均忽略 z 轴，在二维平面内进行。

请注意，物体速度相对于自车当前运动方向计算。如果物体运动方向与自车相反，其速度为负，这会降低下一步碰撞评估中的指标值。

AEB 模块将估计的物体速度及其时间戳加入速度历史队列，然后检查其中是否有过期数据。如果某次速度测量距当前的时间超过 `previous_obstacle_keep_time`，则从队列中移除该速度及时间戳。最后，模块计算剩余速度测量值的中位数，并用该值确定碰撞检查所需的制动距离。

<a id="5-collision-check-with-target-obstacles"></a>

### 5. 与目标障碍物进行碰撞检查

第五步中，AEB 模块检查与最近目标物体是否可能发生碰撞。为此，模块计算防止追尾所需的最小安全制动距离。

模块仅评估最近目标物体，因为安全制动距离可作为所有目标物体的判断阈值。如果到最近目标物体的距离被判定为安全，则认为路径上更远的物体也安全。

制动距离的公式为：

$$
d_{braking} = v_{ego}*t_{response} + v_{ego}^2/(2*a_{min}) -(sign(v_{obj})) * v_{obj}^2/(2*a_{obj_{min}}) + offset
$$

其中：

- $v_{ego}$ 和 $v_{obj}$ 分别是自车与障碍物的当前速度。
- $a_{min}$ 和 $a_{obj\_min}$ 分别是自车与障碍物的最大减速度（最小加速度）。
- $t_{response}$ 是自车开始减速所需的响应时间。

如果到障碍物的实际距离小于计算出的距离（$d_{braking}$），AEB 模块会发送紧急停车信号。

只有被归类为“目标”的物体（定义见第 3 步）才参与碰撞评估。其中，距自车最近的目标障碍物用于计算。如果没有“目标”障碍物，即自车预测路径内（由车宽和扩展余量确定）没有障碍物，则跳过此步骤，并记录最近障碍物的位置，供后续速度计算使用（第 4 步）。此时不会生成紧急停车诊断消息。流程如下图所示。

![制动距离](./image/braking_distance.drawio.svg)

<a id="6-send-emergency-stop-signals-to-diagnostics"></a>

### 6. 向 `/diagnostics` 发送紧急停车信号

如果 AEB 在上一步检测到与点云障碍物的碰撞，则在此步骤向 `/diagnostics` 发送紧急信号。请注意，要启用紧急停车，必须发送 ERROR 级别的紧急状态。此外，AEB 用户应修改配置文件，使紧急级别保持，否则 Autoware 不会持续保持紧急状态。

<a id="use-cases"></a>

## 用例

<a id="front-vehicle-suddenly-brakes"></a>

### 前车突然制动

当前车突然制动，且 AEB 模块检测到碰撞风险时，AEB 可以触发。只要自车与前车距离足够大，且自车紧急制动加速度的幅值足够大，就有可能避免或减轻与突然制动前车的碰撞。注意：AEB 计算制动距离时使用的加速度，不一定是自车实际紧急制动时采用的加速度。可以通过修改 [mrm_emergency stop 的加加速度和加速度值](https://github.com/tier4/autoware_launch/blob/d1b2688f2788acab95bb9995d72efd7182e9006a/autoware_launch/config/system/mrm_emergency_stop_operator/mrm_emergency_stop_operator.param.yaml#L4)来调整实车使用的加速度。

![防止与前车碰撞](./image/front_vehicle_collision.drawio.svg)

<a id="stop-for-objects-that-appear-suddenly"></a>

### 为突然出现的物体停车

如果其他模块未能及时检测到突然出现的物体，AEB 可以作为故障安全措施使自车停车。如果预计物体可能突然切入，可以增大 `expand_width`，让 AEB 在物体进入自车实际路径之前就检测碰撞风险。

![防止与遮挡物体碰撞](./image/occluded_space.drawio.svg)

<a id="preventing-collisions-with-rear-objects"></a>

### 防止与后方物体碰撞

AEB 模块也可以在自车倒车时防止碰撞。

![倒车](./image/backward-driving.drawio.svg)

<a id="preventing-collisions-in-case-of-wrong-odometry-imu-path-only"></a>

### 里程计错误时防止碰撞（仅 IMU 路径）

当车辆里程计信息出错时，MPC 可能无法正确预测自车路径。如果 MPC 预测路径错误，规划模块中的避碰功能可能无法按预期工作。但 AEB 的 IMU 路径不依赖 MPC，因此在其他模块无法预测碰撞时，它仍可能检测到风险。下图展示了一个假设案例：MPC 路径错误，只有 AEB 的 IMU 路径检测到碰撞。

![错误的 MPC 路径](./image/wrong-mpc.drawio.svg)

<a id="parameters"></a>

## 参数

{{ json_to_markdown("control/autoware_autonomous_emergency_braking/schema/autonomous_emergency_braking.schema.json") }}

<a id="limitations"></a>

## 局限性

- 检测到碰撞风险后的停车距离取决于自车速度和减速能力。为了避免碰撞，需要增大检测距离并设置更高的减速度，但这也可能增加不必要的触发次数。因此，应明确本模块应承担的角色，并据此调整参数。

- AEB 可能无法对贴近地面的障碍物做出反应，这取决于点云预处理方法的性能。

- 由于噪声较大，不使用传感器获取的纵向加速度信息。

- 基于传感器数据生成的预测路径，其精度取决于车载传感器的精度。

![AEB 范围](./image/range.drawio.svg)
