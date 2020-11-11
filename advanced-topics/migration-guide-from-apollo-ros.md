# 从阿波罗ROS移植指导

本文描述了从Apollo ROS（Apollo 3.0及以前）迁移到Apollo Cyber RT（Apollo 3.5以后）的项目的基本变化。我们将使用一个ROS项目讲话者（Talker）和倾听者（Listener）作为样例逐步演示迁移指令。

## 构建系统

ROS使用CMake作为其构建系统，而Cyber RT使用bazel。在ROS项目中，需要CmakeLists.txt和package.xml来定义构建配置，比如构建目标、依赖项、消息文件等等。至于Cyber RT组件，只需一个bazel构建（BUILD）文件即可。下面列出了一些关键的构建配置映射。

Cmake

```cmake
project(pb_msgs_example)
add_proto_files(
  DIRECTORY proto
  FILES chatter.proto
)
## Declare a C++ executable
add_executable(pb_talker src/talker.cpp)
target_link_libraries(pb_talker ${catkin_LIBRARIES}pb_msgs_example_proto)
add_executable(pb_listener src/listener.cpp)
target_link_libraries(pb_listener ${catkin_LIBRARIES}  pb_msgs_example_proto)
```

Bazel

```
cc_binary(
  name = "talker",
  srcs = ["talker.cc"],
  deps = [
    "//cyber",
    "//cyber/examples/proto:examples_cc_proto",
    ],
  )
cc_binary(
  name = "listener",
  srcs = ["listener.cc"],
  deps = [
    "//cyber",
    "//cyber/examples/proto:examples_cc_proto",
    ],
  )
```

我们可以很容易的从两个文件中找到映射，例如cmake中的`pb_talker`、`src/talker.cpp`和`add_executable()`映射为构建（BUILD）文件中的`name = "talker"`、`srcs = ["talker.cc"]`和`cc_binary()`

## Proto

为支持proto消息格式，Apollo ROS在`target_link_libraries`中定制了独立的段`add_proto_files`和`projectName_proto(pb_msgs_example_proto)`来以proto格式发送消息。在Cyber RT中定义proto消息，只需要在`deps`设定中添加带有`cc_proto_library`名称的目标proto文件路径，`cc_proto_library`已经在proto文件夹中的构建（BUILD）文件中建立

```
cc_proto_library(
  name = "examples_cc_proto",
  deps = [
    ":examples_proto",
  ],
)
proto_library(
  name = "examples_proto",
  srcs = [
    "examples.proto",
  ],
)
```

在Cyber RT中包（package）的定义也有所改变，在Apollo ROS中一个固定的包`package pb_msgs;`用于proto文件，但Cyber RT中，proto文件路径`package apollo.cyber.examples.proto;`作为代替。

## 文件夹结构

如下所示，Cyber RT移除了src文件夹，所有的源码与构建（BUILD）文件在相同目录。构建（BUILD）文件作用与CMake.lists加pacakag.xml相同。Cyber RT和Apollo ROS 讲述者（Talker）/倾听者（Listener）示例都有一个proto文件夹用于存储消息proto文件，但Cyber RT需要一个为proto文件夹设计的单独的构建（BUILD）文件来设置proto库

### Apollo ROS

  * CMakeLists.txt
  * package.xml
  * proto
    * chatter.proto
  * src
    * listener.cpp
    * talker.cpp

### Cyber RT

  * BUILD
  * listener.cc
  * talker.cc
  * proto
    * BUILD
    * examples.proto (with chatter message)

## 更新源码

### 倾听者（Listener）

Cyber RT

```cpp
#include "cyber/cyber.h"
#include "cyber/examples/proto/examples.pb.h"

void MessageCallback(
    const std::shared_ptr<apollo::cyber::examples::proto::Chatter>& msg) {
  AINFO << "Received message seq-> " << msg->seq();
  AINFO << "msgcontent->" << msg->content();
}

int main(int argc, char* argv[]) {
  // init cyber framework
  apollo::cyber::Init(argv[0]);
  // create listener node
  auto listener_node = apollo::cyber::CreateNode("listener");
  // create listener
  auto listener =
      listener_node->CreateReader<apollo::cyber::examples::proto::Chatter>(
          "channel/chatter", MessageCallback);
  apollo::cyber::WaitForShutdown();
  return 0;
}
```

