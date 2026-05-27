# Linux 程序开发调试

在 Linux 环境下开发程序，调试是每个程序员必须掌握的核心技能。本文将系统介绍 Linux 程序调试的完整技术栈。

## 调试方法选择指南

| 调试阶段   | 推荐工具组合    | 学习成本 | 效果指数 |
| :--------- | :-------------- | :------- | :------- |
| 编译错误   | gcc + 静态分析  | 低       | 高       |
| 运行时错误 | gdb + 日志      | 中       | 中高     |
| 内存问题   | valgrind + asan | 中高     | 高       |
| 性能优化   | perf + 火焰图   | 高       | 中高     |
| 生产故障   | eBPF + 动态追踪 | 很高     | 高       |

## 第一阶段：编译期调试

### 开启编译器警告

```bash
# 终极编译选项（建议收藏）
gcc -Wall -Wextra -Wpedantic -Wshadow -Wpointer-arith \
    -Wcast-align -Wwrite-strings -Wmissing-prototypes \
    -Wmissing-declarations -Wredundant-decls -Wnested-externs \
    -Winline -Wuninitialized -Wconversion -Wstrict-prototypes \
    -g -O0 -o myapp main.c

# C++ 专用选项
g++ -std=c++17 -Wall -Wextra -Wpedantic -Weffc++ \
    -Wold-style-cast -Woverloaded-virtual -Wsign-promo \
    -Wnon-virtual-dtor -g -O0 -o myapp main.cpp
```

### 静态分析工具

```bash
# 安装 clang 静态分析器
sudo apt install clang-tools

# 扫描代码缺陷（生成 HTML 报告）
scan-build -o static-analysis-report gcc -o myapp main.c

# 查看报告
firefox static-analysis-report/index.html
```

### 实战案例

```c
// 有问题的代码
typedef struct {
    char name[32];
    int age;
} Person;

void print_person(Person *p) {
    printf("Name: %s, Age: %d\n", p->name, p.age);  // 编译器警告：p.age 应该是 p->age
}

// 修正后的代码
void print_person(const Person *p) {
    if (p) {
        printf("Name: %s, Age: %d\n", p->name, p->age);
    }
}
```

## 第二阶段：内存调试

### Valgrind 全面检测

```bash
# 安装 valgrind
sudo apt install valgrind

# 内存泄漏检测（最全选项）
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         --verbose \
         --log-file=valgrind.log \
         ./myapp

# 性能分析（生成调用图）
valgrind --tool=callgrind ./myapp
kcachegrind callgrind.out.12345  # 可视化分析
```

### Address Sanitizer（ASan）

```bash
# 编译时启用 ASan（推荐）
gcc -fsanitize=address -g -O1 -fno-omit-frame-pointer -o myapp main.c

# 运行程序（自动检测内存错误）
./myapp

# 输出示例：
# ==12345== ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6020000000f4
```

### 内存调试实战案例

```c
// 内存泄漏示例
void memory_leak_demo() {
    char *buffer = malloc(1024);
    // 忘记 free，造成内存泄漏
}

// 悬垂指针示例
void dangling_pointer_demo() {
    char *buffer = malloc(100);
    free(buffer);
    buffer[0] = 'x';  // 使用已释放的内存
}

// 正确的内存管理
void good_memory_management() {
    char *buffer = malloc(100);
    if (buffer) {
        // 使用 buffer...
        free(buffer);
        buffer = NULL;  // 防止悬垂指针
    }
}
```

## 第三阶段：性能分析

### Perf 性能分析

```bash
# 安装 perf
sudo apt install linux-tools-generic

# 基础性能统计
perf stat ./myapp

# 记录调用栈（生成火焰图）
perf record -g ./myapp
perf report  # 交互式查看

# 实时监控
perf top -p $(pgrep myapp)
```

### 火焰图可视化

```bash
# 安装火焰图工具
git clone https://github.com/brendangregg/FlameGraph.git

# 生成火焰图
perf record -g -F 99 -a -- sleep 30
perf script | FlameGraph/stackcollapse-perf.pl | \
    FlameGraph/flamegraph.pl > perf.svg

# 在浏览器中查看火焰图
firefox perf.svg
```

### 性能优化实战

```c
// 性能问题代码
void slow_function() {
    for (int i = 0; i < 1000000; i++) {
        printf("%d\n", i);  // 频繁的系统调用
    }
}

// 优化后的代码
void fast_function() {
    char buffer[8192];
    int offset = 0;
    for (int i = 0; i < 1000000; i++) {
        offset += snprintf(buffer + offset, sizeof(buffer) - offset,
                          "%d\n", i);
        if (offset > 7000) {  // 批量输出
            printf("%s", buffer);
            offset = 0;
        }
    }
    if (offset > 0) {
        printf("%s", buffer);
    }
}
```

## 第四阶段：多线程调试

### Helgrind 检测竞态条件

```bash
# 安装 helgrind（valgrind 的一部分）
sudo apt install valgrind

# 检测线程错误
valgrind --tool=helgrind ./myapp

# 输出示例：
# ==12345== Possible data race during write of size 4 at 0x601040 by thread #1
```

### GDB 多线程调试

```bash
# 查看所有线程
(gdb) info threads

# 切换线程
(gdb) thread 2

# 设置线程断点
(gdb) break worker.c:42 thread 3

# 锁定调度器（只调试当前线程）
(gdb) set scheduler-locking on

# 同时对所有线程执行命令
(gdb) thread apply all bt
```

### 线程调试实战

```c
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int counter = 0;

void* thread_func(void* arg) {
    for (int i = 0; i < 1000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;  // 保护共享资源
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
```

## 第五阶段：生产环境调试

### eBPF 现代调试技术

```bash
# 安装 bcc 工具集
sudo apt install bpfcc-tools

# 实时系统调用跟踪
sudo execsnoop-bpfcc

# 跟踪特定进程的文件操作
sudo opensnoop-bpfcc -p $(pgrep myapp)

# 网络连接跟踪
sudo tcpconnect-bpfcc

# 函数调用跟踪
sudo funccount-bpfcc 'vfs_*'
```

### 动态追踪工具组合

```bash
# strace 跟踪系统调用
strace -f -e trace=network,signal -p $(pgrep myapp)

# ltrace 跟踪库函数调用
ltrace -f -p $(pgrep myapp)

# 使用 bpftrace（更高级的 eBPF 工具）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s opened %s\n", comm, str(args->filename)); }'
```

## 调试最佳实践总结

### 调试策略金字塔

```
生产环境监控（eBPF）
        ↑
性能优化（perf + 火焰图）
        ↑
内存调试（valgrind + asan）
        ↑
逻辑调试（gdb + 日志）
        ↑
编译期检查（静态分析）
```

### 调试 Checklist

- **编译期**：开启所有警告 + 静态分析
- **内存**：Valgrind + Address Sanitizer 双重检查
- **性能**：建立性能基线 + 定期性能测试
- **并发**：多线程压力测试 + Helgrind 检查
- **生产**：部署监控 + eBPF 动态追踪
