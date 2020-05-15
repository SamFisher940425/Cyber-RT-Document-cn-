# Cyber RT API 教程

此文档对如何创建、操作和使用Cyber RT的API进行了深入的技术研究。

内容列表

  * [讲话者（Talker）和倾听者（Listener）](tutorial/cyber-rt-api-tutorial.md#讲话者（Talker）和倾听者（Listener）)
  * [服务端（Service）的创建与使用](tutorial/cyber-rt-api-tutorial.md#服务端（Service）的创建与使用)
  * [参数（Param）服务](tutorial/cyber-rt-api-tutorial.md#参数（Param）服务)
  * [记录（Log）API](tutorial/cyber-rt-api-tutorial.md#记录（Log）API)
  * [基于组件构建模块](tutorial/cyber-rt-api-tutorial.md#基于组件构建模块)
  * [启动（Launch）](tutorial/cyber-rt-api-tutorial.md#启动（Launch）)
  * [定时器（Timer）](tutorial/cyber-rt-api-tutorial.md#定时器（Timer）)
  * [定时器（Timer）API](tutorial/cyber-rt-api-tutorial.md#定时器（Timer）API)
  * [数据记录（Record）文件：读取与写入](tutorial/cyber-rt-api-tutorial.md#数据记录（Record）文件：读取与写入)
  * [C++ API 词典](tutorial/cyber-rt-api-tutorial.md#C++API词典)
    * [节点（Node）](tutorial/cyber-rt-api-tutorial.md#节点（Node）)
    * [写入者（Writer）](tutorial/cyber-rt-api-tutorial.md#写入者（Writer）)
    * [客户端（Client）](tutorial/cyber-rt-api-tutorial.md#客户端（Client）)
    * [参数服务器（Parameter）](tutorial/cyber-rt-api-tutorial.md#参数服务器（Parameter）)
    * [定时器（Timer）](tutorial/cyber-rt-api-tutorial.md#定时器（Timer）)
    * [时间（Time）](tutorial/cyber-rt-api-tutorial.md#时间（Time）)
    * [持续时间（Duration）](tutorial/cyber-rt-api-tutorial.md#持续时间（Duration）)
    * [速率（Rate）](tutorial/cyber-rt-api-tutorial.md#速率（Rate）)
    * [记录读取者（RecordReader）](tutorial/cyber-rt-api-tutorial.md#记录读取者（RecordReader）)
    * [记录写入者（RecordWriter）](tutorial/cyber-rt-api-tutorial.md#记录写入者（RecordWriter）)

## <a id="讲话者（Talker）和倾听者（Listener）">讲话者（Talker）和倾听者（Listener）</a>

Cyber RT API示例的第一部分是理解讲话者（Talker）和倾听者（Listener）样例。以下是样例中的三个基本概念：节点（基本单元）、读取者（读取消息的工具）和写入者（写入消息的工具）。

### 创建一个节点（Node）

在Cyber RT的框架中，节点是最基本的单元，类似于句柄的角色。在创建特定的功能对象（写入者、读取者等）时，您需要基于现有的节点实例创建它。节点创建界面如下：

```
std::unique_ptr<Node> apollo::cyber::CreateNode(const std::string& node_name, const std::string& name_space = "");
```

  * 参数：

    * 节点名（node_name）：节点的名称，全局唯一标识符

    * 命名空间（name_space）：节点所在的空间名称
      默认情况下，命名空间为空。它是与节点名相连的空间的名称。格式是`/namespace/node_name`

  * 返回值：指向节点的专用智能指针

  * 错误条件：当`cyber::Init()`尚未调用时，系统处于未初始化状态，无法创建节点，返回空指针（nullptr）。

### 创建一个写入者（Writer）

写入者（Writer）是Cyber RT中用来发送消息的基本工具。每个写入者（Writer）都对应于一个具有特定数据类型的通道（Channel）。写入者（Writer）是由节点（Node）类中的`CreateWriter`接口创建的。接口如下所示：

```
template <typename MessageT>
   auto CreateWriter(const std::string& channel_name)
       -> std::shared_ptr<Writer<MessageT>>;
template <typename MessageT>
   auto CreateWriter(const proto::RoleAttributes& role_attr)
       -> std::shared_ptr<Writer<MessageT>>;
```

  * 参数：

    * 通道名称（Channel_name）：要写入的通道的名称

    * 消息类型（MessageT）：将要写入的消息的类型

  * 返回值： 指向写入者（Writer）对象的共享指针

### 创建一个读取者（Reader）

读取者（Reader）是Cyber中用来接收信息的基本设备。读取者在创建时必须绑定到回调函数。新消息到达通道时，将调用回调函数。读取者（Reader）是由节点（Node）类的`CreateReader`接口创建的。接口如下：

```
template <typename MessageT>
auto CreateReader(const std::string& channel_name, const std::function<void(const std::shared_ptr<MessageT>&)>& reader_func)
    -> std::shared_ptr<Reader<MessageT>>;

template <typename MessageT>
auto CreateReader(const ReaderConfig& config,
                  const CallbackFunc<MessageT>& reader_func = nullptr)
    -> std::shared_ptr<cyber::Reader<MessageT>>;

template <typename MessageT>
auto CreateReader(const proto::RoleAttributes& role_attr,
                  const CallbackFunc<MessageT>& reader_func = nullptr)
-> std::shared_ptr<cyber::Reader<MessageT>>;
```

  * 参数：

    * 消息类型（MessageT）：要读取消息的类型
    * 通道名（channel_name）：要接收信息的通道的名称
    * 读取者函数（reader_func）：处理消息的回调函数

  * 返回值：指向读取者（Reader）对象的共享指针

## 示例代码

译者注：讲话者-倾听者描述的是一种拓扑概念，常用作文件名；写入者-读取者描述的是实现函数，常用作函数名。

### 讲话者（cyber/example/talker.cc）

```
#include "cyber/cyber.h"
#include "cyber/proto/chatter.pb.h"
#include "cyber/time/rate.h"
#include "cyber/time/time.h"
using apollo::cyber::Rate;
using apollo::cyber::Time;
using apollo::cyber::proto::Chatter;
int main(int argc, char *argv[]) {
  // init cyber framework
  apollo::cyber::Init(argv[0]);
  // create talker node
  std::shared_ptr<apollo::cyber::Node> talker_node(
      apollo::cyber::CreateNode("talker"));
  // create talker
  auto talker = talker_node->CreateWriter<Chatter>("channel/chatter");
  Rate rate(1.0);
  while (apollo::cyber::OK()) {
    static uint64_t seq = 0;
    auto msg = std::make_shared<apollo::cyber::proto::Chatter>();
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

### 倾听者（cyber/examples/listener.cc）

```
#include "cyber/cyber.h"
#include "cyber/proto/chatter.pb.h"
void MessageCallback(
    const std::shared_ptr<apollo::cyber::proto::Chatter>& msg) {
  AINFO << "Received message seq-> " << msg->seq();
  AINFO << "msgcontent->" << msg->content();
}
int main(int argc, char *argv[]) {
  // init cyber framework
  apollo::cyber::Init(argv[0]);
  // create listener node
  auto listener_node = apollo::cyber::CreateNode("listener");
  // create listener
  auto listener =
      listener_node->CreateReader<apollo::cyber::proto::Chatter>(
          "channel/chatter", MessageCallback);
  apollo::cyber::WaitForShutdown();
  return 0;
}
```

### Bazel 构建（BUILD）文件（cyber/samples/BUILD）

```
cc_binary(
    name = "talker",
    srcs = [ "talker.cc", ],
    deps = [
        "//cyber",
        "//cyber/examples/proto:examples_cc_proto",
    ],
)

cc_binary(
    name = "listener",
    srcs = [ "listener.cc", ],
    deps = [
        "//cyber",
        "//cyber/examples/proto:examples_cc_proto",
    ],
)
```

### 构建与运行

  * 构建：bazel build cyber/examples/...
  * 在另一个终端中运行讲话者（Talker）/倾听者（Listener）：

    * ./bazel-bin/cyber/examples/talker
    * ./bazel-bin/cyber/examples/listener
  * 测试结果：您将在倾听者（Listener）看到消息的打印输出

## <a id="服务端（Service）的创建与使用">服务端（Service）的创建与使用</a>

### 介绍

在自动驾驶系统中，有很多场景需要多模块通信，且不仅只是收发消息。服务（Service）是节点间通信的另一种方式。与通道（Channel）不同，服务（Service）实现了双向通信，例如，节点通过发送请求（request）来获得相应（response）。本节通过实例介绍了Cyber RT API中的服务（Service）模块。

### 样例

问题：通过`Driver.proto`创建一个前后传递的客户端-服务端模型。当客户端发送请求时，服务器解析/处理该请求并返回响应。

演示的实现主要包括以下步骤。

#### 定义请求（request）和响应（response）消息

Cyber中所有消息都是protobuf格式。任何带有序列化/反序列化接口的protobuf消息，都可以用做服务请求和响应消息。在本例中，example.proto中的Driver被用作服务请求和响应。

```
// filename: examples.proto
syntax = "proto2";
package apollo.cyber.examples.proto;
message Driver {
    optional string content = 1;
    optional uint64 msg_id = 2;
    optional uint64 timestamp = 3;
};
```

#### 创建服务端和客户端

```
// filename: cyber/examples/service.cc
#include "cyber/cyber.h"
#include "cyber/examples/proto/examples.pb.h"

using apollo::cyber::examples::proto::Driver;

int main(int argc, char* argv[]) {
  apollo::cyber::Init(argv[0]);
  std::shared_ptr<apollo::cyber::Node> node(
      apollo::cyber::CreateNode("start_node"));
  auto server = node->CreateService<Driver, Driver>(
      "test_server", [](const std::shared_ptr<Driver>& request,
                        std::shared_ptr<Driver>& response) {
        AINFO << "server: I am driver server";
        static uint64_t id = 0;
        ++id;
        response->set_msg_id(id);
        response->set_timestamp(0);
      });
  auto client = node->CreateClient<Driver, Driver>("test_server");
  auto driver_msg = std::make_shared<Driver>();
  driver_msg->set_msg_id(0);
  driver_msg->set_timestamp(0);
  while (apollo::cyber::OK()) {
    auto res = client->SendRequest(driver_msg);
    if (res != nullptr) {
      AINFO << "client: response: " << res->ShortDebugString();
    } else {
      AINFO << "client: service may not ready.";
    }
    sleep(1);
  }

  apollo::cyber::WaitForShutdown();
  return 0;
}
```

#### Bazel构建（Build）文件

```
cc_binary(
    name = "service",
    srcs = [ "service.cc", ],
    deps = [
        "//cyber",
        "//cyber/examples/proto:examples_cc_proto",
    ],
)
```

#### 构建并运行

  * 构建服务端/客户端：`bazel build cyber/examples/…`
  * 运行：`./bazel-bin/cyber/examples/service`
  * 核实结果：您应该在apollo/data/log/service.INFO中看到以下内容：

```
I1124 16:36:44.568845 14965 service.cc:30] [service] server: i am driver server
I1124 16:36:44.569031 14949 service.cc:43] [service] client: response: msg_id: 1 timestamp: 0
I1124 16:36:45.569514 14966 service.cc:30] [service] server: i am driver server
I1124 16:36:45.569932 14949 service.cc:43] [service] client: response: msg_id: 2 timestamp: 0
I1124 16:36:46.570627 14967 service.cc:30] [service] server: i am driver server
I1124 16:36:46.571024 14949 service.cc:43] [service] client: response: msg_id: 3 timestamp: 0
I1124 16:36:47.571566 14968 service.cc:30] [service] server: i am driver server
I1124 16:36:47.571962 14949 service.cc:43] [service] client: response: msg_id: 4 timestamp: 0
I1124 16:36:48.572634 14969 service.cc:30] [service] server: i am driver server
I1124 16:36:48.573030 14949 service.cc:43] [service] client: response: msg_id: 5 timestamp: 0
```

#### 注意事项

  * 注册服务端时，请注意不要使用重复的服务名称
  * 注册服务端和客户端时，应用的节点名也不应重复

## <a id="参数（Param）服务">参数（Param）服务</a>

## <a id="记录（Log）API">记录（Log）API</a>

## <a id="基于组件构建模块">基于组件构建模块</a>

## <a id="启动（Launch）">启动（Launch）</a>

## <a id="定时器（Timer）">定时器（Timer）</a>

## <a id="定时器（Timer）API">定时器（Timer）API</a>

## <a id="数据记录（Record）文件：读取与写入">数据记录（Record）文件：读取与写入</a>

## <a id="C++API词典">C++ API 词典</a>

## <a id="启动（Launch）">启动（Launch）</a>

### <a id="节点（Node）">节点（Node）</a>

### <a id="写入者（Writer）">写入者（Writer）</a>

### <a id="客户端（Client）">客户端（Client）</a>

### <a id="参数服务器（Parameter）">参数服务器（Parameter）</a>

### <a id="定时器（Timer）">定时器（Timer）</a>

### <a id="时间（Time）">时间（Time）</a>

### <a id="持续时间（Duration）">持续时间（Duration）</a>

### <a id="速率（Rate）">速率（Rate）</a>

### <a id="记录读取者（RecordReader）">记录读取者（RecordReader）</a>

### <a id="记录写入者（RecordWriter）">记录写入者（RecordWriter）</a>



