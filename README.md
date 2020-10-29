---
description: 百度阿波罗（Apollo）Cyber-RT中文版文档。由爱好者私人翻译，允许任何非商用的转载。
description: 原文链接：[https://cyber-rt.readthedocs.io/en/latest/CyberRT_Python_API.html] (https://cyber-rt.readthedocs.io/en/latest/CyberRT_Python_API.html)
---

# 介绍

阿波罗Cyber RT是一个特别为自动驾驶场景设计的开源、高性能的运行架构。在集中计算模型的基础上，它针对自动驾驶中高并发性、低延迟和高吞吐量进行了极大的优化。

在近年来自动驾驶技术的发展中，我们从阿波罗早期经验中学到了很多。阿波罗也随着整个行业不断发展。展望未来，阿波罗已经从开发原型转向产品化，随着在真实世界中的批量部署，我们看到了最高级别自动驾驶的鲁棒性和性能上的需求。这也正是我们花费几年时间建立并完善阿波罗Cyber RT的原因，它满足了自动驾驶方案的需求。

使用Cyber RT的关键优势：

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

## 快速入门

* [入门](quick-start/getting-started.md)

* [Cyber RT 术语](quick-start/cyber-rt-terms.md)

* [常见问答](quick-start/f.a.q..md)

## 教程

* [Cyber RT API 教程](tutorial/cyber-rt-api-tutorial.md)

* [Python API 教程](tutorial/python-api-tutorial.md)

* [阿波罗 Cyber RT 开发者工具](tutorial/apollo-cyber-rt-developer-tool.md)

  * [cyber_visualizer 工具](tutorial/apollo-cyber-rt-developer-tool.md#cyber_visualizer工具)

  * [cyber_monitor 工具](tutorial/apollo-cyber-rt-developer-tool.md#cyber_monitor工具)

  * [用cyber_monitor获得与UI界面类似的信息](tutorial/apollo-cyber-rt-developer-tool.md#用cyber_monitor获得与UI界面类似的信息)

  * [cyber_recorder 工具](tutorial/apollo-cyber-rt-developer-tool.md#cyber_recorder工具)

  * [rosbag_to_record 工具](tutorial/apollo-cyber-rt-developer-tool.md#rosbag_to_record工具)
  
## 高级话题

* [在Docker环境中开发](advanced-topics/develop-inside-docker-environment.md)

* [从阿波罗ROS移植指导](advanced-topics/migration-guide-from-apollo-ros.md)

## API参考文档

* [C++ API](api-reference/cpp-api.md)

* [Python API](api-reference/python-api.md)

