<p align="center">
  <a href="https://proxima-ai-tech.com/">
    <img width="500px" src="./images/proxima_logo.png">
  </a>
</p>
<!-- cspell: ignore numba ipynb LSTM -->

<a id="smart-mpc-trajectory-follower"></a>

# Smart MPC 轨迹跟踪器

Smart MPC（模型预测控制）是一种结合模型预测控制与机器学习的控制算法。它继承了模型预测控制的优点，同时利用基于机器学习的数据驱动方法解决建模困难的问题。

只要能够准备好数据采集环境，这项技术就能使原本实现成本较高的模型预测控制更容易投入使用。

<p align="center">
  <a href="https://youtu.be/j7bgK8m4-zg?si=p3ipJQy_p-5AJHOP)">
    <image width="700px" src="./images/autoware_smart_mpc.png">
  </a>
</p>

<a id="requirements"></a>

## 需求

建议在虚拟环境中安装这些依赖。

```bash
pip3 install numba==0.58.1 GPy
```

<a id="provided-features"></a>

## 提供的功能

本功能包提供用于路径跟踪控制的 Smart MPC 逻辑，以及学习和评估机制。下文介绍这些功能。

<a id="trajectory-following-control-based-on-ilqrmppi"></a>

### 基于 iLQR/MPPI 的轨迹跟踪控制

控制模式可选择 "ilqr"、"mppi" 或 "mppi_ilqr"，通过 [mpc_param.yaml](./autoware_smart_mpc_trajectory_follower/param/mpc_param.yaml) 中的 `mpc_parameter:system:mode` 设置。
在 "mppi_ilqr" 模式下，MPPI 的解用作 iLQR 的初始值。

> [!NOTE]
> 默认设置下，由于采样数量不足，"mppi" 模式的性能受限。目前正在通过引入 GPU 支持来解决此问题。

要运行仿真，请执行以下命令：

```bash
ros2 launch autoware_launch planning_simulator.launch.xml map_path:=$HOME/autoware_data/maps/sample-map-planning vehicle_model:=sample_vehicle sensor_model:=sample_sensor_kit trajectory_follower_mode:=smart_mpc_trajectory_follower
```

> [!NOTE]
> 使用 [nominal_param.yaml](./autoware_smart_mpc_trajectory_follower/param/nominal_param.yaml) 中设置的标称模型运行时，请将 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:control_application:use_trained_model` 设为 `false`。使用训练后的模型运行时，将 `trained_model_parameter:control_application:use_trained_model` 设为 `true`，但必须先按以下流程生成训练模型。

<a id="training-of-model-and-reflection-in-control"></a>

### 模型训练与控制应用

要获取训练数据，请启动 autoware，进行驾驶，并使用以下命令录制 rosbag 数据。

```bash
ros2 bag record /localization/kinematic_state /localization/acceleration /vehicle/status/steering_status /control/command/control_cmd /control/trajectory_follower/control_cmd /control/trajectory_follower/lane_departure_checker_node/debug/deviation/lateral /control/trajectory_follower/lane_departure_checker_node/debug/deviation/yaw /system/operation_mode/state /vehicle/status/control_mode /sensing/imu/imu_data /debug_mpc_x_des /debug_mpc_y_des /debug_mpc_v_des /debug_mpc_yaw_des /debug_mpc_acc_des /debug_mpc_steer_des /debug_mpc_X_des_converted /debug_mpc_x_current /debug_mpc_error_prediction /debug_mpc_max_trajectory_err /debug_mpc_emergency_stop_mode /debug_mpc_goal_stop_mode /debug_mpc_total_ctrl_time /debug_mpc_calc_u_opt_time
```

将 [rosbag2.bash](./autoware_smart_mpc_trajectory_follower/training_and_data_check/rosbag2.bash) 移至上述录制的 rosbag 目录，并在该目录下执行以下命令。

```bash
bash rosbag2.bash
```

这会将 rosbag 数据转换为用于模型训练的 CSV 格式。

> [!NOTE]
> 请注意，运行时会自动打开大量终端，rosbag 数据转换完成后会自动关闭。
> 从开始此过程直到所有终端关闭，autoware 都不应运行。

也可以在 Python 环境中执行以下命令，得到相同结果：

```python
from autoware_smart_mpc_trajectory_follower.training_and_data_check import train_drive_NN_model
model_trainer = train_drive_NN_model.train_drive_NN_model()
model_trainer.transform_rosbag_to_csv(rosbag_dir)
```

其中，`rosbag_dir` 表示 rosbag 目录。
此时，首先会自动删除 `rosbag_dir` 中的所有 CSV 文件。

接下来介绍模型训练方法。
如果将 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:memory_for_training:use_memory_for_training` 设为 `true`，则训练包含 LSTM 的模型；设为 `false` 时，则训练不包含 LSTM 的模型。
使用 LSTM 时，根据历史时间序列数据更新细胞状态和隐藏状态，并将其用于预测。

