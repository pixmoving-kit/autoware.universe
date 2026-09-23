# image_diagnostics

<a id="purpose"></a>

## 用途

`image_diagnostics` 是检查输入原始图像状态的节点。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

下图展示了图像诊断节点的流程。每幅图像都被划分为若干小块，以评估各块的状态。

![图像诊断流程图](./image/image_diagnostics_overview.svg)

每个图像小块的状态按下图进行评估。

![图像块状态决策树](./image/block_state_decision.svg)

评估完所有图像块后，按下图汇总整幅图像的状态。

![整幅图像状态决策树](./image/image_status_decision.svg)

<a id="inputs-outputs"></a>

## 输入／输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| ----------------- | ------------------------- | ----------- |
| `input/raw_image` | `sensor_msgs::msg::Image` | 原始图像 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ----------------------------------- | ------------------------------------------------- | ------------------------------------- |
| `image_diag/debug/gray_image` | `sensor_msgs::msg::Image` | 灰度图像 |
| `image_diag/debug/dft_image` | `sensor_msgs::msg::Image` | 离散傅里叶变换图像 |
| `image_diag/debug/diag_block_image` | `sensor_msgs::msg::Image` | 各图像块状态的彩色显示 |
| `image_diag/image_state_diag` | `autoware_internal_debug_msgs::msg::Int32Stamped` | 图像诊断状态值 |
| `/diagnostics` | `diagnostic_msgs::msg::DiagnosticArray` | 诊断信息 |

<a id="parameters"></a>

## 参数

<a id="assumptions-known-limits"></a>

## 前提假设／已知限制

- 这是图像诊断的概念验证实现，算法仍在进一步改进中。

<a id="optional-error-detection-and-handling"></a>

## （可选）错误检测与处理

<a id="optional-performance-characterization"></a>

## （可选）性能特征

<a id="optional-referencesexternal-links"></a>

## （可选）参考资料／外部链接

<a id="optional-future-extensions-unimplemented-parts"></a>

## （可选）后续扩展／尚未实现的部分

- 考虑更具体的图像畸变／遮挡类型，例如雨滴或灰尘。

- 从光学角度考虑雾天或雨天条件下的能见度下降。
