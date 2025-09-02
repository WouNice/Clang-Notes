# C语言函数

## 一、函数的基本概念与重要性

想象一下，你是一位建筑设计师，要建造一座大型商业综合体。若每次都从头开始设计每一个房间、每一段楼梯，那工作量将大得惊人。但要是你有各种预制好的建筑模块，像标准房间、通用楼梯等，就可以根据需求快速组合，高效完成设计。C语言中的函数就如同这些预制模块，是一段具有特定功能的、可重复使用的代码块。

函数的重要性体现在多个方面：

-   **代码复用**：避免重复编写相同的代码。例如，在一个处理学生成绩的程序中，计算平均分的代码可能会在多个地方用到，将其封装成函数后，只需调用该函数即可，无需重复编写计算逻辑。
-   **模块化设计**：将复杂的程序分解为多个小的、功能单一的函数，每个函数负责一个特定的任务。这样可以使程序结构更清晰，易于理解和维护。比如，一个游戏程序可以分为游戏界面绘制、角色移动控制、碰撞检测等多个函数模块。
-   **提高可维护性**：当程序出现问题时，可以更容易地定位和修复问题。因为每个函数的功能相对独立，只需要检查和修改出现问题的函数即可。
-   **团队协作**：在大型项目开发中，团队成员可以分工编写不同的函数，然后将这些函数组合起来形成完整的程序，提高开发效率。

## 二、函数的定义与调用

函数的定义和调用是使用函数的基础操作，下面详细介绍其语法和步骤。

### （一）函数的定义

函数定义的基本语法如下：

```
返回值类型 函数名(参数列表) {
    函数体；
    return 返回值；
}
```

-   **返回值类型**：指定函数返回值的类型，可以是`int`、`float`、`char`等基本数据类型，也可以是自定义的数据类型。如果函数不返回任何值，则使用`void`类型。
-   **函数名**：是函数的标识符，用于在程序中调用该函数。函数名应具有一定的描述性，能清晰表达函数的功能。
-   **参数列表**：指定函数接受的参数，参数之间用逗号分隔。每个参数由数据类型和参数名组成。如果函数不接受任何参数，则参数列表为空，但括号不能省略。
-   **函数体**：是函数的具体实现代码，包含了一系列的语句，用于完成函数的特定功能。
-   **return语句**：用于返回函数的结果。如果函数返回值类型为`void`，则可以省略`return`语句，或者使用`return;`来结束函数。

下面是一个简单的函数定义示例，用于计算两个整数的和：

```c
int add(int a, int b) {
    int sum = a + b;
    return sum;
}
```

### （二）函数的调用

函数定义好后，就可以在其他地方调用它。函数调用的基本语法如下：

```
返回值变量 = 函数名(实际参数列表);
```

-   **返回值变量**：用于接收函数的返回值。如果函数返回值类型为`void`，则不需要使用返回值变量。
-   **实际参数列表**：是传递给函数的具体值，实际参数的数量、类型和顺序必须与函数定义中的参数列表一致。

下面是调用上述`add`函数的示例：

```c
#include <stdio.h>

int add(int a, int b) {
    int sum = a + b;
    return sum;
}

int main() {
    int result = add(3, 5);
    printf("两数之和为: %d\n", result);
    return 0;
}
```

## 三、函数参数的传递方式

在C语言中，函数参数的传递方式主要有值传递和指针传递两种，下面分别介绍它们的特点和应用场景。

### （一）值传递

值传递是指在函数调用时，将实际参数的值复制一份传递给函数的形式参数。在函数内部对形式参数的修改不会影响到实际参数的值。

下面是一个值传递的示例：

```c
#include <stdio.h>

void swap(int x, int y) {
    int temp = x;
    x = y;
    y = temp;
    printf("函数内部交换后: x = %d, y = %d\n", x, y);
}

int main() {
    int a = 3, b = 5;
    printf("交换前: a = %d, b = %d\n", a, b);
    swap(a, b);
    printf("交换后: a = %d, b = %d\n", a, b);
    return0;
}
```

在上述示例中，`swap`函数通过值传递接收`a`和`b`的值，但在函数内部交换的是形式参数`x`和`y`的值，并没有影响到实际参数`a`和`b`的值。

### （二）指针传递

指针传递是指在函数调用时，将实际参数的地址传递给函数的形式参数。在函数内部可以通过指针访问和修改实际参数的值。

