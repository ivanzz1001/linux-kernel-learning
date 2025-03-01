# 使用GNU语法编写HelloWorld程序

文章转载自: 

- [Coder Heart: 使用GNU语法编写HelloWorld程序](https://juejin.cn/post/7198135791234465847)

我们在上一篇学习了GNU汇编的基本语法，本篇我们使用汇编来写一个经典的`HelloWorld`程序。

运行环境:

  - OS: Ubuntu 22.04.4 LTS

  - GCC: gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04) 

在用户空间编写汇编程序输出字符串，有三种方法：

  - 调用C库函数printf()

  - 使用软中断

  - 使用syscall系统调用

下面我们对三种方法，分别进行说明。

## 1. 调用C库函数

为了更好地理解汇编代码，我们先介绍下x86-64架构中函数调用的习惯。

### 1.1 x86-64架构中函数调用的习惯

#### 1.1.1 参数传递

x86-64中，最多允许6个参数通过通用寄存器来传递，多出的参数需要通过栈来传递；参数传递时，参数的顺序与寄存器的关系对应如下：

```text
操作数大小（位）    参数1    参数2    参数3    参数4    参数5    参数6
----------------------------------------------------------------------
64                 %rdi    %rsi     %rdx     %rcx    %r8      %r9
32                 %edi    %esi     %edx     %ecx    %r8d     %r9d
16                 %di     %si      %dx      %cx     %r8w     %r9w
8                  %dil    %sil     %dl      %cl     %r8b     %r9b
```
当参数大于6个时，把超出的参数放到栈上，而参数7位于栈顶。

#### 1.1.2 返回值

被调用函数返回时，把结果放入%rax寄存器中，供调用函数来获取。

#### 1.1.3 栈对齐

根据System V AMD64 ABI文档(下文简称ABI文档)说明（第3.2.2 The Stack Frame节)，在发起call指令之前，栈需要是16字节对齐的。

```text
The end of the input argument area shall be aligned on a 16 (32 or 64, if __m256 or
__m512 is passed on stack) byte boundary. In other words, the stack needs to be 16 (32
or 64) byte aligned immediately before the call instruction is executed.
```

#### 1.1.4 XMM寄存器

根据ABI文档说明（第3.2.3 Parameter Passing节)，当被调用的函数中有浮点数时，需要使用`%xmm0 ~ %xmm7`共8个SSE寄存器来传递参数：

```text
If the class is SSE, the next available vector register is used, the registers are taken

in the order from %xmm0 to %xmm7.
```

另外需要使用`%al`寄存器来指定使用的矢量寄存器的最大数量。

```text
For calls that may call functions that use varargs or stdargs (prototype-less calls or calls
to functions containing ellipsis (. . . ) in the declaration) %al is used as hidden argument
to specify the number of vector registers used. The contents of %al do not need to match
exactly the number of registers, but must be an upper bound on the number of vector
registers used and is in the range 0–8 inclusive.
```

