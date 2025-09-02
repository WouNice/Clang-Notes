# C语言学习指南

C语言作为一门基础且强大的编程语言，在计算机科学领域占据着举足轻重的地位。无论是操作系统、嵌入式开发，还是算法实现，C语言都有着广泛的应用。今天，我们就来深入探讨C语言的核心知识，从基础到高级，带你全面了解C语言的魅力。

## C语言基础

### 变量与数据类型

在C语言中，变量是存储数据的基本单元，而数据类型则决定了变量的存储方式和可存储的数据范围。常见的数据类型包括：

-   基本数据类型：`int`（整型）、`float`（单精度浮点型）、`double`（双精度浮点型）、`char`（字符型）
-   构造数据类型：`struct`（结构体）、`union`（联合体）、`enum`（枚举）
 -   指针类型：通过`*`声明，用于存储变量的地址

```
#include <stdio.h>

int main() {
    int age = 20;
    float height = 175.5;
    double weight = 68.3;
    char gender = 'M';

    printf("Age: %d\n", age);
    printf("Height: %.2f\n", height);
    printf("Weight: %.2f\n", weight);
    printf("Gender: %c\n", gender);

    return0;
}
```

### 运算符

C语言提供了丰富的运算符，包括算术运算符（`+`、`-`、`*`、`/`、`%`）、关系运算符（`>`、`<`、`>=`、`<=`、`==`、`!=`）、逻辑运算符（`&&`、`||`、`!`）以及位运算符（`&`、`|`、`^`、`~`、`<<`、`>>`）等。

```
#include <stdio.h>

int main() {
    int a = 10, b = 3;

    // 算术运算符
    printf("a + b = %d\n", a + b);
    printf("a - b = %d\n", a - b);
    printf("a * b = %d\n", a * b);
    printf("a / b = %d\n", a / b);
    printf("a %% b = %d\n", a % b);

    // 关系运算符
    printf("a > b: %d\n", a > b);
    printf("a < b: %d\n", a < b);
    printf("a == b: %d\n", a == b);

    // 逻辑运算符
    printf("a > b && a > 5: %d\n", (a > b) && (a > 5));
    printf("a < b || a > 5: %d\n", (a < b) || (a > 5));
    printf("! (a > b): %d\n", !(a > b));

    return0;
}
```

### 控制结构

控制结构是程序的核心，用于控制程序的执行流程。C语言的控制结构包括：

-   条件语句：`if`、`else if`、`else`、`switch`

-   循环语句：`for`、`while`、`do-while`

-   跳转语句：`break`、`continue`、`goto`、`return`

```
#include <stdio.h>

int main() {
    int num = 7;

    // 条件语句
    if (num % 2 == 0) {
        printf("%d is even\n", num);
    } else {
        printf("%d is odd\n", num);
    }

    // 循环语句
    for (int i = 0; i < 5; i++) {
        printf("Loop iteration %d\n", i);
    }

    int count = 0;
    while (count < 3) {
        printf("While loop iteration %d\n", count);
        count++;
    }

    return0;
}
```

## 函数

### 函数定义与调用

函数是C语言的基本构建块，用于实现特定的功能。函数的定义包括返回类型、函数名、参数列表和函数体。函数的调用则通过函数名和实际参数来执行函数的功能。

```
#include <stdio.h>

// 函数定义
int add(int a, int b) {
    return a + b;
}

void greet(char *name) {
    printf("Hello, %s!\n", name);
}

int main() {
    // 函数调用
    int sum = add(5, 3);
    printf("Sum: %d\n", sum);

    greet("Alice");

    return0;
}
```

### 参数传递

C语言中，函数的参数传递方式有两种：值传递和地址传递。值传递是将实际参数的值复制给形式参数，而地址传递则是将实际参数的地址传递给形式参数，通过指针操作可以修改实际参数的值。

```
#include <stdio.h>

void swapValues(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    printf("Inside swapValues (value passing): a = %d, b = %d\n", a, b);
}

void swapAddresses(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
    printf("Inside swapAddresses (address passing): a = %d, b = %d\n", *a, *b);
}

int main() {
    int x = 5, y = 10;

    swapValues(x, y);
    printf("After swapValues: x = %d, y = %d\n", x, y);

    swapAddresses(&x, &y);
    printf("After swapAddresses: x = %d, y = %d\n", x, y);

    return0;
}
```

### 递归函数

递归函数是指函数直接或间接调用自身的一种函数。递归函数通常用于解决可以分解为子问题的问题，例如阶乘计算、斐波那契数列等。

```
#include <stdio.h>

// 递归函数计算阶乘
int factorial(int n) {
    if (n == 0 || n == 1) {
        return1;
    } else {
        return n * factorial(n - 1);
    }
}

int main() {
    int num = 5;
    printf("%d! = %d\n", num, factorial(num));

    return0;
}
```

## 数组与指针

### 数组

数组是一组相同类型数据的集合，通过索引来访问数组中的元素。数组的大小在定义时确定，且在程序运行期间不能改变。

```
#include <stdio.h>

int main() {
    int numbers[5] = {1, 2, 3, 4, 5};

    printf("Array elements: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", numbers[i]);
    }
    printf("\n");

    return 0;
}
```

### 指针

指针是C语言中非常重要的概念，它存储变量的内存地址。通过指针，可以间接访问和操作变量的值，实现地址传递、动态内存分配等功能。

```
#include <stdio.h>

int main() {
    int num = 10;
    int *ptr = &num;

    printf("Value of num: %d\n", num);
    printf("Address of num: %p\n", &num);
    printf("Value stored in ptr: %p\n", ptr);
    printf("Value at address stored in ptr: %d\n", *ptr);

    return 0;
}
```

