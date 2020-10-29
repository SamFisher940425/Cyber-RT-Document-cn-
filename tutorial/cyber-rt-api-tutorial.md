# Cyber RT API 教程

此文档对如何创建、操作和使用Cyber RT的API进行了深入的技术研究。

内容列表

  * [讲话者（Talker）和倾听者（Listener）](cyber-rt-api-tutorial.md#讲话者（Talker）和倾听者（Listener）)
  * [服务端（Service）的创建与使用](cyber-rt-api-tutorial.md#服务端（Service）的创建与使用)
  * [参数（Param）服务](cyber-rt-api-tutorial.md#参数（Param）服务)
  * [记录（Log）API](cyber-rt-api-tutorial.md#记录（Log）API)
  * [基于组件构建模块](cyber-rt-api-tutorial.md#基于组件构建模块)
  * [启动（Launch）](cyber-rt-api-tutorial.md#启动（Launch）)
  * [定时器（Timer）](cyber-rt-api-tutorial.md#定时器（Timer）)
  * [时间（Time）API](cyber-rt-api-tutorial.md#时间（Time）API)
  * [数据记录（Record）文件：读取与写入](cyber-rt-api-tutorial.md#数据记录（Record）文件：读取与写入)
  * [C++ API词典](tcyber-rt-api-tutorial.md#C++API词典)
    * [节点（Node）](cyber-rt-api-tutorial.md#节点（Node）)
    * [写入者（Writer）](cyber-rt-api-tutorial.md#写入者（Writer）)
    * [客户端（Client）](cyber-rt-api-tutorial.md#客户端（Client）)
    * [参数服务（Parameter）](cyber-rt-api-tutorial.md#参数服务（Parameter）)
    * [定时器（Timer）](cyber-rt-api-tutorial.md#定时器（Timer）)
    * [时间（Time）](cyber-rt-api-tutorial.md#时间（Time）)
    * [持续时间（Duration）](cyber-rt-api-tutorial.md#持续时间（Duration）)
    * [速率（Rate）](cyber-rt-api-tutorial.md#速率（Rate）)
    * [记录读取者（RecordReader）](cyber-rt-api-tutorial.md#记录读取者（RecordReader）)
    * [记录写入者（RecordWriter）](cyber-rt-api-tutorial.md#记录写入者（RecordWriter）)

## <a id="讲话者（Talker）和倾听者（Listener）">讲话者（Talker）和倾听者（Listener）</a>

Cyber RT API示例的第一部分是理解讲话者（Talker）和倾听者（Listener）样例。以下是样例中的三个基本概念：节点（基本单元）、读取者（读取消息的工具）和写入者（写入消息的工具）。

### 创建一个节点（Node）

在Cyber RT的框架中，节点是最基本的单元，类似于句柄的角色。在创建特定的功能对象（写入者、读取者等）时，您需要基于现有的节点实例创建它。节点创建界面如下：

```cpp
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

```cpp
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

```cpp
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

### 示例代码

译者注：讲话者-倾听者描述的是一种拓扑概念，常用作文件名；写入者-读取者描述的是实现函数，常用作函数名。

#### 讲话者（cyber/example/talker.cc）

```cpp
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

#### 倾听者（cyber/examples/listener.cc）

```cpp
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

#### Bazel 构建（BUILD）文件（cyber/samples/BUILD）

```cpp
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

#### 构建与运行

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

```cpp
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

```cpp
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

```cpp
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

```cpp
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

### 注意事项

  * 注册服务端时，请注意不要使用重复的服务名称
  * 注册服务端和客户端时，应用的节点名也不应重复

## <a id="参数（Param）服务">参数（Param）服务</a>

参数服务（Parameter Service）用于节点之间共享数据，并提供基本的操作，例如`set`、`get`和`list`。参数服务（Parameter Service）基于服务（Service）实现，包含服务端（Service）和客户端（Client）。

### 参数（Parameter）对象

#### 支持的数据类型

通过Cyber RT传递的所有参数都是`apollo::cyber::Parameter`对象，下表列出了5种支持的参数类型。

|Parameter type | C++ data type | protobuf data type |
| :--- | :--- | :--- |
| apollo::cyber::proto::ParamType::INT | int64_t | int64
| apollo::cyber::proto::ParamType::DOUBLE | double | double
| apollo::cyber::proto::ParamType::BOOL | bool |bool
| apollo::cyber::proto::ParamType::STRING | std::string | string
| apollo::cyber::proto::ParamType::PROTOBUF | std::string | string
| apollo::cyber::proto::ParamType::NOT_SET | - | - |

除了上述五种类型外，参数（Parameter）还支持protobuf对象作为传入参数的接口。执行或序列化处理对象并将其转化为字符串（string）类型。

#### 创建参数（Parameter）对象

支持的结构体：

```cpp
Parameter();  // Name is empty, type is NOT_SET 名字为空，类型是NOT_SET
explicit Parameter(const Parameter& parameter);
explicit Parameter(const std::string& name);  // type为NOT_SET
Parameter(const std::string& name, const bool bool_value);
Parameter(const std::string& name, const int int_value);
Parameter(const std::string& name, const int64_t int_value);
Parameter(const std::string& name, const float double_value);
Parameter(const std::string& name, const double double_value);
Parameter(const std::string& name, const std::string& string_value);
Parameter(const std::string& name, const char* string_value);
Parameter(const std::string& name, const std::string& msg_str,
          const std::string& full_name, const std::string& proto_desc);
Parameter(const std::string& name, const google::protobuf::Message& msg);
```

使用参数对象的样例代码：

```cpp
Parameter a("int", 10);
Parameter b("bool", true);
Parameter c("double", 0.1);
Parameter d("string", "cyber");
Parameter e("string", std::string("cyber"));
// proto message Chatter
Chatter chatter;
Parameter f("chatter", chatter);
std::string msg_str("");
chatter.SerializeToString(&msg_str);
std::string msg_desc("");
ProtobufFactory::GetDescriptorString(chatter, &msg_desc);
Parameter g("chatter", msg_str, Chatter::descriptor()->full_name(), msg_desc);
```

#### 接口与数据读取

接口列表：

```cpp
inline ParamType type() const;
inline std::string TypeName() const;
inline std::string Descriptor() const;
inline const std::string Name() const;
inline bool AsBool() const;
inline int64_t AsInt64() const;
inline double AsDouble() const;
inline const std::string AsString() const;
std::string DebugString() const;
template <typename Type>
typename std::enable_if<std::is_base_of<google::protobuf::Message, Type>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_integral<Type>::value && !std::is_same<Type, bool>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_floating_point<Type>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_convertible<Type, std::string>::value, const std::string&>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_same<Type, bool>::value, bool>::type
value() const;
```

如何使用这些接口的样例：

```cpp
Parameter a("int", 10);
a.Name();  // return int
a.Type();  // return apollo::cyber::proto::ParamType::INT
a.TypeName();  // return string: INT
a.DebugString();  // return string: {name: "int", type: "INT", value: 10}
int x = a.AsInt64();  // x = 10
x = a.value<int64_t>();  // x = 10
x = a.AsString();  // Undefined behavior, error log prompt
f.TypeName();  // return string: chatter
auto chatter = f.value<Chatter>();
```

### 参数服务端（Parameter Service）

如果一个节点想要为其他节点提供参数服务（Parameter Service），那么您需要创建`ParameterService`。

```cpp
/**
 * @brief Construct a new ParameterService object
 *
 * @param node shared_ptr of the node handler
 */
explicit ParameterService(const std::shared_ptr<Node>& node);
```

由于所有参数都储存在参数服务对象中，因此可以在`ParameterService`中直接操作这些参数，无需发送服务请求（service request）

**设置参数：**

```cpp
/**
 * @brief Set the Parameter object
 *
 * @param parameter parameter to be set
 */
void SetParameter(const Parameter& parameter);
```

**获取参数：**

```cpp
/**
 * @brief Get the Parameter object
 *
 * @param param_name
 * @param parameter the pointer to store
 * @return true
 * @return false call service fail or timeout
 */
bool GetParameter(const std::string& param_name, Parameter* parameter);
```

**获取参数列表：**

```cpp
/**
 * @brief Get all the Parameter objects
 *
 * @param parameters pointer of vector to store all the parameters
 * @return true
 * @return false call service fail or timeout
 */
bool ListParameters(std::vector<Parameter>* parameters);
```

### 参数客户端（Parameter Client）

如果节点想要使用其他节点的参数服务，您需要创建`ParameterClient`。

```cpp
/**
 * @brief Construct a new ParameterClient object
 *
 * @param node shared_ptr of the node handler
 * @param service_node_name node name which provide a param services
 */
ParameterClient(const std::shared_ptr<Node>& node, const std::string& service_node_name);
```

您还可以执行在[参数服务端（Parameter Service）](cyber-rt-api-tutorial.md#参数（Param）服务)下提到的`SetParameter`、`GetParameter`和`ListParameters`。

### 样例

```cpp
#include "cyber/cyber.h"
#include "cyber/parameter/parameter_client.h"
#include "cyber/parameter/parameter_server.h"

using apollo::cyber::Parameter;
using apollo::cyber::ParameterServer;
using apollo::cyber::ParameterClient;

int main(int argc, char** argv) {
  apollo::cyber::Init(*argv);
  std::shared_ptr<apollo::cyber::Node> node =
      apollo::cyber::CreateNode("parameter");
  auto param_server = std::make_shared<ParameterServer>(node);
  auto param_client = std::make_shared<ParameterClient>(node, "parameter");
  param_server->SetParameter(Parameter("int", 1));
  Parameter parameter;
  param_server->GetParameter("int", &parameter);
  AINFO << "int: " << parameter.AsInt64();
  param_client->SetParameter(Parameter("string", "test"));
  param_client->GetParameter("string", &parameter);
  AINFO << "string: " << parameter.AsString();
  param_client->GetParameter("int", &parameter);
  AINFO << "int: " << parameter.AsInt64();
  return 0;
}
```

#### 构建与运行

  * 构建：`bazel build cyber/examples/…`
  * 运行：`./bazel-bin/cyber/examples/paramserver`

## <a id="记录（Log）API">记录（Log）API</a>

### 记录（Log）库

Cyber 记录库是基于glog之上的，需要包含以下头文件：

```cpp
#include "cyber/common/log.h"
#include "cyber/init.h"
```

### 记录（Log）配置

默认全局配置路径：cyber/setup.bash

以下配置可由开发者更改：

```cpp
export GLOG_log_dir=/apollo/data/log
export GLOG_alsologtostderr=0
export GLOG_colorlogtostderr=1
export GLOG_minloglevel=0
```

### 记录（Log）初始化

在代码中调用初始化方法来初始化记录（Log）：

```cpp
apollo::cyber::cyber::Init(argv[0]) is initialized.
If no macro definition is made in the previous component, the corresponding log is printed to the binary log.
```

### 记录（Log）输出宏

记录库封装了记录打印宏，相关宏如下：

```cpp
ADEBUG << "hello cyber.";
AINFO  << "hello cyber.";
AWARN  << "hello cyber.";
AERROR << "hello cyber.";
AFATAL << "hello cyber.";
```

### 记录（Log）格式

记录文件格式是`<MODULE_NAME>.log.<LOG_LEVEL>.<datetime>.<process_id>`

### 关于记录（Log）文件

目前，本记录文件与默认glog输出行为唯一的不同之处在于同一个模块的不同等级的记录将被写在同一个记录文件中

## <a id="基于组件构建模块">基于组件构建模块</a>

### 关键概念

#### 1.组件（Component）

组件是Cyber RT提供的用于构建应用模块的基础类，每个特定的应用模块都能继承组件类并定义其自己的`Init`和`Proc`函数，以便将其载入Cyber RT框架中。

#### 2.二进制 vs 组件

在应用中使用Cyber RT框架有两种方式：

  * 基于二进制：应用被单独编译成二进制，并通过创建其各自的`读取者（Reader）`和`写入者（Writer）`来与其他模块通信。
  * 基于组件：应用被编译为共享库。通过继承组件类并编写对应的dag描述文件，Cyber RT框架将动态的加载与运行应用。

##### 组件的基本接口

  * 组件的`Init()`函数是类似于执行一些算法初始化的主函数。
  * 组件的`Proc()`函数的工作方式类似于读取者（Reader）的回调函数，当消息到达时被框架调用。

##### 使用组件的优势

  * 组件能够通过启动（Launch）文件被加载入不同进程，部署十分灵活。
  * 组件能够通过修改dag文件的方式修改接收通道的名称，无需重新编译
  * 组件支持接收多种数据类型
  * 组件提供多种融合策略

#### 3.Dag文件格式

Dag文件样例：

```cpp
# Define all coms in DAG streaming.
module_config {
    module_library : "lib/libperception_component.so"
    components {
        class_name : "PerceptionComponent"
        config {
            name : "perception"
            readers {
                channel: "perception/channel_name"
            }
        }
    }
    timer_components {
        class_name : "DriverComponent"
        config {
            name : "driver"
            interval : 100
        }
    }
}
```

  * module_library：如果你想要加载一个.so库，根目录是Cyber的工作目录（与setup.bash同目录）
  * components与timer_component：选择需要加载的基本组件类
  * class_name：加载的组件类的名称
  * name：作为样例的标识符载入的class_name
  * readers：由当前组件接收的数据，支持1-3个数据通道

### 样例

#### Common_component_example(cyber/examples/common_component_example/\*)

头文件定义（common_component_example.h)

```cpp
#include <memory>

#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"
#include "cyber/examples/proto/examples.pb.h"

using apollo::cyber::examples::proto::Driver;
using apollo::cyber::Component;
using apollo::cyber::ComponentBase;

class Commontestcomponent : public Component<Driver, Driver> {
 public:
  bool Init() override;
  bool Proc(const std::shared_ptr<Driver>& msg0,
            const std::shared_ptr<Driver>& msg1) override;
};
CYBER_REGISTER_COMPONENT(Commontestcomponent)
```

Cpp文件实现（common_component_example.cc)

```cpp
#include "cyber/examples/common_component_smaple/common_component_example.h"

#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"

bool Commontestcomponent::Init() {
  AINFO << "Commontest component init";
  return true;
}

bool Commontestcomponent::Proc(const std::shared_ptr<Driver>& msg0,
                               const std::shared_ptr<Driver>& msg1) {
  AINFO << "Start commontest component Proc [" << msg0->msg_id() << "] ["
        << msg1->msg_id() << "]";
  return true;
}
```

#### Timer_component_example(cyber/examples/timer_component_example/\*)

头文件定义（timer_component_example.h）

```cpp
#include <memory>

#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"
#include "cyber/component/timer_component.h"
#include "cyber/examples/proto/examples.pb.h"

using apollo::cyber::examples::proto::Driver;
using apollo::cyber::Component;
using apollo::cyber::ComponentBase;
using apollo::cyber::TimerComponent;
using apollo::cyber::Writer;

class TimertestComponent : public TimerComponent {
 public:
  bool Init() override;
  bool Proc() override;

 private:
  std::shared_ptr<Writer<Driver>> driver_writer_ = nullptr;
};
CYBER_REGISTER_COMPONENT(TimertestComponent)
```

Cpp文件实现（timer_component_example.cc)

```cpp
#include "cyber/examples/timer_component_example/timer_component_example.h"

#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"
#include "cyber/examples/proto/examples.pb.h"

bool TimertestComponent::Init() {
  driver_writer_ = node_->CreateWriter<Driver>("/carstatus/channel");
  return true;
}

bool TimertestComponent::Proc() {
  static int i = 0;
  auto out_msg = std::make_shared<Driver>();
  out_msg->set_msg_id(i++);
  driver_writer_->Write(out_msg);
  AINFO << "timertestcomponent: Write drivermsg->"
        << out_msg->ShortDebugString();
  return true;
}
```

#### 构建与运行

以timer_test_component为例：

  * 构建：`bazel build cyber/examples/timer_component_smaple/…`
  * 运行：`mainboard -d cyber/examples/timer_component_smaple/timer.dag`

### 注意事项

  * 组件需要被注册才可以通过SharedLibrary加载到类。注册接口如下：

  `CYBER_REGISTER_COMPONENT(DriverComponent)`

  如果注册时使用了命名空间，在dag文件中定义时也需要添加命名空间。

  * Component和TimerComponent的配置文件是不同的，请小心不要混淆。

## <a id="启动（Launch）">启动（Launch）</a>

**cyber_launch**是Cyber RT框架的启动器。依据启动文件，cyber会启动多个mainboard，并根据dag文件加载不同的组件到mainboard。cyber_launch 支持两种在子进程中动态加载组件或者启动二进制程序的场景。

### 启动文件格式

```xml
<cyber>
    <module>
        <name>driver</name>
        <dag_conf>driver.dag</dag_conf>
        <process_name></process_name>
        <exception_handler>exit</exception_handler>
    </module>
    <module>
        <name>perception</name>
        <dag_conf>perception.dag</dag_conf>
        <process_name></process_name>
        <exception_handler>respawn</exception_handler>
    </module>
    <module>
        <name>planning</name>
        <dag_conf>planning.dag</dag_conf>
        <process_name></process_name>
    </module>
</cyber>
```

Module：每个加载的组件或者二进制文件都是一个module

  * name：加载模块的名称
  * dag_conf：与组件对应的dag文件
  * process_name：每次启动的mainboard进程，名称相同的组件会在相同的进程中被加载并运行
  * exception_handler：当进程中发生异常时的一个处理方法，它的值可能是下列的exit或respawn（重生）
    * exit：退出意味着当前进程非正常退出时整个进程需要停止运行
    * respawn：非正常退出后当前进程需要重启。如果不存在这个句柄或空句柄则代表不做任何处理，根据进程的特定情况由用户去控制

## <a id="定时器（Timer）">定时器（Timer）</a>

定时器可以用于创建定时任务，以定期运行或运行一次。

### 定时器接口

```cpp
/**
 * @brief Construct a new Timer object
 *
 * @param period The period of the timer, unit is ms
 * @param callback The tasks that the timer needs to perform
 * @param oneshot True: perform the callback only after the first timing cycle
 *                False: perform the callback every timed period
 */
Timer(uint32_t period, std::function<void()> callback, bool oneshot);
```

或者您可以将参数封装到结构体中，如下例的TimerOption：

```cpp
struct TimerOption {
  uint32_t period;                 // The period of the timer, unit is ms
  std::function<void()> callback;  // The tasks that the timer needs to perform
  bool oneshot;  // True: perform the callback only after the first timing cycle
                 // False: perform the callback every timed period
};
/**
 * @brief Construct a new Timer object
 *
 * @param opt Timer option
 */
explicit Timer(TimerOption opt);
```

### 开启定时器

创建定时器实例后，您需要调用`Timer::Start()`来启动定时器。

### 停止定时器

当您需要手动停止已经启动的定时器时，可以调用`Timer::Stop()`接口

### 样例

```cpp
#include <iostream>
#include "cyber/cyber.h"
int main(int argc, char** argv) {
    cyber::Init(argv[0]);
    // Print current time every 100ms
    cyber::Timer timer(100, [](){
        std::cout << cyber::Time::Now() << std::endl;
    }, false);
    timer.Start()
    sleep(1);
    timer.Stop();
}
```

## <a id="时间（Time）API">时间（Time）API</a>

时间（Time）是一个用于管理时间的类，它可以用于获取当前时间，计算耗时，时间转换等工作。

时间类的接口如下：

```cpp
// constructor, passing in a different value to construct Time
Time(uint64_t nanoseconds); //uint64_t, in nanoseconds 纳秒
Time(int nanoseconds); // int type, unit: nanoseconds 
Time(double seconds); // double, in seconds 秒
Time(uint32_t seconds, uint32_t nanoseconds);
// seconds seconds + nanoseconds nanoseconds
Static Time Now(); // Get the current time 获取时间
Double ToSecond() const; // convert to seconds 转换到秒
Uint64_t ToNanosecond() const; // Convert to nanoseconds 转换到纳秒
Std::string ToString() const; // Convert to a string in the format "2018-07-10 20:21:51.123456789" 输出时间字符串
Bool IsZero() const; // Determine if the time is 0 检测时间是否为0
```

以下是样例：

```cpp
#include <iostream>
#include "cyber/cyber.h"
#include "cyber/duration.h"
int main(int argc, char** argv) {
    cyber::Init(argv[0]);
    Time t1(1531225311123456789UL);
    std::cout << t1.ToString() << std::endl; // 2018-07-10 20:21:51.123456789
    // Duration time interval
    Time t1(100);
    Duration d(200);
    Time t2(300);
    assert(d == (t1-t2)); // true
}
```

## <a id="数据记录（Record）文件：读取与写入">数据记录（Record）文件：读取与写入</a>

### 读取读取者（Reader）文件

**数据记录读取器（RecordReader）**是Cyber框架中用于读取消息的组件，每个数据记录读取器（RecordReader）都可以通过调用`open`方法打开一个已有的数据记录（Record）文件。用户只需执行`ReadMessage`来提取数据记录读取器（RecordReader）中最新的消息，然后通过`GetCurrentMessageChannelName`，`GetCurrentRawMessage`，`GetCurrentMessageTime`来获取消息的更多信息。

**数据记录写入器（RecordWriter）**是Cyber框架中用于记录消息的组件，每个数据记录写入器（RecordWriter）都可以通过调用`open`方法创建一个新的数据记录（Record）文件，用户只需执行`WriteMessage`和`WriteChannel`写入消息和通道信息，写入过程是异步的。

### 样例（cybre/example/record.cc）

通过`test_write`方法向`TEST_FILE`中写入100个原始数据（RawMessage），然后通过`test_read`方法将它们读取出来。

```cpp
#include <string>

#include "cyber/cyber.h"
#include "cyber/message/raw_message.h"
#include "cyber/proto/record.pb.h"
#include "cyber/record/record_message.h"
#include "cyber/record/record_reader.h"
#include "cyber/record/record_writer.h"

using ::apollo::cyber::record::RecordReader;
using ::apollo::cyber::record::RecordWriter;
using ::apollo::cyber::record::RecordMessage;
using apollo::cyber::message::RawMessage;

const char CHANNEL_NAME_1[] = "/test/channel1";
const char CHANNEL_NAME_2[] = "/test/channel2";
const char MESSAGE_TYPE_1[] = "apollo.cyber.proto.Test";
const char MESSAGE_TYPE_2[] = "apollo.cyber.proto.Channel";
const char PROTO_DESC[] = "1234567890";
const char STR_10B[] = "1234567890";
const char TEST_FILE[] = "test.record";

void test_write(const std::string &writefile) {
  RecordWriter writer;
  writer.SetSizeOfFileSegmentation(0);
  writer.SetIntervalOfFileSegmentation(0);
  writer.Open(writefile);
  writer.WriteChannel(CHANNEL_NAME_1, MESSAGE_TYPE_1, PROTO_DESC);
  for (uint32_t i = 0; i < 100; ++i) {
    auto msg = std::make_shared<RawMessage>("abc" + std::to_string(i));
    writer.WriteMessage(CHANNEL_NAME_1, msg, 888 + i);
  }
  writer.Close();
}

void test_read(const std::string &readfile) {
  RecordReader reader(readfile);
  RecordMessage message;
  uint64_t msg_count = reader.GetMessageNumber(CHANNEL_NAME_1);
  AINFO << "MSGTYPE: " << reader.GetMessageType(CHANNEL_NAME_1);
  AINFO << "MSGDESC: " << reader.GetProtoDesc(CHANNEL_NAME_1);

  // read all message
  uint64_t i = 0;
  uint64_t valid = 0;
  for (i = 0; i < msg_count; ++i) {
    if (reader.ReadMessage(&message)) {
      AINFO << "msg[" << i << "]-> "
            << "channel name: " << message.channel_name
            << "; content: " << message.content
            << "; msg time: " << message.time;
      valid++;
    } else {
      AERROR << "read msg[" << i << "] failed";
    }
  }
  AINFO << "static msg=================";
  AINFO << "MSG validmsg:totalcount: " << valid << ":" << msg_count;
}

int main(int argc, char *argv[]) {
  apollo::cyber::Init(argv[0]);
  test_write(TEST_FILE);
  sleep(1);
  test_read(TEST_FILE);
  return 0;
}
```

### 构建与运行

  * 构建：`bazel build cyber/examples/…`
  * 运行：`./bazel-bin/cyber/examples/record`
  * 测试结果：

```
I1124 16:56:27.248200 15118 record.cc:64] [record] msg[0]-> channel name: /test/channel1; content: abc0; msg time: 888
I1124 16:56:27.248227 15118 record.cc:64] [record] msg[1]-> channel name: /test/channel1; content: abc1; msg time: 889
I1124 16:56:27.248239 15118 record.cc:64] [record] msg[2]-> channel name: /test/channel1; content: abc2; msg time: 890
I1124 16:56:27.248252 15118 record.cc:64] [record] msg[3]-> channel name: /test/channel1; content: abc3; msg time: 891
I1124 16:56:27.248297 15118 record.cc:64] [record] msg[4]-> channel name: /test/channel1; content: abc4; msg time: 892
I1124 16:56:27.248378 15118 record.cc:64] [record] msg[5]-> channel name: /test/channel1; content: abc5; msg time: 893
...
I1124 16:56:27.250422 15118 record.cc:73] [record] static msg=================
I1124 16:56:27.250434 15118 record.cc:74] [record] MSG validmsg:totalcount: 100:100
```

## <a id="C++API词典">C++ API词典</a>

### <a id="节点（Node）">节点（Node）API</a>

更多信息和样例参见[节点（Node）](cyber-rt-api-tutorial.md#讲话者（Talker）和倾听者（Listener）)

### API列表

```cpp
//create writer with user-define attr and message type
auto CreateWriter(const proto::RoleAttributes& role_attr)
    -> std::shared_ptr<transport::Writer<MessageT>>;
//create reader with user-define attr, callback and message type
auto CreateReader(const proto::RoleAttributes& role_attr,
    const croutine::CRoutineFunc<MessageT>& reader_func)
    -> std::shared_ptr<transport::Reader<MessageT>>;
//create writer with specific channel name and message type
auto CreateWriter(const std::string& channel_name)
    -> std::shared_ptr<transport::Writer<MessageT>>;
//create reader with specific channel name, callback and message type
auto CreateReader(const std::string& channel_name,
    const croutine::CRoutineFunc<MessageT>& reader_func)
    -> std::shared_ptr<transport::Reader<MessageT>>;
//create reader with user-define config, callback and message type
auto CreateReader(const ReaderConfig& config,
                  const CallbackFunc<MessageT>& reader_func)
    -> std::shared_ptr<cybertron::Reader<MessageT>>;
//create service with name and specific callback
auto CreateService(const std::string& service_name,
    const typename service::Service<Request, Response>::ServiceCallback& service_calllback)
    -> std::shared_ptr<service::Service<Request, Response>>;
//create client with name to send request to server
auto CreateClient(const std::string& service_name)
    -> std::shared_ptr<service::Client<Request, Response>>;
```

## <a id="写入者（Writer）">写入者（Writer）API</a>

更多信息及样例参见[写入者（Writer）](cyber-rt-api-tutorial.md#讲话者（Talker）和倾听者（Listener）)

### API列表

```cpp
bool Write(const std::shared_ptr<MessageT>& message);
```

## <a id="客户端（Client）">客户端（Client）API</a>

更多信息及样例参见[客户端（Client）](cyber-rt-api-tutorial.md#服务端（Service）的创建与使用)

### API列表

```cpp
SharedResponse SendRequest(SharedRequest request,
                           const std::chrono::seconds& timeout_s = std::chrono::seconds(5));
SharedResponse SendRequest(const Request& request,
                           const std::chrono::seconds& timeout_s = std::chrono::seconds(5));
```

## <a id="参数服务（Parameter）">参数服务（Parameter）API</a>

用户进行参数相关操作的接口：

  * 设置参数相关API
  * 读取参数相关API
  * 创建一个参数服务（ParameterService）来为其他节点提供参数服务相关APIs
  * 创建一个参数客户端（ParameterClient）来使用其他节点提供的参数

更多信息及样例参见[参数服务（Parameter）](cyber-rt-api-tutorial.md#参数（Param）服务)

### API列表-设置参数

```cpp
Parameter();  // Name is empty, type is NOT_SET
explicit Parameter(const Parameter& parameter);
explicit Parameter(const std::string& name);  // Type is NOT_SET
Parameter(const std::string& name, const bool bool_value);
Parameter(const std::string& name, const int int_value);
Parameter(const std::string& name, const int64_t int_value);
Parameter(const std::string& name, const float double_value);
Parameter(const std::string& name, const double double_value);
Parameter(const std::string& name, const std::string& string_value);
Parameter(const std::string& name, const char* string_value);
Parameter(const std::string& name, const std::string& msg_str,
          const std::string& full_name, const std::string& proto_desc);
Parameter(const std::string& name, const google::protobuf::Message& msg);
```

### API列表-读取参数

```cpp
inline ParamType type() const;
inline std::string TypeName() const;
inline std::string Descriptor() const;
inline const std::string Name() const;
inline bool AsBool() const;
inline int64_t AsInt64() const;
inline double AsDouble() const;
inline const std::string AsString() const;
std::string DebugString() const;
template <typename Type>
typename std::enable_if<std::is_base_of<google::protobuf::Message, Type>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_integral<Type>::value && !std::is_same<Type, bool>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_floating_point<Type>::value, Type>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_convertible<Type, std::string>::value, const std::string&>::type
value() const;
template <typename Type>
typename std::enable_if<std::is_same<Type, bool>::value, bool>::type
value() const;
```

### API列表-创建参数服务

```cpp
explicit ParameterService(const std::shared_ptr<Node>& node);
void SetParameter(const Parameter& parameter);
bool GetParameter(const std::string& param_name, Parameter* parameter);
bool ListParameters(std::vector<Parameter>* parameters);
```

### API列表-创建参数客户端

```cpp
ParameterClient(const std::shared_ptr<Node>& node, const std::string& service_node_name);
bool SetParameter(const Parameter& parameter);
bool GetParameter(const std::string& param_name, Parameter* parameter);
bool ListParameters(std::vector<Parameter>* parameters);
```

## <a id="定时器（Timer）">定时器（Timer）API</a>

可以设定定时器参数并调用start和stop接口来开始或停止定时器，更多信息及样例参见[定时器（Timer）](cyber-rt-api-tutorial.md#定时器（Timer）)

### API列表

```cpp
Timer(uint32_t period, std::function<void()> callback, bool oneshot);
Timer(TimerOption opt);
void SetTimerOption(TimerOption opt);
void Start();
void Stop();
```

## <a id="时间（Time）">时间（Time）API</a>

更多信息及样例详见[时间（Time）](cyber-rt-api-tutorial.md#时间（Time）)

### API列表

```cpp
static const Time MAX;
static const Time MIN;
Time() {}
explicit Time(uint64_t nanoseconds);
explicit Time(int nanoseconds);
explicit Time(double seconds);
Time(uint32_t seconds, uint32_t nanoseconds);
Time(const Time& other);
static Time Now();
static Time MonoTime();
static void SleepUntil(const Time& time);
double ToSecond() const;
uint64_t ToNanosecond() const;
std::string ToString() const;
bool IsZero() const;
```

## <a id="持续时间（Duration）">持续时间（Duration）API</a>

间隔相关接口，用于指示时间间隔，能够初始化为特定的纳秒或秒

### API列表

```cpp
Duration() {}
Duration(int64_t nanoseconds);
Duration(int nanoseconds);
Duration(double seconds);
Duration(uint32_t seconds, uint32_t nanoseconds);
Duration(const Rate& rate);
Duration(const Duration& other);
double ToSecond() const;
int64_t ToNanosecond() const;
bool IsZero() const;
void Sleep() const;
```

## <a id="速率（Rate）">速率（Rate）API</a>

频率接口通常用于初始化以特定频率初始化的对象后的睡眠频率时间

### API列表

```cpp
Rate(double frequency);
Rate(uint64_t nanoseconds);
Rate(const Duration&);
void Sleep();
void Reset();
Duration CycleTime() const;
Duration ExpectedCycleTime() const { return expected_cycle_time_; }
```

## <a id="记录读取者（RecordReader）">记录读取者（RecordReader）API</a>

读取数据记录文件的接口，用于读取数据记录文件中的消息及通道信息

### API列表

```cpp
RecordReader();
bool Open(const std::string& filename, uint64_t begin_time = 0,
          uint64_t end_time = UINT64_MAX);
void Close();
bool ReadMessage();
bool EndOfFile();
const std::string& CurrentMessageChannelName();
std::shared_ptr<RawMessage> CurrentRawMessage();
uint64_t CurrentMessageTime();
```

## <a id="记录写入者（RecordWriter）">记录写入者（RecordWriter）API</a>

写入数据记录文件的接口，用于记录消息及通道信息到数据记录文件中

### API列表

```cpp
RecordWriter();
bool Open(const std::string& file);
void Close();
bool WriteChannel(const std::string& name, const std::string& type,
                  const std::string& proto_desc);
template <typename MessageT>
bool WriteMessage(const std::string& channel_name, const MessageT& message,
                  const uint64_t time_nanosec,
                  const std::string& proto_desc = "");
bool SetSizeOfFileSegmentation(uint64_t size_kilobytes);
bool SetIntervalOfFileSegmentation(uint64_t time_sec);
```