ROS

```cpp
#include "ros/ros.h"
#include "chatter.pb.h"

void MessageCallback(const boost::shared_ptr<pb_msgs::Chatter>& msg) {
  ROS_INFO_STREAM("Time: " << msg->stamp().sec() << "." << msg->stamp().nsec());
  ROS_INFO("I heard pb Chatter message: [%s]", msg->content().c_str());
}

int main(int argc, char** argv) {
  ros::init(argc, argv, "listener");
  ros::NodeHandle n;
  ros::Subscriber pb_sub = n.subscribe("chatter", 1000, MessageCallback);
  ros::spin();
  return 0;
}
```

从上面两个倾听者（Listener）示例代码中可以容易的看到，Cyber RT为开发者提供了一个从ROS移植来的与之非常相近的API

  * `ros::init(argc, argv, "listener");` --> `apollo::cyber::Init(argv[0]);`
  * `ros::NodeHandle n;` --> `auto listener_node = apollo::cyber::CreateNode("listener");`
  * `ros::Subscriber pb_sub = n.subscribe("chatter", 1000, MessageCallback);` --> `auto listener = listener_node->CreateReader("channel/chatter", MessageCallback);`
  * `ros::spin();` --> `apollo::cyber::WaitForShutdown();`

注：Cyber RT中，一个倾听者（Listener）节点需要用`node->CreateReader<messageType>(channelName, callback)`从通道中读取数据

### 讲述者（Talker）

Cyber RT

```cpp
#include "cyber/cyber.h"
#include "cyber/examples/proto/examples.pb.h"

using apollo::cyber::examples::proto::Chatter;

int main(int argc, char *argv[]) {
  // init cyber framework
  apollo::cyber::Init(argv[0]);
  // create talker node
  auto talker_node = apollo::cyber::CreateNode("talker");
  // create talker
  auto talker = talker_node->CreateWriter<Chatter>("channel/chatter");
  Rate rate(1.0);
  while (apollo::cyber::OK()) {
    static uint64_t seq = 0;
    auto msg = std::make_shared<Chatter>();
    msg->set_timestamp(Time::Now().ToNanosecond());
    msg->set_lidar_timestamp(Time::Now().ToNanosecond());
    msg->set_seq(seq++);
    msg->set_content("Hello, apollo!");
    talker->Write(msg);
    AINFO << "talker sent a message!";
    rate.Sleep();
  }
  return 0;
}
```

ROS

```cpp
#include "ros/ros.h"
#include "chatter.pb.h"

#include <sstream>

int main(int argc, char** argv) {
  ros::init(argc, argv, "talker");
  ros::NodeHandle n;
  ros::Publisher chatter_pub = n.advertise<pb_msgs::Chatter>("chatter", 1000);
  ros::Rate loop_rate(10);
  int count = 0;
  while (ros::ok()) {
    pb_msgs::Chatter msg;
    ros::Time now = ros::Time::now();
    msg.mutable_stamp()->set_sec(now.sec);
    msg.mutable_stamp()->set_nsec(now.nsec);
    std::stringstream ss;
    ss << "Hello world " << count;
    msg.set_content(ss.str());
    chatter_pub.publish(msg);
    ros::spinOnce();
    loop_rate.sleep();
  }
  return 0;
}
```

大多数映射关系已经在上述示例中列出，其余的如下所示

  * `ros::Publisher chatter_pub = n.advertise<pb_msgs::Chatter>("chatter", 1000);` –> `auto talker = talker_node->CreateWriter<Chatter>("channel/chatter");`
  * `chatter_pub.publish(msg);` –> `talker->Write(msg);`

## 工具映射

|  ROS  | Cyber RT |  Note  |
|:----- |  :-----  | :----- |
| rosbag | cyber_recorder | data file |
| scripts/diagnostics.sh | cyber_monitor | channel debug |
| offline_lidar_visualizer_tool | cyber_visualizer | point cloud visualizer |

## ROS数据包移植

Cyber RT中，数据记录从ROS bag变为Cyber record。Cyber RT为使用者提供了一个移植工具`rosbag_to_record`，其可以轻易的将Apollo 3.0以前的(ROS)数据文件迁移至Cyber RT中，示例如下

`rosbag_to_record demo_3.0.bag demo_3.5.record`

