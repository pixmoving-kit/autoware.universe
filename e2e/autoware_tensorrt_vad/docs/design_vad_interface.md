<a id="vadinterface-design"></a>

# VadInterface 设计

- 代码：[vad_interface.cpp](../lib/vad_interface.cpp) [vad_interface.hpp](../src/vad_interface.hpp)

<a id="responsibilities"></a>

## 职责

- 将 `VadInputTopicData` 转换为 `VadInputData`，并将 `VadOutputData` 转换为 `VadOutputTopicData`
  - 通过 [`CoordinateTransformer`](../src/coordinate_transformer.hpp) 管理 TF 查询
    - 通过 TF 缓冲区提供相机坐标系变换（base_link → 相机坐标系）
    - 注意：对于 CARLA，VAD 坐标 = Autoware base_link 坐标（无需转换）
  - 转换输入数据（所有转换器均位于 `vad_interface::` 命名空间）
    - 通过 [`InputImageConverter`](../src/input_converter/image_converter.hpp) 转换输入图像
    - 通过 [`InputTransformMatrixConverter`](../src/input_converter/transform_matrix_converter.hpp) 转换输入变换矩阵
      - 使用 `CoordinateTransformer::lookup_base2cam()` 构建变换矩阵
    - 通过 [`InputCanBusConverter`](../src/input_converter/can_bus_converter.hpp) 转换输入里程计数据
    - 通过 [`InputBEVShiftConverter`](../src/input_converter/bev_shift_converter.hpp) 计算 BEV 偏移
  - 转换输出数据（所有转换器均位于 `vad_interface::` 命名空间）
    - 通过 [`OutputTrajectoryConverter`](../src/output_converter/trajectory_converter.hpp) 转换输出规划轨迹
    - 通过 [`OutputObjectsConverter`](../src/output_converter/objects_converter.hpp) 转换输出预测目标
    - 通过 [`OutputMapConverter`](../src/output_converter/map_converter.hpp) 转换输出地图标记

- 负责仅使用 CPU 的预处理和后处理（不使用 CUDA）
- 首次成功计算后缓存 `vad_base2img` 变换矩阵，避免重复 TF 查询

<a id="processing-flowchart"></a>

## 处理流程图

```mermaid
flowchart TD
    VadNode["VadNode"]
    VadNode2["VadNode"]
    VadInputTopicData((VadInputTopicData))
    VadOutputData((VadOutputData))
    VadNode --> VadInputTopicData
    VadInputTopicData --> ConvertInput
    subgraph ConvertInput["VadInterface::convert_input()"]
        ImageConv
        MatrixConv
        CanBusConv
        BEVShiftConv
    end
    ConvertInput --> VadInputData((VadInputData))
    VadInputData --> VadNode2
    style VadInputData fill:#fff,stroke:#1976d2,stroke-width:2px,color:#000000
    VadNode --> VadOutputData
    VadOutputData --> ConvertOutput
    subgraph ConvertOutput["VadInterface::convert_output()"]
        TrajConv
        ObjConv
        MapConv
    end
    ConvertOutput --> VadOutputTopicData((VadOutputTopicData))
    VadOutputTopicData --> VadNode2

    subgraph VadInterface["VadInterface"]
        subgraph ConvertInput["VadInterface::convert_input()"]
            subgraph ImageConv["InputImageConverter"]
                ImageProc["process_image()"]
            end
            subgraph MatrixConv["InputTransformMatrixConverter"]
                MatrixProc["process_vad_base2img()"]
            end
            subgraph CanBusConv["InputCanBusConverter"]
                CanBusProc["process_can_bus()"]
            end
            subgraph BEVShiftConv["InputBEVShiftConverter"]
                BEVShiftProc["process_shift()"]
            end
        end
        subgraph ConvertOutput["VadInterface::convert_output()"]
            subgraph TrajConv["OutputTrajectoryConverter"]
                TrajProc["process_trajectory()"]
                TrajCandProc["process_candidate_trajectories()"]
            end
            subgraph ObjConv["OutputObjectsConverter"]
                ObjProc["process_predicted_objects()"]
            end
            subgraph MapConv["OutputMapConverter"]
                MapProc["process_map_points()"]
            end
        end
    end
    style VadOutputTopicData fill:#fff,stroke:#1976d2,stroke-width:2px,color:#000000

    style VadInputTopicData fill:#fff,stroke:#1976d2,stroke-width:2px,color:#000000
    style VadOutputData fill:#fff,stroke:#1976d2,stroke-width:2px,color:#000000
    style VadInterface fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000000
    style VadNode fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000000
    style VadNode2 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000000
    %% Blue style for ConvertInput/ConvertOutput subgraphs
    style ConvertInput fill:#bbdefb,stroke:#1976d2,stroke-width:2px,color:#000000
    style ConvertOutput fill:#bbdefb,stroke:#1976d2,stroke-width:2px,color:#000000

    %% Clickable links for code navigation (must be after all style statements)
    click VadNode "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/vad_node.cpp" "VadNode implementation"
    click VadNode2 "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/vad_node.cpp" "VadNode implementation"
    click VadInputTopicData "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/data_types.hpp" "VadInputTopicData definition"
    click VadOutputData "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/data_types.hpp" "VadOutputData definition"
    click ConvertInput "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/vad_interface.hpp" "VadInterface::convert_input()"
    click ConvertOutput "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/vad_interface.hpp" "VadInterface::convert_output()"
    click VadInputData "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/data_types.hpp" "VadInputData definition"
    click VadOutputTopicData "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/data_types.hpp" "VadOutputTopicData definition"
    click ImageProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/input_converter/image_converter.hpp" "InputImageConverter::process_image()"
    click MatrixProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/input_converter/transform_matrix_converter.hpp" "InputTransformMatrixConverter::process_vad_base2img()"
    click CanBusProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/input_converter/can_bus_converter.hpp" "InputCanBusConverter::process_can_bus()"
    click BEVShiftProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/input_converter/bev_shift_converter.hpp" "InputBEVShiftConverter::process_shift()"
    click TrajProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/output_converter/trajectory_converter.hpp" "OutputTrajectoryConverter::process_trajectory()"
    click TrajCandProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/output_converter/trajectory_converter.hpp" "OutputTrajectoryConverter::process_candidate_trajectories()"
    click ObjProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/output_converter/objects_converter.hpp" "OutputObjectsConverter::process_predicted_objects()"
    click MapProc "https://github.com/autowarefoundation/autoware_universe/tree/main/e2e/autoware_tensorrt_vad/src/output_converter/map_converter.hpp" "OutputMapConverter::process_map_points()"
```

