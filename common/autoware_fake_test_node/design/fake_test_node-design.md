<a id="fake-test-node"></a>

# 模拟测试节点

<a id="what-this-package-provides"></a>

## 此软件包提供的功能

在 C++ 中使用 GTest 编写节点集成测试时，通常需要编写大量样板代码来创建模拟节点，使其在指定话题上发布预期消息，并订阅其他话题的消息。这一般通过自定义 GTest 测试夹具实现。

此软件包中的库提供了两个实用类，可代替上述自定义测试夹具，为节点编写集成测试：

- `autoware::fake_test_node::FakeTestNode`：用作 `TEST_F` 测试的自定义测试夹具
- `autoware::fake_test_node::FakeTestNodeParametrized`：用作参数化 `TEST_P` 测试的自定义测试夹具（接受一个模板参数，并将其传递给 `testing::TestWithParam<T>`）

这些测试夹具负责初始化和重新初始化 rclcpp，并检查所有订阅器和发布器是否匹配，从而减少用户需要编写的样板代码。

<a id="how-to-use-this-library"></a>

## 使用方法

包含相关头文件后，用户可以通过 typedef 自定义测试夹具名称，并直接将提供的类用作 `TEST_F` 和 `TEST_P` 测试的夹具。

<a id="example-usage"></a>

### 使用示例

假设需要测试一个 `NodeUnderTest` 节点。该节点订阅 `std_msgs::msg::Int32` 消息，并发布 `std_msgs::msg::Bool` 消息，表示输入是否为正数。可以使用以下代码，通过 `autoware::fake_test_node::FakeTestNode` 测试此节点：

```cpp
using FakeNodeFixture = autoware::fake_test_node::FakeTestNode;

/// @test Test that we can use a non-parametrized test.
TEST_F(FakeNodeFixture, Test) {
  Int32 msg{};
  msg.data = 15;
  const auto node = std::make_shared<NodeUnderTest>();

  Bool::SharedPtr last_received_msg{};
  auto fake_odom_publisher = create_publisher<Int32>("/input_topic");
  auto result_odom_subscription = create_subscription<Bool>("/output_topic", *node,
    [&last_received_msg](const Bool::SharedPtr msg) {last_received_msg = msg;});

  const auto dt{std::chrono::milliseconds{100LL}};
  const auto max_wait_time{std::chrono::seconds{10LL}};
  auto time_passed{std::chrono::milliseconds{0LL}};
  while (!last_received_msg) {
    fake_odom_publisher->publish(msg);
    rclcpp::spin_some(node);
    rclcpp::spin_some(get_fake_node());
    std::this_thread::sleep_for(dt);
    time_passed += dt;
    if (time_passed > max_wait_time) {
      FAIL() << "Did not receive a message soon enough.";
    }
  }
  EXPECT_TRUE(last_received_msg->data);
  SUCCEED();
}
```

此处仅展示 `TEST_F` 示例；`TEST_P` 的用法非常相似，只需增加少量用于设置各参数值的样板代码。使用示例见 `test_fake_test_node.cpp`。
