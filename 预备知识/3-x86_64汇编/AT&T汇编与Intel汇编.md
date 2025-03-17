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

   