给定用于训练和验证的 rosbag 目录路径 `dir_0`、`dir_1`、`dir_2`、……、`dir_val_0`、`dir_val_1`、`dir_val_2`、……，以及保存模型的目录 `save_dir`，可以在 Python 环境中按以下方式保存模型：

```python
from autoware_smart_mpc_trajectory_follower.training_and_data_check import train_drive_NN_model
model_trainer = train_drive_NN_model.train_drive_NN_model()
model_trainer.add_data_from_csv(dir_0, add_mode="as_train")
model_trainer.add_data_from_csv(dir_1, add_mode="as_train")
model_trainer.add_data_from_csv(dir_2, add_mode="as_train")
...
model_trainer.add_data_from_csv(dir_val_0, add_mode="as_val")
model_trainer.add_data_from_csv(dir_val_1, add_mode="as_val")
model_trainer.add_data_from_csv(dir_val_2, add_mode="as_val")
...
model_trainer.get_trained_model()
model_trainer.save_models(save_dir)
```

如果未指定 `add_mode`，或未添加验证数据，则会拆分训练数据，分别用于训练和验证。

执行多项式回归后，可以按以下方式针对残差训练神经网络：

```python
model_trainer.get_trained_model(use_polynomial_reg=True)
```

> [!NOTE]
> 默认设置下，使用若干预先选定的多项式进行回归。
> 将 get_trained_model 的参数 `use_selected_polynomial=False` 时，可以通过 `deg` 设置使用的多项式最高次数。

如果只进行多项式回归，不使用神经网络模型，请执行以下命令：

```python
model_trainer.get_trained_model(use_polynomial_reg=True,force_NN_model_to_zero=True)
```

将 `save_dir` 中保存的 `model_for_test_drive.pth` 和 `polynomial_reg_info.npz` 移至用户主目录，并将 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:control_application:use_trained_model` 设为 `true`，即可将训练模型用于控制。

<a id="performance-evaluation"></a>

### 性能评估

这里以 sample_vehicle 的实际轴距为 2.79 m、但控制器错误使用 2.0 m 的情况为例，说明如何验证自适应性能。
要让控制器使用 2.0 m 轴距，请将 [nominal_param.yaml](./autoware_smart_mpc_trajectory_follower/param/nominal_param.yaml) 中的 `nominal_parameter:vehicle_info:wheel_base` 设为 2.0，并执行以下命令：

```bash
python3 -m smart_mpc_trajectory_follower.clear_pycache
```

<a id="test-on-autoware"></a>

#### 在 autoware 中测试

要在 autoware 中使用训练前的标称模型进行控制测试，请确认 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:control_application:use_trained_model` 为 `false`，再按“基于 iLQR/MPPI 的轨迹跟踪控制”中的方法启动 autoware。本次测试使用以下路线：

<p><img src="images/test_route.png" width=712pix></p>

