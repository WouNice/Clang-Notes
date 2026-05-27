# Pthread 简介

## 什么是 Pthread

Pthread（POSIX Threads）是 IEEE POSIX 标准定义的线程编程接口，为类 Unix 系统提供了标准化的多线程编程API。

> **POSIX** = Portable Operating System Interface，可移植操作系统接口

## 支持平台

| 平台 | 支持情况 | 说明 |
|------|---------|------|
| Linux | ✅ 原生支持 | 通过 `libpthread.so` 或 `libc.so`（glibc 2.34+） |
| macOS | ✅ 原生支持 | 符合 POSIX 标准 |
| FreeBSD/NetBSD/OpenBSD | ✅ 原生支持 | 类 Unix 系统 |
| Windows | ⚠️ 需移植 | 通过 [pthreads-win32](https://sourceforge.net/projects/pthreads4w/) 或类似库 |

## API 命名规范

Pthread API 遵循 C 语言命名风格，以 `pthread_` 为前缀：

```c
// 线程管理
pthread_create()   // 创建线程
pthread_exit()     // 线程退出
pthread_join()     // 等待线程结束
pthread_detach()   // 分离线程

// 互斥锁
pthread_mutex_init()    // 初始化互斥锁
pthread_mutex_lock()    // 加锁
pthread_mutex_unlock()  // 解锁

// 条件变量
pthread_cond_init()     // 初始化条件变量
pthread_cond_wait()     // 等待条件
pthread_cond_signal()   // 发送信号
```

## 核心优势

1. **标准化**：跨平台的统一接口
2. **成熟稳定**：经过数十年生产环境验证
3. **细粒度控制**：可直接管理线程属性、栈大小、调度策略等
4. **与 C/C++ 无缝集成**：底层API，性能开销小

## 在 Linux 中的实现

Linux 中 Pthread 的实现经历了两个阶段：

- **LinuxThreads**（早期）：使用 `clone()` 模拟线程，每个线程对应一个进程
- **NPTL**（Native POSIX Thread Library，现代）：Linux 2.6 引入，真正的1:1线程模型，性能更好

```bash
# 查看当前使用的线程实现
getconf GNU_LIBPTHREAD_VERSION
# 输出示例：NPTL 2.35
```
