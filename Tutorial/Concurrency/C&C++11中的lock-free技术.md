# C/C++11中的lock-free技术

>   原文：https://mp.weixin.qq.com/s/UsnyPxxCneRYfNWDddJxsQ

## 多线程编程中需要注意的细节

### 程序员角度的一条语句可能包含很多条机器指令

```c
counter += 1；
```

对counter进行+1操作，对应的汇编如下，

```c
mov     rax, QWORD PTR counter[rip]
sub     rax, 1
mov     QWORD PTR counter[rip], rax
```

-   读取变量counter的值；
-   将读来的值+1；
-   将+1后更新的值写入变量couter中。

### 编译器的指令重排

C/C++代码需要经过源码编写-->编译、链接-->执行三个步骤才能运行起来。编译器为了提高程序运行的效率，会对编译的代码进行指令重排(和源代码的顺序不一致)。以下面的代码为例：

```c
int A, B;

void foo()
{
    A = B + 1;
    B = 0;
}
```

未加编译器优化，汇编指令的顺序和代码书写的顺序(program order)相同。

```c
A:
        .zero   4
B:
        .zero   4
foo():
        push    rbp
        mov     rbp, rsp
        mov     eax, DWORD PTR B[rip]
        add     eax, 1
        mov     DWORD PTR A[rip], eax
        mov     DWORD PTR B[rip], 0
        nop
        pop     rbp
        ret
```

加入编译器优化后的汇编代码（-O2），会对Line 5和Line 6进行指令重排，先对B赋值，然后才对A赋值。

```c
foo():
        mov     eax, DWORD PTR B[rip]
        mov     DWORD PTR B[rip], 0
        add     eax, 1
        mov     DWORD PTR A[rip], eax
        ret
B:
        .zero   4
A:
        .zero   4
```

### CPU的指令乱序执行

```c
a = 1 ①
b = a + 3 ②
c = 7 ③
d = 9 ④
e = c + d ⑤
```

如上面的代码(这里先忽略编译器的指令重排，假设没有进行优化重排)，按照语法规则，先执行①，然后是②...，最后是⑤。CPU遵循这个顺序，将它们拆分成微操作(机器指令)，然后送入流水线(==指令流水线是在单个处理器内实现指令级并行的一种技术。流水线试图使处理器的每个部分忙于处理一些指令，方法是将传入的指令分成一系列连续的步骤（即“流水线”），由不同的处理器单元执行，并并行处理指令的不同部分==)，这样做是为了尽量填满流水线，提高CPU的效率，在单位时间内执行更多的指令。

流水线中指令的执行是并行的，一旦进入流水线，所有的指令就具有了同时执行的特征：前面的指令还没有执行完毕，后面的指令已经开始执行，这就造成了所谓的指令乱序执行。但是指令的执行可能会卡在某些环节上，比如，要访问的数据不在高速缓存中，需要执行一个慢速的内存访问以及高速缓存填充的操作。再比如，当前指令的执行依赖于其他指令的执行结果。不管是什么原因，都可能导致前面的指令后执行完毕，而后面的指令反而先执行完毕。

因此上面的语句的可能执行顺序如下，

```c
a = 1 ①
c = 7 ③
b = a + 3 ②
d = 9 ④
e = c + d ⑤
```

注意，乱序执行肯定要遵循的第一个原则是正确性，由于指令间的依赖关系，②不会在①之前执行完毕，⑤不会在③和④之前执行完毕(这里都是针对单个CPU核心而言，即上述5条指令都是在CPU0中执行。在我的认知中，目前单个线程是无法在多个CPU core中并行运行的)。

### 缓存一致性问题

在多核处理器的环境下，由于上述多级缓存、内存的存在以及多个线程运行在多个不同的CPU core上，导致的一个问题就是缓存的一致性。目前有很多的协议(Cache Coherence Protocols)来处理该问题，例如MSI Protocol，MOSI Protocol，MESI Protocol，MOESI Protocol。

## 多线程编程中的lock-free

基于以上的几个原因，在多线程编程中，对于一些共享资源的同步，最早接触到的是一些基于锁的技术，例如各种锁(Mutex...)，信号量等，这些技术已经基本可以解决99%的问题，可以称为基于锁(lock-based)的多线程编程技术。

既然有基于锁的，当然应该也有无锁的多线程编程技术(lock-free)。

>   无锁编程是一种技术，它允许共享数据结构的并发更新，而不需要在线程之间执行代价高昂的同步。此方法确保线程不会阻塞任意长的时间，并且在存在多个线程时保证某些线程的进程。
>
>   无锁算法是精心设计的数据结构和函数，允许多个线程尝试彼此独立地进行进程。这意味着在执行关键区域之前，不需要尝试获取锁。相反，您可以独立地更新部分数据结构的本地副本，然后使用CAS（Compare-And-Swap）自动地将其应用于共享结构。

上面提到，lock-free编程不会再像基于lock的方式让多个线程之间出现阻塞，主要为了“提高效率”。注意，lock-free的效率不一定高于lock相关的技术。

下面是lock-free技术的一些优点：

-   可以在必须避免锁的地方使用，例如中断处理程序
-   避免阻塞带来的麻烦，如死锁和优先级反转
-   提高多核处理器的性能

### lock-free相关的技术

一个大的应用程序不可能都是基于lock-free的技术，不现实，也没必要。在程序中，我们一般会使用一些无锁的“组件”或者“函数”，例如lock-free队列，例如这个concurrentqueue(https://github.com/cameron314/concurrentqueue)，支持lock-free的enqueue、try_enqueue、try_dequeue等操作。

关于lock-free编程的一些技术：atomic operations、Read Modify Write、Compare And Swap、Acquire-Release semantic、memory barriers(内存屏障)、ABA problem、Sequential Consistency等等，