按“模型训练与控制应用”中的方法录制 rosbag 并训练模型，将生成的 `model_for_test_drive.pth` 和 `polynomial_reg_info.npz` 移至用户主目录。
示例模型可从 [sample_models/wheel_base_changed](./sample_models/wheel_base_changed/) 获取，它要求 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:memory_for_training:use_memory_for_training` 设为 `true`。

> [!NOTE]
> 虽然训练数据较少，但为简化演示，我们将观察这点数据能够带来多大的性能提升。

要使用此处得到的训练模型控制车辆，请将 `trained_model_parameter:control_application:use_trained_model` 设为 `true`，以相同方式启动 autoware，沿同一路线行驶并录制 rosbag。
行驶完成后，按“模型训练与控制应用”中的方法将 rosbag 文件转换为 CSV 格式。
对标称模型和训练模型的 rosbag 文件 `rosbag_nominal` 与 `rosbag_trained`，分别按以下方式运行 `control/autoware_smart_mpc_trajectory_follower/autoware_smart_mpc_trajectory_follower/training_and_data_check/data_checker.ipynb` 中的 `lateral_error_visualize` 函数，即可绘制横向偏差：

```python
lateral_error_visualize(dir_name=rosbag_nominal,ylim=[-1.2,1.2])
lateral_error_visualize(dir_name=rosbag_trained,ylim=[-1.2,1.2])
```

得到以下结果。

<div style="display: flex; justify-content: center; align-items: center;">
    <img src="images/lateral_error_nominal_model.png">
    <img src="images/lateral_error_trained_model.png">
</div>

<a id="test-on-python-simulator"></a>

#### 在 Python 仿真器中测试

首先，为使 Python 仿真器使用 2.79 m 轴距，创建以下文件，以 `sim_setting.json` 为名保存在 `control/autoware_smart_mpc_trajectory_follower/autoware_smart_mpc_trajectory_follower/python_simulator` 中：

```json
{ "wheel_base": 2.79 }
```

然后进入 `control/autoware_smart_mpc_trajectory_follower/autoware_smart_mpc_trajectory_follower/python_simulator`，执行以下命令，在 Python 仿真器中使用标称控制测试蛇形行驶：

```bash
python3 run_python_simulator.py nominal_test
```

行驶结果保存在 `test_python_nominal_sim` 中。

得到以下结果。

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_nominal_model_wheel_base.png" width="712px">
</p>

上排中间的图表示横向偏差。

执行以下命令，使用纯追踪控制下的“8”字行驶数据进行训练。

要使用“8”字行驶进行训练，并基于得到的模型行驶，请执行以下命令：

```bash
python3 run_python_simulator.py
```

行驶结果保存在 `test_python_trined_sim` 中。

将 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:memory_for_training:use_memory_for_training` 设为 `true` 时，得到以下结果。

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_trained_model_lstm_wheel_base.png" width="712px">
</p>

将 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中的 `trained_model_parameter:memory_for_training:use_memory_for_training` 设为 `false` 时，得到以下结果。

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_trained_model_wheel_base.png" width="712px">
</p>

可以看到，横向偏差显著改善。
但是否使用 LSTM 的行驶差异并不明显。

为观察差异，可以尝试调整 steer_time_delay 等参数。

首先，将 [nominal_param.yaml](./autoware_smart_mpc_trajectory_follower/param/nominal_param.yaml) 中的 `nominal_parameter:vehicle_info:wheel_base` 设为 2.79，恢复标称模型的默认设置，并执行以下命令：

```bash
python3 -m smart_mpc_trajectory_follower.clear_pycache
```

然后按以下方式修改 `sim_setting.json`：

```json
{ "steer_time_delay": 1.01 }
```

这样即可在 `steer_time_delay` 为 1.01 秒的条件下进行实验。

使用标称模型的行驶结果如下：

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_nominal_model_steer_time_delay.png" width="712px">
</p>

使用包含 LSTM 的训练模型的行驶结果如下：

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_trained_model_lstm_steer_time_delay.png" width="712px">
</p>

使用不包含 LSTM 的训练模型的行驶结果如下：

<p style="text-align: center;">
    <img src="images/python_sim_lateral_error_trained_model_steer_time_delay.png" width="712px">
</p>

可以看到，包含 LSTM 的模型性能明显优于不包含 LSTM 的模型。

可传给 Python 仿真器的参数如下。

