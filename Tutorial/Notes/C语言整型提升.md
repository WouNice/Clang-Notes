# 整型提升

整型提升是C程序设计语言中的一项规定：

>   在表达式计算时，各种整形首先要提升为int类型，如果int类型不足以表示的话，就需要提升为unsigned int类型，然后再执行表达式的运算。

整型提升按照以下规则进行：

-   有符号的int整数和无符号的int整数运算自动转换为无符号数

-   当宽度小于int类型的无符号数和有符号整数进行算法时，首先转换为int类型的有符号数

-   当宽度小于int类型的无符号数和无符号整数进行算法时，转换为int类型的无符号数

举例子来了解一下整形提升：

一些数据类型（比如char，short int）比int占用更少的字节数，对它们执行操作时，这些数据类型会自动提升为int或unsigned int，例如，在较小的类型（如char，short和enum）上不会进行算术计算，代码如下：

```c
#include <stdio.h>

int main() {
    char a = 30, b = 40, c = 10;
    char d = (a * b) / c;
    printf("%d ", d);
    return 0;
}
// 输出结果：120
```

直接看代码，表达式`（a * b）/ c`似乎引起算术溢出，因为带符号的字符只能具有-128至127的值（在大多数C编译器中），而子表达式的值`（a * b）= 1200`，大于128。

但是整数提升是在char类型进行算术运算时发生的，我们得到了适当的结果而没有任何溢出。

## 整型提升的意义

虽然机器指令中可能有出现两个8比特字节这种字节相加指令，但是一般用途的CPU是难以直接实现这样的字节相加运算的。

所以，表达式中各种长度可能小于int长度的整型值，都必须先转换为int或unsigned int，然后才能送入CPU去执行运算。

CPU内整型运算器(ALU)的操作数的字节长度一般就是int的字节长度，同时也是CPU的通用寄存器的长度。而表达式的整型运算要在CPU的相应运算器件内执行。

因此，两个char类型的树进行相加运算时，是在CPU中执行，自然而然的需要先转换为CPU内整型操作数的标准长度。

## 隐式转换规则

如下代码的输出结果是？为什么？

```c
#include <stdio.h>

int main(void) {
    unsigned int a = 6;
    int b = -20;
    if (a + b > 6)
        printf("a+b大于6\n");
    else
        printf("a+b小于6\n");
    return 0;
}
```

程序输出结果为

```
a+b大于6
```

原因是因为编译器会将有符号数b转换成为一个无符号数，即此处 a+b 等价于`a + (unsigned int)b`。

该程序运行在32bit环境下，b的值为 `0xFFFFFFFF-20+1 = 4294967276`，即a+b将远远大于6。

C 语言按照一定的规则来进行此类运算的转换，这种规则称为正常算术转换，转换的顺序为：

```c
double>float>unsigned long>long>unsigned int>int>char
```

即操作数类型排在后面的与操作数类型排在前面的进行运算时，排在后面的类型将 隐式转换 为排在前面的类型。

## 应用举例

### short int的长度 = int的长度的情况

C语言标准中仅规定了：

```c
char的长度 ≤ short int的长度 ≤ int的长度
```

这意味着short int与int的长度相等的可能，这种情形下，unsigned short就无法提升为int表示，只能提升为unsigned int，代码如下：

```c
#include <stdio.h>

int main() {
    char a = 0xb6;
    short b = 0xb600;
    int c = 0xb6000000;
    if (a == 0xb6) printf("a");
    if (b == 0xb600) printf("b");
    if (c == 0xb6000000) printf("c");
    return 0;
}
// 输出结果：c
```

C语言标准没有规定char类型是有符号还是无符号，在这些环境下，编译器把char定义为signed char。

表达式`a==0xb6`被整型提升，其中`char类型的a`提升为int类型并表示为一个负值，因此这个表达式的结果为false；

表达式`b==0xb600`被整型提升，其中`short类型的b`提升为int类型并为一个负值，因此这个表达式的结果为false；

表达式`c == 0xb6000000`没有做整型提升，==运算符的两段都是int类型的负值，其结果为true。

我们再考虑以下程序作为另一个示例。

```c
#include <stdio.h>

int main() {
    char a = 0xfb;
    unsigned char b = 0xfb;
    printf("a = %c\n", a);
    printf("b = %c\n", b);
    if (a == b) {
        printf("Same\n");
    } else {
        printf("Not Same\n");
    }
    return 0;
}
```

输出结果：

```
a =
b =
Not Same
```

当我们打印“a”和“b”时，将打印相同的字符，但是当我们比较它们时，输出的结果却不相同。

-   “a”和“b”与char具有相同的二进制表示形式，但是，当对“a”和 ”b”执行比较操作时，它们首先会转换为int。
-   “a”是一个有符号的字符，当转换为int时，其值变为-5（有符号的值0xfb）。
-   “b”是无符号字符，当将其转换为int时，其值变为251。

值-5和251具有不同的int表示形式，因此我们得到的输出为“Not Same”。

### 前缀+的情况

C语言的单操作数的+运算符（即“前缀+”），一个主要作用就是实现对操作数的整型提升。例如：

```c
#include <stdio.h>

int main() {
    char a = 1;
    printf("%u\n", sizeof(a));
    printf("%u\n", sizeof(+a));
    return 0;
}
```

输出结果：

```
1
4
```

从结果中我们可以看到，前缀+把大小给提升了。
