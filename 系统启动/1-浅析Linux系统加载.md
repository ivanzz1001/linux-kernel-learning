# 浅析Linux系统加载

文章的大部分内容参考自:

- [浅析linux系统加载: 从CPU上电到用户态，讲讲BIOS、UEFI、MBR引导、GRUB引导](https://zhuanlan.zhihu.com/p/10518502692)

对里面的部分图进行了重新制作，在段落安排及叙述方面也做了一些调整。

# 1. 整体介绍

由于解密固件方面的逆向需求，所以需要了解Linux系统加载机制。花了好几天来来回回缕清关系和学习，学的时候还是慎用语言大模型不然容易被它绕进去。

本文默认读者学习过操作系统等课堂上能学到的知识，尝试使用通俗的语言+各个地方的截图+常见疑惑，结合来描述。

从CPU上电到用户态，大概流程如下：


# 2. BIOS与UEFI对比

`BIOS`和`UEFI`是两种用于初始化硬件并引导操作系统的固件接口。它们运行在主板的芯片上，是操作系统启动前最先执行的软件。

## 2.1 基本定义

- **BIOS(Basic Input/Output System)**

    - 启动方式：传统启动（Legacy Boot）
  
    - 引导位置：MBR（主引导记录）
  
    - 用户界面：文字界面，键盘操作
  
    - 硬盘支持：最大支持 2 TB，最多 4 个主分区
  
    - 执行文件：bootloader（如 GRUB）
  
    - 启动速度：较慢
  
    - 安全特性：基本无

- **UEFI(Unified Extensible Firmware Interface)**

    - 启动方式：支持现代启动（UEFI Boot）
 
    - 引导位置：GPT（GUID 分区表）

    - 用户界面：图形界面，支持鼠标
 
    - 硬盘支持：支持超过 2 TB 和 128+ 分区
 
    - 执行文件: `.efi` 可执行文件（如`grubx64.efi`）
 
    - 启动速度: 更快
 
    - 安全特性：支持 Secure Boot（防止恶意软件启动）
 
## 2.2 启动流程对比

- **BIOS启动流程**

    - 上电后执行BIOS固件
 
    - 检查硬件(POST: Power On Self Test)
 
    - 读取 MBR（前 512 字节）并跳转到启动加载器
 
    - 加载 OS（如通过 GRUB 加载 Linux） 

- **UEFI启动流程**

    - 执行 UEFI 固件
 
    - 初始化硬件
 
    - 从 EFI 系统分区（ESP）加载 `.efi` 文件（如`grubx64.efi`）
 
    - 加载 OS
 
## 2.3 启动分区区别

```text
类型       分区类型                            描述
-------------------------------------------------------------------------------
BIOS	    MBR                          启动记录写在磁盘开头的 MBR 中

UEFI	GPT + EFI 系统分区（ESP           .efi 文件位于 EFI/ 路径中（通常是 FAT32）
```

## 2.4 如何判断系统是 BIOS 还是 UEFI？

1. 在Linux系统中

  ```bash
  # [ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"
  ```

2. 在 Windows 系统中

    - `msinfo32` 命令查看“BIOS 模式”：显示为“UEFI”或“Legacy”

## 2.5 常见用途对比


| 功能/特性         | BIOS  | UEFI                                    |
| ---------------- | --------- | ----------------------------------- |
| 安装 Windows 11   | ❌        | ✅ （必须启用 UEFI + Secure Boot）     |
| 多系统启动         | 较麻烦     | 易管理，支持 EFI Boot Manager         |
| 自动修复启动       | 不支持     | 支持 EFI Shell、备份启动项             |
| 启动自定义工具      | 需特殊处理 | 可直接引导 EFI 工具（如 `rEFInd`）      |


## 2.6 总结

- 对于10年前旧设备：推荐BIOS 启动

- 对于新设备、Win11、Linux 现代发行版： 推荐UEFI 启动（配 GPT)
