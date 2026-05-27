# GDB 调试案例

本文档通过实际案例演示 GDB 在问题定位中的应用。

## 案例一：死锁调试

## 模拟死锁问题的产生

下面，我们用代码来模拟死锁问题的产生。

首先，我们先创建 2 个线程，分别为线程 A 和 线程 B，然后有两个互斥锁，分别是 mutex_A 和 mutex_B，代码如下：

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

pthread_mutex_t mutex_A = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_B = PTHREAD_MUTEX_INITIALIZER;

// 线程函数 A
void *threadA_proc(void *data) {
    printf("thread A waiting get ResourceA \n");
    pthread_mutex_lock(&mutex_A);
    printf("thread A got ResourceA \n");

    sleep(1);

    printf("thread A waiting get ResourceB \n");
    pthread_mutex_lock(&mutex_B);
    printf("thread A got ResourceB \n");

    pthread_mutex_unlock(&mutex_B);
    pthread_mutex_unlock(&mutex_A);
    return (void *) 0;
}

// 线程函数 B
void *threadB_proc(void *data) {
    printf("thread B waiting get ResourceB \n");
    pthread_mutex_lock(&mutex_B);
    printf("thread B got ResourceB \n");

    sleep(1);

    printf("thread B waiting get ResourceA \n");
    pthread_mutex_lock(&mutex_A);
    printf("thread B got ResourceA \n");

    pthread_mutex_unlock(&mutex_A);
    pthread_mutex_unlock(&mutex_B);
    return (void *) 0;
}


int main() {
    pthread_t tidA, tidB;

    pid_t pid = getpid();
    printf("func %s pid is %lld\n", __FUNCTION__, pid);

    //创建两个线程
    pthread_create(&tidA, NULL, threadA_proc, NULL);
    pthread_create(&tidB, NULL, threadB_proc, NULL);

    pthread_join(tidA, NULL);
    pthread_join(tidB, NULL);

    printf("exit\n");

    return 0;
}
```

线程 A 函数的过程：

- 先获取互斥锁 A，然后睡眠 1 秒；
- 再获取互斥锁 B，然后释放互斥锁 B；
- 最后释放互斥锁 A；

线程 B 函数的过程：

- 先获取互斥锁 B，然后睡眠 1 秒；
- 再获取互斥锁 A，然后释放互斥锁 A；
- 最后释放互斥锁 B；

我们运行这个程序，运行结果如下：

```apl
[root@localhost Ctest]# gcc -g test.c -o deadlock_example
[root@localhost Ctest]# ./deadlock_example
func main pid is 35104
thread A waiting get ResourceA
thread A got ResourceA
thread B waiting get ResourceB
thread B got ResourceB
thread A waiting get ResourceB
thread B waiting get ResourceA
// 阻塞
```

可以看到线程 B 在等待互斥锁 A 的释放，线程 A 在等待互斥锁 B 的释放，双方都在等待对方资源的释放，很明显，产生了死锁问题。

### 进程状态洞察

在怀疑程序出现死锁后，我们首先可以借助shell命令来初步观察进程的状态，获取一些关键信息，为后续深入排查死锁提供线索。

#### 使用ps aux查看进程概况

ps aux是一个非常实用的shell命令，它可以显示当前系统中所有用户的所有进程的详细信息。通过这个命令，我们可以获取进程的 CPU 使用率（% CPU）、内存使用情况（% MEM）等关键数据。在排查死锁时，这些信息能够帮助我们初步判断进程是否陷入了异常状态。

```apl
$ ps aux | grep deadlock_example
root        35104  0.0  0.1  19020  3424 pts/0    Sl+  08:38   0:00 ./deadlock_example
root        2124  0.0  0.1   3880  1920 pts/1    S+   08:39   0:00 grep --color=auto deadlock_example
```

在这个输出中，%CPU表示进程占用的 CPU 百分比，%MEM表示占用内存的百分比。如果一个进程陷入死锁，它通常无法正常执行任务，CPU 利用率会非常低，甚至接近于 0。同时，由于线程被阻塞，进程可能会保持对某些资源的占用，内存使用情况可能不会有明显变化，但也不会释放已占用的内存。所以，当我们看到一个进程的 CPU 利用率持续处于较低水平，且内存占用没有明显的波动时，就需要警惕死锁的可能性了。

#### top -Hp深入线程分析

top命令是一个动态实时查看进程信息的工具，而top -Hp则是top命令的一个强大扩展，它可以深入查看指定进程内每个线程的 CPU 和内存占用情况。这对于我们排查死锁非常有帮助，因为死锁往往发生在线程层面，通过查看线程的状态，我们可以更精确地识别是否存在死锁的迹象。

当我们执行top -Hp <pid>（<pid>为ps aux命令查找到的进程 ID）时，会进入一个实时更新的界面，显示该进程内各个线程的详细信息，包括线程 ID（PID）、用户（USER）、CPU 使用率（% CPU）、内存使用情况（% MEM）等。

```apl
[root@192 Ctest]# top -Hp 35104
top - 08:40:48 up 4 min,  0 users,  load average: 0.01, 0.03, 0.00
Threads:   3 total,   0 running,   3 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.3 hi,  0.2 si,  0.0 st
MiB Mem :   1739.0 total,    490.1 free,    889.4 used,    519.8 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.    849.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   35104 root      20   0   19020   3424   1280 S   0.0   0.2   0:00.00 deadlock_exampl
   2045 root      20   0   19020   3424   1280 S   0.0   0.2   0:00.00 deadlock_exampl
   2046 root      20   0   19020   3424   1280 S   0.0   0.2   0:00.00 deadlock_exampl
