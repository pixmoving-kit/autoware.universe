<a id="landmark-based-localizer"></a>

# 基于地标的定位器

此目录包含基于地标进行定位的功能包。

地标可以是：

- 相机检测到的 AR 标签
- 激光雷达通过强度特征检测到的标志板

等。

这些地标容易检测并估计位姿，因此，只要提前将地标位姿记录在地图中，就可以根据检测到的地标位姿计算自车位姿。

目前假定地标为平面。

下图以 `ar_tag_based_localizer` 为例说明定位原理。

![原理](./doc_image/principle.png)

计算出的自车位姿传递给 EKF，与速度信息融合，以估计更准确的自车位姿。

<a id="node-diagram"></a>

## 节点图

![节点图](./doc_image/node_diagram.drawio.svg)

### `landmark_manager`

下一节介绍地图中地标的定义，请参阅“地图规范”。

`landmark_manager` 是从地图加载地标的工具包。

- 平移：地标四个顶点的中心
- 旋转：如下节所示，将顶点按逆时针顺序编号为 1、2、3、4。方向定义为从 1 指向 2 的向量与从 2 指向 3 的向量的叉积方向。

用户可以将地标定义为 Lanelet2 的四顶点多边形。
此时，四个顶点可能并不位于同一平面上，这种情况下很难计算地标方向。
因此，如果将这四个顶点视为四面体，且其体积超过 `volume_threshold` 参数，便不会为该地标发布 tf_static。

<a id="landmark-based-localizer-packages"></a>

### 基于地标的定位功能包

- ar_tag_based_localizer
- 等。

<a id="map-specifications"></a>

## 地图规范

参见 <https://github.com/autowarefoundation/autoware_lanelet2_extension/blob/main/autoware_lanelet2_extension/docs/lanelet2_format_extension.md#localization-landmarks>

<a id="about-consider_orientation"></a>

## 关于 `consider_orientation`

`LandmarkManager` 类的 `calculate_new_self_pose` 函数包含一个名为 `consider_orientation` 的布尔参数，用于决定根据检测地标和地图地标计算新自车位姿的方法。下图展示两种方法的区别。

![是否考虑姿态的区别](./doc_image/consider_orientation.drawio.svg)

### `consider_orientation = true`

在此模式下，计算新的自车位姿，使“基于当前自车位姿检测到的地标”的相对位姿，等于“从新自车位姿观察地图地标”的相对位姿。
此方法可以校正姿态，但容易受到地标检测姿态误差的较大影响。

### `consider_orientation = false`

在此模式下，计算新的自车位姿，仅保证 x、y、z 方向上的相对位置正确。

此方法无法校正姿态，但不受地标检测姿态误差的影响。
