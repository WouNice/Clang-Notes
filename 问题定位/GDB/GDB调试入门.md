# GDB调试入门

本篇讲解使用GDB调试Linux应用程序，以下以`hellowld.c`为例介绍 GDB 的调试入门：

## 编写代码

```c
#include <stdio.h>

int main(int argc, char **argv)
{
    int i;
    int result = 0;

    if(1 >= argc)
    {
        printf("Helloworld.\n");
    }
    printf("Hello World %s!\n",argv[1]);

    for(i = 1; i <= 100; i++)  {
        result += i;
    }

    printf("result = %d\n", result );

    return 0;
}
```

编译时加上 `-g` 参数：

```apl
gcc helloworld.c -o hellowrld -g
```

## 启动调试

```oz
$ gdb helloWorld
GNU gdb (GDB) Red Hat Enterprise Linux 8.2-12.el8
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from helloworld...done.
(gdb) run                  <----------------------------- 不带参数运行
Starting program: /home/zhuzhg/helloworld
Missing separate debuginfos, use: yum debuginfo-install glibc-2.28-101.el8.x86_64
helloworld.
result = 5050
[Inferior 1 (process 1069013) exited normally]
(gdb) run China            <----------------------------- 带参数运行
Starting program: /home/zhuzhg/helloworld China
Hello World China!
result = 5050
[Inferior 1 (process 1071086) exited normally]
(gdb)
```

## 断点

### 设置断点

-   文件行号断点：`break hellowrld.c:9`
-   函数断点：`break main`
-   条件断点：`break helloworld.c:17 if c == 10`
-   临时断点, 假设某处的断点只想生效一次，那么可以设置临时断点，这样断点后面就不复存在了：`tbreak helleworld.c:9`
-   禁用或启动断点：

```oz
disable                 # 禁用所有断点
disable bnum            # 禁用标号为bnum的断点
enable                  # 启用所有断点
enable bnum             # 启用标号为bnum的断点
enable delete bnum      # 启动标号为bnum的断点，并且在此之后删除该断点
```

-   断点清除：

```oz
clear                   # 删除当前行所有breakpoints
clear function          # 删除函数名为function处的断点
clear filename:function # 删除文件filename中函数function处的断点
clear lineNum           # 删除行号为lineNum处的断点
clear f:lename:lineNum  # 删除文件filename中行号为lineNum处的断点
delete                  # 删除所有breakpoints,watchpoints和catchpoints
delete bnum             # 删除断点号为bnum的断点
```

## 变量查看

**变量查看:** 最常见的使用便是使用print（可简写为p）打印变量内容。

以上述程序为例：

```oz
gdb helloworld
break helloworld.c:17 if i == 0
(gdb) run
Starting program: /home/book/helloworld
helloworld.

Breakpoint 2, main (argc=1, argv=0x7fffffffdca8) at helloworld.c:17
17            result += i;
(gdb) print i                <------------------ 查看变量 i 当前的值
$1 = 10
(gdb) print result           <------------------ 查看变量 result 当前的值
$2 = 45
(gdb) print argc             <------------------ 查看变量 argc 当前的值
$3 = 1
(gdb) print str
$4 = 0x4006c8 "Hello World" <------------------ 查看变量 str 当前的值
```

**查看内存：** examine(简写为x)可以用来查看内存地址中的值。语法如下：

```oz
x/[n][f][u] addr
```

其中：

单元类型常见有如下：

示例：

```oz
(gdb) x/4b str
0x4006c8:    01001000    01100101    01101100    01101100
```

可以看到，变量 str 的四个字节都以二进制的方式打印出来了。

-   b 字节
-   h 半字，即双字节
-   w 字，即四字节
-   g 八字节
-   n 表示要显示的内存单元数，默认值为1
-   f 表示要打印的格式，前面已经提到了格式控制字符
-   u 要打印的单元长度
-   addr 内存地址

**查看寄存器内容：**info registers

```oz
ra             0x3ff7ef2282     0x3ff7ef2282 <__libc_start_main+160>
sp             0x3ffffffaa0     0x3ffffffaa0
gp             0x2aaaaac800     0x2aaaaac800
tp             0x3ff7fdd250     0x3ff7fdd250
t0             0x3ff7ed60b0     274742468784
t1             0x3ff7ef21e2     274742583778
t2             0x2aaaaac4f0     183251944688
fp             0x3ffffffab0     0x3ffffffab0
s1             0x0      0
a0             0x1      1
a1             0x3ffffffc28     274877905960
a2             0x3ffffffc38     274877905976
a3             0x0      0
a4             0x3ffffffad8     274877905624
a5             0x0      0
a6             0x3ff7fd88a8     274743527592
(内容过多未显示完全)
```

## 单步调试

**单步执行-next：**

next命令（可简写为n）用于在程序断住后，继续执行下一条语句，假设已经启动调试，并在第12行停住，如果要继续执行，则使用n执行下一条语句，如果后面跟上数字num，则表示执行该命令num次，就达到继续执行n行的效果了：

```oz
gdb helloworld                     <------------------------------- 加载程序
(gdb) break helloworld.c:18        <------------------------------- 设置断点
(gdb) run                          <------------------------------- 启动调试
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/book/helloworld
Helleo World.

Breakpoint 2, main (argc=1, argv=0x7fffffffdca8) at helloworld.c:18         <-------- 程序在 18 行暂停
18            result += i;
Breakpoint 2, main (argc=1, argv=0x7fffffffdca8) at helloworld.c:18
18            result += i;
(gdb) next                                    <--------  单步执行

17        for(i = 1; i <= 100; i++)  {

Breakpoint 2, main (argc=1, argv=0x7fffffffdca8) at helloworld.c:18
18            result += i;
(gdb) next 2                                  <--------  执行两次

Breakpoint 2, main (argc=1, argv=0x7fffffffdca8) at helloworld.c:18
18            result += i;
```

**单步进入-step：**

如果我们想跟踪函数内部的情况，可以使用step命令（可简写为s），它可以单步跟踪到函数内部，但前提是该函数有调试信息并且有源码信息。

**断点继续-continue：**

continue命令（可简写为c），它会继续执行程序，直到再次遇到断点处。
