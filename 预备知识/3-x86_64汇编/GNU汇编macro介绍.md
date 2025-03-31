# GNU汇编macro介绍


文章参考:

- [Using as](https://sourceware.org/binutils/docs/as/index.html)

- [as PDF](https://sourceware.org/binutils/docs-2.38/as.pdf)


在 GNU 汇编器中，macro的语法与其他汇编器（如 NASM）有所不同，这里简单介绍。

# 1. GNU汇编宏的基本语法

GNU汇编使用`.macro`和`.endm`指令来定义宏，支持参数化和局部标签。其基本语法格式如下：

```assembly
.macro macname macargs
    ; 宏定义体
    ; 使用 \参数1, \参数2  来引用参数
.endm
```

**代码示例如下**

1. 简单无参宏

    ```assembly
    .macro PRINT_NEWLINE
        mov $0x0A, %al   # 换行符
        call putchar
        mov $0x0D, %al   # 回车符
        call putchar
    .endm
    
    # 调用宏
    PRINT_NEWLINE
    ```

2. 带参数的宏

    ```assembly
    .macro ADD_AND_STORE result, a, b
        mov \a, %eax
        add \b, %eax
        mov %eax, \result
    .endm
    
    # 调用宏
    ADD_AND_STORE result, $10, $20
    ```