### 指针与数组的关系

指针和数组在C语言中有着紧密的联系。数组名本质上是一个指向数组首元素的指针，可以通过指针操作数组元素。

```
#include <stdio.h>

int main() {
    int arr[] = {1, 2, 3, 4, 5};

    printf("Using array index:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    printf("Using pointer arithmetic:\n");
    for (int *ptr = arr; ptr < arr + 5; ptr++) {
        printf("%d ", *ptr);
    }
    printf("\n");

    return0;
}
```

### 指针与函数

指针在函数中有着广泛的应用，例如通过指针作为函数参数实现地址传递，或者使用函数指针实现回调函数等。

```
#include <stdio.h>

void modifyValue(int *value) {
    (*value)++;
}

int main() {
    int num = 5;
    modifyValue(&num);
    printf("Modified value: %d\n", num);

    return 0;
}
```

## 结构体与联合体

### 结构体

结构体是一种用户自定义的数据类型，可以将不同类型的数据组合成一个整体，便于管理和操作。

```
#include <stdio.h>

struct Student {
    char name[50];
    int age;
    float gpa;
};

int main() {
    struct Student student1 = {"Alice", 20, 3.8};
    struct Student student2 = {"Bob", 22, 3.5};

    printf("Student 1: %s, %d, %.1f\n", student1.name, student1.age, student1.gpa);
    printf("Student 2: %s, %d, %.1f\n", student2.name, student2.age, student2.gpa);

    return0;
}
```

### 联合体

联合体也是一种用户自定义的数据类型，但它与结构体不同的是，联合体中的所有成员共享同一块内存空间。这意味着在任意时刻，联合体中只能有一个成员被赋值。

```
#include <stdio.h>

union Data {
    int i;
    float f;
    char str[20];
};

int main() {
    union Data data;

    data.i = 10;
    printf("Integer: %d\n", data.i);

    data.f = 20.5;
    printf("Float: %.1f\n", data.f);

    // 注意：赋值后之前的值会被覆盖
    strcpy(data.str, "Hello");
    printf("String: %s\n", data.str);

    return0;
}
```

## 文件操作

### 文件指针

在C语言中，文件操作是通过文件指针来实现的。文件指针指向一个文件结构体，用于管理文件的读写操作。

```
#include <stdio.h>

int main() {
    FILE *file = fopen("example.txt", "w");
    if (file == NULL) {
        printf("Unable to open file\n");
        return1;
    }

    fprintf(file, "Hello, World!\n");
    fclose(file);

    return0;
}
```

### 文件读写

文件的读写操作可以通过多种函数实现，如`fread`、`fwrite`、`fscanf`、`fprintf`等。根据文件的打开模式（如`"r"`、`"w"`、`"a"`、`"rb"`、`"wb"`等），可以选择不同的读写方式。

```
#include <stdio.h>

int main() {
    FILE *file = fopen("data.txt", "w");
    if (file == NULL) {
        printf("Unable to open file\n");
        return1;
    }

    int numbers[] = {1, 2, 3, 4, 5};
    fwrite(numbers, sizeof(int), 5, file);
    fclose(file);

    file = fopen("data.txt", "r");
    if (file == NULL) {
        printf("Unable to open file\n");
        return1;
    }

    int readNumbers[5];
    fread(readNumbers, sizeof(int), 5, file);
    printf("Numbers read from file: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", readNumbers[i]);
    }
    printf("\n");

    fclose(file);

    return0;
}
```

## 动态内存分配

### 内存分配函数

C语言提供了动态内存分配的函数，如`malloc`、`calloc`、`realloc`和`free`。通过这些函数，可以在程序运行时动态地分配和释放内存。

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *arr = (int *)malloc(5 * sizeof(int));
    if (arr == NULL) {
        printf("Memory allocation failed\n");
        return1;
    }

    for (int i = 0; i < 5; i++) {
        arr[i] = i + 1;
    }

    printf("Array elements: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    free(arr);

    return0;
}
```

### 内存管理

动态内存分配需要注意内存的释放，避免内存泄漏。同时，要确保分配的内存足够大，避免越界访问导致程序崩溃。

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    int size;
    printf("Enter array size: ");
    scanf("%d", &size);

    int *arr = (int *)calloc(size, sizeof(int));
    if (arr == NULL) {
        printf("Memory allocation failed\n");
        return1;
    }

    for (int i = 0; i < size; i++) {
        arr[i] = i + 1;
    }

    printf("Array elements: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    free(arr);

    return0;
}
```

### 预处理器指令

7.1 宏定义

预处理器指令在编译之前对源代码进行处理。常用的预处理器指令包括`#define`（宏定义）、`#include`（文件包含）、`#ifdef`、`#ifndef`、`#endif`（条件编译）等。

```
#include <stdio.h>

#define PI 3.14159

int main() {
    float radius = 5.0;
    float area = PI * radius * radius;

    printf("Area of circle: %.2f\n", area);

    return 0;
}
```

### 文件包含

通过`#include`指令可以包含头文件，头文件中通常包含函数的声明、宏定义等。常见的头文件有`stdio.h`（标准输入输出）、`stdlib.h`（标准库函数）、`math.h`（数学函数）等。

```
#include <stdio.h>
#include <math.h>

int main() {
    double num = 16.0;
    double squareRoot = sqrt(num);

    printf("Square root of %.2f: %.2f\n", num, squareRoot);

    return 0;
}
```