| 参数 | 类型 | 说明 |
| ------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| steer_bias | float | 转向偏置 [rad] |
| steer_rate_lim | float | 转向角速度限制 [rad/s] |
| vel_rate_lim | float | 加速度限制 [m/s^2] |
| wheel_base | float | 轴距 [m] |
| steer_dead_band | float | 转向死区 [rad] |
| adaptive_gear_ratio_coef | list[float] | 长度为 6 的浮点数列表，指定从轮胎转角到方向盘转角的随车速变化的传动比信息。 |
| acc_time_delay | float | 加速度时间延迟 [s] |
| steer_time_delay | float | 转向时间延迟 [s] |
| acc_time_constant | float | 加速度时间常数 [s] |
| steer_time_constant | float | 转向时间常数 [s] |
| accel_map_scale | float | 放大加速度输入值与实际加速度之间映射失真的参数。<br> 映射信息保存在 `control/autoware_smart_mpc_trajectory_follower/autoware_smart_mpc_trajectory_follower/python_simulator/accel_map.csv` 中。 |
| acc_scaling | float | 加速度缩放系数 |
| steer_scaling | float | 转向缩放系数 |
| vehicle_type | int | 取值为 0 至 4，对应预定义车辆类型。<br> 各车辆类型的说明见下文。 |

例如，要在仿真端设置 0.01 [rad] 的转向偏置和 0.001 [rad] 的转向死区，请按以下方式编辑 `sim_setting.json`。

```json
{ "steer_bias": 0.01, "steer_dead_band": 0.001 }
```

##### vehicle_type_0

此车辆类型与控制中使用的默认车辆类型一致。

| 参数 | 值 |
| ------------------- | ----- |
| wheel_base          | 2.79  |
| acc_time_delay      | 0.1   |
| steer_time_delay    | 0.27  |
| acc_time_constant   | 0.1   |
| steer_time_constant | 0.24  |
| acc_scaling         | 1.0   |

##### vehicle_type_1

此车辆类型面向重型客车。

| 参数 | 值 |
| ------------------- | ----- |
| wheel_base          | 4.76  |
| acc_time_delay      | 1.0   |
| steer_time_delay    | 1.0   |
| acc_time_constant   | 1.0   |
| steer_time_constant | 1.0   |
| acc_scaling         | 0.2   |

##### vehicle_type_2

此车辆类型面向轻型客车。

| 参数 | 值 |
| ------------------- | ----- |
| wheel_base          | 4.76  |
| acc_time_delay      | 0.5   |
| steer_time_delay    | 0.5   |
| acc_time_constant   | 0.5   |
| steer_time_constant | 0.5   |
| acc_scaling         | 0.5   |

##### vehicle_type_3

此车辆类型面向小型车辆。

| 参数 | 值 |
| ------------------- | ----- |
| wheel_base          | 1.335 |
| acc_time_delay      | 0.3   |
| steer_time_delay    | 0.3   |
| acc_time_constant   | 0.3   |
| steer_time_constant | 0.3   |
| acc_scaling         | 1.5   |

##### vehicle_type_4

此车辆类型面向小型机器人。

| 参数 | 值 |
| ------------------- | ----- |
| wheel_base          | 0.395 |
| acc_time_delay      | 0.2   |
| steer_time_delay    | 0.2   |
| acc_time_constant   | 0.2   |
| steer_time_constant | 0.2   |
| acc_scaling         | 1.0   |

<a id="auto-test-on-python-simulator"></a>

#### 在 Python 仿真器中自动测试

本节介绍如何在控制端保持模型参数不变、在仿真端按预设范围改变模型参数，以测试自适应性能。

要在 [run_sim.py](./autoware_smart_mpc_trajectory_follower/python_simulator/run_sim.py) 设定的参数变化范围内进行行驶实验，例如可进入 `control/autoware_smart_mpc_trajectory_follower/autoware_smart_mpc_trajectory_follower/python_simulator`，执行以下命令：

```bash
python3 run_sim.py --param_name steer_bias
```

这里介绍了转向偏置的实验流程，其他参数也可以使用相同方法。

要一次性测试除限制参数之外的所有参数，请执行以下命令：

```bash
python3 run_auto_test.py
```

结果保存在 `auto_test` 目录中。
执行完成后，运行 [plot_auto_test_result.ipynb](./autoware_smart_mpc_trajectory_follower/python_simulator/plot_auto_test_result.ipynb)，得到以下结果：

