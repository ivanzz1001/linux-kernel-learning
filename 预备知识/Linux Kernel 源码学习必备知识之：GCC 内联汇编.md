# Linux Kernel 源码学习必备知识之：GCC 内联汇编

文章转载自: 

- [Coder Heart: Linux Kernel 源码学习必备知识之-GCC 内联汇编](https://juejin.cn/post/7200008514390491195)


# 1. 内联汇编简介

## 1.1 什么是内联汇编

内联汇编称为inline assembly，GCC支持在C代码中直接嵌入汇编代码，所以称为GCC inline assembly。

内联汇编按格式分为两大类：基本内联汇编和扩展内联汇编。基本内联汇编没有操作数，而扩展内联汇编可以有一个或多个操作数。当在C函数里混合使用C和汇编语言时，首选扩展汇编；当汇编语言出现在模块顶级时（也就是不在模块的函数中，在函数外使用），必须使用基本内联汇编。

## 1.2 为什么要使用内联汇编

因为一些底层操作，C语言不支持（比如寄存器操作），而汇编语言可以；为了提高C语言的能力，因此有了内联汇编。

# 2. 基本内联汇编

基本内联汇编是最简单的内联形式，其格式为：

```assembly
asm [qualifiers]("assembly code")
```

各关键字之间可以用空格或制表符分隔，也可以紧凑挨在一起不分隔。

关键字 asm 用于声明内联汇编表达式，这是内联汇编固定的部分，不可缺少。`asm` 和``__asm__`` 是同义的，是由 gcc 定义的宏：`#define __asm__ asm` 。

## 2.1 限定符(qualifiers)

基本内联汇编有两种限定符，分别为`volatile`和`inline`。

1. volatile限定符

    因为GCC有一个优化选项`-O`，可以指定优化级别。当用`-O`来编译时，gcc 按照自己的意图优化代码，说不定就会把自己写的代码修改了。关键字 volatile 的作用是告诉 gcc：“不要修改我写的汇编代码，请原样保留”。

    volatile和`__volatile__`是同义的，是由GCC定义的宏：`#define __volatile__ volatile`

    在基本内联汇编中，volatile是可选的，而且对功能没有影响。因为基本内联汇编代码块，默认都是 volatile 的

1. inline限定符

    如果使用 inline 限定符, 则出于内联目的, asm 语句的大小将被视为可能的最小大小（参见[Size of an asm](https://gcc.gnu.org/onlinedocs/gcc/Size-of-an-asm.html#Size-of-an-asm)）

## 2.2 汇编代码(assembly code)

"assembly code"是我们写的汇编代码，它必须位于圆括号内，而且必须用双引号引起来。这是格式要求，只要满足了这个格式`asm [qualifiers] ("")`, assembly code甚至可以为空。

assembly code的规则：

- 指令必须用双引号 引起来，无论双引号中是一条指令或多条指令；

- 一对双引号不能跨行，如果跨行需要在结尾用反斜杠 `"\"` 转义；

- 指令之间用分号";"、换行符"\n"或换行符加制表符"\n\t"分隔。

提醒一下，即使是指令分布在多个双引号中，gcc 最终也要把 它们合并到一起来处理，合并之后，指令间要有分隔符。所以，当指令在多个双引号中时，除最后一个双引号外，其余双引号中的代码最后一定要有分隔符，这和其它编程语言中表示代码结束部分的分隔符是一样的，如：

```assembly
asm("mov $9, %eax;""pushl %eax")    # 正确
asm("mov $9, %eax""pushl %eax")     # 错误
```

## 2.3 代码示例

```assembly
asm("movl %ecx %eax");      /* moves the contents of ecx to eax */
__asm__("movb %bh (%eax)"); /* moves the byte from bh to the memory pointed by eax */

__asm__ ("movl %eax, %ebx\n\t"
         "movl $56, %esi\n\t"
         "movl %ecx, $label(%edx,%ebx,$4)\n\t"
         "movb %ah, (%ebx)");
```

说明: 上面`movl %ecx, $label(%edx,%ebx,$4)`可能存在语法错误，可能应该写成`movl %ecx, label(%edx,%ebx,4)`

## 2.4 基本内联汇编的用武之地

使用扩展汇编通常会生成更短、更安全以及更高效的代码；在大部分情况下使用扩展汇编是比基本汇编更好的解决方案。但是，在两种情况下只能使用基本内联汇编：

- 扩展内联汇编只能在C函数内部使用，所以当需要在文件域（最顶级）及函数外部使用时，只能使用基本内联汇编。函数外部使用的基本内联汇编，不能使用任何限定符。

- 带 nake 属性声明的函数，必须使用基本内联汇编（参见 [Function Attributes](https://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html#Function-Attributes)）。


## 2.5 基本内联汇编的缺点

如果我们在汇编代码里改变了寄存器内容，但是从汇编返回时没有对其进行恢复，就会产生不良后果。因为 GCC 根本不知道到我们修改了寄存器，而我们修改寄存器时也没有通知 GCC，它会认为寄存器里的值没有被修改而继续使用，这样就会产生问题而导致程序崩溃。扩展汇编提供了解决这些问题的能力。

# 3. 扩展内联汇编

由于基本内联汇编功能太薄弱了，不能满足使用需求，所以对它进行了扩展。扩展后的内联汇编格式如下：

```assembly
asm [qualifiers] (assembler template 
                 [: output operands]           /* 可选的 */
                 [: input operands ]           /* 可选的 */
                 [: list of clobber/modify]    /* 可选的 */
                 );
```
和基本内联汇编相比，扩展内联汇编在圆括号中变成了 4 部分，多了 output、input 和 clobber/modify 三项。

其中的每一部分都可以省略, 甚至包括assembler template。省略的部分要保留冒号分隔符来占位。如果省略的是最靠后的一个或多个连续的部分，分隔符也不用保留，比如省略了 clobber/modify ，不需要保留 input 后面的冒号。

在学习各部分功能之前，我们先来看一个例子，对扩展内联汇编有一个基本的印象：

```assembly
#include <stdio.h> 
void main() {
    int a = 10, b;
    asm ("movl %1, %%eax;" 
        "movl %%eax, %0;"
        :"=r"(b)          /* output */
        :"r"(a)           /* input */
        :"%eax"          /* clobbered register */
        );
    printf("Now, b is: %d\n", b);
}
```
在这段代码中，我们实现了让 b 等于 a的功能，代码说明如下：

- `b`是输出操作数，被`%0`引用；`a`是输入操作数，被`%1`引用

- `r` 是对操作数的约束，它通知 GCC 可以使用任何寄存器来存储操作数。`=` 是 output 的约束描述符，它说明这个输出操作数是只写的。

- 在寄存器名称前面有 2 个`%`，这是因为单个`%`被 GCC 用做操作数占位符，为了区分操作数和寄存器，GCC 在寄存器名称前使用了 2 个 `%`。

- `%eax` 出现在 clobber/modify 里， 它告诉 GCC `%eax` 的值会被修改，所以不要用它来存储其它值。

代码输出结果如下：

```text
# gcc -o test inline_assign.c
# ./test 
Now, b is: 10
```

## 3.1 限定符(qualifiers)

扩展汇编包括如下三种限定符：

- volatile

  volite 限定符主要是禁止 GCC做优化，参见 2.1 节。

- inline

  如果使用 inline 限定符，则出于内联目的，asm 语句的大小将被视为可能的最小大小（参见[Size of an asm](https://gcc.gnu.org/onlinedocs/gcc/Size-of-an-asm.html#Size-of-an-asm)）。

- goto

  此限定符通知编译器，汇编语句可能会执行一个跳转，跳转目的是 goto 标签里的一个（参见 [GotoLabels](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#GotoLabels)）。

这里把这些限定符列举出来，只是为了文章的完整性，我们不会对这些限定符做深入讨论。如果大家想做深入研究，可以参考 gcc 相关文档。

## 3.2 汇编模板(assembler template)

汇编模板包含一组汇编代码，格式同 2.2 节。但与基本内联汇编不同的是，扩展汇编的代码中允许存在操作数占位符，如 `%0`，`%1`等。

占位符分为`序号占位符`和`名称占位符`两种:

1. **序号占位符**

    序号占位符是对在 output 和 input 中的操作数的引用，按照它们从左到右出现的次序从 0 开始编号，一直到 9，引用它的格式是%0~9，也就是说最多支持 10 个序号占位符。

    在操作数自身的序号前面加 1 个百分号 `%` 便是对相应操作数的引用。一定要切记，占位符指代约束所对应的操作数，也就是在汇编中的操作数，并不是圆括号中的 C 变量。

    举个例子：

    ```assembly
    asm("addl %%rbx, %%rax":"=a"(out_sum):"a"(in_a),"b"(in_b));
    ```

    等价于:

    ```assembly
    asm("addl %2, %1":"=a"(out_sum):"a"(in_a),"b"(in_b));
    ```

    其中:

    - "=a"(out_sum)序号为0, %0 对应的是 rax。

    - "a"(in_a) 序号为1, %1对应的是 rax

    - "b"(in_b) 序号为2, %2 对应的是 rbx

    前文说过，由于扩展内联汇编中的占位符要有前缀 %，为了区别占位符和寄存器，只好在寄存器前用两个 % 做前缀。

    我们写一个简单的包含占位符的例子：

    ```c
    #include <stdio.h>

    int main(void)
    {
        __uint64_t ret;
        asm volatile
        (
            "movq $0x1122334455667788, %%rax;"
            "movq %%rax, %0"
            :"=r"(ret)
        );
        
        printf("ret is: %ld\n", ret);
        return 0;
    }
    ```

    编译并运行，其输出结果如下：

    ```text
    # gcc -o test test.c
    # ./test
     ret is: 1234605616436508552
    ```

    我们知道，指令的操作数大小并不一致，有的指令操作数大小是 64 位，有的是 32 位，有的是 16 位，有的是 8 位。一个 64 位寄存器，我们既可以当做 64 位来使用，也可以当做 32 位、16位、8位来使用。比如，对于寄存器 %rax，当我们使用全部 64 位时，我们称它为 %rax；使用低 32 位时，为 %eax；依次类推，低 16 位为 %ax，低 8 位为 %al，高 8 位为 %ah。有些情况下，编译器能够推断出操作数的位数；但在某些特殊情况下，编译器会报错。比如下面这段代码：

    ```c
    #include <stdio.h>
    int main(void)
    {
         __uint64_t x = 0x1122334455667788, y = 0;
         asm volatile
         (
            "movl %1, %0;"
            : "=m"(y)
            : "a"(x)
        );
        printf("x's lower double word is: %ld\n", y);

        y = 0;
        asm volatile
        (
            "movw %1, %0;"
            : "=m"(y)
            : "a"(x)
        );
        printf("x's lower word is: %ld\n", y);
        return 0;
    }
    ```
    在这段代码中，我们想获取变量 x 的低 32 位（双字）和低 16 位（单字）。虽然我们在汇编指令中分别用 `movl` 和 `movw` 指定了操作数大小，但是在编译时依然会报错：

    ```text
    # gcc -o test test.c 
    test.c: Assembler messages:
    test.c:5: Error: incorrect register `%rax' used with `l' suffix
    test.c:14: Error: incorrect register `%rax' used with `w' suffix
    ```
    从报错信息中可以看到，虽然我们在汇编指令中使用了指定操作数对应的后缀，但是编译器依然使用的是 %rax寄存器，并给我们报了操作数大小不匹配的错误。

    为了解决这个问题, GCC 提供了操作数描述符，让我们来手动指定操作数的大小及其它功能，比较常用的操作数描述符有以下几个：

    ```text
    Modifier  Description                                                   Operand        ‘att’
    -------------------------------------------------------------------------------------------------
    A         Print an absolute memory reference.                            %A0           *%rax
    b         Print the QImode name of the register.                         %b0           %al
    
    d         print duplicated register operand for
              AVX instruction.                                               %d5           %xmm0
              
    E         Print the address in Double Integer
              (DImode) mode (8 bytes) when the target                        %E1           %(rax)
              is 64-bit. Otherwise mode is unspecified
              (VOIDmode).                                     
            
    h         Print the QImode name for a “high” 
              register.                                                      %h0           %ah
              
    H         Add 8 bytes to an offsettable memory 
              reference. Useful when accessing the                           %H0           8(%rax)
              high 8 bytes of SSE values. For a memref
              in (%rax), it generates               
              
    k         Print the SImode name of the register.                         %k0           %eax
    q         Print the DImode name of the register.                         %q0           %rax
    w         Print the HImode name of the register.                         %w0           %ax
    ```

    注：完整的描述符列表可以参考 GCC 官方文档 [Extended Asm:x86 operand modifiers](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#x86Operandmodifiers)。

    从上表可以看到，如果我们想用寄存器低 32 位如%eax, 需要在 % 和 占位序号之间加上 k，即%eax；想用寄存器低 16 位如%ax，需要在 % 和 占位序号之间加上 w，即%w0，等等。

    根据描述符要求，我们把代码修改一下：

    ```c
    #include <stdio.h>
    int main(void)
    {
        __uint64_t x = 0x1122334455667788, y = 0;
        asm volatile
        (
            "movl %k1, %0;"
            : "=m"(y)
            : "a"(x)
        );
        printf("x's lower double word is: %ld\n", y);

        y = 0;
        asm volatile
        (
            "movw %w1, %0;"
            : "=m"(y)
            : "a"(x)
        );
        printf("x's lower word is: %ld\n", y);
        return 0;
    }
    ```
    再次编译并运行：

    ```text
    # gcc -o test test.c 
    # ./test
     x's lower double word is: 1432778632
     x's lower word is: 30600
    ```

1. **名称占位符**

    名称占位符与序号占位符不同，序号占位符靠本身出现在output和input中的位置就能被编译器识别出来。而名称占位符需要在output和input中给操作数显式的起个名字，用名字来标识操作数，格式如下：

    ```text
    [名称] "约束名" (C变量)
    ```

    这样，该约束对应的汇编操作数就有了名字，在 assembly template 中应用操作数时，采用 `%[名称]` 的形式就可以了。

    示例如下：

    ```C
    #include <stdio.h>

    void main(){
        int a = 18, b = 3, out = 0;

        asm("divb %[divisor]; movb %%al, %[result]" \
            :[result]"m="(out)
            :"a"(a), [divisor]"m"(b)
        );

        printf("result is %d\n", out);
    }
    ```

    编译并运行：

    ```bash
    # gcc -o test name_placeholder_test.c
    # ./test
    result is 6
    ```

## 3.3 操作数

### 3.3.1 输出操作数(output operands)

output用来指定汇编代码的数据如何输出给C代码使用。内嵌的汇编指令运行结束后，如果想将运行结果保存到C变量中，就用此项指定输出的位置。output中每个操作数格式为：

```text
"操作数修饰符约束名"(C变量名)
```

其中的引号和括号不能少，操作数修饰符通常为等号"="。多个操作数之间用逗号","分割。

### 3.3.2 输入操作数(input operands)

input 用来指定 C 中数据如何输入给汇编使用。要想让汇编使用C中的变量作为参数，就要使用如下的方式来进行设定。input中每个操作数的格式为:

```text
"操作数修饰符约束名"(C变量名)
```

其中的引号和括号不能少，操作数修饰符为可选项。多个操作数之间用逗号","分割。


单独强调一下，以上的output()和input()括号中的是C代码中的变量，output(C变量)和input(C变量)就像C语言中的函数，将C变量(值或变量地址)转换成汇编代码的操作数。

## 3.4  破坏列表（list of colbber/modify）

汇编代码执行后会破坏一些内存或寄存器资源，通过此项通知编译器，可能造成寄存器或内存数据的破坏，这样 gcc 就知道那些寄存器或内存需要提前保护起来。

怎样通知 gcc 我们修改了哪些寄存器？

这个很简单，只要在 clobber/modify 部分明确写出来就行了，记得要用双引号把寄存器名称引起来，多个寄存器之间用逗号","分隔，这里的寄存器不用再加两个'%'啦，只写名称即可，如:

```assembly
asm("movl %%eax, %0; movl %%eax %%ebx"
    :"=m"(ret_value)
    :
    : "bx"
   )
```
大家看，虽然修改的是寄存器 ebx ，但只要在 clobber/modify 声明 bx 就可以了，甚至可以声明 bl 。原因是即使寄存器只变动一部分，它的整体也会全跟着受影响，所以在 clobber/modiy 句中声明寄存器时，可以用低 8 位名称、低 16 位名称或低 32 位或全 64 位名称，如"al"、"ax"、"eax"、"rax" 都是指 rax 寄存器，其他通用寄存器也是一样的。


如果我们的内联汇编代码修改了标志寄存器 eflags 中的标志位，同样需要在 clobber/modiy 命中用"cc"声明。

如果我们修改了内存，我们需要在clobber/modify中用"memory"声明。

## 3.5 约束

约束描述了操作数是否使用了寄存器，用的是哪种寄存器；是否引用了内存，用的是哪种地址；是否是立即数，是哪种类型的立即数等等。它所起的作用就是把 C 代码中的操作数（变量、立即数）映射为汇编中所使用的操作数，实际就是描述 C 中的操作数如何变成汇编操作数。这些约束的作用域是 input 和 output 部分，约束分为以下几类：

### 3.5.1 常用约束

1. **Register operand constraint(r)**

   当操作数使用此约束时，该操作数会被存放在通用寄存器(General Purpose Registers)中。例如：

   ```assembly
   asm ("movl %%eax, %0\n" :"=r"(myval));
   ```

   这里变量`myval`会被保存在一个寄存器中，eax寄存器的值会被copy到该寄存器，之后`myval`的值又会从该寄存器更新回内存。当`r`被指定时，gcc可能会将myval的值保存在任何一个可用的通用寄存器。如果我们想更明确的指定使用哪个寄存器，那么使用特定的寄存器约束：

      - `a`: 表示 a 系列寄存器 rax/eax/ax/al

      - `b`: 表示 b 系列寄存器 rbx/ebx/bx/bl

      - `c`: 表示 c 系列寄存器 rcx/ecx/cx/cl

      - `d`: 表示 d 系列寄存器 rdx/edx/dx/dl

      - `D`: 表示寄存器 rdi/edi/di

      - `S`: 示寄存器 rsi/esi/si

      - `q`: 表示 rax/rbx/rcx/rdx 这 4 个通用寄存器中任意一个

      - `r`: 表示 rax/rbx/rcx/rdx/rsi/rdi 这 6 个通用寄存器中任意一个

      - `g`: 表示可以存放到任意地点（寄存器和内存）。相当于除了同 q 一样外，还可以让 gcc 安排在内存中

      - `A`: a 和 d 寄存器，用于返回结果保存在a 和 d 寄存器的指令

      - `f`: 表示浮点寄存器

      - `t`: 表示第 1 个浮点寄存器

      - `u`: 表示第 2 个浮点寄存器

      - `p`: 表示内存地址，给 "load address" 和 "push address" 指令使用

    值得注意的一点这是，在基本内联汇编中，寄存器表示方法和直接写汇编没什么区别，都是以%开头，后面跟着寄存器名称，比如 %eax；但是在扩展内联汇编中，单个`%` 有了新的用途，用来表示占位符，所以在扩展内联汇编中的寄存器名称前要用两个%做前缀。下面对比一下：

      - **基本内联汇编**

        ```C
        #include <stdio.h>
        
        int a = 1, b = 2, sum;
        void main(){
            asm("push %rax;                   \
            push %rbx;                   \
            movl a, %eax;                \
            movl b, %ebx;                \
            addl %ebx, %eax;             \
            movl %eax, sum;              \
            pop %rbx;                    \
            pop %rax;");
        
            printf("sum is %d\n", sum);
        }
        ```

      - **扩展内联汇编**

        ```C
        #include <stdio.h>
        
        void main(){
            int a = 1, b = 2, sum;
        
            asm("addl %%ebx, %%eax":"=a"(sum):"a"(a),"b"(b));
        
            printf("sum is %d\n", sum);
        }
        ```

    可以看到，a 和 b 是在 input 部分中输入的，用约束名 a 为C变量a指定了用寄存器eax; 用约束名b为C变量b指定了用寄存器ebx。addl 指令的结果保存到寄存器eax中，在 output 中用约束名 a 指定了把寄存器 eax 的值存储到C变量 sum 中。output 中的 `=` 号是操作数类型修饰符，表示只写，其实就是 sum = eax 的意思。


1. **Memory operand constraint(m)**

    内存约束是要求 gcc 直接将位于 input 和 output 中的 C 变量的内存地址作为内联汇编代码的操作数，不需要寄存器做中转，直接进行内存读写，也就是汇编代码的操作数是 C 变量的指针。

      - `m`: 表示内存操作数，可以是计算机支持的任何类型的地址

      - `o`: 表示内存操作数，但访问它是通过偏移量的形式访问，即包含 offset_address 的格式

    代码示例:

      ```C
      #include <stdio.h>

      int main(void)
      {
          static unsigned long arr[3] = {0, 1, 2};
      
          static unsigned long element;
      
          asm volatile("movq 16+%1, %0" : "=r"(element) : "o"(arr));
          printf("%lu\n", element);
          reurn 0x0;
      }
      ```

1. **立即数约束**

    立即数即常数，此约束要求 gcc 在传值的时候不通过寄存器或内存，直接作为立即数传给汇编代码。由于立即数不是变量，只能作为右值，所以只能放在 input 中。

      - `i`: 表示操作数为整数立即数

      - `E`: 表示操作数为浮点数立即数

      - `I`(大写的i): 表示操作数为0~31之间的立即数

      - `J`: 表示操作数为0~63之间的立即数

      - `N`: 表示操作数为0~255之间的立即数

      - `O`: 表示操作数为0~32之间的立即数

      - `X`: 表示操作数为任何类型立即数

1. **通用约束**

    - 0~9：此约束只用在 input 部分，表示该 input 操作数与 output 中第 n 个操作数用相同的寄存器或内存。


### 3.5.2 约束修正符(constraint modifiers)

在约束中还有操作数类型修饰符，用来修饰所约束的操作数。

1. **在 output 中有以下 3 种**

    - `=`: 表示操作数是只写，相当于为 output 括号中的 C 变量赋值，如 "=a"(c_var)，此修饰符相当于 c_var = eax。

    - `+`：表示操作数是可读写的，告诉 gcc 所约束的寄存器或内存先被读入，再被写入。

    - `&`: 表示此 output 中的操作数要独占所约束（分配）的寄存器，只供 output 使用，任何 input 中所分配的寄存器不能与此相同。注意，当表达式中有多个修饰符时，& 要与约束名挨着，不能分隔

1. **在input中**

    - `%`：该操作数可以和下一个输入操作数互换

一般情况下，input 中的 C 变量是只读的，output中的 C 变量是只写的。修饰符 "=" 只用在output中，表示 C 变量是只写的。

修饰符"+" 也只用在 output 中，但它具备读、写的属性，也就是它既可以作为输入，同时也可以作为输出，所以省去了在 input 中声明约束。示例如下：

```C
#include <stdio.h>

void main(){
    int a = 1, b = 2;

    asm("addl %%ebx, %%eax;":"+a"(in_a):"b"(in_b));

    printf("a is %d\n", a);
}
```

# 4 参考文献

- 《操作系统真象还原》第 6.4 节

- [GCC 文档：How to Use Inline Assembly Language in C Code](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html#Using-Assembly-Language-with-C)

- [GCC-Inline-Assembly-HOWTO](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)

- [Inline assembly](https://0xax.gitbooks.io/linux-insides/content/Theory/linux-theory-3.html)

- [brennan att inline assembly](https://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)
