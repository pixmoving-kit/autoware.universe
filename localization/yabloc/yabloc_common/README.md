# yabloc_common

此功能包包含一些与地图相关的可执行节点，并提供 YabLoc 通用库。

- [ground_server](#ground_server)
- [ll2_decomposer](#ll2_decomposer)

## ground_server

<a id="purpose"></a>

### 用途

根据 Lanelet2 估计地面的高度和倾斜程度。

<a id="input-outputs"></a>

### 输入／输出

<a id="input"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------ | --------------------------------------- | ------------------- |
| `input/vector_map` | `autoware_map_msgs::msg::LaneletMapBin` | 矢量地图 |
| `input/pose` | `geometry_msgs::msg::PoseStamped` | 估计的自车位姿 |

<a id="output"></a>

#### 输出

| 名称 | 类型 | 说明 |
| ----------------------- | ---------------------------------- | ------------------------------------------------------------------------------- |
| `output/ground` | `std_msgs::msg::Float32MultiArray` | 估计的地面参数，包含 x、y、z、normal_x、normal_y、normal_z。 |
| `output/ground_markers` | `visualization_msgs::msg::Marker` | 估计地面平面的可视化 |
| `output/ground_status` | `std_msgs::msg::String` | 地面平面估计的状态日志 |
| `output/height` | `std_msgs::msg::Float32` | 高程 |
| `output/near_cloud` | `sensor_msgs::msg::PointCloud2` | 从 Lanelet2 提取、用于估计地面倾斜程度的点云 |

<a id="parameters"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_common/schema/ground_server.schema.json") }}

## ll2_decomposer

<a id="purpose_1"></a>

### 用途

此节点从 Lanelet2 提取与路面标线和 YabLoc 相关的元素。

<a id="input-outputs_1"></a>

### 输入／输出

<a id="input_1"></a>

#### 输入

| 名称 | 类型 | 说明 |
| ------------------ | --------------------------------------- | ----------- |
| `input/vector_map` | `autoware_map_msgs::msg::LaneletMapBin` | 矢量地图 |

<a id="output_1"></a>

#### 输出

| 名称 | 类型 | 说明 |
| -------------------------- | -------------------------------------- | --------------------------------------------- |
| `output/ll2_bounding_box` | `sensor_msgs::msg::PointCloud2` | 从 Lanelet2 提取的包围盒 |
| `output/ll2_road_marking` | `sensor_msgs::msg::PointCloud2` | 从 Lanelet2 提取的路面标线 |
| `output/ll2_sign_board` | `sensor_msgs::msg::PointCloud2` | 从 Lanelet2 提取的交通标志牌 |
| `output/sign_board_marker` | `visualization_msgs::msg::MarkerArray` | 交通标志牌的可视化 |

<a id="parameters_1"></a>

### 参数

{{ json_to_markdown("localization/yabloc/yabloc_common/schema/ll2_decomposer.schema.json") }}
