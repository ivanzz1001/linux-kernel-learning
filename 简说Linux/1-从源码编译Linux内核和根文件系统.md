# 从源码编译Linux内核

本文参考如下：

- [linux内核开发第1讲：从源码编译 linux-4.9.229 内核和 busybox 文件系统](https://www.bilibili.com/video/BV1Vo4y117Xx/?spm_id_from=333.788)

- [带你阅读linux内核源码：下载源码、编译内核并运行一个最小系统](https://www.bilibili.com/read/cv7118525/?spm_id_from=333.999.0.0)

- [深入了解Linux内核编译与管理](https://zhuanlan.zhihu.com/p/638449345)

- [Linux内核配置与编译步骤(超详细)](https://zhuanlan.zhihu.com/p/475992952?utm_id=0)

- [Linux内核配置以及Make menuconfig过程分析](https://blog.csdn.net/lizuobin2/article/details/51429937)

- [源码菜单配置](https://blog.csdn.net/Ybc_csdn/article/details/130892594)


本文编译所使用的环境如下：

* Ubuntu版本: Ubuntu22.04



## 1. 下载指定版本Linux内核源代码

Linux内核源代码可以到`https://mirrors.edge.kernel.org/pub/linux/kernel/`下载，这里我们下载`4.9.229`版本：
<pre>
# wget https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.9.229.tar.gz
# ls -alh
total 136M
drwxr-xr-x 2 root root 4.0K  3月 24 16:17 .
drwxr-xr-x 6 root root 4.0K  3月 24 16:16 ..
-rw-r--r-- 1 root root 136M  7月  1  2020 linux-4.9.229.tar.gz


# tar -zxvf linux-4.9.229.tar.gz
# cd linux-4.9.229/
# ls
arch     crypto         include  kernel       net             security
block    Documentation  init     lib          README          sound
certs    drivers        ipc      MAINTAINERS  REPORTING-BUGS  tools
COPYING  firmware       Kbuild   Makefile     samples         usr
CREDITS  fs             Kconfig  mm           scripts         virt
</pre>

源代码目录的相关介绍如下：

* arch：与硬件平台有关的选项，大部分指的是CPU的类别，例如 x86，x86_64，Xen虚拟支持等；

* block：与区块设备较相关的设置数据，区块数据通常指的是大量存储媒体，还包括ext3等文件系统的支持是否允许等；

* crypto：内核所支持的加密技术，如 md5或des等；

* Documention：与内核有关的帮助文档；

* drivers：一些硬件的驱动程序，如显卡、网卡、PCI相关硬件；

* firmware：一些旧式硬件的微指令（固件）数据；

* fs：内核所支持的filesystems，如vfat、nfs等；

* include：一些可让其他程序调用的头（header）定义数据；

* init：一些内核初始化的定义功能，包括挂载与init程序的调用等(ps: start_kernel()函数就在此包中)；

* ipc：定义Linux操作系统内各程序的通信；

* kernel：定义内核的程序、内核状态、线程、程序的调度（schedule）、程序的信号（signal）等；

* lib：一些函数库；

* mm：与内存单元有关的各种信息，包括swap与虚拟内存等；

* net：与网络有关的各项协议信息，还有防火墙（net/ipv4/netfilter/*）等；

* security：包括SELinux等安全性设置；

* sound：与音效有关的各项模块；

* virt：与虚拟化及其有关的信息，目前内核支持的是KVM（Kernel base Virtual Machine）

## 2. 前期准备

这里根据自己编译过程中遇到的相关问题可能需要预先安装如下包或工具:
<pre>
# apt install make
# apt install libncurses5-dev  
</pre>


## 3. 内核的配置

内核的目的在管理硬件与提供系统内核功能，因此你必须要先找到系统硬件，并且规划主机的未来任务，这样才能够编译出适合这部主机的内核。所以，真个内核编译的重要工作就是在 “挑选我们想要的功能”。

### 3.1 硬件环境查看与内核功能要求
<pre>
# lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         45 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  2
  On-line CPU(s) list:   0,1
Vendor ID:               GenuineIntel
  Model name:            11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz
    CPU family:          6
    Model:               140
    Thread(s) per core:  1
    Core(s) per socket:  1
    Socket(s):           2
    Stepping:            1
    BogoMIPS:            5606.39
    Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx pdpe1gb rdtscp lm co
                         nstant_tsc arch_perfmon rep_good nopl xtopology tsc_reliable nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4
                         _1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_
                         single ssbd ibrs ibpb stibp ibrs_enhanced fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx5
                         12ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves arat avx512vbmi umip pku ospke avx512_vbmi2 
                         gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq rdpid movdiri movdir64b fsrm avx512_vp2intersect md_clear flush_l1d 
                         arch_capabilities
Virtualization features: 
  Hypervisor vendor:     VMware
  Virtualization type:   full
Caches (sum of all):     
  L1d:                   96 KiB (2 instances)
  L1i:                   64 KiB (2 instances)
  L2:                    2.5 MiB (2 instances)
  L3:                    24 MiB (2 instances)
NUMA:                    
  NUMA node(s):          1
  NUMA node0 CPU(s):     0,1
Vulnerabilities:         
  Gather data sampling:  Unknown: Dependent on hypervisor status
  Itlb multihit:         KVM: Mitigation: VMX unsupported
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Mmio stale data:       Not affected
  Retbleed:              Not affected
  Spec rstack overflow:  Not affected
  Spec store bypass:     Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:            Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:            Mitigation; Enhanced / Automatic IBRS, IBPB conditional, RSB filling, PBRSB-eIBRS SW sequence
  Srbds:                 Not affected
  Tsx async abort:       Not affected


# dmidecode | grep "Base Board Information"          //主板
Base Board Information

# lspci | grep Ethernet                             //网卡
02:01.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)

# lspci | grep vga                                  //显卡
</pre>


### 3.2 保持干净的源代码：make mrproper

了解了硬件相关的信息后，我们还得要处理一下内核源代码下面的残留文件才行。可以通过以下命令在此目录下处理这些“编译过程的目标文件（*.o）以及设置文件”:
<pre>
# make help
Cleaning targets:
  clean           - Remove most generated files but keep the config and
                    enough build support to build external modules
  mrproper        - Remove all generated files + config + various backup files
  distclean       - mrproper + remove editor backup and patch files

Configuration targets:
  config          - Update current config utilising a line-oriented program
  nconfig         - Update current config utilising a ncurses menu based
                    program
  menuconfig      - Update current config utilising a menu based program
  xconfig         - Update current config utilising a Qt based front-end
  gconfig         - Update current config utilising a GTK+ based front-end
  oldconfig       - Update current config utilising a provided .config as base
  localmodconfig  - Update current config disabling modules not loaded
  localyesconfig  - Update current config converting local mods to core
  silentoldconfig - Same as oldconfig, but quietly, additionally update deps
  defconfig       - New config with default from ARCH supplied defconfig
  savedefconfig   - Save current config as ./defconfig (minimal config)
  allnoconfig     - New config where all options are answered with no
  allyesconfig    - New config where all options are accepted with yes
  allmodconfig    - New config selecting modules when possible
  alldefconfig    - New config with all symbols set to default
  randconfig      - New config with random answer to all options
  listnewconfig   - List new options
  olddefconfig    - Same as silentoldconfig but sets new symbols to their
                    default value
  kvmconfig       - Enable additional options for kvm guest kernel support
  xenconfig       - Enable additional options for xen dom0 and guest kernel support
  tinyconfig      - Configure the tiniest possible kernel

# make mrproper
</pre>


### 3.3 配置内核

1) 指定CPU arch

这里我们编译出来的内核是需要运行在x86上面的，因此这里执行如下命令导出ARCH环境变量:
<pre>
# export ARCH=x86
</pre>
>ps: 如果我们不想使用此方式导出ARCH环境变量，那么我们可以在执行make命令时显示传递ARCH值。例如 make ARCH=x86 menuconfig


2) 配置board config

当我们在执行make menuconfig时，其会执行scripts/kconfig/Makefile => arch/$(ARCH)/Makefile, 然后根据一定的规则来自动选择board。因为在Makefile自动推导board类型时会假设我们要编译出的内核是要运行在当前编译机上，而事实上我们可能需要运行在平台上，因此这里我们直接手动指定board类型即可:
<pre>
# make x86_64_defconfig 
</pre>
执行完成后会在源代码根目录生成`.config`配置文件。



3) 对x86_64_defconfig进行微调

这一步其实是对第2步的菜单进行微调，我们需要内核支持ramdisk驱动，所以需要选中如下配置:

>ps: 在执行menuconfig的时候，根目录下的Makefile文件如果发现有`.config`文件(参看KCONFIG_CONFIG变量设置)，那么就会加载该配置文件，此时就相当于对第2步的菜单进行微调

`
General setup  --->
      ----> [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support

Device Drivers  --->
    [*] Block devices  --->
            <*>   RAM block device support
            (65536) Default RAM disk size (kbytes) 
`
按上述修改之后记得保存。

>ps: 关于RAM filesystem的作用，参看
>
> - [initramfs 在内核中的作用与实现](https://blog.csdn.net/song_lee/article/details/106027410)
> - [Linux引导启动过程详细分析](https://zhuanlan.zhihu.com/p/567076094)


题外话，menuconfig的大体菜单样式如下，这里我们简单看一下：
`
# make menuconfig
[*] 64-bit kernel
	General setup  --->
[*] Enable loadable module support  --->
[*] Enable the block layer  --->
	Processor type and features  --->
	Power management and ACPI options  --->
	Bus options (PCI etc.)  --->
	Executable file formats / Emulations  --->
[*] Networking support  --->
	Device Drivers  --->
	Firmware Drivers  --->
	File systems  --->
	Kernel hacking  --->
	Security options  --->
-*- Cryptographic API  --->
[*] Virtualization  ---> 
	Library routines  --->
`

## 4. 编译
这里直接执行make命令编译即可，编译成功后的内核位于：arch/x86_64/boot/bzImage 
``
# make 
# ls -ahl arch/x86_64/boot/ 
total 8.0K
drwxr-xr-x 2 root root 4.0K  3月 24 21:25 .
drwxr-xr-x 3 root root 4.0K  3月 24 21:25 ..
lrwxrwxrwx 1 root root   22  3月 24 21:25 bzImage -> ../../x86/boot/bzImage

# ls -alh arch/x86/boot/bzImage 
-rw-r--r-- 1 root root 6.6M  3月 24 21:25 arch/x86/boot/bzImage
``







