<a id="shift-decider"></a>

# 挡位决策器

<a id="purpose"></a>

## 目的

`autoware_shift_decider` 模块根据阿克曼控制命令决定挡位。

<a id="inner-workings-algorithms"></a>

## 内部机制／算法

<a id="flow-chart"></a>

### 流程图

```plantuml
@startuml
skinparam monochrome true

title update current shift
start
if (absolute target velocity is less than threshold) then (yes)
    :set previous shift;
else(no)
if (target velocity is positive) then (yes)
    :set shift DRIVE;
else
    :set shift REVERSE;
endif
endif
    :publish current shift;
note right
    publish shift for constant interval
end note
stop
@enduml
```

<a id="algorithms"></a>

### 算法

<a id="inputs-outputs"></a>

## 输入与输出

<a id="input"></a>

### 输入

| 名称 | 类型 | 说明 |
| --------------------- | ------------------------------------- | ---------------------------- |
| `~/input/control_cmd` | `autoware_control_msgs::msg::Control` | 车辆控制命令。 |

<a id="output"></a>

### 输出

| 名称 | 类型 | 说明 |
| ------------------ | ----------------------------------------- | ---------------------------------- |
| `~output/gear_cmd` | `autoware_vehicle_msgs::msg::GearCommand` | 前进或后退的挡位。 |

<a id="parameters"></a>

## 参数

无。

<a id="assumptions-known-limits"></a>

## 假设与已知限制

待确定。
