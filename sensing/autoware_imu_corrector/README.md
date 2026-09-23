# autoware_imu_corrector

## imu_corrector

`imu_corrector_node` 是校正 IMU 数据的节点。

1. 读取参数，校正横摆角速度偏置 $b$。
2. 读取参数，校正横摆角速度标准差 $\sigma$。
3. 以 NDT 位姿为真值，校正横摆角速度比例因子 $s$。

数学上，假设满足以下关系式：

$$
\tilde{\omega}(t) = s(t) * \omega(t) + b(t) + n(t)
$$

其中，$\tilde{\omega}$ 表示观测角速度，$\omega$ 表示真实角速度，$b$ 表示偏置，$s$ 表示比例因子，$n$ 表示高斯噪声。
同时假设 $n\sim\mathcal{N}(0, \sigma^2)$。

<!-- Use the value estimated by [deviation_estimator](https://github.com/autowarefoundation/autoware_tools/tree/main/localization/deviation_estimation_tools) as the parameters for this node. -->

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| -------- | ----------------------- | ------------ |
| `~input` | `sensor_msgs::msg::Imu` | 原始 IMU 数据 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| --------- | ----------------------- | ------------------ |
| `~output` | `sensor_msgs::msg::Imu` | 校正后的 IMU 数据 |

<a id="parameters"></a>

### 参数

| 名称 | 类型 | 说明 |
| ---------------------------- | ------ | ------------------------------------------------ |
| `angular_velocity_offset_x` | double | imu_link 坐标系中的滚转角速度偏置 [rad/s] |
| `angular_velocity_offset_y` | double | imu_link 坐标系中的俯仰角速度偏置 [rad/s] |
| `angular_velocity_offset_z` | double | imu_link 坐标系中的横摆角速度偏置 [rad/s] |
| `angular_velocity_stddev_xx` | double | imu_link 坐标系中的滚转角速度标准差 [rad/s] |
| `angular_velocity_stddev_yy` | double | imu_link 坐标系中的俯仰角速度标准差 [rad/s] |
| `angular_velocity_stddev_zz` | double | imu_link 坐标系中的横摆角速度标准差 [rad/s] |
| `acceleration_stddev` | double | imu_link 坐标系中的加速度标准差 [m/s^2] |

**注意：** 角速度偏置值会引入固定补偿，陀螺仪偏置估计不会考虑该补偿。如果同时启用 `on_off_correction.correct_for_dynamic_bias` 和 `on_off_correction.correct_for_static_bias` 标志，则会自动禁用 `correct_for_static_bias`，以避免校正错误。

## gyro_bias_estimator

`gyro_bias_validator` 是验证陀螺仪偏置的节点。它订阅 `sensor_msgs::msg::Imu` 话题，并验证陀螺仪偏置是否处于指定范围内。

注意，该节点仅在车辆停止时对陀螺仪数据求平均，以计算偏置。

<a id="input_1"></a>

### 输入

| 名称 | 类型 | 说明 |
| ------------------ | ----------------------------------------------- | ---------------- |
| `~/input/imu_raw` | `sensor_msgs::msg::Imu` | **原始** IMU 数据 |
| `~/input/pose` | `geometry_msgs::msg::PoseWithCovarianceStamped` | NDT 位姿 |
| `~/input/odometry` | `nav_msgs::msg::Odometry` | 里程计数据 |

注意，这里假定输入位姿足够准确。例如，使用 NDT 时，假定 NDT 已正确收敛。

目前，Autoware 的 `pose_source` 可以使用 NDT 之外的方法，但精度较低的方法不适合用于 IMU 偏置估计。

将来，如果能妥善处理位姿误差，利用 NDT 估计的 IMU 偏置除了用于验证，还可能用于在线标定。

比例因子估计使用扩展卡尔曼滤波器（EKF）。以 NDT 位姿为真值，并假定其精度足以保证对正确比例因子观测的长期收敛。EKF 滤波器包含初始化检查，需要连续 N 个样本位于范围内（`samples_in_bounds_to_init`）才能初始化；否则会重试，达到 `max_reinitialization_retries` 设定的次数后触发错误。

<a id="output_1"></a>

### 输出

| 名称 | 类型 | 说明 |
| --------------------- | ------------------------------------ | ------------------------------------------- |
| `~/output/gyro_bias` | `geometry_msgs::msg::Vector3Stamped` | 陀螺仪偏置 [rad/s] |
| `~/output/gyro_scale` | `geometry_msgs::msg::Vector3Stamped` | 估计的陀螺仪比例因子 [无量纲] |

<a id="parameters-bias-estimation"></a>

### 参数（偏置估计）

注意，该节点还使用 `imu_corrector.param.yaml` 中的 `angular_velocity_offset_x`、`angular_velocity_offset_y`、`angular_velocity_offset_z` 参数。

| 名称 | 类型 | 说明 |
| ------------------------------------- | ------ | ------------------------------------------------------------------------------------------- |
| `gyro_bias_threshold` | double | 陀螺仪偏置阈值 [rad/s] |
| `timer_callback_interval_sec` | double | 定时器回调间隔 [sec] |
| `diagnostics_updater_interval_sec` | double | 诊断更新周期 [sec] |
| `straight_motion_ang_vel_upper_limit` | double | 横摆角速度上限，超过该值则不视为直线运动 [rad/s] |

<a id="parameters-scale-estimation"></a>

### 参数（比例因子估计）

| 名称 | 类型 | 说明 |
| ---------------------------------------- | ------ | ---------------------------------------------------------------------- |
| `estimate_scale_init` | double | 比例因子估计初始值 |
| `min_allowed_scale` | double | 允许的最小比例因子 |
| `max_allowed_scale` | double | 允许的最大比例因子 |
| `threshold_to_estimate_scale` | double | 估计比例因子所需的最小横摆角速度 |
| `percentage_scale_rate_allow_correct` | double | 校正时相对于当前比例因子所允许的变化百分比 |
| `alpha` | double | 比例因子滤波系数（互补滤波器） |
| `delay_gyro_ms` | int | 应用于陀螺仪数据的延迟，单位为毫秒 |
| `samples_filter_pose_rate` | int | 位姿变化率滤波的样本数 |
| `samples_filter_gyro_rate` | int | 陀螺仪角速度滤波的样本数 |
| `alpha_gyro` | double | 陀螺仪角速度滤波系数 |
| `buffer_size_gyro` | int | 陀螺仪数据缓冲区大小 |
| `alpha_ndt_rate` | double | NDT 角速度滤波系数 |
| `ekf_rate.max_variance_p` | double | EKF 角速度估计允许的最大方差 |
| `ekf_rate.variance_p_after` | double | EKF 角速度估计初始化后的方差 |
| `ekf_rate.process_noise_q` | double | EKF 角速度估计的过程噪声 |
| `ekf_rate.process_noise_q_after` | double | EKF 角速度估计初始化后的过程噪声 |
| `ekf_rate.measurement_noise_r` | double | EKF 角速度估计的测量噪声 |
| `ekf_rate.measurement_noise_r_after` | double | EKF 角速度估计初始化后的测量噪声 |
| `ekf_rate.samples_to_init` | int | 初始化 EKF 角速度估计所需的样本数 |
| `ekf_rate.min_covariance` | double | EKF 角速度估计的最小协方差 |
| `ekf_angle.process_noise_q_angle` | double | EKF 角度估计的过程噪声 |
| `ekf_angle.variance_p_angle` | double | EKF 角度估计的初始方差 |
| `ekf_angle.measurement_noise_r_angle` | double | EKF 角度估计的测量噪声 |
| `ekf_angle.min_covariance_angle` | double | EKF 角度估计的最小协方差 |
| `ekf_angle.decay_coefficient` | double | EKF 角度估计的衰减系数 |
| `ekf_angle.samples_in_bounds_to_init` | int | 初始化 EKF 角度估计所需的范围内样本数 |
| `ekf_angle.max_reinitialization_retries` | int | EKF 角度估计重新初始化的最大重试次数 |

<a id="imu-scalebias-injection"></a>

## IMU 比例因子／偏置注入

为测试陀螺仪比例因子和偏置的估计结果，可以通过以下参数选择向原始 IMU 数据中注入人为设置的比例因子和偏置。可以通过输出数据观察 IMU 比例因子的影响，并将该输出重映射为 'imu_corrector' 的输入。

<a id="output_2"></a>

### 输出

| 名称 | 类型 | 说明 |
| --------------------- | ----------------------- | ------------------------------- |
| `~/output/imu_scaled` | `sensor_msgs::msg::Imu` | 比例因子校正后的 IMU 数据 |

<a id="parameters_1"></a>

### 参数

| 名称 | 类型 | 说明 |
| ------------------ | ------ | ------------------------------------------------------------------- |
| `modify_imu_scale` | bool | 启用或禁用比例因子注入 |
| `scale_on_purpose` | double | 注入的比例因子值 |
| `bias_on_purpose` | double | 注入的偏置值 |
| `drift_scale` | double | 每次循环向比例因子增加的值，用于模拟比例因子漂移 |
| `drift_bias` | double | 每次循环向偏置增加的值，用于模拟偏置漂移 |

<a id="imu-correction-control"></a>

## IMU 校正控制

这些参数控制陀螺仪偏置和比例因子的校正方式。**同一时间只应启用一种偏置校正方法。**  
如果两个标志均启用，**静态偏置校正会自动禁用**。

<a id="static-bias-correction-correct_for_static_bias"></a>

### 静态偏置校正（`correct_for_static_bias`）

- 偏置值必须预先计算，并存储在配置文件的以下参数中：
  `angular_velocity_offset_[x, y, z]`
- 将这些偏置直接应用于原始陀螺仪数据：
  `~/input/imu_raw`
  -仅应在陀螺仪偏置不随时间变化时使用。

<a id="dynamic-bias-correction-correct_for_dynamic_bias"></a>

### 动态偏置校正（`correct_for_dynamic_bias`）

- 使用以下话题发布的偏置估计值：
  `~/output/gyro_bias`
- 借助以下里程计数据估计偏置：
  `~/input/odometry`
- 将估计的偏置应用于原始陀螺仪数据，以进行校正：
  `~/input/imu_raw`
  -应在陀螺仪偏置随时间变化且能获取里程计数据时使用。

<a id="parameters_2"></a>

### 参数

| 名称 | 类型 | 说明 |
| -------------------------------------------- | ---- | ---------------------------------------------------------- |
| `on_off_correction.correct_for_static_bias` | bool | 启用或禁用静态偏置校正（默认：true） |
| `on_off_correction.correct_for_dynamic_bias` | bool | 启用或禁用动态偏置校正（默认：false） |
| `on_off_correction.correct_for_scale` | bool | 启用或禁用比例因子校正（默认：false） |
