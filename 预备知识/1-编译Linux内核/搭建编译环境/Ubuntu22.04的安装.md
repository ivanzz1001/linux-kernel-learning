
# Ubuntu22.04的安装

参看如下文章:

- [VMware安装ubuntu-22.04虚拟机 原创](https://blog.51cto.com/musnow/13303531)

# 1. 下载Ubuntu系统镜像

打开阿里云镜像站点：https://developer.aliyun.com/mirror/

找到如图所示位置，选择Ubuntu 22.04.3(destop-amd64)系统

![ubuntu-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/aliyun-ubuntu-download.webp)

![ubuntu-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/aliyun-ubuntu-2204.webp)

Ubuntu 22.04.3(destop-amd64)系统镜像下载连接如下：

```text
https://mirrors.aliyun.com/ubuntu-releases/jammy/ubuntu-22.04.3-desktop-amd64.iso
```
如果你不需要使用图形化界面，可以选择22.04.3(live-server-amd64)版本，这样系统的运行资源消耗会更低。

# 2. 安装系统

## 2.1 创建虚拟机并选择ISO镜像

打开VMware，选择新建虚拟机

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step01.webp)

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step02.webp)

在下图位置选择刚刚下载好的系统ISO镜像:

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step03.webp)

VMware中提供了针对Ubuntu的快速安装功能，我们直接使用此功能，就能在基本不需要做任何操作的情况下安装好虚拟机。

## 2.2 配置主机名,用户名和密码

因为是快速安装，所以我们只需要配置自己的主机名、用户名和密码就可以了:

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step04.webp)

## 2.3 安装路径选择

安装路径是虚拟机的文件存放路径，这里请选择你的电脑上剩余空间较多的磁盘，并在内部创建一个新的文件夹来存放虚拟机文件。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step05.webp)

## 2.4 CPU核心数选择

虚拟机CPU核数请根据你的电脑的CPU来选择，比如我的笔记本CPU是8核16线程的（ctrl+alt+delete打开你电脑的任务管理器来查看）

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step06.webp)

因为我的笔记本是8核16线程的，所以这里我选择了4核，内核数量选择2，最终生成的虚拟机就可以认为是一个4核8线程的主机。

更复杂的配置选择逻辑我就不提了，只需要满足一个原则，虚拟机的处理器配置中最终的处理器内核总数需要少于你的笔记本任务管理器中显示的逻辑处理器数量。

>ps: 如果你不知道怎么选择，那就设置处理器数量为2，每个处理器的内核数量为2即可。我们的学习对CPU的性能要求并不高

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step07.webp)