# GDB 调试案例集

## 案例一：死锁调试

### 问题描述

多线程程序中，两个线程互相等待对方持有的锁，导致程序永久阻塞。

### 示例代码

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

pthread_mutex_t mutex_A = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_B = PTHREAD_MUTEX_INITIALIZER;

void *threadA_proc(void *data) {
    printf("thread A waiting get ResourceA \n");
    pthread_mutex_lock(&mutex_A);
    printf("thread A got ResourceA \n");
    sleep(1);
    printf("thread A waiting get ResourceB \n");
    pthread_mutex_lock(&mutex_B);  // 等待 mutex_B
    printf("thread A got ResourceB \n");
    pthread_mutex_unlock(&mutex_B);
    pthread_mutex_unlock(&mutex_A);
    return (void *) 0;
}

void *threadB_proc(void *data) {
    printf("thread B waiting get ResourceB \n");
    pthread_mutex_lock(&mutex_B);
    printf("thread B got ResourceB \n");
    sleep(1);
    printf("thread B waiting get ResourceA \n");
    pthread_mutex_lock(&mutex_A);  // 等待 mutex_A
    printf("thread B got ResourceA \n");
    pthread_mutex_unlock(&mutex_A);
    pthread_mutex_unlock(&mutex_B);
    return (void *) 0;
}

int main() {
    pthread_t tidA, tidB;
    pthread_create(&tidA, NULL, threadA_proc, NULL);
    pthread_create(&tidB, NULL, threadB_proc, NULL);
    pthread_join(tidA, NULL);
    pthread_join(tidB, NULL);
    printf("exit\n");
    return 0;
}
```

### 运行现象

```bash
$ gcc -g test.c -o deadlock_example -pthread
$ ./deadlock_example
thread A waiting get ResourceA
thread A got ResourceA
thread B waiting get ResourceB
thread B got ResourceB
thread A waiting get ResourceB
thread B waiting get ResourceA
# 程序阻塞，不再输出
```

### 调试步骤

#### 步骤 1：使用 ps 查看进程状态

```bash
$ ps aux | grep deadlock_example
root  35104  0.0  0.1  19020  3424 pts/0  Sl+  08:38  0:00 ./deadlock_example
```

#### 步骤 2：使用 top 查看线程状态

```bash
$ top -Hp 35104
Threads: 3 total, 0 running, 3 sleeping, 0 stopped, 0 zombie
  PID USER  PR  NI  VIRT  RES  SHR S %CPU %MEM  TIME+ COMMAND
35104 root  20   0 19020 3424 1280 S  0.0  0.2  0:00.00 deadlock_exampl
 2045 root  20   0 19020 3424 1280 S  0.0  0.2  0:00.00 deadlock_exampl
 2046 root  20   0 19020 3424 1280 S  0.0  0.2  0:00.00 deadlock_exampl
```

所有线程都处于 `S`（sleeping）状态，CPU 使用率为 0。

#### 步骤 3：使用 GDB attach 调试

```bash
$ gdb -p 35104
```

#### 步骤 4：查看所有线程堆栈

```bash
(gdb) thread apply all bt

Thread 3 (Thread 0x7f9e0d7fe640 (LWP 2046)):
#0  __lll_lock_wait () from /lib64/libc.so.6
#1  pthread_mutex_lock () from /lib64/libc.so.6
#2  threadB_proc (data=0x0) at test.c:34
#3  start_thread () from /lib64/libc.so.6

Thread 2 (Thread 0x7f9e0dfff640 (LWP 2045)):
#0  __lll_lock_wait () from /lib64/libc.so.6
#1  pthread_mutex_lock () from /lib64/libc.so.6
#2  threadA_proc (data=0x0) at test.c:17
#3  start_thread () from /lib64/libc.so.6

Thread 1 (Thread 0x7f9e0e2dd600 (LWP 35104)):
#0  __futex_abstimed_wait_common () from /lib64/libc.so.6
#1  __pthread_clockjoin_ex () from /lib64/libc.so.6
#2  main () at test.c:53
```

#### 步骤 5：查看锁的持有情况

```bash
(gdb) thread 2
[Switching to thread 2]

(gdb) frame 2
#2 threadA_proc (data=0x0) at test.c:17
17          pthread_mutex_lock(&mutex_B);

(gdb) p mutex_A
$1 = {__data = {__lock = 2, __count = 0, __owner = 35105, ...}}

(gdb) p mutex_B
$2 = {__data = {__lock = 2, __count = 0, __owner = 35106, ...}}
```

**分析**：
- `mutex_A` 被线程 35105（线程 A）持有
- `mutex_B` 被线程 35106（线程 B）持有
- 线程 A 在等待 `mutex_B`，线程 B 在等待 `mutex_A`
- **确认发生了死锁**

### 解决方案

1. **锁顺序一致性**：所有线程按相同顺序获取锁
2. **使用 trylock**：获取锁失败时释放已持有的锁
3. **使用超时锁**：`pthread_mutex_timedlock`

## 案例二：段错误调试

### 问题描述

程序访问非法内存地址（如空指针解引用），导致 `Segmentation fault`。

### 示例代码

```c
#include <stdio.h>