<p style="text-align: center;">
    <img src="images/proxima_test_result_with_lstm.png" width="712px">
</p>

橙线表示使用纯追踪“8”字行驶数据训练出的中间模型；蓝线表示结合中间模型行驶数据和“8”字行驶数据训练出的最终模型。
大多数情况下性能足够好，但面向重型客车的 `vehicle_type_1` 出现了约 2 m 的横向偏差，效果不理想。

可以在 `run_sim.py` 中设置以下参数：

| 参数 | 类型 | 说明 |
| ------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| USE_TRAINED_MODEL_DIFF | bool | 是否在控制中使用训练模型的导数 |
| DATA_COLLECTION_MODE | DataCollectionMode | 采集训练数据所用的方法。<br> "DataCollectionMode.ff"：使用前馈输入直线行驶。<br> "DataCollectionMode.pp"：使用纯追踪控制进行“8”字行驶。<br> "DataCollectionMode.mpc"：使用 MPC 进行蛇形行驶。 |
| USE_POLYNOMIAL_REGRESSION | bool | 是否在神经网络训练前进行多项式回归 |
| USE_SELECTED_POLYNOMIAL | bool | USE_POLYNOMIAL_REGRESSION 为 True 时，是否仅使用部分预选多项式进行回归。<br> 这些多项式的选择旨在基于车辆标称模型吸收某些参数偏移的影响。 |
| FORCE_NN_MODEL_TO_ZERO | bool | 是否强制将神经网络模型置零（即消除神经网络模型的贡献）。<br> USE_POLYNOMIAL_REGRESSION 为 True 时，将 FORCE_MODEL_TO_ZERO 设为 True，可使控制仅使用多项式回归结果，而不使用神经网络模型。 |
| FIT_INTERCEPT | bool | 多项式回归是否包含偏置。<br> 若为 False，则使用一次或更高次的多项式进行回归。 |
| USE_INTERCEPT | bool | 执行包含偏置的多项式回归时，是否使用所得偏置信息。<br> 仅当 FIT_INTERCEPT 为 True 时有意义。<br> 若为 False，即使回归时包含偏置，也会丢弃多项式回归中的偏置，期望由神经网络模型消除偏置项。 |

> [!NOTE]
> 运行 `run_sim.py` 时，其中设置的 `use_trained_model_diff` 优先于 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 中设置的 `trained_model_parameter:control_application:use_trained_model_diff`。

<a id="kernel-density-estimation-of-pure-pursuit-driving-data"></a>

#### 纯追踪行驶数据的核密度估计

可以使用核密度估计展示纯追踪行驶数据的分布。为此，请运行 [density_estimation.ipynb](./autoware_smart_mpc_trajectory_follower/python_simulator/density_estimation.ipynb)。

密度估计的最小值与行驶结果的横向偏差相关性较低。目前正在开发能够更好预测横向偏差的标量指标。

<a id="change-of-nominal-parameters-and-their-reloading"></a>

## 修改并重新加载标称参数

可以通过编辑 [nominal_param.yaml](./autoware_smart_mpc_trajectory_follower/param/nominal_param.yaml) 修改车辆模型的标称参数。
修改标称参数后，必须运行以下命令删除缓存：

```bash
python3 -m smart_mpc_trajectory_follower.clear_pycache
```

标称参数包括：

| 参数 | 类型 | 说明 |
| ------------------------------------------------ | ----- | ------------------------------ |
| nominal_parameter:vehicle_info:wheel_base | float | 轴距 [m] |
| nominal_parameter:acceleration:acc_time_delay | float | 加速度时间延迟 [s] |
| nominal_parameter:acceleration:acc_time_constant | float | 加速度时间常数 [s] |
| nominal_parameter:steering:steer_time_delay | float | 转向时间延迟 [s] |
| nominal_parameter:steering:steer_time_constant | float | 转向时间常数 [s] |

<a id="change-of-control-parameters-and-their-reloading"></a>

## 修改并重新加载控制参数

