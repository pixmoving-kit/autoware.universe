# obstacle_collision_checker

<a id="purpose"></a>

## 目的

`obstacle_collision_checker` 模块用于检查预测轨迹是否与障碍物碰撞，并在发现碰撞时发布诊断错误。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="flow-chart"></a>

### 流程图

```plantuml
@startuml
skinparam monochrome true

title obstacle collision checker : update
start

:calculate braking distance;

:resampling trajectory;
note right
to reduce calculation cost
end note
:filter point cloud by trajectory;

:create vehicle foot prints;

:create vehicle passing area;

partition will_collide {

while (has next ego vehicle foot print) is (yes)
  :found collision with obstacle foot print;
  if (has collision with obstacle) then (yes)
      :set diag to ERROR;
      stop
  endif
end while (no)
:set diag to OK;
stop
}

@enduml
```

<a id="algorithms"></a>

### 算法

<a id="check-data"></a>

### 数据检查

检查 `obstacle_collision_checker` 是否收到去地面点云、predicted_trajectory、参考轨迹和当前速度数据。

<a id="diagnostic-update"></a>

### 诊断更新

如果预测路径上发现碰撞，本模块将诊断状态设为 `ERROR`；否则设为 `OK`。

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ---------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| `~/input/trajectory` | `autoware_planning_msgs::msg::Trajectory` | 参考轨迹 |
| `~/input/trajectory` | `autoware_planning_msgs::msg::Trajectory` | 预测轨迹 |
| `/perception/obstacle_segmentation/pointcloud` | `sensor_msgs::msg::PointCloud2` | 自车应停车或避让的障碍物点云 |
| `/tf` | `tf2_msgs::msg::TFMessage` | TF |
| `/tf_static` | `tf2_msgs::msg::TFMessage` | 静态 TF |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ---------------- | -------------------------------------- | ------------------------ |
| `~/debug/marker` | `visualization_msgs::msg::MarkerArray` | 可视化标记 |

<a id="parameters"></a>

## 参数

| 名称 | 类型 | 说明 | 默认值 |
| :------------------ | :------- | :------------------------------------------------- | :------------ |
| `delay_time` | `double` | 车辆延迟时间 [s] | 0.3 |
| `footprint_margin` | `double` | 车辆轮廓余量 [m] | 0.0 |
| `max_deceleration` | `double` | 自车停车的最大减速度 [m/s^2] | 2.0 |
| `resample_interval` | `double` | 轨迹重采样间隔 [m] | 0.3 |
| `search_radius` | `double` | 从轨迹搜索点云的距离 [m] | 5.0 |

<a id="assumptions-known-limits"></a>

## 假设与已知限制

要正确执行碰撞检查，需要获得合理的预测轨迹和无噪声的障碍物点云。
