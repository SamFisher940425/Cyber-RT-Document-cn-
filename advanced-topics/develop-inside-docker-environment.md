# 在Docker环境中开发

为简化工作，阿波罗Cyber RT发布了一个Docker镜像以及一些脚本来帮助开发者构建和应用Cyber RT。

Cyber RT官方Docker镜像是基于Ubuntu 18.04构建的，它提供了构建Cyber RT及其驱动的全部支持，因此如果您只对Cyber RT感兴趣，这将是一个非常理想的起点。

**注：最近添加了ARM平台支持并与Apollo开发环境完全集成。您将可以在x86以及ARM平台上使用相同的脚本开发Cyber RT。然而，由于ARM平台仅在Nvidia Driver PX上验证过，因此若您在Driver PX以外的其他开发板上尝试时遇到问题，请联系我们。**

一下几节将展示如何开始使用Cyber RT Docker并从头构建您自己的Docker镜像

## 在Docker中构建并测试Cyber RT

启动官方的Cyber RT Docker，您首先需要运行以下几个命令：

注：首次运行这些命令可能需要花费一定的时间，因为您将下载整个Docker镜像，这取决于您的网络带宽

注：如果您之前运行过这个命令，您将丢失之前Docker中所做的全部更改。除非您想启动一个新的Docker环境。

`./docker/scripts/cyber_start.sh`

进入Docker环境中您只需启动以下命令：

注：只要您愿意，您可以多次进入和退出，Docker环境将一直存在，直到您重启Docker。

`./docker/scripts/cyber_into.sh`

仅构建Cyber RT并测试：

`./apollo.sh build_cyber
bazel test cyber/...`

您应当在开发自己的工程前通过所有的测试

在Cyber RT中构建驱动

`./apollo.sh build_drivers`

**注：仅适用于ARM平台的启动说明**

由于Docker在Driver PX平台上的一些限制，您需要在上面的过程后执行以下步骤

首次运行cyber_into.sh后，进入Cyber RT容器，运行以下两条命令：

`/apollo/scripts/docker_adduser.sh
su nvidia`

**退出时，请使用`ctrl+p ctrl+q`而不是`exit`**，否则您将丢失容器中运行的程序

**注：仅适用于ARM平台的说明结束**

## 构建Cyber RT Docker镜像

为构建您自己的Docker镜像，在x86平台上请运行以下命令：

`cd docker/build/
./build_cyber.sh cyber.x86_64.dockerfile`

ARM平台下：

`cd docker/build/
./build_cyber.sh cyber.aarch64.dockerfile`

由于ARM平台的性能，为节省时间，您可以通过以下选项去下载一些预编译的包

`cd docker/build/
./build_cyber.sh cyber.aarch64.dockerfile download`

如果想要长时间保存Docker镜像，使用以下示例命令将镜像储存在您个人的Docker仓库中，使用`docker images`查看您自己的镜像名称

`docker push [YOUR_OWN_DOCKER_REGISTRY]:cyber-x86_64-18.04-20190531_1115`

