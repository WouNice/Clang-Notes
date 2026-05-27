# 基础C语法

## static的作用

在C语言中，关键字static有三个明显的作用：

- 在函数体修饰变量：一个被声明为静态的变量在这一函数被调用过程中维持其值不变。
- 在模块内（但在函数体外）修饰变量：一个被声明为静态的变量可以被模块内所用函数访问，但不能被模块外其它函数访问。它是一个本地的全局变量。
- 在模块内修饰函数：一个被声明为静态的函数只可被这一模块内的其它函数调用。那就是，这个函数被限制在声明它的模块的本地范围内使用。

## const的作用

下面的声明都是什么意思：

```c
const int a;
int const a;
const int *a;
int * const a;
int const * a const;
```

- 前两个的作用是一样，a是一个常整型数。
- 第三个意味着a是一个指向常整型数的指针（也就是，整型数是不可修改的，但指针可以）。
- 第四个意思a是一个指向整型数的常指针（也就是说，指针指向的整型数是可以修改的，但指针是不可修改的）。
- 最后一个意味着a是一个指向常整型数的常指针（也就是说，指针指向的整型数是不可修改的，同时指针也是不可修改的）。

## volatile的作用

volatile是C和C++中的一个关键字，它的作用是告知编译器，被该关键字修饰的变量可能会在程序的外部被改变，因此编译器在优化代码时不能对该变量进行一些可能导致错误的优化操作，具体如下：

（1）确保对变量的每次访问都是从内存中读取

编译器通常会将频繁访问的变量缓存到寄存器中，以提高访问速度。但对于volatile修饰的变量，编译器会每次都从内存中读取其值，而不是使用寄存器中的缓存值。

例如，在多线程程序中，一个线程修改了volatile修饰的变量，其他线程能立即读取到最新值，而不是旧的缓存值。

（2）禁止编译器对代码进行优化重排

编译器为了提高程序的执行效率，可能会对代码进行优化重排，改变语句的执行顺序。但对于volatile变量的读写操作，编译器不会将其与其他指令重排，以保证程序按照代码的书写顺序来访问volatile变量。

具体来说，`volatile`有以下几个主要用途：

- **硬件寄存器访问**：在访问硬件寄存器时，由于硬件可能随时改变寄存器的值，所以需要用`volatile`来确保每次读取或写入都是从实际的硬件寄存器而不是可能被优化过的缓存中进行。例如在嵌入式系统开发中，当与外设进行交互时，外设的状态寄存器可能会被硬件随时更新，使用`volatile`可以确保程序正确地读取到最新的状态。
- **多线程编程**：在多线程环境中，如果一个变量可能被多个线程同时访问和修改，而没有使用合适的同步机制，那么这个变量就可能在不同的线程中被意外地改变。将这个变量声明为`volatile`可以提醒编译器不要对该变量进行过度的优化，从而避免一些难以调试的错误。例如，一个标志变量被一个线程设置为某个特定值以通知另一个线程执行特定的操作，这个标志变量就应该被声明为`volatile`。
- **中断处理**：在处理中断服务程序时，被中断服务程序修改的变量应该被声明为`volatile`。因为中断可能在任何时候发生，并且中断服务程序可能会修改这些变量的值，所以编译器不能对这些变量进行优化，必须每次都从内存中读取它们的值。

### 常见volatile面试题

**1. 一个参数既可以是const还可以是volatile吗？解释为什么。**

是的。一个例子是只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它。

**2. 一个指针可以是volatile吗？解释为什么。**

是的。尽管这并不很常见。一个例子是当一个中断服务子程序修改一个指向一个buffer的指针时。

**3. 下面的函数有什么错误：**

```c
int square(volatile int *ptr) {
    return *ptr * *ptr;
}
```

这段代码的目的是用来返回指针*ptr指向值的平方，但是，由于*ptr指向一个volatile型参数，编译器将产生类似下面的代码：

```c
int square(volatile int *ptr) {
    int a, b;
    a = *ptr;
    b = *ptr;
    return a * b;
}
```

由于*ptr的值可能在两次取值语句之间发生改变，因此a和b可能是不同的。结果，这段代码可能返回的不是你所期望的平方值！正确的代码如下：

```c
long square(volatile int *ptr) {
    int a;
    a = *ptr;
    return a * a;
}
```

