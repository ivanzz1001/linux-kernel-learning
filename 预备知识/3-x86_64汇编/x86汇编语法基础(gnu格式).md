# x86汇编语法基础(gnu格式)

文章转载自: 

- [Coder Heart: x86汇编语法基础](https://juejin.cn/post/7195191160603115580)

## 1. 寄存器

### 1.1 通用寄存器

x86_64 CPU包含16个64位的通用寄存器。另外，这些寄存器的低32位、16位、8位都可以作为独立的寄存器来进行访问。

![x64-registers](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/3-x86_64%E6%B1%87%E7%BC%96/image/x64_registers.png)

1. **caller-save registers**

    `%rax` , `%rcx` , `%rdx` , `%rdi` , `%rsi` , `%rsp` , `%r8-r11` 通常被认为是caller-save寄存器， 其中：

      - `%rax` 通常用于保存函数的返回值

      - `%rcx` 通常用于保存函数调用时的第4个参数

      - `%rdx` 通常用于保存函数调用时的第3个参数

      - `%rsi` 通常用于保存函数调用时的第2个参数

      - `%rdi` 通常用于保存函数调用时的第1个参数

      - `%rsp` 用于保存栈指针，指向当前栈顶元素

      - `%r8` 通常用于保存函数调用时的第5个参数

      - `%r9` 通常用于保存函数调用时的第6个参数

1. **callee-save registers**

    `%rbx` , `%rbp` , `%r12-r15`通常被认为是callee-save寄存器，其中：

      - `%rbp` 保存栈基址


>Tips: 当指令以寄存器作为目标时，对于生成小于8字节结果的指令，寄存器中剩下的字节如何处理，有两条规则
>
> - 生成1字节和2字节数字的指令会保持剩下的字节不变
> - 生成4字节的指令会把高位4字节置为0
>
> 后面这条规则是作为从IA32到x86-64的扩展的一部分而采用的

### 1.2 标志寄存器EFLAGS

EFLAGS标志寄存器包含有`状态标志位`、`控制标志位`以及`系统标志位`， 处理器在初始化时将EFLAGS标志寄存器赋值为00000002H。

下图描绘了EFLAGS标志寄存器各位的功能，其中第1, 3, 5, 15以及22-31位保留未使用。由于64位模式不再支持VM和NT标志位，所以处理器不应该再置位这两个标志位。

![x64-registers](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E9%A2%84%E5%A4%87%E7%9F%A5%E8%AF%86/3-x86_64%E6%B1%87%E7%BC%96/image/x64-elfags.awebp)


