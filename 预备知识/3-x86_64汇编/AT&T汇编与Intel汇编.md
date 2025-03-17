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
  
1. **内存操作数**

   Intel语法中将基址寄存器放在`[]`中，而AT&T语法中将基址寄存器放在`()`中。
    
