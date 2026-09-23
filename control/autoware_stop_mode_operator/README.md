# autoware_stop_mode_operator

此软件包发布选择停止模式时使用的停车指令。

<a id="auto-parking"></a>

## 自动挂入驻车挡

如果 `enable_auto_parking` 设置为 `true`，当路线未设置或已到达终点，且车辆停止时，将自动切换到驻车挡。
由于停止模式需要独立于定位功能工作，速度应从车辆状态获取，而不是从运动学状态获取。