> **注意**：volatile只能保证可见性，不能保证原子性。

## register关键字的作用

在C语言中，register关键字用于建议编译器将变量存储在寄存器中，而不是常规的内存中。其作用主要有以下几点：

（1）**提高访问速度**：寄存器是CPU内部的高速存储单元，对寄存器的访问速度远远快于对内存的访问速度。将频繁使用的变量声明为register，可以让编译器优先将其存储在寄存器中，这样在程序运行时，对该变量的读写操作就能更快地完成，从而提高程序的执行效率。

（2）**减少内存访问**：对于一些需要频繁读写的变量，如循环变量，使用register关键字可以减少对内存的访问次数，降低内存总线的负载，同时也有助于提高整个系统的性能。

不过，register只是一个建议，编译器可以根据实际情况决定是否将变量放入寄存器。例如，当寄存器资源紧张或者变量的使用方式不适合放在寄存器中时，编译器可能会忽略这个建议。另外，register变量不能取地址，因为它可能没有存储在内存中固定的地址上。

## restrict关键字的作用

`restrict`是C99引入的关键字，用于修饰指针，表明该指针是访问其指向对象的唯一方式。

- 作用：告诉编译器，对于被`restrict`修饰的指针，只有它能访问所指向的数据，编译器可以据此进行更激进的优化。
- 使用场景：常见于高性能计算和库函数中，如`memcpy`、`memmove`等函数的参数声明。

```c
void *memcpy(void *restrict dest, const void *restrict src, size_t n);
```

这表示dest和src指向的内存区域不重叠，编译器可以生成更高效的代码。

## sizeof和strlen的区别

| 特性 | sizeof | strlen |
|------|--------|--------|
| **类型** | 运算符 | 函数 |
| **计算时机** | 编译时 | 运行时 |
| **参数类型** | 任意数据类型、变量、数组、指针 | 字符指针（char*） |
| **返回值** | 整个对象占用的字节数（含\0对于字符数组） | 字符串有效字符数（不含\0） |
| **数组行为** | 不退化，返回数组总大小 | 退化为指针，按指针处理 |

### sizeof示例

```c
char *ss = "0123456789";
sizeof(ss);   // 8字节（64位系统指针大小）
sizeof(*ss);  // 1字节（char类型大小）

char ss[] = "0123456789";
sizeof(ss);   // 11字节（10个字符 + 1个\0）
sizeof(*ss);  // 1字节
```

### strlen示例

```c
char ss[] = "0123456789";
strlen(ss);   // 10（有效字符数，不含\0）

char p[] = "a\n";
strlen(p);   // 2（\n是转义字符但仍是1个字符）
```

## 指针和引用的区别

> C语言没有引用，引用是C++的概念。这里主要讨论C中的指针。

（1）**定义与性质**

- **指针**：是一个变量，其值为另一个变量的地址，通过访问指针所存储的地址来间接访问它所指向的变量。可以被赋值为不同的地址，从而指向不同的变量，并且可以为空指针NULL。
- **引用**（C++）：是一个已存在变量的别名，它和被引用的变量在内存中具有相同的地址，从本质上讲，它就是被引用的变量本身，不占用额外的内存空间来存储地址，且必须在定义时初始化，之后不能再引用其他变量。

（2）**操作符**

- **指针**：使用`*`运算符来访问指针所指向的变量的值，称为解引用操作。例如`*p`表示访问指针p所指向的变量。另外，使用`&`运算符可以获取变量的地址，用于给指针赋值。
- **引用**（C++）：在定义引用时使用`&`符号，但在使用引用时，直接使用引用名就可以访问其所引用的变量，不需要额外的操作符来解引用。

（3）**内存分配与释放**

- **指针**：可以动态地分配和释放内存，通过malloc、calloc等函数分配内存，使用free函数释放内存。如果指针指向的内存没有被正确释放，会导致内存泄漏。
- **引用**（C++）：本身不涉及内存的分配和释放，它只是对已存在变量的别名，其生命周期与被引用的变量相同，由系统自动管理内存。

（4）**作为函数参数**

- **指针**：将指针作为函数参数时，传递的是变量的地址。在函数内部可以通过修改指针所指向的内容来改变实参的值，但如果不小心修改了指针本身的值（即改变了它所指向的地址），不会影响到实参指针变量的值。
- **引用**（C++）：作为函数参数时，传递的是变量本身的别名。在函数内部对引用的操作等同于对实参变量的操作，任何对引用的修改都会直接反映到实参上。

