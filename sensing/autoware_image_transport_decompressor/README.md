# image_transport_decompressor

<a id="purpose"></a>

## 用途

`image_transport_decompressor` 是用于解压图像的节点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| -------------------------- | ----------------------------------- | ---------------- |
| `~/input/compressed_image` | `sensor_msgs::msg::CompressedImage` | 压缩图像 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| -------------------- | ------------------------- | ------------------ |
| `~/output/raw_image` | `sensor_msgs::msg::Image` | 解压后的图像 |

<a id="parameters"></a>

## 参数

{{ json_to_markdown("sensing/autoware_image_transport_decompressor/schema/image_transport_decompressor.schema.json") }}

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

支持的场景是 8 位 RGB 或 BGR 相机，并将 `encoding` 设置为 `rgb8` 或 `bgr8`。其他情况
仍然会解码并发布，但结果会出现以下两类错误之一，因为无论相机发送什么数据，
解码结果始终为三通道的 8 位 BGR。

- **其他相机，且 `encoding: rgb8` 或 `bgr8`。** 消息格式正确，因此使用该消息的节点
  不会报错，但像素值已不再是相机实际测得的值。
- **其他任何 `encoding`。** 发布消息的 `encoding` 仍使用发送方指定的值，但数据内容
  已与该编码不符。对于 16 位编码，`cv_bridge` 会抛出异常；对于其他编码，则会静默地错误解读数据。

具体如下：

| 相机图像 | `encoding: rgb8` 或 `bgr8` | 其他任何 `encoding` |
| ------------------ | ---------------------------------- | ----------------------------------- |
| `rgb8`, `bgr8` | 与发送内容一致 | 与发送内容一致 |
| `rgba8`, `bgra8` | 丢弃 alpha 通道 | alpha 通道替换为 255 |
| `mono8` | 将单通道复制为三通道 | **三通道数据被标为 `mono8`** |
| `mono16` | 取高 8 位并复制为三通道 | **三通道数据被标为 `mono16`** |
| `rgb16`, `bgr16` | 取高 8 位 | **8 位采样值被标为 16 位** |
| `rgba16`, `bgra16` | 取高 8 位并丢弃 alpha 通道 | **8 位采样值被标为 16 位** |
| `bayer_rggb8` | 复制拜耳模式，未恢复颜色 | **三通道数据被标为拜耳格式** |
| `yuv422` | 与发送内容一致 | **BGR 像素被标为 `yuv422`** |

加粗的单元格属于第二种情况：数据内容与其发布时声明的编码不符。

无法解码的数据会被丢弃，并通过 `RCLCPP_ERROR` 报告。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分
