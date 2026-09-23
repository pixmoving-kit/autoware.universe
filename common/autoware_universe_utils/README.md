# autoware_universe_utils

<a id="purpose"></a>

## 用途

此软件包包含许多供其他软件包使用的通用函数，请按需参考。

<a id="for-developers"></a>

## 开发者说明

已移除 `autoware_universe_utils.hpp` 头文件，因为直接或间接包含此文件的源文件需要很长的预处理时间。

## `autoware::universe_utils`

### `systems`

#### `autoware::universe_utils::TimeKeeper`

<a id="constructor"></a>

##### 构造函数

```cpp
template <typename... Reporters>
explicit TimeKeeper(Reporters... reporters);
```

- 使用报告器列表初始化 `TimeKeeper`。

<a id="methods"></a>

##### 方法

- `void add_reporter(std::ostream * os);`
  - 添加报告器，将处理时间输出到 `ostream`。
  - `os`: 指向 `ostream` 对象的指针。

- `void add_reporter(rclcpp::Publisher<ProcessingTimeDetail>::SharedPtr publisher);`
  - 添加报告器，通过 `rclcpp` 发布器发布处理时间。
  - `publisher`: 指向 `rclcpp` 发布器的共享指针。

- `void add_reporter(rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher);`
  - 添加报告器，通过 `rclcpp` 发布器使用 `std_msgs::msg::String` 发布处理时间。
  - `publisher`: 指向 `rclcpp` 发布器的共享指针。

- `void start_track(const std::string & func_name);`
  - 开始跟踪函数的处理时间。
  - `func_name`: 要跟踪的函数名称。

- `void end_track(const std::string & func_name);`
  - 结束对函数处理时间的跟踪。
  - `func_name`: 要结束跟踪的函数名称。

- `void comment(const std::string & comment);`
  - 为当前正在跟踪的函数添加备注。
  - `comment`: 要添加的备注。

<a id="note"></a>

##### 注意

- 可以使用 `start_track` 和 `end_track` 开始和结束计时，如下所示：

  ```cpp
  time_keeper.start_track("example_function");
  // Your function code here
  time_keeper.end_track("example_function");
  ```

- 为保证安全并确保正确跟踪，建议使用 `ScopedTimeTrack`。

<a id="example"></a>

##### 示例

```cpp
#include <rclcpp/rclcpp.hpp>

#include <std_msgs/msg/string.hpp>

#include <chrono>
#include <iostream>
#include <memory>
#include <thread>

class ExampleNode : public rclcpp::Node
{
public:
  ExampleNode() : Node("time_keeper_example")
  {
    publisher_ =
      create_publisher<autoware::universe_utils::ProcessingTimeDetail>("processing_time", 1);

    time_keeper_ = std::make_shared<autoware::universe_utils::TimeKeeper>(publisher_, &std::cerr);
    // You can also add a reporter later by add_reporter.
    // time_keeper_->add_reporter(publisher_);
    // time_keeper_->add_reporter(&std::cerr);

    timer_ =
      create_wall_timer(std::chrono::seconds(1), std::bind(&ExampleNode::func_a, this));
  }

private:
  std::shared_ptr<autoware::universe_utils::TimeKeeper> time_keeper_;
  rclcpp::Publisher<autoware::universe_utils::ProcessingTimeDetail>::SharedPtr publisher_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_str_;
  rclcpp::TimerBase::SharedPtr timer_;

  void func_a()
  {
    // Start constructing ProcessingTimeTree (because func_a is the root function)
    autoware::universe_utils::ScopedTimeTrack st("func_a", *time_keeper_);
    std::this_thread::sleep_for(std::chrono::milliseconds(1));
    time_keeper_->comment("This is a comment for func_a");
    func_b();
    // End constructing ProcessingTimeTree. After this, the tree will be reported (publishing
    // message and outputting to std::cerr)
  }

  void func_b()
  {
    autoware::universe_utils::ScopedTimeTrack st("func_b", *time_keeper_);
    std::this_thread::sleep_for(std::chrono::milliseconds(2));
    time_keeper_->comment("This is a comment for func_b");
    func_c();
  }

  void func_c()
  {
    autoware::universe_utils::ScopedTimeTrack st("func_c", *time_keeper_);
    std::this_thread::sleep_for(std::chrono::milliseconds(3));
    time_keeper_->comment("This is a comment for func_c");
  }
};

int main(int argc, char ** argv)
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<ExampleNode>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

- 输出（控制台）

  ```text
  ==========================
  func_a (6.243ms) : This is a comment for func_a
      └── func_b (5.116ms) : This is a comment for func_b
          └── func_c (3.055ms) : This is a comment for func_c
  ```

- 输出（`ros2 topic echo /processing_time`）

  ```text
  ---
  nodes:
  - id: 1
    name: func_a
    processing_time: 6.366
    parent_id: 0
    comment: This is a comment for func_a
  - id: 2
    name: func_b
    processing_time: 5.237
    parent_id: 1
    comment: This is a comment for func_b
  - id: 3
    name: func_c
    processing_time: 3.156
    parent_id: 2
    comment: This is a comment for func_c
  ```

#### `autoware::universe_utils::ScopedTimeTrack`

<a id="description"></a>

##### 说明

用于在作用域内自动跟踪函数处理时间的类。

<a id="constructor_1"></a>

##### 构造函数

```cpp
ScopedTimeTrack(const std::string & func_name, TimeKeeper & time_keeper);
```

- `func_name`: 要跟踪的函数名称。
- `time_keeper`: 对 `TimeKeeper` 对象的引用。

<a id="destructor"></a>

##### 析构函数

```cpp
~ScopedTimeTrack();
```

- 销毁 `ScopedTimeTrack` 对象，结束对函数的跟踪。