## typedef与define的区别

（1）#define之后不带分号，typedef之后带分号。

（2）#define可以使用其他类型说明符对宏类型名进行扩展，而typedef不能这样做。如：

```c
#define INT1 int
unsigned INT1 n;  // 没问题

typedef int INT2;
unsigned INT2 n;  // 有问题
```

INT1可以使用类型说明符unsigned进行扩展，而INT2不能使用unsigned进行扩展。

（3）在连续定义几个变量的时候，typedef能够保证定义的所有变量均为同一类型，而#define则无法保证。如：

```c
#define PINT1 int*;
PINT1 p1, p2;  // 即int *p1, p2; p2是int而非int*

typedef int* PINT2;
PINT2 p1, p2;  // p1、p2类型相同，都是int*
```

## 指针函数和函数指针

### 指针函数

本质是一个函数，函数的返回值是一个指针。其定义形式为`type *function_name(parameters)`。

```c
int *func(int a, int b);  // 指针函数，返回int*类型
```

### 函数指针

本质是一个指针，该指针指向一个函数。其定义形式为`type (*pointer_name)(parameters)`。

```c
int (*ptr)(int a, int b);  // 函数指针，指向返回int、参数为(int, int)的函数
```

### 使用场景

- **回调函数**：如`qsort`中的比较函数
- **状态机**：根据状态选择不同函数执行
- **函数指针数组**：实现菜单驱动系统
- **面向对象模拟**：C语言中模拟多态

## 数组指针和指针数组

### 数组指针

是一个指针变量，它指向一个数组。其定义形式为`type (*pointer_name)[array_size]`。

```c
int (*p)[5];  // p是指向包含5个int元素的数组的指针
```

### 指针数组

是一个数组，数组中的每个元素都是一个指针。其定义形式为`type *pointer_array_name[array_size]`。

```c
int *p[5];  // p是包含5个int指针的数组
```

## union和struct存储的区别

（1）**内存占用**

- **struct**：结构体中的每个成员都有独立的内存空间，其占用的内存大小是所有成员变量大小之和，可能会存在内存对齐的情况。
- **union**：联合的所有成员共享同一块内存空间，其占用的内存大小是其最大成员变量的大小。

（2）**数据存储方式**

- **struct**：可以同时存储多个不同类型的数据成员，每个成员都有自己独立的存储空间，互不影响。
- **union**：在同一时刻只能存储一个成员的值，当给联合的一个成员赋值时，会覆盖其他成员的值。

```c
struct DataStruct {
    int num;      // 4字节
    char ch;      // 1字节
    float f;      // 4字节
};
// sizeof(struct DataStruct) = 12字节（可能有填充）

union DataUnion {
    int num;      // 4字节
    char ch;      // 1字节
    float f;      // 4字节
};
// sizeof(union DataUnion) = 4字节（所有成员共享同一空间）
```

## 大端模式和小端模式

（1）**数据存储顺序**

- **大端模式（Big Endian）**：数据的高位字节存于低地址，低位字节存于高地址。

```
例如0x12345678在大端模式下的内存布局：
地址:  0x00  0x01  0x02  0x03
数据:  0x12  0x34  0x56  0x78
```

- **小端模式（Little Endian）**：数据的低位字节存于低地址，高位字节存于高地址。x86架构采用小端模式。

```
例如0x12345678在小端模式下的内存布局：
地址:  0x00  0x01  0x02  0x03
数据:  0x78  0x56  0x34  0x12
```

（2）**判断大小端的方法**

```c
// 方法1：联合类型判断法
int check_endian() {
    union {
        int num;
        char ch;
    } un;
    un.num = 1;
    return un.ch == 1;  // 返回1为小端，0为大端
}

// 方法2：指针强制类型转换判断法
int check_endian2() {
    int num = 1;
    char *p = (char *)&num;
    return *p == 1;  // 返回1为小端，0为大端
}
```

## 内存对齐

（1）**基本原理**

现代计算机的内存通常按字节编址，但在访问数据时，并非以单个字节为单位进行读写，而是以一定的块大小（如4字节、8字节等）进行操作。内存对齐就是让数据的存储地址满足特定的对齐要求，通常是数据类型大小的整数倍。