下面是一个指针传递的示例：

```
#include <stdio.h>

void swap(int *x, int *y) {
    int temp = *x;
    *x = *y;
    *y = temp;
    printf("函数内部交换后: *x = %d, *y = %d\n", *x, *y);
}

int main() {
    int a = 3, b = 5;
    printf("交换前: a = %d, b = %d\n", a, b);
    swap(&a, &b);
    printf("交换后: a = %d, b = %d\n", a, b);
    return0;
}
```

在上述示例中，`swap`函数通过指针传递接收`a`和`b`的地址，在函数内部通过指针操作交换了`a`和`b`的值。

## 四、函数的作用域与存储类别

函数的作用域和存储类别决定了函数和变量的可见性和生命周期，下面分别介绍它们的概念和特点。

### （一）作用域

作用域是指变量和函数的可见范围，C语言中主要有局部作用域和文件作用域两种。

-   **局部作用域**：在函数内部或代码块内部定义的变量和函数，只在其定义的函数或代码块内部可见。例如：

```
#include <stdio.h>

void test() {
    int num = 10;  // 局部变量，只在test函数内部可见
    printf("num = %d\n", num);
}

int main() {
    test();
    // 这里无法访问test函数内部的num变量
    return 0;
}
```

-   **文件作用域**：在函数外部定义的变量和函数，在整个源文件内可见。如果要在其他源文件中使用这些变量和函数，需要使用`extern`关键字进行声明。例如：

```
// file1.c
#include <stdio.h>

int global_num = 20;  // 全局变量，具有文件作用域

void print_global_num() {
    printf("global_num = %d\n", global_num);
}

// file2.c
#include <stdio.h>

externint global_num;  // 声明外部全局变量
extern void print_global_num();  // 声明外部函数

int main() {
    printf("从另一个文件访问全局变量: %d\n", global_num);
    print_global_num();
    return0;
}
```

### （二）存储类别

存储类别决定了变量的生命周期和存储位置，C语言中主要有`auto`、`static`、`extern`和`register`四种存储类别。

-   **auto**：默认的存储类别，用于定义局部变量。局部变量在函数调用时创建，函数调用结束后销毁。例如：

```
#include <stdio.h>

void test() {
    auto int num = 30;  // 等价于 int num = 30;
    printf("num = %d\n", num);
}

int main() {
    test();
    return 0;
}
```

-   **static**：用于定义静态变量和静态函数。静态变量在程序运行期间只初始化一次，其生命周期贯穿整个程序。静态函数只能在定义它的源文件中使用。例如：

```
#include <stdio.h>

void test() {
    staticint count = 0;  // 静态局部变量
    count++;
    printf("count = %d\n", count);
}

int main() {
    test();
    test();
    test();
    return0;
}
```

-   **extern**：用于声明外部变量和外部函数，表示这些变量和函数在其他源文件中定义。例如前面提到的`file1.c`和`file2.c`的示例。
-   **register**：用于建议编译器将变量存储在寄存器中，以提高访问速度。但寄存器的数量有限，编译器可能会忽略这个建议。例如：

```
#include <stdio.h>

void test() {
    register int num = 40;  // 建议将num存储在寄存器中
    printf("num = %d\n", num);
}

int main() {
    test();
    return 0;
}
```

## 五、数组与函数

数组在函数中有着广泛的应用，下面介绍数组作为函数参数的传递方式和相关注意事项。

### （一）一维数组作为函数参数

一维数组作为函数参数时，实际上传递的是数组的首地址，即数组名代表的是数组首元素的地址。函数可以通过这个地址访问和修改数组的元素。

下面是一个一维数组作为函数参数的示例，用于计算数组元素的和：

```
#include <stdio.h>

int array_sum(int arr[], int size) {
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += arr[i];
    }
    return sum;
}

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    int size = sizeof(arr) / sizeof(arr[0]);
    int result = array_sum(arr, size);
    printf("数组元素的和为: %d\n", result);
    return0;
}
```

### （二）二维数组作为函数参数

二维数组作为函数参数时，需要指定数组的列数。因为二维数组在内存中是按行存储的，知道列数才能正确计算元素的地址。

下面是一个二维数组作为函数参数的示例，用于打印二维数组的元素：