<a id="function-roles"></a>

### 函数职责

<a id="api-functions-public"></a>

### API 函数（公开）

`VadNode` 在推理前调用 `convert_input()`，在推理后调用 `convert_output()`。

- [`convert_input(const VadInputTopicData&)`](../lib/vad_interface.cpp)：将 `VadInputTopicData` 转换为 `VadInputData`
  - 通过 [`InputTransformMatrixConverter::process_vad_base2img()`](../src/input_converter/transform_matrix_converter.hpp) 验证并缓存 `vad_base2img` 变换
  - 通过 [`InputCanBusConverter::process_can_bus()`](../src/input_converter/can_bus_converter.hpp) 处理 CAN 总线数据
  - 通过 [`InputBEVShiftConverter::process_shift()`](../src/input_converter/bev_shift_converter.hpp) 计算 BEV 偏移
  - 通过 [`InputImageConverter::process_image()`](../src/input_converter/image_converter.hpp) 处理图像
  - 更新 `prev_can_bus_` 供下一帧使用

- [`convert_output(const VadOutputData&, ...)`](../lib/vad_interface.cpp)：将 `VadOutputData` 转换为 `VadOutputTopicData`
  - 通过 [`OutputTrajectoryConverter::process_candidate_trajectories()`](../src/output_converter/trajectory_converter.hpp) 转换候选轨迹
  - 通过 [`OutputTrajectoryConverter::process_trajectory()`](../src/output_converter/trajectory_converter.hpp) 转换主轨迹
  - 通过 [`OutputMapConverter::process_map_points()`](../src/output_converter/map_converter.hpp) 转换地图折线
  - 通过 [`OutputObjectsConverter::process_predicted_objects()`](../src/output_converter/objects_converter.hpp) 转换预测目标

<a id="converter-architecture"></a>

### 转换器架构

所有转换器类均位于 `autoware::tensorrt_vad::vad_interface::` 命名空间，并继承基础 `Converter` 类，该类提供对 `CoordinateTransformer` 和配置的访问。

<a id="key-design-details"></a>

## 关键设计细节

### CoordinateTransformer

- 封装 TF 缓冲区，并提供 `lookup_base2cam(frame_id)` 用于相机变换
- 对于 CARLA：VAD 坐标与 Autoware base_link 完全相同（无需坐标转换）
- 由 `InputTransformMatrixConverter` 使用以构建 `vad_base2img` 矩阵

<a id="caching-strategy"></a>

### 缓存策略

- 首次成功计算后缓存 `vad_base2img_transform_`
- 缓存前验证变换（检查非零值）
- 避免每帧重复 TF 查询

<a id="converter-dependency-injection"></a>

### 转换器依赖注入

- 所有转换器均在构造函数中接收 `CoordinateTransformer` 引用和配置
- 支持单元测试和关注点分离

<a id="todo"></a>

## 待办事项

- 尽管数据并非来自 CAN BUS，仍使用“CanBus”这一名称。这种命名优先遵循 [VAD 代码使用的记法](https://github.com/hustvl/VAD/blob/36047b6b5985e01832d8a2ecb0355d7f3c753ee1/projects/mmdet3d_plugin/datasets/nuscenes_vad_dataset.py#L1375-L1382)。但它可能引起混淆，因此应考虑更合适的名称。
