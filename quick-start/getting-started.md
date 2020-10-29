# 入门

阿波罗Cyber RT框架是基于组件概念构建的。作为阿波罗Cyber RT框架的基本组成模块，每个组件都包含一个特定的算法模块，该模块处理一组输入数据并产生一组输出数据。

为了成功的创建和启动一个新组件，需要执行四个步骤：

* 建立组件文件结构
* 实现组件类
* 设置配置文件
* 启动组件

下面的示例演示了如何创建一个简单的组件，然后构建、运行并在屏幕上看到最终的输出。如果您想进一步探索阿波罗Cyber RT，您可以在_**/apollo/cyber/examples/**_目录下找到几个展示了如何使用框架的不同功能的示例。

**注:**示例必须在阿波罗的docker环境中运行，它是用Bazel编译的。

## 建立组件的文件结构

请创建以下文件，假定在以下路径：

_**/apollo/cyber/example/common\_component\_example/**_

* 头文件：common\_component\_example.h
* 源文件：common\_component\_example.cc
* 构建文件：BUILD
* DAG依赖文件：common.dag
* 启动文件：common.launch

## 实现组件的类

### 实现组件头文件

实现_**common\_component\_example.h**_需要：

* 继承组件的类
* 定义您自己的_**Init**_和_**Proc**_函数，_**Proc**_函数需要指定其输入数据类型
* 使用_**CYBER\_REGISTER\_COMPONENT**_将您的组件类注册为全局类

```cpp
#include <memory>
#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"
#include "cyber/examples/proto/examples.pb.h"

using apollo::cyber::examples::proto::Driver;
using apollo::cyber::Component;
using apollo::cyber::ComponentBase;

class CommonComponentSample : public Component<Driver, Driver> {
 public:
  bool Init() override;
  bool Proc(const std::shared_ptr<Driver>& msg0,
            const std::shared_ptr<Driver>& msg1) override;
};

CYBER_REGISTER_COMPONENT(CommonComponentSample)
```

### 为示例组件实现源文件

在_**common\_component\_example.cc**_中，_**Init**_函数和_**Proc**_函数需要被实现。

```cpp
#include "cyber/examples/common_component_example/common_component_example.h"
#include "cyber/class_loader/class_loader.h"
#include "cyber/component/component.h"

bool CommonComponentSample::Init() {
  AINFO << "Commontest component init";
  return true;
}

bool CommonComponentSample::Proc(const std::shared_ptr<Driver>& msg0,
                               const std::shared_ptr<Driver>& msg1) {
  AINFO << "Start common component Proc [" << msg0->msg_id() << "] ["
        << msg1->msg_id() << "]";
  return true;
}
```

### 为示例组件创建构建文件

创建Bazel构建文件。

```cpp
load("//tools:cpplint.bzl", "cpplint")

package(default_visibility = ["//visibility:public"])

cc_binary(
    name = "libcommon_component_example.so",
    deps = [":common_component_example_lib"],
    linkopts = ["-shared"],
    linkstatic = False,
)

cc_library(
    name = "common_component_example_lib",
    srcs = [
        "common_component_example.cc",
    ],
    hdrs = [
        "common_component_example.h",
    ],
    deps = [
        "//cyber",
        "//cyber/examples/proto:examples_cc_proto",
    ],
)

cpplint()
```

## 建立配置文件

### 配置DAG依赖文件

要配置DAG依赖文件（common.dag），请指定以下项目：

* 通道名称（name）：用于数据的输入输出
* 库路径（component\_library）：从组件类生成的库
* 类名称（class\_name）：组件的类名称

```cpp
# Define all coms in DAG streaming.
component_config {
    component_library : "/apollo/bazel-bin/cyber/examples/common_component_example/libcommon_component_example.so"
    components {
        class_name : "CommonComponentSample"
        config {
            name : "common"
            readers {
                channel: "/apollo/prediction"
            }
            readers {
                channel: "/apollo/test"
            }
        }
    }
}
```

### 配置启动文件

要配置启动文件（common.launch），请指定以下项目：

* 组件名称
* 前面步骤中创建的DAG依赖文件
* 组件中运行的进程的名称

```cpp
<cyber>
    <component>
        <name>common</name>
        <dag_conf>/apollo/cyber/examples/common_component_example/common.dag</dag_conf>
        <process_name>common</process_name>
    </component>
</cyber>
```

## 启动组件

运行以下命令来构建组件：

`bash /apollo/apollo.sh build`

注：确保示例组件构建完好

然后配置环境：

```cpp
cd /apollo/cyber
source setup.bash
```

有两种方式启动组件：

* 通过启动文件来启动组件（推荐）：

  `cyber_launch start /apollo/cyber/examples/common_component_example/common.launch`

* 通过DAG依赖文件来启动组件：

  `mainboard -d /apollo/cyber/examples/common_component_example/common.dag`