```
#include <stdio.h>

void print_2d_array(int arr[][3], int rows) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d ", arr[i][j]);
        }
        printf("\n");
    }
}

int main() {
    int arr[][3] = {{1, 2, 3}, {4, 5, 6}};
    int rows = sizeof(arr) / sizeof(arr[0]);
    print_2d_array(arr, rows);
    return0;
}
```

## 六、函数指针

函数指针是指向函数的指针变量，它可以存储函数的入口地址，通过函数指针可以动态调用不同的函数。

函数指针的定义语法如下：

```
返回值类型 (*指针变量名)(参数列表);
```

下面是一个函数指针的示例，用于实现简单的计算器功能：

```
#include <stdio.h>

// 加法函数
int add(int a, int b) {
    return a + b;
}

// 减法函数
int sub(int a, int b) {
    return a - b;
}

// 计算器函数，接受函数指针作为参数
int calculator(int (*func)(int, int), int a, int b) {
    return func(a, b);
}

int main() {
    int a = 5, b = 3;
    int result;

    // 使用加法函数
    result = calculator(add, a, b);
    printf("加法结果: %d\n", result);

    // 使用减法函数
    result = calculator(sub, a, b);
    printf("减法结果: %d\n", result);

    return0;
}
```

## 七、递归函数

递归函数是指在函数内部调用自身的函数，递归函数通常用于解决可以分解为相同子问题的问题。递归函数需要满足两个条件：

-   **基线条件**：也称为终止条件，用于结束递归调用，避免无限递归。
-   **递归步骤**：将原问题分解为更小的子问题，并通过递归调用自身来解决这些子问题。

下面是一个递归函数的示例，用于计算阶乘：

```
#include <stdio.h>

// 递归函数计算阶乘
int factorial(int n) {
    if (n == 0 || n == 1) {
        return1;  // 基线条件
    } else {
        return n * factorial(n - 1);  // 递归步骤
    }
}

int main() {
    int num = 5;
    int result = factorial(num);
    printf("%d的阶乘是: %d\n", num, result);
    return0;
}
```

## 八、可变参数函数

可变参数函数是指可以接受可变数量和类型的参数的函数。C语言中通过`stdarg.h`头文件提供的宏来实现可变参数函数。

下面是一个可变参数函数的示例，用于计算多个整数的和：

```
#include <stdio.h>
#include <stdarg.h>

// 可变参数函数计算多个整数的和
int sum(int count, ...) {
    va_list args;
    va_start(args, count);

    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }

    va_end(args);
    return total;
}

int main() {
    int result = sum(3, 1, 2, 3);
    printf("多个整数的和为: %d\n", result);
    return0;
}
```

## 九、函数的高级应用

函数在实际编程中还有许多高级应用场景，下面介绍一些常见的应用。

### （一）回调函数

回调函数是指将一个函数作为参数传递给另一个函数，在另一个函数中调用这个传递进来的函数。回调函数常用于事件处理、排序算法等场景。

下面是一个回调函数的示例，用于实现简单的排序算法：

```
#include <stdio.h>

// 比较函数，用于比较两个整数的大小
int compare(int a, int b) {
    return a - b;
}

// 排序函数，接受回调函数作为参数
void sort(int arr[], int size, int (*cmp)(int, int)) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = i + 1; j < size; j++) {
            if (cmp(arr[i], arr[j]) > 0) {
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}

int main() {
    int arr[] = {3, 1, 4, 2, 5};
    int size = sizeof(arr) / sizeof(arr[0]);

    // 调用排序函数，传入回调函数
    sort(arr, size, compare);

    // 打印排序后的数组
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    return0;
}
```

### （二）库函数的使用

C语言提供了丰富的标准库函数，这些函数可以帮助我们完成各种常见的任务，如字符串处理、数学计算、文件操作等。在使用库函数时，需要包含相应的头文件。

下面是一些常见库函数的使用示例：

```
#include <stdio.h>
#include <string.h>
#include <math.h>

int main() {
    // 字符串处理函数
    char str1[] = "Hello";
    char str2[] = "World";
    char str3[20];
    strcpy(str3, str1);  // 复制字符串
    strcat(str3, " ");   // 连接字符串
    strcat(str3, str2);
    printf("拼接后的字符串: %s\n", str3);

    // 数学函数
    double num = 25.0;
    double sqrt_num = sqrt(num);  // 计算平方根
    printf("%f的平方根是: %f\n", num, sqrt_num);

    return0;
}
```

