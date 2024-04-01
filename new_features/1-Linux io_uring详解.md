# Linux io_uring详解

文章参考如下：

- [liburing官网](https://github.com/axboe/liburing)

- [Linux I/O 神器之 io_uring](https://zhuanlan.zhihu.com/p/626799855?utm_id=0)

- [Linux IO 模式之 io_uring](https://zhuanlan.zhihu.com/p/389978597)

- [ovoice: io_uring](https://github.com/0voice/kernel_new_features/tree/main/io_uring)

- [How io_uring and eBPF Will Revolutionize Programming in Linux](https://thenewstack.io/how-io_uring-and-ebpf-will-revolutionize-programming-in-linux/)

- [inux 异步 I/O 框架 io_uring：基本原理、程序示例与性能压测](https://arthurchiao.art/blog/intro-to-io-uring-zh/)

- [Understanding Modern Storage APIs:A systematic study of libaio, SPDK, and io_urin](https://animeshtrivedi.github.io/files/papers/2022-systor-Understanding%20modern%20storage%20APIs:%20a%20systematic%20study%20of%20libaio,%20SPDK,%20and%20io_uring.pdf)

- [xNVMe and io_uring NVMe passthrough](https://storagedeveloper.org/sites/default/files/SDC/2023/presentations/SNIA-SDC23-Lund-xNVMe-and-io-uring-NVMe-passthrough_1.pdf)

- [RingGuard: Guard io_uring with eBPF](https://dl.acm.org/doi/pdf/10.1145/3609021.3609304)

- [eBPF document](https://ebpf.io/what-is-ebpf/)


io_uring是2019年Linux 5.1内核首次引入的高性能异步IO框架，能显著加速IO密集型应用的性能。但如果你的应用已经在使用传统Linux AIO了，并且使用方式恰当，那io_uring并不会带来太大的性能提升————根据原文测试（以及我们自己的复现），即便打开高级特性，也只有5%。除非你真的需要这5%的额外性能，否则切换成io_uring代价可能也挺大，因为需要重写应用来适配io_uring(或者让依赖的平台或框架去适配，总之需要改代码）。

既然性能跟传统 AIO 差不多，那为什么还称 io_uring 为革命性技术呢？

- 首先和最大的贡献在于统一了Linux异步IO框架
    - Linux AIO只支持Direct IO模式的存储文件，而且主要用于数据库这一细分领域；
    
    - io_uring 支持存储文件和网络文件(network sockets)，也支持更多的异步系统调用(accept/openat/stat...)，而非仅限于read/write系统调用
    
- 在设计上是真正的异步 I/O，作为对比，Linux AIO 虽然也是异步的，但仍然可能会阻塞，某些情况下的行为也无法预测。似乎之前 Windows 在这块反而是领先的，更多参考：
    - [浅析开源项目之 io_uring](https://zhuanlan.zhihu.com/p/361955546)  
    - [Is there really no asynchronous block I/O on Linux?](https://stackoverflow.com/questions/13407542/is-there-really-no-asynchronous-block-i-o-on-linux)

- 灵活性和可扩展性非常好，甚至能基于 io_uring 重写所有系统调用，而 Linux AIO 设计时就没考虑扩展性。

eBPF 也算是异步框架（事件驱动），但与 io_uring 没有本质联系，二者属于不同子系统， 并且在模型上有一个本质区别：

- eBPF 对用户是透明的，只需升级内核（到合适的版本），应用程序无需任何改造；

- io_uring 提供了新的系统调用和用户空间 API，因此需要应用程序做改造

eBPF 作为动态跟踪工具，能够更方便地排查和观测 io_uring 等模块在执行层面的具体问题。


----------

本文介绍 Linux 异步 I/O 的发展历史，io_uring 的原理和功能， 并给出了一些程序示例和性能压测结果（我们在 5.10 内核做了类似测试，结论与原文差不多）。

很多人可能还没意识到，Linux 内核在过去几年已经发生了一场革命。这场革命源于 两个激动人心的新接口的引入：eBPF 和 io_uring。 我们认为，二者将会完全改变应用与内核交互的方式，以及 应用开发者思考和看待内核的方式。

本文介绍 io_uring（我们在 ScyllaDB 中有 io_uring 的深入使用经验），并略微提及一下 eBPF。

<br />

## 1. Linux I/O 系统调用演进

### 1.1 基于fd的阻塞式 I/O：read()/write()

作为大家最熟悉的读写方式，Linux 内核提供了基于文件描述符的系统调用， 这些描述符指向的可能是存储文件（storage file），也可能是 network sockets

`
ssize_t read(int fd, void *buf, size_t count);

ssize_t write(int fd, const void *buf, size_t count);
`
二者称为阻塞式系统调用（blocking system calls），因为程序调用 这些函数时会进入 sleep 状态，然后被调度出去（让出处理器），直到 I/O 操作完成：

- 如果数据在文件中，并且文件内容已经缓存在 page cache 中，调用会立即返回；

- 如果数据在另一台机器上，就需要通过网络（例如 TCP）获取，会阻塞一段时间；

- 如果数据在硬盘上，也会阻塞一段时间。

但很容易想到，随着存储设备越来越快，程序越来越复杂， 阻塞式（blocking）已经这种最简单的方式已经不适用了。

### 1.2 非阻塞式 I/O：select()/poll()/epoll()
阻塞式之后，出现了一些新的、非阻塞的系统调用，例如 select()、poll() 以及更新的 epoll()。 应用程序在调用这些函数读写时不会阻塞，而是立即返回，返回的是一个 已经 ready 的文件描述符列表。

![epoll](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/new_features/image/io_uring/epoll.png)