```

在正常情况下，我们希望看到各个线程都在积极地工作，CPU 使用率有一定的波动，表明线程在执行任务。然而，如果发生死锁，可能会出现一些异常情况。例如，部分线程的 CPU 使用率一直为 0，处于阻塞状态，而同时又有其他线程在尝试获取被阻塞线程持有的资源，导致这些线程也无法继续执行，从而出现活跃线程与阻塞线程的矛盾。如果我们观察到这种情况，就可以进一步确认死锁的可能性，为后续使用gdb进行更深入的调试指明方向。

### 调试定位死锁

通过shell命令初步判断程序可能出现死锁后，接下来就需要借助强大的调试工具gdb进行更深入的分析，精准定位死锁发生的位置。

#### gdb attach 附加进程

gdb的attach命令允许我们将调试器附加到一个正在运行的进程上，就像是给正在行驶的汽车安装一个实时监测系统，能够对进程内部的运行状态进行详细的观察和调试。在使用gdb attach之前，我们需要先获取目标进程的 ID（PID），这可以通过前面提到的ps aux命令来完成。

```apl
gdb -p 35104
```

执行上述命令后，gdb会暂停目标进程，此时我们就可以使用gdb的各种调试命令来对进程进行分析了。需要注意的是，在生产环境中使用attach命令时要格外小心，因为附加操作可能会导致进程暂停一段时间，影响其正常运行。

#### thread apply all bt 查看堆栈

一旦gdb成功附加到进程，我们就可以使用thread apply all bt命令来查看所有线程的堆栈信息。堆栈信息就像是程序运行的 “脚印”，记录了每个线程在执行过程中调用的函数以及函数的参数等重要信息。通过分析这些堆栈信息，我们能够了解每个线程的执行状态，进而找到死锁发生的代码行。

在gdb中执行thread apply all bt命令后，会得到类似下面的输出：

```apl
(gdb) thread apply all bt