（2）**对齐规则**

| 类型 | 典型对齐值 |
|------|-----------|
| char | 1字节 |
| short | 2字节 |
| int | 4字节 |
| long | 4字节（32位）/ 8字节（64位） |
| float | 4字节 |
| double | 8字节 |

（3）**结构体对齐规则**

- 结构体的对齐值为其最大成员的对齐值
- 结构体大小必须是最大对齐值的整数倍
- 成员偏移量必须是其自身对齐值的整数倍

```c
struct Example {
    char a;    // 偏移0，占1字节
    int b;     // 偏移4（前面填充3字节），占4字节
    char c;    // 偏移8，占1字节
    // 结构体大小需要是4的倍数，所以总大小为12字节
};
// sizeof(struct Example) = 12
```

（4）**控制对齐方式**

```c
// 方法1：#pragma pack
#pragma pack(push, 1)
struct Packed {
    char a;
    int b;
};
#pragma pack(pop)

// 方法2：__attribute__ ((packed))
struct __attribute__((packed)) Packed2 {
    char a;
    int b;
};
```

## const和define的区别

| 特性 | const | #define |
|------|-------|---------|
| **类型** | 关键字 | 预处理指令 |
| **类型检查** | 有类型，编译器检查 | 无类型，不检查 |
| **作用域** | 有作用域（函数内有效） | 整个文件，需#undef取消 |
| **内存分配** | 分配空间（只读变量） | 不分配，替换 |
| **调试** | 可调试（可见名） | 调试困难（替换后） |
| **安全性** | 更安全 | 可能产生意外替换 |

## 堆栈溢出怎么办

（1）**检查递归调用**

- 原因：递归函数若缺少正确的终止条件，或递归层数过深，易导致堆栈溢出。
- 解决方法：仔细检查递归函数，确保有合理的终止条件。可考虑增加递归深度的限制，或使用迭代方式替代递归。

（2）**优化局部变量使用**

- 原因：函数中定义的大量局部变量会占用栈空间，当函数调用层次较多时，可能引发栈溢出。
- 解决方法：尽量减少不必要的局部变量，对于占用空间较大的局部变量，可考虑将其定义为动态分配的内存（如使用malloc），以避免占用栈空间。

（3）**调整堆栈大小**

- 解决方法：在编译器或链接器中设置更大的堆栈空间。例如，在GCC中可使用`-Wl,--stack`选项来指定栈大小。

## C语言函数返回值

C语言函数返回值类型由函数定义时指定的返回值类型决定，可以是基本数据类型、指针类型、结构体类型、枚举类型等。

```c
int add(int a, int b) {
    return a + b;  // 返回int类型
}

float divide(int a, int b) {
    return (float)a / b;  // 返回float类型
}

void hello() {
    return;  // 无返回值
}
```

## C语言变量存储方式

（1）**自动存储（Automatic storage）**

局部变量，存储在栈上。作用域仅限于定义它的代码块，当代码块执行完毕后自动释放。

```c
void function() {
    int a = 10;  // 自动存储
}
```

（2）**静态存储（Static storage）**

全局变量或static修饰的变量，存储在静态存储区（数据段），程序整个生命周期内存在。

```c
int global_var;  // 全局变量

void function() {
    static int static_local = 0;  // 静态局部变量
}
```

（3）**动态存储（Dynamic storage）**

通过malloc/calloc/realloc在堆上分配，需手动free释放。

```c
int *ptr = (int *)malloc(sizeof(int));
free(ptr);
```

（4）**寄存器存储（Register storage）**

使用register关键字，建议存储在寄存器中。

```c
register int r_variable;
```

## 变量未初始化值

| 变量类型 | 初始值 |
|---------|--------|
| 局部变量 | 随机值（垃圾值） |
| 全局变量 | 0 |
| 静态全局变量 | 0 |
| 静态局部变量 | 0 |
| 寄存器变量 | 随机值 |

```c
int global_var;  // 全局变量，初始值为0

void function() {
    int local_var;        // 局部变量，初始值为随机值
    static int static_var;  // 静态局部变量，初始值为0
}
```

## C11/C99 关键字补充

### `_Alignas`和`_Alignof`（C11）

```c
#include <stdio.h>

int main() {
    printf("Alignment of int: %zu\n", _Alignof(int));

    _Alignas(16) char aligned_char;
    printf("Alignment of aligned_char: %zu\n", _Alignof(_Alignas(16) char));

    return 0;
}
```

