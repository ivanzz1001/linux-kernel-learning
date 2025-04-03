
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

## 2.5 内存选择

根据你电脑的内存选择，建议选择低于总内存一半的内存数，否则可能会导致宿主机卡顿。

比如电脑是16GB，建议只选到8GB，否则虚拟机运行占用太多内存，容易弄得笔记本也卡卡的，干不了啥其他事情。

> 这里还需要纠正小白的一部分错误认识，内存是指运行内存，不是你电脑的硬盘容量！！！右键你电脑桌面上的此电脑，点击属性，就能看到你电脑有多少内存。任务管理器中也能看到当前的内存总量。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step08.webp)


现在的电脑都是8G内存打底了，主流价位的笔记本也普及了16GB，所以大家笔记本运行虚拟机都是没问题的。因为我们安装的是带图形化界面的Ubuntu系统，所以内存建议至少设置为4GB，2GB可能会有点卡（请忽略下图中的注释）

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step09.webp)


## 2.6 网络

建议选择NAT模式，这样即便你的宿主机没有连上网络，我们依旧可以在无网络时连接上虚拟机Linux进行学习（前提是提前在有网络情况下配置好了编程环境）

> 如果使用桥接模式，由于它需要从父路由器获取IP地址，是没有办法在宿主机无网络的时候ssh连接上虚拟机的。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step10.webp)

NAT是网络链接的协议之一，常用于路由器上，在本站的IP网络协议博客中有提到这个协议。

## 2.7 IO和磁盘

选择推荐的，不用修改。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step11.webp)

磁盘类型选择vm推荐的就行（不同电脑/虚拟机系统，推荐的可能不一样）

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step12.webp)


![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step13.webp)

磁盘最少选择20GB，根据自己电脑硬盘容量自行选择。选择的硬盘容量并不会立马占满，而是随着使用时长慢慢增加的；所以强烈建议 强烈建议 强烈建议选择一个较大的硬盘值，避免后续空间不够需要扩容。

虚拟磁盘勾选单个文件，多个文件非常不方便我们管理虚拟机。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step14.webp)

磁盘文件名字不用修改。

## 2.8 完成

点击完成按钮，虚拟机就创建好了

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step15.webp)

随后，VMware会自动开启这个虚拟机，进入Ubuntu的配置界面。因为是快速安装，所以这个流程都是全自动的，不需要我们进行任何操作

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step16.webp)


# 3. Ubuntu初始化

随后，我们就会进入Ubuntu的初始化设置界面。

## 3.1 语言设置

Linux学习中，英文永远是首选语言。不建议改成中文。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step17.webp)

选择语言后点击Continue，就会进入下图所示界面，保持默认值就行了，继续点击Continue:

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step18.webp)

## 3.2 安装方式

这里会显示安装方式，默认选中的这个是会擦除硬盘上所有数据（格式化后安装）我们是虚拟机，所以就用这个模式就行了。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step19.webp)

随后会弹出来一个警告，代表这个模式会擦除硬盘数据，不用担心，擦除的是虚拟机硬盘。点击Continue就行了。会开始Ubuntu的安装进度条。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step20.webp)

## 3.3 时区选择

随后会进入时区选择界面，选择东八区Shanghai就行了（国际时区中，我国大陆的时区是用Asia/Shanghai来标定，而不是北京）

## 3.4 用户名和密码

这里又需要输入用户名和密码，最终的用户名和密码是以这里的设定为准的。

这里还可以选择是否自动登录，因为是虚拟机，勾选自动登录会方便一点。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step21.webp)

## 3.5 最终安装流程

随后就会进入最终的安装流程，等进度条跑完。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step22.webp)

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step23.webp)

最终会出现下图所示重启按钮，点击它，就可以了

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step24.webp)

会出现黑屏白字的信息，这里我们不需要操作

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step25.webp)

## 3.6  安装成功

成功进入桌面，下面的这个导引一直点skip就行，我们不需要登录它。

![ubuntu-install](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/1-%E7%BC%96%E8%AF%91Linux%E5%86%85%E6%A0%B8/%E6%90%AD%E5%BB%BA%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/image/ubuntu-install-step26.webp)


**Over！**

安装Ubuntu虚拟机完成，教程结束