Thread 3 (Thread 0x7f9e0d7fe640 (LWP 2046) "deadlock_exampl"):
#0  0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1  0x00007f9e0e08d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2  0x000000000040124b in threadB_proc (data=0x0) at test.c:34
#3  0x00007f9e0e08a19a in start_thread () from /lib64/libc.so.6
#4  0x00007f9e0e10f210 in clone3 () from /lib64/libc.so.6

Thread 2 (Thread 0x7f9e0dfff640 (LWP 2045) "deadlock_exampl"):
#0  0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1  0x00007f9e0e08d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2  0x00000000004011de in threadA_proc (data=0x0) at test.c:17
#3  0x00007f9e0e08a19a in start_thread () from /lib64/libc.so.6
#4  0x00007f9e0e10f210 in clone3 () from /lib64/libc.so.6

Thread 1 (Thread 0x7f9e0e2dd600 (LWP 35104) "deadlock_exampl"):
#0  0x00007f9e0e08722a in __futex_abstimed_wait_common () from /lib64/libc.so.6
#1  0x00007f9e0e08bca4 in __pthread_clockjoin_ex () from /lib64/libc.so.6
#2  0x00000000004012e0 in main () at test.c:53
```

在这个输出中，每一行都代表了一个函数调用，#0表示当前线程正在执行的函数，从#0往上依次是调用当前函数的其他函数。通过观察这些堆栈信息，我们可以看到线程1和线程2都卡在了`pthread_mutex_lock`函数处，这就是死锁发生的关键线索。结合代码行号（at test.c:34和at test.c:17），我们可以进一步定位到死锁发生的具体代码位置。

#### info threads 辅助分析

除了thread apply all bt命令，info threads命令也是我们在调试多线程程序时的得力助手。info threads命令可以列出所有线程的状态和索引，方便我们逐个分析每个线程的情况。

在gdb中执行info threads命令后，会得到如下输出：

```apl
(gdb) info threads
  Id   Target Id                                          Frame
* 1    Thread 0x7f9e0e2dd600 (LWP 35104) "deadlock_exampl" 0x00007f9e0e08722a in __futex_abstimed_wait_common () from /lib64/libc.so.6
  2    Thread 0x7f9e0dfff640 (LWP 2045) "deadlock_exampl" 0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
  3    Thread 0x7f9e0d7fe640 (LWP 2046) "deadlock_exampl" 0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
```

在这个输出中，Id列表示线程的索引，Target Id包含了线程的 LWP（轻量级进程 ID）和线程的名称，Frame则显示了线程当前所处的函数位置。通过info threads命令，我们可以快速了解每个线程的大致状态。

如果我们对某个线程特别关注，可以使用thread <线程ID>命令切换到该线程，然后再使用bt命令查看其具体的堆栈信息。例如，要查看线程2的堆栈信息，可以执行以下操作：

```apl
(gdb) thread 2
[Switching to thread 2 (Thread 0x7f9e0dfff640 (LWP 2045))]
#0  0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
(gdb) bt
#0  0x00007f9e0e0873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1  0x00007f9e0e08d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2  0x00000000004011de in threadA_proc (data=0x0) at test.c:17
#3  0x00007f9e0e08a19a in start_thread () from /lib64/libc.so.6
#4  0x00007f9e0e10f210 in clone3 () from /lib64/libc.so.6
```

通过这种方式，我们可以更细致地分析每个线程的执行情况，进一步确定引发死锁的代码部分。

### 利用工具排查死锁问题

由于当前的死锁代码例子是 C 写的，在 Linux 下，我们可以使用 `pstack` + `gdb` 工具来定位死锁问题。

pstack 命令可以显示每个线程的栈跟踪信息（函数调用过程），它的使用方式也很简单，只需要 `pstack <pid>` 就可以了。

那么，在定位死锁问题时，我们可以多次执行 pstack 命令查看线程的函数调用过程，多次对比结果，确认哪几个线程一直没有变化，且是因为在等待锁，那么大概率是由于死锁问题导致的。

```c
[root@localhost Ctest]# pstack 35104  // pid main
Thread 3 (Thread 0x7f06aaffe640 (LWP 35106) "test"):
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1 0x00007f06ab88d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2 0x000000000040124b in threadB_proc (data=0x0) at test.c:34
#3 0x00007f06ab88a19a in start_thread () from /lib64/libc.so.6
#4 0x00007f06ab90f210 in clone3 () from /lib64/libc.so.6
Thread 2 (Thread 0x7f06ab7ff640 (LWP 35105) "test"):
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1 0x00007f06ab88d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2 0x00000000004011de in threadA_proc (data=0x0) at test.c:17
#3 0x00007f06ab88a19a in start_thread () from /lib64/libc.so.6
#4 0x00007f06ab90f210 in clone3 () from /lib64/libc.so.6
Thread 1 (Thread 0x7f06aba49600 (LWP 35104) "test"):
#0 0x00007f06ab88722a in __futex_abstimed_wait_common () from /lib64/libc.so.6
#1 0x00007f06ab88bca4 in __pthread_clockjoin_ex () from /lib64/libc.so.6
#2 0x00000000004012e0 in main () at test.c:53

