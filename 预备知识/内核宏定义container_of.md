# Linux内核宏定义container_of 

Linux container_of宏定义在include/linux/container_of.h中，它的主要作用就是根据我们结构体中的已知的成员变量的地址,来寻求该结构体的首地址。

参看:

- [Linux中常见的宏](https://zhuanlan.zhihu.com/p/512761413)

- [Linux内核中的宏定义](https://zhuanlan.zhihu.com/p/544043183)

- [GCC在线手册](https://gcc.gnu.org/onlinedocs/)

## 1.  __same_type宏

在include/linux/compiler_types.h中，定义了`__same_type`宏:

```c
/* Are two types/vars the same type (ignoring qualifiers)? */
#define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
```

### 1.1 __builtin_types_compatible_p()

是GCC的一个内置函函数，用于在编译期检查两个类型是否兼容(compatible)， 函数原型如下:
```c
int __builtin_types_compatible_p(type1, type2);
```
若`type1`与`type2`兼容(compatible)，则返回1；否则返回0。

1. `compatible`的含义

    如果两个类型是严格相同的，那么肯定满足compatible。此外，还需要注意以下场景：

      - 两个类型的sizeof值跟它的类型判断是相互不影响的，例如：在i386架构下，`sizeof(char *)` 和 `sizeof(int)` 的值是相同的，但是他们的类型却是不相同的

      - 限定符会被忽略，例如const long和long他们的类型是相同的

      - 相同的类型不同的指针级数是不相同的例如：`double*`和`double**`是不相同的

      - 连个用typedef定义的类型，当且仅当它们定义的类型相同的时候，才是兼容的

      - enum类型是一种特殊的情况，两个enum类型是不相同的

2. 使用示例

    ```c
    #include <stdio.h>

    int main(int argc, char *argv[]) 
    {
        if (__builtin_types_compatible_p(int, int)) {
            printf("int and int are compatible.\n");
        } else {
            printf("int and int are not compatible.\n");
        }

        if (__builtin_types_compatible_p(int, unsigned int)) {
            printf("int and unsigned int are compatible.\n");
        } else {
            printf("int and unsigned int are not compatible.\n");
        }

        typedef int my_int;
        if (__builtin_types_compatible_p(int, my_int)) {
            printf("int and my_int are compatible.\n");
        } else {
            printf("int and my_int are not compatible.\n");
        }

        return 0;
    }
    ```
    输出结果：
    ```bash
    int and int are compatible.
    int and unsigned int are not compatible.
    int and my_int are compatible.
    ```

3. 参考

    - [__builtin_types_compatible_p函数](https://blog.csdn.net/akakakak250/article/details/64129401)

    - [借用gcc内置函数帮助C来实现函数重载](https://www.cnblogs.com/imreW/p/17304116.html)

## 2. static_assert()

static_assert 是 C++11 引入的编译时断言特性，允许在编译期进行条件检查，并在条件不满足时产生编译错误。

这一特性非常有用，因为它可以在编译阶段就发现潜在的错误，而不是等到运行时。这对于模板编程、类型检查、常量表达式的验证等场景非常重要。

### 2.1 语法格式

static_assert 的基本语法如下：

```c
static_assert( constant_expression, error_message );
```
其中:
  - `constant_expression`：一个在编译时可求值的常量表达式。如果表达式的结果为 false，则会产生编译错误。

  - `error_message`：当断言失败时，编译器将显示的错误信息。这个消息必须是一个字符串字面量。

### 2.2 使用示例

1. 检查类型大小

    在跨平台开发时，确保特定类型具有预期的大小是非常重要的。可以使用 static_assert 来验证类型的大小。

    ```c
    #include <iostream>
    #include <cstdint>

    int main() {
        static_assert(sizeof(int) == 4, "int must be 4 bytes");
        static_assert(sizeof(std::int64_t) == 8, "64-bit integer must be 8 bytes");

        std::cout << "Type sizes are as expected." << std::endl;
        return 0;
    }
    ```
    程序输出:
    
    ```bash
    Type sizes are as expected.
    ```

    如果在某个平台上 int 不是 4 字节或者 std::int64_t 不是 8 字节，上述代码将无法编译，编译器会输出相应的错误信息。

2. 模板编程中的条件验证

    static_assert 可以在模板编程中用来验证模板参数满足特定条件。

    ```c
    #include <iostream>
    #include <type_traits> // 提供std::is_integral

    template<typename T>
    void mustBeIntegral(T value) {
        static_assert(std::is_integral<T>::value, "Template parameter T must be an integral type.");
        std::cout << "Value is: " << value << std::endl;
    }

    int main() {
        mustBeIntegral(10);           // 正确：T为int，是整型
        // mustBeIntegral(3.14);      // 错误：T为double，编译将失败
        return 0;
    }
    ```

    程序输出：
    
    ```bash
    Value is: 10
    ```

    如果尝试将非整型作为模板参数 T 传递给 mustBeIntegral 函数，编译器将显示 static_assert 指定的错误消息。

    例如上述程序中，把 mustBeIntegral(3.14) 这行代码打开，编译时的错误信息如下：

    ```c
    main.cpp: In instantiation of ‘void mustBeIntegral(T) [with T = double]’:
    main.cpp:12:19:   required from here
    main.cpp:6:40: error: static assertion failed: Template parameter T must be an integral type.
     6 |     static_assert(std::is_integral<T>::value, "Template parameter T must be an integral type.");
      |                                        ^~~~~
    main.cpp:6:40: note: ‘std::integral_constant::value’ evaluates to false
    ```

3. 常量表达式验证

    在需要编译时计算的场景下，可以使用 static_assert 来验证常量表达式的结果。

    ```c
    constexpr int getArraySize() {
        return 42;
    }

    int main() {
        static_assert(getArraySize() == 42, "Array size must be 42.");
        int myArray[getArraySize()]; // 使用常量表达式作为数组大小

        std::cout << "Array size is as expected." << std::endl;
        return 0;
    }
    ```

    程序输出：

    ```bash
    Array size is as expected.
    ```

    这个例子中，static_assert 用于验证由 constexpr 函数 getArraySize 返回的大小是否符合预期，确保数组大小的定义是正确的。

### 2.3 总结及参考
通过 static_assert，C++ 程序员可以更容易地在编译时捕获错误和强制执行约束，这有助于提高代码质量和稳定性。

参考：
  
  - [C++11 新特性：编译时断言 static_assert](https://zhuanlan.zhihu.com/p/688155758)


## 3. _Generic

`_Generic`是在C11标准中(ISO/IEC 9899:2011）为C语言引入的轻量级泛型编程。尽管 C 语言不像 C++ 那样支持面向对象编程和模板，但它通过 _Generic 提供了一种在编译时根据表达式的类型选择不同代码路径的方式。这使得 C 语言能够在某种程度上实现类似于泛型编程的设计。

### 3.1 什么是泛型编程？

泛型编程是一种编程范式，它允许程序员在编写代码时使用一些将来才会指定的类型。这些类型在代码实例化时作为参数指明。例如，在 C++ 中，可以通过模板来支持泛型编程。

```c
std::vector<T>, std::list<T>, std::set<T>
```

### 3.2 C语言中的泛型编程

在 C 语言中，虽然没有真正意义上的泛型编程，但 C11 标准中的 _Generic 关键字提供了一种在编译时根据赋值表达式的类型在泛型关联表中选择一个表达式的方法。这样可以将一组功能相同但类型不同的函数抽象为一个统一的接口。

### 3.3 _Generic 的语法形式

语法形式如下:

```c
_Generic(expression, type1: result1, type2: result2, ..., default: default_result)
```

参数说明：

- expression：该表达式的类型会被评估(evaluate)

- type1, type2, ...: 表达式所匹配的类型列表

- result1, result2, ...: 表达式所匹配类型对应的值

- default: 可选，当前面的类型均不匹配时的默认结果


参看:

  - [C语言泛型 _Generic](https://zhuanlan.zhihu.com/p/718600927)

  - [deepseek: _Generic() Linux](https://chat.deepseek.com/)


## 4. offsetof()

offsetof宏‌是C和C++语言中的一个宏，定义在stddef.h头文件中。它的主要作用是计算结构体成员在内存中的偏移量，即获取结构体成员相对于结构体起始地址的字节偏移.

1. 工作原理

    offsetof宏的工作原理是通过将0地址强制转换为结构体类型的指针，然后访问该结构体的成员，并取出该成员的地址。由于结构体的起始地址为0，因此成员地址即为相对于结构体起始地址的偏移量.

    如下是offsetof()宏的一种实现参考:

    ```c
    #define offsetof(type, member) ((size_t)&(((type *)0)->member))
    ```


## 5. container_of宏

```c
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:	the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:	the name of the member within the struct.
 *
 * WARNING: any const qualifier of @ptr is lost.
 */
#define container_of(ptr, type, member) ({				\
	void *__mptr = (void *)(ptr);					\
	static_assert(__same_type(*(ptr), ((type *)0)->member) ||	\
		      __same_type(*(ptr), void),			\
		      "pointer type mismatch in container_of()");	\
	((type *)(__mptr - offsetof(type, member))); })
```

有了前面的介绍，这里container_of就很好理解了。

## 6. container_of_const宏

```c
/**
 * container_of_const - cast a member of a structure out to the containing
 *			structure and preserve the const-ness of the pointer
 * @ptr:		the pointer to the member
 * @type:		the type of the container struct this is embedded in.
 * @member:		the name of the member within the struct.
 */
#define container_of_const(ptr, type, member)				\
	_Generic(ptr,							\
		const typeof(*(ptr)) *: ((const type *)container_of(ptr, type, member)),\
		default: ((type *)container_of(ptr, type, member))	\
	)
```

