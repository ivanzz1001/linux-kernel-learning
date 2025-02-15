# caller saved与callee saved寄存器

我们在看x86_64 CPU架构芯片手册的时候，经常会看到caller saved registers、callee saved registers相关描述。这里我们介绍一下两者。

1. `caller saved registers`

    也就是调用者保存的寄存器，是指在函数调用过程中，如果调用者(caller)希望在这些寄存器中保留某些值，则它必须在调用子函数(callee)之前保存这些寄存器的内容，因为子函数可能覆盖它们。

1. `callee saved registers`

    也就是被调用者保存的寄存器，被调用函数如果要用到这些寄存器，必须先保存它们原来的值，并在返回前恢复，这样调用者可以放心这些寄存器的值不会被改变。

caller saved registers与callee saved registers的核心区别在于保存寄存器的责任方不同

参看:
  - [X86-64 Architecture Guide](http://6.s081.scripts.mit.edu/sp18/x86-64-architecture-guide.html)

  - [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)


## 1. caller saved寄存器

1. `责任方`

    由调用者负责保存

1. `行为准则`

    - 如果调用者希望在函数调用后继续使用这些寄存器中的值，必须主动在调用前保存它们的值（例如压入栈中)

    - 被调用函数（Callee）可以随意修改这些寄存器，无需恢复原始值。

1. `典型用途`

    存放临时计算结果或参数传递

1. `常见示例(x86_64架构)`

    ```plaintext
    rax, rcx, rdx, rsi, rdi, r8-r11
    ```

## 2. callee saved寄存器

1. `责任方`

    由被调用者负责保存.

1. `行为准则`

    - 如果被调用函数需要使用这些寄存器，必须在函数入口保存其原始值（例如压入栈中），并在返回前恢复

    - 调用者可以假设这些寄存器的值在函数调用后保持不变。

1. `典型用途`

    存放需要长期保留的变量.

1. `常见示例(x86_64架构)`

    ```plaintext
    rbx, rsp, rbp, r12-r15
    ```

## 3. 应用场景

假设函数**A**调用函数**B**:

1. `caller saved场景`

    - A 在调用 B 前，若 rax 中有重要数据（Caller-Saved），需手动保存

      ```assembly
      push rax                 ; 保存 rax
      call B
      pop rax                  ; 恢复 rax
      ```

    - B 可以自由使用 rax，无需恢复。

1. `callee saved场景`

    - 若 B 需要使用 rbx（Callee-Saved），需在内部保存和恢复

      ```assembly
      B:
          push rbx                     ; 保存原始 rbx
          ...                          ; 使用 rbx
          pop rbx                      ; 恢复 rbx
          ret
      ```
    - A 无需关心 rbx 是否被修改

## 4. 设计原因

可能用户会有疑问，为什么需要这样的划分？这样的设计是为了减少保存和恢复寄存器的开销。因为不是所有的寄存器都需要在每次调用时都保存，根据责任划分，调用者可以根据需要保存必要的寄存器，而被调用者负责保存那些可能被自己使用的、需要长期保留的寄存器。

1. `性能优化`

    减少不必要的保存/恢复操作。Caller-Saved 寄存器适用于短期数据，Callee-Saved 适用于长期数据。

1. `责任分离`

    调用者更清楚哪些临时值需要保留，而被调用者负责维护稳定环境

## 5. 常见问题

- Q：为什么需要区分两者？

  A：平衡性能与代码复杂性。若所有寄存器由一方保存，会导致冗余操作。

- Q：如何确定寄存器的类别？

  A：由架构的 ABI（应用程序二进制接口） 规定（如 x86-64 的 System V ABI）。