[root@localhost Ctest]# pstack 35105  // pid threadA_proc
Thread 1 (Thread 0x7f06ab7ff640 (LWP 35105) "test"):
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1 0x00007f06ab88d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2 0x00000000004011de in threadA_proc (data=0x0) at test.c:17
#3 0x00007f06ab88a19a in start_thread () from /lib64/libc.so.6
#4 0x00007f06ab90f210 in clone3 () from /lib64/libc.so.6

[root@localhost Ctest]# pstack 35106  // pid threadB_proc
Thread 1 (Thread 0x7f06aaffe640 (LWP 35106) "test"):
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1 0x00007f06ab88d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2 0x000000000040124b in threadB_proc (data=0x0) at test.c:34
#3 0x00007f06ab88a19a in start_thread () from /lib64/libc.so.6
#4 0x00007f06ab90f210 in clone3 () from /lib64/libc.so.6
```

可以看到，Thread 2 和 Thread 3 一直阻塞获取锁（pthread_mutex_lock）的过程，而且 pstack 多次输出信息都没有变化，那么可能大概率发生了死锁。

但是，还不能够确认这两个线程是在互相等待对方的锁的释放，因为我们看不到它们是等在哪个锁对象，于是我们可以使用 gdb 工具进一步确认。

整个 gdb 调试过程，如下：

```apl
// gdb 命令
$ gdb -p 35104

// 打印所有的线程信息
(gdb) info thread
  Id   Target Id                                Frame
* 1    Thread 0x7f06aba49600 (LWP 35104) "test" 0x00007f06ab88722a in __futex_abstimed_wait_common () from /lib64/libc.so.6
  2    Thread 0x7f06ab7ff640 (LWP 35105) "test" 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
  3    Thread 0x7f06aaffe640 (LWP 35106) "test" 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
//最左边的 * 表示 gdb 锁定的线程，切换到第二个线程去查看

// 切换到第1个线程
(gdb) thread 2
[Switching to thread 2 (Thread 0x7f06ab7ff640 (LWP 35105))]
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6

// bt 可以打印函数堆栈，却无法看到函数参数，跟 pstack 命令一样
(gdb) bt
#0 0x00007f06ab8873f0 in __lll_lock_wait () from /lib64/libc.so.6
#1 0x00007f06ab88d582 in pthread_mutex_lock@@GLIBC_2.2.5 () from /lib64/libc.so.6
#2 0x00000000004011de in threadA_proc (data=0x0) at test.c:17
#3 0x00007f06ab88a19a in start_thread () from /lib64/libc.so.6
#4 0x00007f06ab90f210 in clone3 () from /lib64/libc.so.6