可以通过编辑 [mpc_param.yaml](./autoware_smart_mpc_trajectory_follower/param/mpc_param.yaml) 和 [trained_model_param.yaml](./autoware_smart_mpc_trajectory_follower/param/trained_model_param.yaml) 修改控制参数。
可以通过重启 autoware 使参数修改生效，也可以执行以下命令应用修改：

```bash
ros2 topic pub /pympc_reload_mpc_param_trigger std_msgs/msg/String "data: ''" --once
```

主要控制参数如下。

### `mpc_param.yaml`

| 参数 | 类型 | 说明 |
| ------------------------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mpc_parameter:system:mode | str | 控制模式。<br> "ilqr"：iLQR 模式。<br> "mppi"：MPPI 模式。<br> "mppi_ilqr"：使用 MPPI 的解作为 iLQR 初始值。 |
| mpc_parameter:cost_parameters:Q | list[float] | 状态的阶段代价。<br> 长度为 8 的列表，依次为纵向偏差、横向偏差、速度偏差、偏航角偏差、加速度偏差、转向偏差、加速度输入偏差、转向输入偏差的代价权重。 |
| mpc_parameter:cost_parameters:Q_c | list[float] | 状态在下述 timing_Q_c 对应预测步上的代价。<br> 列表各分量的对应关系与 Q 相同。 |
| mpc_parameter:cost_parameters:Q_f | list[float] | 状态的终端代价。<br> 列表各分量的对应关系与 Q 相同。 |
| mpc_parameter:cost_parameters:R | list[float] | 长度为 2 的列表，R[0] 是加速度输入变化率的代价权重，R[1] 是转向输入变化率的代价权重。 |
| mpc_parameter:mpc_setting:timing_Q_c | list[int] | 将状态阶段代价设为 Q_c 的预测步编号。 |
| mpc_parameter:compensation:acc_fb_decay | float | MPC 外部补偿器对观测加速度与预测加速度之差积分时的衰减系数。 |
| mpc_parameter:compensation:acc_fb_gain | float | 加速度补偿增益。 |
| mpc_parameter:compensation:max_error_acc | float | 最大加速度补偿量（m/s^2） |
| mpc_parameter:compensation:steer_fb_decay | float | MPC 外部补偿器对观测转向值与预测转向值之差积分时的衰减系数。 |
| mpc_parameter:compensation:steer_fb_gain | float | 转向补偿增益。 |
| mpc_parameter:compensation:max_error_steer | float | 最大转向补偿量（rad） |

### `trained_model_param.yaml`

| 参数 | 类型 | 说明 |
| ------------------------------------------------------------------- | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| trained_model_parameter:control_application:use_trained_model | bool | 是否在控制中使用训练模型。 |
| trained_model_parameter:control_application:use_trained_model_diff | bool | 是否在控制中使用训练模型的导数。<br> 仅当 use_trained_model 为 True 时有意义；若为 False，则动力学导数使用标称模型，训练模型仅用于预测。 |
| trained_model_parameter:memory_for_training:use_memory_for_training | bool | 是否使用包含 LSTM 的模型进行学习。 |
| trained_model_parameter:memory_for_training:use_memory_diff | bool | 是否在控制中使用相对于 LSTM 上一时刻细胞状态和隐藏状态的导数。 |

<a id="request-to-release-the-slow-stop-mode"></a>

## 请求解除缓慢停车模式

如果预测轨迹与目标轨迹偏差过大，系统会进入缓慢停车模式，车辆停止运动。
要取消缓慢停车模式，使车辆重新具备行驶条件，请执行以下命令：

```bash
ros2 topic pub /pympc_stop_mode_reset_request std_msgs/msg/String "data: ''" --once
```

<a id="limitation"></a>

## 局限性

- 初始位置或姿态与目标相差较大时，可能无法起步。

- 首次控制开始时需要编译 numba 函数，因此规划完成前可能需要一些时间。

- 接近目标点停车时，本控制器会切换到另一种简单控制律，因此除目标点附近外，停车功能可能无法正常工作。如果加速度映射存在明显偏移，也很难停车。

- 如果实际动力学与标称模型偏差过大，例如面向重型客车的 `vehicle_type_1`，可能无法实现良好控制。
