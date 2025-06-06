
# CSM、EFI、UEFI、ESP、Windows boot manager的概念与作用

本文转载自如下:

- [CSM、EFI、UEFI、ESP、Windows boot manager的概念与作用](https://www.zhihu.com/tardis/bd/art/1901198379981182382?source_id=1001)

# 1. CSM(Compatibility Support Module)

是BIOS中的一个功能模块，全称为`兼容性支持模块`。它主要用在现代的UEFI(统一可扩展固件接口）固件中提供对传统BIOS启动模式的支持。

它允许UEFI固件模拟传统BIOS行为，支持基于MBR分区表的Legacy BIOS启动方式。例如，在UEFI主板上安装Windows 7等旧系统时，需要启用CSM以识别MBR格式的引导记录。

## 1.1 CSM的主要作用

1. 兼容传统启动模式


1. 支持老旧操作系统


1. 保障硬件兼容性