// 打印第二帧信息，每次函数调用都会有压栈的过程，而 frame 则记录栈中的帧信息
(gdb) frame 2
#2 0x00000000004011de in threadA_proc (data=0x0) at test.c:17
17          pthread_mutex_lock(&mutex_B);

// 打印mutex_A的值 ,  __owner表示gdb中标示线程的值，即LWP
(gdb) p mutex_A
$3 = {__data = {__lock = 2, __count = 0, __owner = 35105, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}},
  __size = "\002\000\000\000\000\000\000\000!\211\000\000\001", '\000' <repeats 26 times>, __align = 2}

// 打印mutex_B的值 ,  __owner表示gdb中标示线程的值，即LWP
(gdb) p mutex_B
$5 = {__data = {__lock = 2, __count = 0, __owner = 35106, __nusers = 1, __kind = 0, __spins = 0, __elision = 0, __list = {__prev = 0x0, __next = 0x0}},
  __size = "\002\000\000\000\000\000\000\000\"\211\000\000\001", '\000' <repeats 26 times>, __align = 2}
```

我来解释下，上面的调试过程：

1. 通过 `info thread` 打印了所有的线程信息，可以看到有 3 个线程，一个是主线程（LWP 35104），另外两个都是我们自己创建的线程（LWP 35105 和 35106）；
2. 通过 `thread 2`，将切换到第 2 个线程（LWP 35105）；
3. 通过 `bt`，打印线程的调用栈信息，可以看到有 threadB_proc 函数，说明这个是线程 B 函数，也就说 LWP 35105 是线程 B;
4. 通过 `frame 2`，打印调用栈中的第三个帧的信息，可以看到线程 B 函数，在获取互斥锁 A 的时候阻塞了；
5. 通过 `p mutex_A`，打印互斥锁 A 对象信息，可以看到它被 LWP 为 35105（线程 A）的线程持有着；
6. 通过 `p mutex_B`，打印互斥锁 B 对象信息，可以看到他被 LWP 为 35106（线程 B）的线程持有着；

因为线程 B 在等待线程 A 所持有的 mutex_A, 而同时线程 A 又在等待线程 B 所拥有的 mutex_B, 所以可以断定该程序发生了死锁。

## 对空指针的解引用

示例代码引入一个内存访问错误：对空指针的解引用。这种错误类型经常导致程序崩溃并生成核心转储（core dump），是使用GDB进行调试的一个常见场景。

```c
#include <stdio.h>

int addNumbers(const int *a, const int *b) {    // 在此错误地解引用传入指针
    // 未检查指针是否为NULL，如果是NULL，解引用将导致段错误
    return *a + *b;
}

