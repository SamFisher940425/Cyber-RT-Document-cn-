# 常见问答

## 阿波罗Cyber RT是什么？

阿波罗Cyber RT是一个专门为自动驾驶场景设计的开源的运行框架。在集中计算模型的基础上，它在性能、延迟和数据吞吐量方面进行了高度优化。

## 我们为什么要再一个新的运行框架下开发？

  * 自动驾驶技术多年来的发展中，我们从阿波罗前期的经验中学到了很多。在自动驾驶场景下，我们需要一个高效的集中计算模型，它需要有极高的性能，包括高并发性、低延迟和高吞吐量。

  * 阿波罗也正随着整个行业不断发展。展望未来，阿波罗已经从开发转向生产，随着现实世界中的大量部署，我们看到了对最高的鲁棒性和高性能的需求。这就是我们花费数年时间打造阿波罗Cyber RT的原因，它满足了自动驾驶解决方案的需求。

## 这个新的运行框架有何优势？

* 加速开发
  * 定义良好的数据融合任务接口
  * 一系列开发工具
  * 大量的传感器驱动
* 简化开发
  * 高效且适应性强的通信
  * 具有资源意识的可配置的用户级调度程序
  * 可移植、依赖更少
* 赋能您自己的自动驾驶车辆
  * 默认开源的运行框架
  * 专为自动驾驶设计的编译模块
  * 即插即用的自动驾驶系统

## 我们还能使用已经采集好的数据吗？

* 如果您收集的数据与阿波罗以前的版本兼容，您可以使用我们推荐的转换工具使数据与我们新的运行框架兼容。

* 如果您创建了自定义的数据格式，则新的运行框架将不支持以前生成的数据。

## 你们还会继续支持ROS吗？

我们将继续支持以前基于ROS的阿波罗版本（3.0以前的版本）。我们非常感谢您继续与我们共同成长，并非常建议您移植到阿波罗3.5。虽然我们知道一些开发者更愿意在ROS上工作，但我们希望您能够理解阿波罗作为一个团队不在未来版本中继续支持ROS的原因，因为我们正在努力开发一个符合汽车标准的更加全面的平台。

## 阿波罗Cyber RT会影响常规的代码开发吗？

如果您在运行框架层没有任何修改，并只在阿波罗模块代码的基础上工作，您将不会受到新的运行框架的影响，因为大多数时候您只需要重新接入输入输出数据的访问。在[Cyber](https://github.com/ApolloAuto/apollo/tree/master/docs/cyber/)github的其他文档中还有更多的细节。

## 阿波罗Cyber RT的推荐设置

* 目前运行框架只支持在Trusty（Ubuntu 14.04）上运行。（译者注：阿波罗5.5版本官方推荐Ubuntu 18.04）
* 运行框架仍使用阿波罗Docker环境
* 建议在打开一个新终端时候“source”一下“setup.bash”文件
* Fork并克隆在[apollo/cyber](https://github.com/ApolloAuto/apollo/tree/master/cyber/)目录下带有新框架代码的阿波罗的仓库

## 如何使能SHM来降低延迟

为了减少线程数，Cyber RT中更改了共享内存的可读指示机制。默认的机制是UDP多播，系统调用（sendto）会导致一定的延迟。

因此，要减少延迟，可以更改机制，步骤如下：

1. 更新Cyber RT到最新版本；
2. 在https://github.com/ApolloAuto/apollo/blob/master/cyber/conf/cyber.pb.conf 中取消对***transport_conf***的注释；
3. 把***shm_conf***的***notifier_type***从“multicast”改到“condition”；
4. 使用后缀选项编译Cyber RT，如`bazel build -c opt --copt=-fpic //cyber/...`；
5. 运行talker和listener；

注：您可以根据节点间的关系选择相应的传输方式。例如，在进程中默认的配置是进程内***INTRA***、主机进程间***SHM***、主机间***RTPS***。当然您可以将三者都改成RTPS。或者将`same_proc`和`diff_proc`改成***SHM***;

## 如何使用无序列化消息？

Cyber RT支持的消息类型包括序列化结构化数据，例如protobuf，以及原始字节序列。您可以参考示例代码：

* apollo::cyber::message::RawMessage
* talker: https://github.com/gruminions/apollo/blob/record/cyber/examples/talker.cc
* listener: https://github.com/gruminions/apollo/blob/record/cyber/examples/listener.cc

## 如何配置多主机通信？

确保两台主机（或多台主机）位于局域网中的相同网段内，例如***192.168.10.6***和***192.168.10.7***。

您只需要更改`/apollo/cyber/setup.bash`中的`CYBER_IP`：

```
export CYBER_IP=127.0.0.1
```

假设您有两台主机A和B，A的IP是192.168.10.6，B的IP是192.168.10.7。然后在主机A上将CYBER_IP设置为192.168.10.6，在主机B上将CYBER_IP设置为192.168.10.7。现在主机A可以与主机B通信了。

更多的常见问答待续……