### _Static_assert（C11）

编译时断言，比assert更早发现问题。

```c
_Static_assert(sizeof(int) >= 4, "int must be at least 4 bytes");
_Static_assert(sizeof(void*) >= 4, " pointers must be at least 4 bytes");
```

### _Noreturn（C11）

表示函数不会返回。

```c
#include <stdio.h>
#include <stdnoreturn.h>

noreturn void fatal_error(void) {
    printf("Fatal error occurred\n");
    exit(1);
}
```

## 位操作（Bit Manipulation）

### 常见位操作技巧

| 操作 | 代码 | 说明 |
|------|------|------|
| 设置第n位为1 | `x |= (1 << n)` | 第n位设为1 |
| 设置第n位为0 | `x &= ~(1 << n)` | 第n位清零 |
| 切换第n位 | `x ^= (1 << n)` | 第n位取反 |
| 读取第n位 | `(x >> n) & 1` | 获取第n位的值 |
| 判断奇偶 | `x & 1` | 结果为1是奇数，0是偶数 |

### 实战例子

```c
// 1. 计算二进制中1的个数（Brian Kernighan算法）
int count_ones(unsigned int x) {
    int count = 0;
    while (x) {
        x &= (x - 1);  // 清除最低位的1
        count++;
    }
    return count;
}

// 2. 判断是否为2的幂
int is_power_of_two(unsigned int x) {
    return (x != 0) && ((x & (x - 1)) == 0);
}

// 3. 交换两个变量的值（不用临时变量）
void swap(int *a, int *b) {
    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}
```

## 内存管理常见错误

### 内存泄漏（Memory Leak）

```c
void leak_example(void) {
    int *p = malloc(sizeof(int) * 10);
    // 没有free就返回了，导致内存泄漏
    return;  // p指向的内存无法再被释放
}
```

**检测方法**：

- Valgrind（Linux）：`valgrind --leak-check=full ./program`
- Dr. Memory（Windows）
- AddressSanitizer：`gcc -fsanitize=address -g file.c`

### 悬空指针（Dangling Pointer）

```c
void dangling_example(void) {
    int *p = malloc(sizeof(int));
    *p = 10;
    free(p);  // p变成悬空指针
    // *p = 20;  // 危险！未定义行为
}
```

**最佳实践**：free后立即将指针设为NULL：

```c
free(p);
p = NULL;  // 防止悬空指针
```

### 双重释放（Double Free）

```c
void double_free_example(void) {
    int *p = malloc(sizeof(int));
    free(p);
    // free(p);  // 危险！双重释放
}
```

### 越界访问（Buffer Overflow）

```c
void overflow_example(void) {
    int arr[10];
    for (int i = 0; i <= 10; i++) {  // 越界访问arr[10]
        arr[i] = i;
    }
}
```

## 柔性数组（Flexible Array Member）

C99标准支持柔性数组，常用于变长数据结构：

```c
struct dynamic_array {
    size_t length;
    int data[];  // 柔性数组，必须是最后一个成员
};

// 使用
struct dynamic_array *create(size_t n) {
    struct dynamic_array *p = malloc(sizeof(struct dynamic_array) + n * sizeof(int));
    if (p == NULL) return NULL;
    p->length = n;
    for (size_t i = 0; i < n; i++) {
        p->data[i] = 0;
    }
    return p;
}
```

**优点**：

1. 一次性分配内存，减少内存碎片
2. 只需要管理一个指针
3. 缓存局部性更好

## 函数指针与回调函数

### 函数指针的定义和使用

```c
// 定义函数指针类型
typedef int (*CompareFunc)(const void *, const void *);

// 使用函数指针作为参数（回调函数）
void bubble_sort(int *arr, int n, CompareFunc compare) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (compare(&arr[j], &arr[j+1]) > 0) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

// 比较函数
int ascending(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}
```

### 函数指针的实际应用

```c
typedef void (*StateHandler)(void *context);

void state_idle(void *ctx) { /* 处理空闲状态 */ }
void state_running(void *ctx) { /* 处理运行状态 */ }
void state_error(void *ctx) { /* 处理错误状态 */ }

StateHandler state_machine[] = {state_idle, state_running, state_error};

// 调用：state_machine[current_state](context);
```