int main() {
    int number = 4, sum = 0;
    int *ptr1 = NULL; // 故意设置为NULL，模拟错误的指针使用
    int *ptr2 = &number;
    printf("请输入两个整数：");
    // 传入一个空指针和一个有效指针，这会引起运行时错误
    sum = addNumbers(ptr1, ptr2);
    printf("两个数的和为：%d\n", sum);
    return 0;
}
```

在这个代码中，addNumbers函数预期接收两个整数指针，但main函数中故意将ptr1设置为NULL，而这将导致在尝试解引用NULL指针时发生运行时错误（通常是段错误）。

**编译配置以支持调试信息**

开始之前，确保您的代码在编译时包含了必要的调试信息。使用GCC或G++时，添加选项是关键：

```oz
gcc -g add_numbers.c -o add_numbers
```

**启动GDB**

打开终端，你可以用GDB调试这个程序，观察它在何处以及如何失败：

```oz
[root@192 build]# gdb test_example
GNU gdb (Rocky Linux) 14.2-4.el9
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from test_example...
(gdb)
```

**设置断点**

在程序的某一行设置断点，当程序执行到这一行时会暂停，允许你查看和修改程序状态。设置断点的命令为break或简写b，后面跟上函数名或行号，例如：

```oz
break main
```

输出：

```oz
(gdb) break main
Breakpoint 1 at 0x40114a: file /home/spider/Ctest/example.c, line 9.
```

**运行程序**

使用run或r命令开始运行程序。如果设置了断点，程序会在到达第一个断点时暂停。

```oz
(gdb) r
Starting program: /home/spider/Ctest/build/test_example

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.rockylinux.org/>
Enable debuginfod for this session? (y or [n])
Debuginfod has been disabled.
To make this setting permanent, add 'set debuginfod enabled off' to .gdbinit.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1.1, main () at /home/spider/Ctest/example.c:9
9           int number = 4, sum = 0;
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.34-168.el9_6.19.x86_64 gtest-1.11.0-1.el9.x86_64 libgcc-11.5.0-5.el9_5.x86_64 libstdc++-11.5.0-5.el9_5.x86_64
```

**单步执行**

-   next 或 n ：执行下一行代码，如果当前行是一个函数调用，则不会进入该函数内部。
-   step 或 s：执行下一行代码或进入函数内部。

```oz
(gdb) n
10          int *ptr1 = NULL; // 故意设置为NULL，模拟错误的指针使用
```

**继续执行**

当程序暂停后，可以使用 continue 或 c 命令继续执行，直到遇到下一个断点。

**查看变量和表达式**

在程序暂停时，可以使用 print 或 p 命令查看变量值或计算表达式，如：

```oz
(gdb) p b
$1 = (const int *) 0x7fffffffdde4
```

**发现 segmentation fault 并修复**

```oz
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401136 in addNumbers (a=0x0, b=0x7fffffffdde4) at /home/spider/Ctest/example.c:5
5           return *a + *b;
```

可以从`a=0x0`得出a为空指针

## GDB调试core文件

段错误(**Segmentation fault**)是折磨程序员的一个难题，然而我们可以利用 GDB 快速定位到出错的地方，这确实是一个利好消息。

### 产生 Segmentation fault 程序

下面是一个**没有递归终止条件**的程序，显然会发生 Segmentation fault 错误。

```c
int recursion(int n) {   //无终止条件的递归
    recursion(n - 1);
}

int main(int argc, char const *argv[]) {
    int num = 10;
    recursion(num);
    return 0;
}
```

此时编译运行的时候是没有 core 文件的，效果如图 5 所示：

```apl
[root@localhost Ctest]## gcc -g test.c -o test
[root@localhost Ctest]## ./test
Segmentation fault (core dumped)
[root@localhost Ctest]## ls
test  test.c
```

这是因为我们没有配置生成 core 文件。

### 生成 core 文件

我们先使用如下命令查看 core 文件大小，若结果为 0 表示还没有生成 core 文件。

```apl
[root@localhost Ctest]## ulimit -c  // 查看 core 文件大小
unlimited
[root@localhost Ctest]## ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) unlimited
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 6782
max locked memory           (kbytes, -l) 8192
max memory size             (kbytes, -m) unlimited
open files                          (-n) 524288
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 6782
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

使用如下命令修改 core 文件大小不受限制：