int addNumbers(const int *a, const int *b) {
    // 未检查指针是否为 NULL
    return *a + *b;  // 如果 a 或 b 为 NULL，将产生段错误
}

int main() {
    int number = 4, sum = 0;
    int *ptr1 = NULL;  // 空指针
    int *ptr2 = &number;

    sum = addNumbers(ptr1, ptr2);  // 传入空指针
    printf("两个数的和为：%d\n", sum);
    return 0;
}
```

### 调试步骤

#### 步骤 1：编译并运行

```bash
$ gcc -g add_numbers.c -o add_numbers
$ ./add_numbers
Segmentation fault (core dumped)
```

#### 步骤 2：使用 GDB 调试

```bash
$ gdb ./add_numbers
```

#### 步骤 3：运行程序

```bash
(gdb) run
Starting program: /home/user/add_numbers

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401136 in addNumbers (a=0x0, b=0x7fffffffdde4) at add_numbers.c:5
5           return *a + *b;
```

**关键信息**：
- 信号：`SIGSEGV`（段错误）
- 位置：`addNumbers` 函数第 5 行
- 参数：`a=0x0`（空指针）

#### 步骤 4：查看调用栈

```bash
(gdb) bt
#0  addNumbers (a=0x0, b=0x7fffffffdde4) at add_numbers.c:5
#1  0x0000000000401156 in main () at add_numbers.c:15
```

### 解决方案

添加空指针检查：

```c
int addNumbers(const int *a, const int *b) {
    if (a == NULL || b == NULL) {
        fprintf(stderr, "Error: NULL pointer\n");
        return 0;
    }
    return *a + *b;
}
```

## 案例三：Core Dump 分析

### 问题描述

程序崩溃后生成 core 文件，需要分析崩溃原因。

### 示例代码

```c
int recursion(int n) {
    // 无终止条件的递归，导致栈溢出
    recursion(n - 1);
}

int main() {
    int num = 10;
    recursion(num);
    return 0;
}
```

### 配置 Core Dump

#### 步骤 1：启用 Core Dump

```bash
# 临时设置
$ ulimit -c unlimited

# 永久设置（/etc/security/limits.conf）
* soft core unlimited

# 设置 core 文件名格式
$ echo "core-%e-%p-%t" | sudo tee /proc/sys/kernel/core_pattern
```

#### 步骤 2：运行程序生成 core 文件

```bash
$ gcc -g test.c -o test
$ ./test
Segmentation fault (core dumped)
$ ls
test  test.c  core-test-12345-1640765384
```

### 分析 Core Dump

#### 步骤 1：加载 core 文件

```bash
$ gdb ./test core-test-12345-1640765384
```

#### 步骤 2：查看崩溃信息

```bash
(gdb) bt
#0  recursion (n=-523456) at test.c:3
#1  0x0000000000401136 in recursion (n=-523455) at test.c:3
#2  0x0000000000401136 in recursion (n=-523454) at test.c:3
...
#523460 0x0000000000401136 in recursion (n=10) at test.c:3
#523461 0x0000000000401156 in main () at test.c:9
```

**分析**：
- 递归调用深度达到 523460 层
- 栈溢出导致段错误
- 原因：`recursion` 函数缺少终止条件

### 解决方案

添加递归终止条件：

```c
int recursion(int n) {
    if (n <= 0) return 0;  // 终止条件
    return recursion(n - 1);
}
```

## 调试技巧总结

### 死锁调试技巧

| 步骤 | 命令 | 目的 |
| :--- | :--- | :--- |
| 1 | `ps aux \| grep <program>` | 确认进程存在 |
| 2 | `top -Hp <pid>` | 查看线程状态 |
| 3 | `gdb -p <pid>` | attach 到进程 |
| 4 | `thread apply all bt` | 查看所有线程堆栈 |
| 5 | `p <mutex>` | 查看锁的持有情况 |

### 段错误调试技巧

| 步骤 | 命令 | 目的 |
| :--- | :--- | :--- |
| 1 | `gdb ./program` | 启动调试 |
| 2 | `run` | 运行程序 |
| 3 | `bt` | 查看崩溃位置 |
| 4 | `frame <n>` | 切换到相关帧 |
| 5 | `info locals` | 查看局部变量 |

### Core Dump 调试技巧

| 步骤 | 命令 | 目的 |
| :--- | :--- | :--- |
| 1 | `gdb ./program core` | 加载 core 文件 |
| 2 | `bt` | 查看崩溃调用栈 |
| 3 | `frame <n>` | 切换到相关帧 |
| 4 | `info locals` | 查看局部变量 |
| 5 | `info registers` | 查看寄存器状态 |

