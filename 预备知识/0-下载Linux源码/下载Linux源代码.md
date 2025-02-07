# 如何下载Linux源码，看这篇就够了！

在工作中，我们难免会遇到需要去找某个版本的linux源码的情况，今天这篇文章就手把手教大家如何找到自己想要的linux源码版本。

参看：

- [如何下载Linux源码，看这篇就够了！](https://blog.csdn.net/qq_43257914/article/details/134344756)

- [Linux内核版本：探秘版本号的奥秘](https://baijiahao.baidu.com/s?id=1772040171585220927&wfr=spider&for=pc)


## 1. Linux官网

Linux官网：https://www.kernel.org/


## 2. 下载长期维护版本版本

尽量去下载这种带有 longterm 标注的，这种表示长期维护的版本
![kernel-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/0-%E4%B8%8B%E8%BD%BDLinux%E6%BA%90%E7%A0%81/image/kernel-download-1.png)


## 3. 下载指定发布版本
如果你想下载指定的版本，则按下面的步骤:

- 点击 https://www.kernel.org/pub/ 进入发布界面

![kernel-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/0-%E4%B8%8B%E8%BD%BDLinux%E6%BA%90%E7%A0%81/image/kernel-download-2.png)


- 选择linux选项

![kernel-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/0-%E4%B8%8B%E8%BD%BDLinux%E6%BA%90%E7%A0%81/image/kernel-download-3.png)

- 选择kernel，也就是我们想要找到的 linux源码

![kernel-download](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/0-%E4%B8%8B%E8%BD%BDLinux%E6%BA%90%E7%A0%81/image/kernel-download-4.png)

>ps: 这里值得注意的是，我们可以从Historic文件夹下载到一些特别古老的版本，如果想研究的话可以去下载


## 4. Linux内核版本：探秘版本号的奥秘


在Linux世界中，你可能听说过各种奇怪的内核版本号，比如4.4、5.10、5.13等等。这些版本号究竟代表着什么意思呢？本文将带你探秘Linux内核版本号的奥秘，让你了解它们的命名规则和背后的故事。

![kernel-version](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/0-%E4%B8%8B%E8%BD%BDLinux%E6%BA%90%E7%A0%81/image/kernel-version.png)

### 4.1 Linux内核版本号的命名规则
Linux内核的版本号采用X.Y.Z的形式，其中X表示主版本号、Y表示次版本号、Z表示修订号。当内核版本更新时，不同部分的号码会有相应的增加。

- 主版本号(X)：当内核发生重大改变或完全重写时，增加主版本号。例如，从2.x升级到3.x，就意味着内核经历了一次重大的演进。

- 次版本号(Y)：当内核有新功能添加或接口变更时，增加次版本号。例如，从5.11升级到5.12，表示添加了新的功能或进行了一些接口改动。

- 修订号(Z)：当内核有bug修复或小的改进时，增加修订号。例如，从5.13.1升级到5.13.2，表示进行了一些小的修复和改进。


### 4.2 内核版本号的背后故事
Linux内核版本号背后有着丰富的故事和有趣的命名传统。Linux的创始人Linus Torvalds曾经在邮件列表中透露了一些有趣的内核版本号的命名方式。

他表示，偶数的主版本号通常代表稳定版本，例如2.0、4.0、6.0等。而奇数的主版本号则代表开发版本，例如1.1、3.3、5.9等。这种方式让用户可以根据版本号来判断内核的稳定性和可靠性。

此外，每个内核版本的次版本号(Y)通常也会与一个特定的主题相关联，例如，5.8版本的次版本号是“Hello Kitty”。这些有趣的命名方式为内核版本增添了一份乐趣和趣味。


### 4.3 内核版本的选择和更新
对于普通用户来说，选择合适的内核版本是很重要的。通常，发行版的维护者会根据不同的用途和需求选择合适的内核版本，并在系统升级时自动更新内核。

对于高级用户和开发者来说，可能会选择更新的内核版本来体验新的功能和性能优化。他们可以通过源代码或者软件包管理器来安装和更新内核。