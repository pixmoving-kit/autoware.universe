# bytetrack

<a id="purpose"></a>

## 用途

核心算法 `ByteTrack` 主要用于多目标跟踪。
该算法会关联几乎所有检测框，包括检测分数较低的框，
因此使用它有望减少漏检数量。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="cite"></a>

### 引用

<!-- cspell: ignore Yifu Peize Jiang Dongdong Fucheng Weng Zehuan Xinggang -->

- Yifu Zhang, Peize Sun, Yi Jiang, Dongdong Yu, Fucheng Weng, Zehuan Yuan, Ping Luo, Wenyu Liu, and Xinggang Wang,
  "ByteTrack: Multi-Object Tracking by Associating Every Detection Box", in the proc. of the ECCV
  2022, [[ref](https://arxiv.org/abs/2110.06864)]
- 此功能包由[此仓库](https://github.com/ifzhang/ByteTrack/tree/main/deploy/TensorRT/cpp) 移植到 Autoware
  （ByteTrack 作者提供的 C++ 实现）

<a id="2d-tracking-modification-from-original-codes"></a>

### 相较原始代码的 2D 跟踪修改

论文仅说明 2D 跟踪算法采用简单的卡尔曼滤波器。
原始代码使用 `top-left-corner`、`aspect ratio` 和 `size` 作为状态向量。

由于遮挡可能改变宽高比，这种方式有时不稳定。
因此我们使用 `top-left` 和 `size` 作为状态向量。

可通过 `config/bytetrack_node.param.yaml` 中的参数控制卡尔曼滤波器设置。

<a id="inputs-outputs"></a>

## 输入／输出

### bytetrack_node

<a id="input"></a>

#### 输入

| 名称      | 类型                                               | 说明                                 |
| --------- | -------------------------------------------------- | ------------------------------------------- |
| `in/rect` | `tier4_perception_msgs/DetectedObjectsWithFeature` | 带有 2D 包围框的检测目标 |

<a id="output"></a>

#### 输出

| 名称                     | 类型                                               | 说明                                               |
| ------------------------ | -------------------------------------------------- | --------------------------------------------------------- |
| `out/objects`            | `tier4_perception_msgs/DetectedObjectsWithFeature` | 带有 2D 包围框的检测目标               |
| `out/objects/debug/uuid` | `tier4_perception_msgs/DynamicObjectArray`         | 每个目标的通用唯一标识符（UUID） |

### bytetrack_visualizer

<a id="input_1"></a>

#### 输入

| 名称       | 类型                                                 | 说明                                               |
| ---------- | ---------------------------------------------------- | --------------------------------------------------------- |
| `in/image` | `sensor_msgs/Image` or `sensor_msgs/CompressedImage` | 执行目标检测的输入图像    |
| `in/rect`  | `tier4_perception_msgs/DetectedObjectsWithFeature`   | 带有 2D 包围框的检测目标               |
| `in/uuid`  | `tier4_perception_msgs/DynamicObjectArray`           | 每个目标的通用唯一标识符（UUID） |

<a id="output_1"></a>

#### 输出

| 名称        | 类型                | 说明                                                       |
| ----------- | ------------------- | ----------------------------------------------------------------- |
| `out/image` | `sensor_msgs/Image` | 绘制了检测包围框及其 UUID 的图像 |

<a id="parameters"></a>

## 参数

### bytetrack_node

| 名称                  | 类型 | 默认值 | 说明                                              |
| --------------------- | ---- | ------------- | -------------------------------------------------------- |
| `track_buffer_length` | int  | 30            | 将轨迹片段视为丢失的帧数 |

### bytetrack_visualizer

| 名称      | 类型 | 默认值 | 说明                                                                                   |
| --------- | ---- | ------------- | --------------------------------------------------------------------------------------------- |
| `use_raw` | bool | false         | 控制节点在 `sensor_msgs/Image` 与 `sensor_msgs/CompressedImage` 输入之间切换的标志 |

<a id="assumptionsknown-limits"></a>

## 假设与已知限制

<a id="reference-repositories"></a>

## 参考仓库

- <https://github.com/ifzhang/ByteTrack>

## License

The codes under the `lib` directory are copied from [the original codes](https://github.com/ifzhang/ByteTrack/tree/72ca8b45d36caf5a39e949c6aa815d9abffd1ab5/deploy/TensorRT/cpp) and modified.
The original codes belong to the MIT license stated as follows, while this ported packages are provided with Apache License 2.0:

> MIT License
>
> Copyright (c) 2021 Yifu Zhang
>
> Permission is hereby granted, free of charge, to any person obtaining a copy
> of this software and associated documentation files (the "Software"), to deal
> in the Software without restriction, including without limitation the rights
> to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
> copies of the Software, and to permit persons to whom the Software is
> furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in all
> copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
> IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
> FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
> AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
> LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
> OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
> SOFTWARE.
