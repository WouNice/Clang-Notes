# stdarg.h

`stdarg.h`定义于函数的可变参数相关的一些方法。

-   va_list 类型
-   va_start()
-   va_arg()：获取当前参数
-   va_end()
-   va_copy()：它以完全相同的状态复制va_list变量。
-   va_copy()：如果你需要浏览参数，但也需要记住你当前的位置，这是很有用的。

接受可变函数作为参数的一些方法。

- vprintf()
- vfprintf()
- vsprintf()
- vsnprintf()

```c
#include <stdio.h>
#include <stdarg.h>

int my_printf(int serial, const char *format, ...)
{
    va_list va;

    // Do my custom work
    printf("The serial number is: %d\n", serial);

    // Then pass the rest off to vprintf()
    va_start(va, format);
    int rv = vprintf(format, va);
    va_end(va);

    return rv;
}

int main(void)
{
    int x = 10;
    float y = 3.2;

    my_printf(3490, "x is %d, y is %f\n", x, y);
}
```

