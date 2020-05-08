# Cyber RT API 教程

此文档对如何创建、操作和使用Cyber RT的API进行了深入的技术研究。

内容列表

  * [讲话者（Talker）和倾听者（Listener）](tutorial/cyber-rt-api-tutorial.md#讲话者（Talker）和倾听者（Listener）)
  * [服务端（Service）的创建与使用](tutorial/cyber-rt-api-tutorial.md#)
  * [参数（Param）服务](tutorial/cyber-rt-api-tutorial.md#)
  * [记录（Log）API](tutorial/cyber-rt-api-tutorial.md#)
  * [基于组件构建模块](tutorial/cyber-rt-api-tutorial.md#)
  * [启动（Launch）](tutorial/cyber-rt-api-tutorial.md#)
  * [定时器（Timer）](tutorial/cyber-rt-api-tutorial.md#)
  * [定时器（Timer）API](tutorial/cyber-rt-api-tutorial.md#)
  * [数据记录（Record）文件：读取与写入](tutorial/cyber-rt-api-tutorial.md#)
  * [C++ API 词典](tutorial/cyber-rt-api-tutorial.md#)
    * [节点（Node）](tutorial/cyber-rt-api-tutorial.md#)
    * [写入者（Writer）](tutorial/cyber-rt-api-tutorial.md#)
    * [客户端（Client）](tutorial/cyber-rt-api-tutorial.md#)
    * [参数服务器（Parameter）](tutorial/cyber-rt-api-tutorial.md#)
    * [定时器（Timer）](tutorial/cyber-rt-api-tutorial.md#)
    * [时间（Time）](tutorial/cyber-rt-api-tutorial.md#)
    * [持续时间（Duration）](tutorial/cyber-rt-api-tutorial.md#)
    * [速率（Rate）](tutorial/cyber-rt-api-tutorial.md#)
    * [记录读取者（RecordReader）](tutorial/cyber-rt-api-tutorial.md#)
    * [记录写入者（RecordWriter）](tutorial/cyber-rt-api-tutorial.md#)

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

  