```apl
[root@localhost Ctest]## ulimit -c unlimited // 修改 core 文件大小不受限制
[root@localhost Ctest]## ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) unlimited
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 6782
max locked memory           (kbytes, -l) 8192
max memory size             (kbytes, -m) unlimited
open files                          (-n) 524288
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 6782
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

>   【RockyLinux设置core dump在当前目录生成】使用以下命令临时更改核心转储文件的存储位置：
>
>   ```apl
>   echo 'core' | sudo tee /proc/sys/kernel/core_pattern
>   ```
>
>   这会将核心转储文件的名称设置为简单的“core”，并在当前工作目录下生成。
>
>   来源：https://juejin.cn/post/7317104726767599670

再次编译运行就能够看到 core 文件了，效果下所示：

```apl
[root@localhost Ctest]## ./test
Segmentation fault (core dumped)
[root@localhost Ctest]## ll
-rw-------. 1 root root 8556544 Jul 12 22:15 core.32638
-rwxr-xr-x. 1 root root   18488 Jul 12 22:13 test
-rw-r--r--. 1 root root     171 Jul 12 20:44 test.c
```

### 调试 core 文件

输入如下命令调试 core 文件，直接定位到问题语句：

```apl
[root@localhost Ctest]## gdb ./test core.32638 // test 为可执行文件
Excess command line arguments ignored. (// ...)
GNU gdb (Rocky Linux) 14.2-4.el9
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./test...
[New LWP 32638]

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.rockylinux.org/>
Enable debuginfod for this session? (y or [n]) y
Debuginfod has been enabled.
To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
--Type <RET> for more, q to quit, c to continue without paging--
Downloading separate debug info for /lib64/libc.so.6
Download failed: No route to host.  Continuing without separate debug info for /lib64/libc.so.6.
Download failed: No route to host.  Continuing without separate debug info for system-supplied DSO at 0x7ffce8367000.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
Core was generated by `./test'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0 0x000000000040110e in recursion (n=<error reading variable: Cannot access memory at address 0x7ffce7a88ffc>) at test.c:1
1       int recursion(int n) {   //无终止条件的递归
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.34-168.el9_6.19.x86_64
(gdb)
```

接下来可以打印当前堆栈信息分析问题：

```apl
backtrace  // 打印当前堆栈信息，backtrace 可以用 bt 的缩写替代
```

效果如下所示：

```apl
(gdb) backtrace
#0 0x000000000040110e in recursion (n=<error reading variable: Cannot access memory at address 0x7ffce7a88ffc>) at test.c:1
#1 0x000000000040111e in recursion (n=-261839) at test.c:2
#2 0x000000000040111e in recursion (n=-261838) at test.c:2
#3 0x000000000040111e in recursion (n=-261837) at test.c:2
#4 0x000000000040111e in recursion (n=-261836) at test.c:2
#5 0x000000000040111e in recursion (n=-261835) at test.c:2
#6 0x000000000040111e in recursion (n=-261834) at test.c:2
#7 0x000000000040111e in recursion (n=-261833) at test.c:2
#8 0x000000000040111e in recursion (n=-261832) at test.c:2
#9 0x000000000040111e in recursion (n=-261831) at test.c:2
#10 0x000000000040111e in recursion (n=-261830) at test.c:2
#11 0x000000000040111e in recursion (n=-261829) at test.c:2
#12 0x000000000040111e in recursion (n=-261828) at test.c:2
#13 0x000000000040111e in recursion (n=-261827) at test.c:2
#14 0x000000000040111e in recursion (n=-261826) at test.c:2
#15 0x000000000040111e in recursion (n=-261825) at test.c:2
#16 0x000000000040111e in recursion (n=-261824) at test.c:2
#17 0x000000000040111e in recursion (n=-261823) at test.c:2
#18 0x000000000040111e in recursion (n=-261822) at test.c:2
#19 0x000000000040111e in recursion (n=-261821) at test.c:2
#20 0x000000000040111e in recursion (n=-261820) at test.c:2
#21 0x000000000040111e in recursion (n=-261819) at test.c:2
```

上图中每一个编号就是一个栈帧，我们还能够切换到对应的栈帧，并且查看栈帧的信息。

```
frame n  // 切换到 n 号栈帧，frame 可以用 f 替代
info frame  // 查看当前栈帧信息
info registers  // 查看寄存器的值
```

效果如下所示：

```c
(gdb) frame 1
#1 0x000000000040111e in recursion (n=-261839) at test.c:2
2           recursion(n - 1);
(gdb) info f
Stack level 1, frame at 0x7ffce7a89030:
 rip = 0x40111e in recursion (test.c:2); saved rip = 0x40111e
 called by frame at 0x7ffce7a89050, caller of frame at 0x7ffce7a89010
 source language c.
 Arglist at 0x7ffce7a89020, args: n=-261839
 Locals at 0x7ffce7a89020, Previous frame's sp is 0x7ffce7a89030
 Saved registers:
  rbp at 0x7ffce7a89020, rip at 0x7ffce7a89028
(gdb) info registers
rax            0xfffc0130          4294705456
rbx            0x0                 0
rcx            0x403e48            4210248
rdx            0x7ffce8286c98      140724203449496
rsi            0x7ffce8286c88      140724203449480
rdi            0xfffc0130          4294705456
rbp            0x7ffce7a89020      0x7ffce7a89020
rsp            0x7ffce7a89010      0x7ffce7a89010
r8             0x7f085bffaf30      139673879949104
r9             0x7f085c1830c0      139673881555136
r10            0x7ffce82868e0      140724203448544
r11            0x206               518
r12            0x7ffce8286c88      140724203449480
r13            0x401121            4198689
r14            0x403e48            4210248
r15            0x7f085c1b4000      139673881755648
rip            0x40111e            0x40111e <recursion+24>
eflags         0x10206             [ PF IF RF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
--Type <RET> for more, q to quit, c to continue without paging--
fs             0x0                 0
gs             0x0                 0
fs_base        0x7f085c177600      139673881507328
gs_base        0x0                 0
(gdb)
```

### GDB反汇编

我们还可以将可执行文件进行反汇编，**即将机器语言转换为我们可阅读的汇编语言**。反汇编应用场景：从汇编层面分析程序问题时，包括但不限于**软件破解**、**性能分析**、**编译器 bug 追踪**等。

**objdump 命令主要是用来查看文件中的各个段(代码段、数据段等)的详细信息。**

```
objdump -d test  // demo 为可执行文件
```

图 12 为部分反汇编输出信息：

```a
[root@localhost Ctest]## objdump -d test
......
0000000000401121 <main>:
  401121:       55                      push   %rbp
  401122:       48 89 e5                mov    %rsp,%rbp
  401125:       48 83 ec 20             sub    $0x20,%rsp
  401129:       89 7d ec                mov    %edi,-0x14(%rbp)
  40112c:       48 89 75 e0             mov    %rsi,-0x20(%rbp)
  401130:       c7 45 fc 0a 00 00 00    movl   $0xa,-0x4(%rbp)
  401137:       8b 45 fc                mov    -0x4(%rbp),%eax
  40113a:       89 c7                   mov    %eax,%edi
  40113c:       e8 c5 ff ff ff          callq  401106 <recursion>
  401141:       b8 00 00 00 00          mov    $0x0,%eax
  401146:       c9                      leaveq
  401147:       c3                      retq
......
```

除了利用 objdump 查看整个文件的反汇编程序外，我们还可以在 **GDB 调试过程查看函数的反汇编程序**。

```apl
(gdb) disassemble main  // main 可替换为指定函数
Dump of assembler code for function main:
   0x0000000000401121 <+0>:     push   %rbp
   0x0000000000401122 <+1>:     mov    %rsp,%rbp
   0x0000000000401125 <+4>:     sub    $0x20,%rsp
   0x0000000000401129 <+8>:     mov    %edi,-0x14(%rbp)
   0x000000000040112c <+11>:    mov    %rsi,-0x20(%rbp)
   0x0000000000401130 <+15>:    movl   $0xa,-0x4(%rbp)
   0x0000000000401137 <+22>:    mov    -0x4(%rbp),%eax
   0x000000000040113a <+25>:    mov    %eax,%edi
   0x000000000040113c <+27>:    call   0x401106 <recursion>
   0x0000000000401141 <+32>:    mov    $0x0,%eax
   0x0000000000401146 <+37>:    leave
   0x0000000000401147 <+38>:    ret
End of assembler dump.
```

