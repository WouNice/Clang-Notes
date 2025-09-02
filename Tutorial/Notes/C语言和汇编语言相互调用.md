# C语言和汇编语言相互调用

在C语言中可以内嵌汇编代码，在汇编程序中同样也可以调用C程序。在调用的时候，要注意根据ATPCS规则来完成参数的传递，并配置好C程序传递参数和保存局部变量所依赖的堆栈环境，然后使用BL指令直接跳转即可。

一个C和汇编相互调用的示例代码：

main.c：

```c
#include <stdio.h>

int sum(int a, int b) {
    int result;
    result = a + b;
    printf("result = %d\n", result);
    return result;
}

void sum_asm(void);

int main(void) {
    sum_asm();
    return 0;
}
```

SUM.S：

```
.text
.global sum_asm
.arm
.type sum_asm, %function
sum_asm:
    mov r0, #3
    mov r1, #4
    bl sum
    bx lr
```

编译运行这两个文件，并运行：

```
# arm-linux-gnueabi-gcc -o a.out main.c SUM.S
# ./a.out
```
