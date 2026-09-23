<a id="perception-evaluator"></a>

# 感知评估器

用于评估感知系统输出的节点。

<a id="purpose"></a>

## 用途

本模块无须标注即可评估感知结果的准确性。它能检查性能，并评估几秒前的结果，从而支持在线运行。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

评估指标如下：

- predicted_path_deviation
- predicted_path_deviation_variance
- lateral_deviation
- yaw_deviation
- yaw_rate
- total_objects_count
- average_objects_count
- interval_objects_count

<a id="predicted-path-deviation-predicted-path-deviation-variance"></a>

### 预测路径偏差与预测路径偏差方差

将过去物体的预测路径与其实际行驶路径比较，确定**运动物体**的偏差。对每个物体，计算指定时间步内预测路径点与实际路径对应点之间的平均距离，即平均位移误差（ADE）。被评估的目标物体是 $T_N$ 秒前的物体，其中 $T_N$ 是预测时域 $[T_1, T_2, ..., T_N]$ 的最大值。

> [!NOTE]
> 所有指标均以 $T_N$ 秒前的物体为目标，以统一各指标的目标物体时间。

![各物体的路径偏差](./images/path_deviation_each_object.drawio.svg)

$$
\begin{align}
n_{points} = T / dt \\
ADE = \Sigma_{i=1}^{n_{points}} d_i / n_{points}　\\
Var = \Sigma_{i=1}^{n_{points}} (d_i - ADE)^2 / n_{points}
\end{align}
$$

- $n_{points}$：预测路径中的点数。
- $T$：预测评估的时间范围。
- $dt$：预测路径的时间间隔。
- $d_i$：路径点 $i$ 处预测路径与实际行驶路径之间的距离。
- $ADE$：目标物体预测路径的平均偏差。
- $Var$：目标物体预测路径偏差的方差。

最终的预测路径偏差指标按同类物体汇总，计算各物体预测路径平均偏差的均值、最大值和最小值。

![路径偏差](./images/path_deviation.drawio.svg)

$$
\begin{align}
ADE_{mean} = \Sigma_{j=1}^{n_{objects}} ADE_j / n_{objects} \\
ADE_{max} = max(ADE_j) \\
ADE_{min} = min(ADE_j)
\end{align}
$$

$$
\begin{align}
Var_{mean} = \Sigma_{j=1}^{n_{objects}} Var_j / n_{objects} \\
Var_{max} = max(Var_j) \\
Var_{min} = min(Var_j)
\end{align}
$$

- $n_{objects}$：物体数量。
- $ADE_{mean}$：所有物体预测路径平均偏差的均值。
- $ADE_{max}$：所有物体预测路径平均偏差的最大值。
- $ADE_{min}$：所有物体预测路径平均偏差的最小值。
- $Var_{mean}$：所有物体预测路径偏差方差的均值。
- $Var_{max}$：所有物体预测路径偏差方差的最大值。
- $Var_{min}$：所有物体预测路径偏差方差的最小值。

实际指标名称由物体类别和预测时域决定，例如 `predicted_path_deviation_variance_CAR_5.00`。

<a id="lateral-deviation"></a>

### 横向偏差

计算平滑后的实际行驶轨迹与感知位置之间的横向偏差，评估**运动物体**横向位置识别的稳定性。实际行驶轨迹通过中心移动平均滤波器平滑，窗口大小由 `smoothing_window_size` 指定。将平滑轨迹与时间戳为 $T=T_n$ 秒前的物体感知位置比较，计算横向偏差。静止物体的平滑行驶轨迹不稳定，因此不计算此指标。

![横向偏差](./images/lateral_deviation.drawio.svg)

<a id="yaw-deviation"></a>

### 偏航角偏差

对**运动物体**，计算过去物体识别出的偏航角与平滑实际行驶轨迹方位角之间的偏差。实际行驶轨迹通过中心移动平均滤波器平滑，窗口大小由 `smoothing_window_size` 指定。将平滑轨迹的偏航方位角与时间戳为 $T=T_n$ 秒前的物体感知朝向比较，计算偏航角偏差。
静止物体的平滑行驶轨迹不稳定，因此不计算此指标。

![偏航角偏差](./images/yaw_deviation.drawio.svg)

<a id="yaw-rate"></a>

### 偏航角速度

根据物体相较于前一时间步的偏航角变化计算偏航角速度。此指标针对**静止物体**，评估偏航角速度识别的稳定性。通过比较过去物体的偏航角与上一周期收到的物体偏航角进行计算。这里的 t2 是 $T_n$ 秒前的时间戳。

![偏航角速度](./images/yaw_rate.drawio.svg)

<a id="object-counts"></a>

### 物体计数

统计指定检测范围内各类别的检测数量。这些指标针对最新物体，而非过去的物体。

![检测计数](./images/detection_counts.drawio.svg)

图中的范围 $R$ 由半径列表（例如 $r_1, r_2, \ldots$）与高度列表（例如 $h_1, h_2, \ldots$）组合确定。
例如：

- 范围 $R = (r_1, h_1)$ 内的 CAR 数量为 1。
- 范围 $R = (r_1, h_2)$ 内的 CAR 数量为 2。
- 范围 $R = (r_2, h_1)$ 内的 CAR 数量为 3。
- 范围 $R = (r_2, h_2)$ 内的 CAR 数量为 4。

<a id="total-object-count"></a>

