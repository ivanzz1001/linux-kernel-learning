# AT&T汇编与Intel汇编比较

汇编存在多种形式，目前主流的有`AT&T汇编`和`Intel汇编`，这两种形式极其容易混淆。其中AT&T汇编语法被Linux和GCC广泛支持，而Intel汇编则由Windows支持，故而对于长期涉及到两种操作系统编程的人而言，这两种汇编语法虽本质相同，但语法形式的差异常让人混淆。这里便介绍一下两者的主要区别。

参看:

- [GCC-Inline-Assembly-HOWTO](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html#s4)

- [Brennan's Guide to Inline Assembly](https://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)


---

1. **Source-Destination顺序**

    AT&T语法中操作数的方向与Intel相反。Intel语法中，第一个操作数为Destination, 第二个操作数为Source； 而AT&T语法中，第一个操作数为Source，第二个操作数为Destination。比如：

    - AT&T: `Op-code src dst`
      
    - Intel: `Op-code dst src`

    

1. **寄存器命名**

    AT&T语法中，寄存器的名字都以`%`开头。比如要使用eax寄存器：

    - AT&T: `%eax`
  
    - Intel: `eax`

1. **立即数与常量**

   AT&T语法中，立即数与常量值以`$`开头； 对于静态"C"变量也以`$`开头。Intel语法中，对于十六进制常量以`h`结尾，而AT&T语法中十六进制常量以`0x`开头。

   如下我们将静态"C"变量`booga`的地址加载到eax寄存器:

    - AT&T: `movl $_booga, %eax`
  
       > 说明：`$_booga`表示 _booga 这个符号的地址，前面的 $ 表示取 立即数（即 _booga 的地址值，而不是 _booga 指向的内容）
  
    - Intel: `mov eax, _booga`
  
   如下我们将0xd00d加载到ebx寄存器：

     - AT&T: `movl $0xd00d, %ebx`
  
     - Intel: `mov ebx, d00dh` 

1. **操作数大小**

    AT&T语法中，内存操作数的大小由`Op-code`名称的最后一个字符决定，`Op-code`以`b`、`w`、`l`结尾分别表示byte(8-bit)、word(16-bit)、long(32-bit)长度的内存引用。Intel语法中，是通过在内存操作数(note: 非Op-code)前加上`byte ptr`、`word ptr`、`dword ptr`来指定。

    >Note: 在AT&T语法中，如果省略了Op-code中最后一个标识大小的字符，GAS (GNU assembler) 会尝试进行猜测；如果我们不想让GAS来猜，或者为了避免GAS猜测错误，请不要忘记指定。

    - AT&T:  `movw %ax, %bx`
  
    - Intel: mov bx, ax
  
1. **内存引用**

   Intel语法中将基址寄存器放在`[]`中，而AT&T语法中将基址寄存器放在`()`中。如下是AT&T及Intel汇编间接内存引用的语法：

   - AT&T: `section:disp(base, index, scale)`
  
   - Intel: `section:[base + index*scale + disp]`
  
   我们需要记住的其中一个点是：当disp/scale为常数时，不能使用`$`前缀。下面我们再详细介绍一下AT&T内存引用语法各个部分的含义：

   ```text
    segment:offset(base, index, scale) 
    ```

    其中:

      - segment: 可以是 x86 架构的任意段寄存器。segment 是可选的，如果指定的话，后面要跟上冒号来与offset隔离开； 如果未指定的话，默认为数据段寄存器`%ds`

      - offset: 是一个立即数偏移量，是可选的。

      - base： 表示基址寄存器，可以是16个通用寄存器中的任意一个

      - index: 表示变址寄存器，可以是16个通用寄存器中的任意一个

      - scale: 表示比例因子，scale会与index相乘再加上base来表示内存地址。比例因子必须是1,2,4或者8；若比例因子为指定，默认为1

    有效地址被计算为：`D[segment]+offset+R[base]+R[index]*scale`，其中`D[]`表示对应段寄存器的数据; `R[]`表示通用寄存器里的数。我们用`M[addr]`来表示内存地址addr。


    内存地址操作示例：

    ```text
    指令                              说明
    ----------------------------------------------------------------------
    movl var, %eax                   把内存地址M[var]处的数据传送到eax寄存器
    movl %cs:var, %eax               把代码段偏移量为var处的内存数据传送到eax寄存器
    movl $var, %eax                  把立即数var传送到eax寄存器
    movl var(%esi), %eax             把内存地址 M[R[%esi] + var] 处的数据传送到eax寄存器
    movl (%ebx, %esi, 4), %eax       把内存地址 M[R[%ebx] + R[%esi]*4] 处的数据传送到eax寄存器
    movl var(%ebx, %esi, 4), %eax    把内存地址 M[var+R[%ebx] + R[%esi]*4] 处的数据传送到eax寄存器
    ```


# AT&T汇编与Intel汇编差异示例

上面我们讲述了AT&T汇编与Intel汇编的主要差异，要想了解更详细信息，请参考GNU Assembler文档。如下我们给出一些示例以更好的理解两者之间的差异：

```text
+------------------------------+------------------------------------+
|       Intel Code             |      AT&T Code                     |
+------------------------------+------------------------------------+
| mov     eax,1                |  movl    $1,%eax                   |   
| mov     ebx,0ffh             |  movl    $0xff,%ebx                |   
| int     80h                  |  int     $0x80                     |   
| mov     ebx, eax             |  movl    %eax, %ebx                |
| mov     eax,[ecx]            |  movl    (%ecx),%eax               |
| mov     eax,[ebx+3]          |  movl    3(%ebx),%eax              | 
| mov     eax,[ebx+20h]        |  movl    0x20(%ebx),%eax           |
| add     eax,[ebx+ecx*2h]     |  addl    (%ebx,%ecx,0x2),%eax      |
| lea     eax,[ebx+ecx]        |  leal    (%ebx,%ecx),%eax          |
| sub     eax,[ebx+ecx*4h-20h] |  subl    -0x20(%ebx,%ecx,0x4),%eax |
+------------------------------+------------------------------------+
```
   
    