x86-64函数调用习惯，也可以参考维基百科上的文档，地址在这里: [System V AMD64 ABI](https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI);
另外，关于ABI最新文档，可以从这里获取: [x86-64 ABI](https://gitlab.com/x86-psABIs/x86-64-ABI)


### 1.2 打印HelloWorld

代码如下：

```text
.section .data

msg:
    .asciz "Hello world!\n"     # 定义了字符串 'Hello world!'，由于是使用.asciz 定义的，会自动在字符串后面加上字符 '\0'，以满足 C 语言习惯。

.section .text
.globl main
main:
    /* 调用 printf() 函数打印 "Hello world!" */
    /* printf函数原型: int printf(char *fmt, ...) */
    subq $8, %rsp               # 发起 CALL 调用之前，栈必须是对齐到16字节，否则会报 segment fault 错误
    xorq %rax, %rax             # 被调用函数参数中有浮点数时， %al 寄存器中保存的是需要传送到 XMM 寄存器的参数数量
    mov $msg, %rdi              # 字符串地址
    call printf                 # 调用C库函数 printf

    /* return */
    xorq %rax, %rax             # main函数返回值，rax = 0
    addq $8, %rsp               # 恢复原来的栈地址
    ret                         # 从 main 函数返回
```

编译运行：
```bash
# gcc -o helloworld -no-pie helloworld.s
# ./helloworld
Hello world
# echo $?
0
```

需要说明的是，我们在程序运行完成后，使用`echo $?`来检查函数的返回值，这个返回值就是我们调用`ret`指令之前，
`%rax`里保存的值。我们可以把`%rax`里的值改成其他值，比如100(`movq $100, $rax`)来验证下。

> Note: 上面我们在编译时指定了`-no-pie`选项，请参看[can not be used when making a PIE object](https://linuxtut.com/en/4fc41123ed41cf443a6b/)

### 1.3 打印包含浮点数的格式化字符串

上面举了个最简单的输出 Hello world 的例子，如果说我们输出的参数里有变量，而且是个浮点数，该如何处理呢？根据函数调用习惯，我们把代码稍微修改一下，让它可以打印出 Hello world!1234.56，并且让函数返回100：

```text 
.section .data

msg:
    .asciz "Hello world!%.2f\n"
f:
    .double 1234.56

.section .text
.globl main
main:
    /* 调用 printf() 函数打印 "Hello world!" */
    /*  printf函数原型: int printf(char *fmt, ...) */
    subq $8, %rsp       # 发起 CALL 调用之前，栈必须是对齐到16字节，否则汇报 segment fault 错误
    movl $1, %eax       # 被调用函数参数中有浮点数时， %al寄存器中保存的是需要传送到XMM寄存器的参数数量，我们传入了1个浮点数，所以为1
    mov $msg, %rdi      # 字符串地址
    movsd f, %xmm0      # 参数为浮点数时，需要使用%xmm系列寄存器来传参
    call printf         # 调用C库函数 printf

    /* return */
    movq $100, %rax     # main函数返回值，rax = 100
    addq $8, %rsp       # 恢复原来的栈地址
    ret
```

编译运行：
```bash 
# gcc -o helloworld -no-pie helloworld.s
# ./helloworld
Hello world!1234.56
# echo $?
100
```

可以看到，运行后输出了浮点数，且返回值为100。

## 2. 应用程序、C库和内核之间的关系

调用printf()函数时，应用程序、C库和内核之间的关系如下图所示：

![relationship](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/3-x86_64%E6%B1%87%E7%BC%96/image/app-clib-kernel.awebp)

从图中可以看到，我们调用C库函数printf()时，最终会调用内核的write()系统调用，那么我们就可以绕过C库，直接使用系统调用来输出字符串。

在Linux x86系统上，系统调用可以通过多种方式来实现。在32位系统上，可以通过`int 0x80`或`sysenter`来实现；在64位系统上，使用`syscall`来实现。其中`int 0x80`是传统的系统调用方式，被称为legacy system call； `sysenter`和`syscall`是后来添加的指令，被称为Fast System Call。

## 3. 软中断int 0x80

### 3.1 参数传递

当使用`int 0x80`进行系统调用时，参数与寄存器的对应关系如下图所示：

```text

系统调用号    参数1    参数2    参数3    参数4    参数5    参数6
-------------------------------------------------------------
%rax         %rbx     %rcx    %rdx     %rsi     %rdi     %rbp
```
该对应关系可以从Linux kernel源码arch/x86/entry/entry_32.S里找到。如果大家不方便下载源码，可以从源码阅读网站查看，各版本的内核源码都有，地址在这里：[Linux kernel在线阅读网站](https://elixir.bootlin.com/linux/v6.13.5/source)

下面是5.0版本内核文件里的调用参数介绍，[文档地址](https://elixir.bootlin.com/linux/v5.0/source/arch/x86/entry/entry_32.S)

```c
/*
 * 32-bit legacy system call entry.
 *
 * 32-bit x86 Linux system calls traditionally used the INT $0x80
 * instruction.  INT $0x80 lands here.
 *
 * This entry point can be used by any 32-bit perform system calls.
 * Instances of INT $0x80 can be found inline in various programs and
 * libraries.  It is also used by the vDSO's __kernel_vsyscall
 * fallback for hardware that doesn't support a faster entry method.
 * Restarted 32-bit system calls also fall back to INT $0x80
 * regardless of what instruction was originally used to do the system
 * call.  (64-bit programs can use INT $0x80 as well, but they can
 * only run on 64-bit kernels and therefore land in
 * entry_INT80_compat.)
 *
 * This is considered a slow path.  It is not used by most libc
 * implementations on modern hardware except during process startup.
 *
 * Arguments:
 * eax  system call number
 * ebx  arg1
 * ecx  arg2
 * edx  arg3
 * esi  arg4
 * edi  arg5
 * ebp  arg6
 */
```

### 3.2 系统调用号
在 x86-64 系统上，虽然仍然可以使用 int 0x80 来进行系统调用，但它执行的是32位的系统调用，使用的是32位的系统调用表，且效率低下，不应该再使用；在64位系统上，应该使用syscall系统调用，来使用64位的系统调用表。

32位系统调用表，可以在[这里](https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_32.tbl)获取。下面列出了32位系统的部分调用及编号，可以看到，write()的系统调用编号为 4 ，exit()系统调用编号为 1。

```c
# SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note
#
# 32-bit system call numbers and entry vectors
#
# The format is:
# <number> <abi> <name> <entry point> [<compat entry point> [noreturn]]
#
# The __ia32_sys and __ia32_compat_sys stubs are created on-the-fly for
# sys_*() system calls and compat_sys_*() compat system calls if
# IA32_EMULATION is defined, and expect struct pt_regs *regs as their only
# parameter.
#
# The abi is always "i386" for this file.
#
0	i386	restart_syscall		sys_restart_syscall
1	i386	exit			sys_exit			-			noreturn
2	i386	fork			sys_fork
3	i386	read			sys_read
4	i386	write			sys_write
5	i386	open			sys_open			compat_sys_open
6	i386	close			sys_close
7	i386	waitpid			sys_waitpid
8	i386	creat			sys_creat
9	i386	link			sys_link
10	i386	unlink			sys_unlink
11	i386	execve			sys_execve			compat_sys_execve
```

### 3.3 函数原型

1. write()系统调用函数原型

    ```c
    ssize_t write(int fd, const void *buf, size_t count);
    ```
1. exit()系统调用函数原型

    ```c
    void _exit(int status);
    ```

### 3.4 汇编代码

```text
.section .data

msg:
    .ascii "Hello world!\n"
len = . - msg

.section .text
.globl main
main:
    /* write(2) 系统调用, 打印 "Hello world!" */
    /* write(2)原型：ssize_t write(int fd, const void *buf, size_t count); */
    movq $4, %rax       # write()系统调用号，4
    movq $1, %rbx       # 第一个参数，fd
    movq $msg, %rcx     # 第二个参数，buf
    movq $len, %rdx     # 第三个参数，count
    int $0x80

    /* exit(2) 系统调用  */
    /* exit()原型：void _exit(int status); */
    movq $1, %rax       # exit()系统调用号，1
    movq $0, %rbx       # 状态码，status
    int $0x80
```
编译执行：

```bash
# gcc -o helloworld -no-pie helloworld.s
# ./helloworld
Hello world!
# echo $?
0
```

说明：

- 这里使用了 `.ascii` 来定义一个字符串，而没有使用 `.asciz`，是因为我们不再需要兼容C的习惯，我们需要自己计算字符串的长度。

- `len = . - msg` 里`.`表示当前地址。

## 4. syscall系统调用

### 4.1 参数传递

4.1 参数传递
当使用 syscall进行系统调用时，参数与寄存器的对应关系如下图所示：

```text
系统调用号    参数1    参数2    参数3    参数4    参数5    参数6
---------------------------------------------------------------
%rax         %rdi     %rsi    %rdx     %r10     %r8     %r9
```

该对应关系可以从 linux kernel 源码 arch/x86/entry/entry_64.S 里找到。下面是 5.0 版本内核文件里的调用参数介绍，[文档地址](https://elixir.bootlin.com/linux/v5.0/source/arch/x86/entry/entry_64.S)。

```c
/*
 * 64-bit SYSCALL instruction entry. Up to 6 arguments in registers.
 *
 * This is the only entry point used for 64-bit system calls.  The
 * hardware interface is reasonably well designed and the register to
 * argument mapping Linux uses fits well with the registers that are
 * available when SYSCALL is used.
 *
 * SYSCALL instructions can be found inlined in libc implementations as
 * well as some other programs and libraries.  There are also a handful
 * of SYSCALL instructions in the vDSO used, for example, as a
 * clock_gettimeofday fallback.
 *
 * 64-bit SYSCALL saves rip to rcx, clears rflags.RF, then saves rflags to r11,
 * then loads new ss, cs, and rip from previously programmed MSRs.
 * rflags gets masked by a value from another MSR (so CLD and CLAC
 * are not needed). SYSCALL does not save anything on the stack
 * and does not change rsp.
 *
 * Registers on entry:
 * rax  system call number
 * rcx  return address
 * r11  saved rflags (note: r11 is callee-clobbered register in C ABI)
 * rdi  arg0
 * rsi  arg1
 * rdx  arg2
 * r10  arg3 (needs to be moved to rcx to conform to C ABI)
 * r8   arg4
 * r9   arg5
 * (note: r12-r15, rbp, rbx are callee-preserved in C ABI)
 *
 * Only called from user space.
 *
 * When user can change pt_regs->foo always force IRET. That is because
 * it deals with uncanonical addresses better. SYSRET has trouble
 * with them due to bugs in both AMD and Intel CPUs.
 */
```

### 4.2 系统调用号

64位系统调用表，可以在[这里](https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl)获取。下面列出了64位系统的部分调用及编号，可以看到，write()的系统调用编号为 1 ，exit()系统调用编号为 60。

```c
# SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note
#
# 64-bit system call numbers and entry vectors
#
# The format is:
# <number> <abi> <name> <entry point> [<compat entry point> [noreturn]]
#
# The __x64_sys_*() stubs are created on-the-fly for sys_*() system calls
#
# The abi is "common", "64" or "x32" for this file.
#
0	common	read			sys_read
1	common	write			sys_write
2	common	open			sys_open
3	common	close			sys_close
4	common	stat			sys_newstat
5	common	fstat			sys_newfstat
6	common	lstat			sys_newlstat
7	common	poll			sys_poll
8	common	lseek			sys_lseek
9	common	mmap			sys_mmap
10	common	mprotect		sys_mprotect
```

### 4.3 函数原型

1. write()系统调用函数原型

    ```c
    ssize_t write(int fd, const void *buf, size_t count);
    ```
1. exit()系统调用函数原型

    ```c
    void _exit(int status);
    ```

  ### 4.4 汇编代码

  ```text
.section .data
msg:
    .ascii "Hello World!\n"
len = . - msg

.section .text
.globl  main
main:
    # ssize_t write(int fd, const void *buf, size_t count)
    mov $1, %rdi            # fd
    mov $msg, %rsi          # buffer
    mov $len, %rdx          # count
    mov $1, %rax            # write(2)系统调用号，64位系统为1
    syscall

    # exit(status)
    mov $0, %rdi            # status
    mov $60, %rax           # exit(2)系统调用号，64位系统为60
    syscall
  ```
编译执行：

```bash
# gcc -o helloworld -no-pie helloworld.s
# ./helloworld
Hello world!
# echo $?
0
```

提示：

- 同样的系统调用函数，在32位系统和64位系统里，其调用号是不一样的，因为使用的是不同的系统调用表。