#### 物体总数

统计指定检测范围内各类别的不同物体数量，计算如下：

$$
\begin{align}
\text{Total Object Count (Class, Range)} = \left| \bigcup_{t=0}^{T_{\text{now}}} \{ \text{uuid} \mid \text{class}(t, \text{uuid}) = C \wedge \text{position}(t, \text{uuid}) \in R \} \right|
\end{align}
$$

其中：

- $\bigcup$ 表示从 $t = 0$ 到 $T_{\text{now}}$ 的所有帧的并集，确保每个 uuid 只计数一次。
- $\text{class}(t, \text{uuid}) = C$ 表示时刻 $t$ 对应 uuid 的物体属于类别 $C$。
- $\text{position}(t, \text{uuid}) \in R$ 表示时刻 $t$ 对应 uuid 的物体位于指定范围 $R$ 内。
- $\left| \{ \ldots \} \right|$ 表示集合基数，即所有考察时刻中满足类别与范围条件的不同 uuid 数量。

<a id="average-object-count"></a>

#### 平均物体数量

统计指定检测范围内各类别的平均物体数量。此指标衡量每帧检测到多少物体，不考虑 uuid。计算如下：

$$
\begin{align}
\text{Average Object Count (Class, Range)} = \frac{1}{N} \sum_{t=0}^{T_{\text{now}}} \left| \{ \text{object} \mid \text{class}(t, \text{object}) = C \wedge \text{position}(t, \text{object}) \in R \} \right|
\end{align}
$$

其中：

- $N$ 表示截至 $T\_{\text{now}}$ 的时间段内总帧数（时间段具体由 `detection_count_purge_seconds` 确定）。
- $text{object}$ 表示时刻 $t$ 满足类别与范围条件的物体数量。

<a id="interval-object-count"></a>

#### 时间窗口内的物体数量

统计最近 `objects_count_window_seconds` 内，指定检测范围中各类别的平均物体数量。此指标衡量每帧检测到多少物体，不考虑 uuid。计算如下：

$$
\begin{align}
\text{Interval Object Count (Class, Range)} = \frac{1}{W} \sum_{t=T_{\text{now}} - T_W}^{T_{\text{now}}} \left| \{ \text{object} \mid \text{class}(t, \text{object}) = C \wedge \text{position}(t, \text{object}) \in R \} \right|
\end{align}
$$

其中：

- $W$ 表示最近 `objects_count_window_seconds` 内的总帧数。
- $T_W$ 表示时间窗口 `objects_count_window_seconds`。

<a id="inputs-outputs"></a>

## 输入／输出

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------------------------------- | ----------------------------------------------- |
| `~/input/objects` | `autoware_perception_msgs::msg::PredictedObjects` | 待评估的预测物体。 |
| `~/metrics` | `tier4_metric_msgs::msg::MetricArray` | 感知精度的指标信息。 |
| `~/markers` | `visualization_msgs::msg::MarkerArray` | 用于调试和可视化的标记。 |

<a id="parameters"></a>

## 参数

| 名称 | 类型 | 说明 |
| ------------------------------------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `selected_metrics` | List | 待评估指标，例如横向偏差、偏航角偏差和预测路径偏差。 |
| `smoothing_window_size` | Integer | 路径平滑的窗口大小，应为奇数。 |
| `prediction_time_horizons` | list[double] | 预测评估的时间范围，单位秒。 |
| `stopped_velocity_threshold` | double | 判断车辆是否停止的速度阈值。 |
| `detection_radius_list` | list[double] | 待评估物体的检测半径（仅用于物体计数）。 |
| `detection_height_list` | list[double] | 待评估物体的检测高度（仅用于物体计数）。 |
| `detection_count_purge_seconds` | double | 清理物体检测计数的时间窗口。 |
| `objects_count_window_seconds` | double | 保留物体检测计数的时间窗口，此窗口内的检测数量存储在 `detection_count_vector_` 中。 |
| `target_object.*.check_lateral_deviation` | bool | 是否检查指定物体类型（汽车、卡车等）的横向偏差。 |
| `target_object.*.check_yaw_deviation` | bool | 是否检查指定物体类型（汽车、卡车等）的偏航角偏差。 |
| `target_object.*.check_predicted_path_deviation` | bool | 是否检查指定物体类型（汽车、卡车等）的预测路径偏差。 |
| `target_object.*.check_yaw_rate` | bool | 是否检查指定物体类型（汽车、卡车等）的偏航角速度。 |
| `target_object.*.check_total_objects_count` | bool | 是否统计指定物体类型（汽车、卡车等）的物体总数。 |
| `target_object.*.check_average_objects_count` | bool | 是否统计指定物体类型（汽车、卡车等）的平均物体数量。 |
| `target_object.*.check_interval_average_objects_count` | bool | 是否统计指定物体类型（汽车、卡车等）的时间窗口平均物体数量。 |
| `debug_marker.*` | bool | 标记可视化的调试参数（历史路径、预测路径等）。 |

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

假设 PredictedObjects 的当前位置具有合理精度。

<a id="future-extensions-unimplemented-parts"></a>

## 后续扩展与尚未实现的部分

- 各类别识别数量的增长率。
- 物理行为异常物体的指标（例如穿过围栏）。
- 物体分裂的指标。
- 通常静止的物体却发生移动的问题指标。
- 物体消失指标。
