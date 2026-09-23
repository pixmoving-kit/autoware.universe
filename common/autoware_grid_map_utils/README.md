<a id="grid-map-utils"></a>

# 栅格地图工具

<a id="overview"></a>

## 概述

此软件包重新实现了 `grid_map::PolygonIterator`，用于遍历栅格地图中位于指定多边形内部的所有单元格。

<a id="algorithm"></a>

## 算法

此实现采用[扫描线算法](https://en.wikipedia.org/wiki/Scanline_rendering)，这是一种在栅格化图像上绘制多边形的常用算法。
将该算法应用于栅格地图时，其主要思路如下：

- 计算栅格地图各行与多边形边界的交点；
- 逐行计算每对交点之间的列；
- 得到的 `(row, column)` 索引位于多边形内部。

有关扫描线算法的更多信息，请参阅参考资料。

## API

`autoware::grid_map_utils::PolygonIterator` 与原始 [`grid_map::PolygonIterator`](https://docs.ros.org/en/kinetic/api/grid_map_core/html/classgrid__map_1_1PolygonIterator.html) 使用相同的 API。

<a id="assumptions"></a>

## 假设

仅当多边形边界没有_恰好_穿过任何单元格中心时，才能保证 `autoware::grid_map_utils::PolygonIterator` 与 `grid_map::PolygonIterator` 的行为一致。
当边界恰好穿过单元格中心时，浮点精度误差可能导致两种实现对该单元格位于多边形内部还是外部的判断不同。

<a id="performances"></a>

## 性能

基准测试代码位于 `test/benchmarking.cpp`，同时用于验证 `autoware::grid_map_utils::PolygonIterator` 与 `grid_map::PolygonIterator` 的行为完全一致。

下图比较了此软件包实现（`autoware_grid_map_utils`）与原始实现（`grid_map`）的运行时间。
测量时间包含迭代器构造及遍历全部索引的耗时，并使用对数坐标显示。
测试改变正方形栅格地图的边长，取值为 `100 <= n <= 1000`（size=`n` 表示包含 `n x n` 个单元格的栅格），同时使用顶点数为 `3 <= m <= 100` 的随机多边形；每组参数 `(n,m)` 重复测试 10 次。

![运行时间比较](media/runtime_comparison.png)

<a id="future-improvements"></a>

## 后续改进

扫描线算法存在适用于多个多边形的变体。
如果需要遍历至少位于多个多边形之一内部的单元格，可以实现这些变体。

当前实现沿用原始 `grid_map::PolygonIterator` 的行为：当单元格的中心位于多边形内部时，选中该单元格。
也可以调整此行为，例如返回所有与多边形重叠的单元格。

<a id="references"></a>

## 参考资料

- <https://en.wikipedia.org/wiki/Scanline_rendering>
- <https://web.cs.ucdavis.edu/~ma/ECS175_S00/Notes/0411_b.pdf>
