# glog_component

此软件包以 ROS 2 组件库的形式提供 glog（Google 日志库）功能，可通过组件容器动态加载 glog。

有关功能详情，请参阅 [glog GitHub 仓库](https://github.com/google/glog)。

<a id="example"></a>

## 示例

在容器中加载 `glog_component` 时，启动文件可以按如下方式编写：

```py
glog_component = ComposableNode(
    package="autoware_glog_component",
    plugin="autoware::glog_component::GlogComponent",
    name="glog_component",
)

container = ComposableNodeContainer(
    name="my_container",
    namespace="",
    package="rclcpp_components",
    executable=LaunchConfiguration("container_executable"),
    composable_node_descriptions=[
        component1,
        component2,
        glog_component,
    ],
)
```
