# 阿波罗 Cyber RT 开发者工具

阿波罗Cyber RT框架提供了一系列用于日常开发的有用工具，包括一个可视化工具***cyber_visualizer***和两个命令行工具***cyber_monitor***与***cyber_recorder***

注：使用这些工具需要基于阿波罗Docker环境，请遵照阿波罗wiki进入正确的Docker容器

阿波罗Cyber RT中所有的工具都依赖阿波罗Cyber RT库，因此在使用阿波罗Cyber RT之前必须“source”目录中的“setup.bash”文件，请参考如下命令：

`username@computername:~$: source /your-path-to-apollo-install-dir/cyber/setup.bash`

## cyber_visualizer 工具

### 安装与运行

***cyber_visualizer***是为了展示阿波罗Cyber RT中各通道数据的可视化工具

```
username@computername:~$: source /your-path-to-apollo-install-dir/cyber/setup.bash
username@computername:~$: cyber_visualizer
```

### 与可视化工具交互

* 在运行cyber_visualizer之后，您将看到以下界面：

![Cyber RT 界面1](/images/cyber_visualizer1.png)

* 在Cyber RT中，当数据流通过通道传输，所有通道的列表将会在***ChannelNames***中展示，如下图所示。例如，您可以看到Cyber RT的数据记录工具（cyber_recorder）在另一个终端中回放数据，***cyber_visualizer***将接收所有活动通道的信息（从回放数据中）并展示它们。

![Cyber RT 界面2](/images/cyber_visualizer2.png)