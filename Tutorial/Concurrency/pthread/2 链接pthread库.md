# 链接 Pthread 库

## 检查系统是否安装 Pthread

在大多数 Linux 发行版中，Pthread 是预装的。可通过以下命令验证：

```bash
# 查看动态链接库
ldconfig -p | grep libpthread

# 或查看已安装的包（Debian/Ubuntu）
dpkg -l | grep libpthread

# 或查看已安装的包（RHEL/CentOS/Fedora）
rpm -qa | grep glibc
```

## 编译示例代码

以下是一个简单的 Pthread 示例：

```c
#include <pthread.h>
#include <stdio.h>

void *do_work(void *arg) {
    printf("Hello from thread!\n");
    return NULL;
}

int main() {
    pthread_t thread_id;
    printf("Creating thread...\n");
    pthread_create(&thread_id, NULL, do_work, NULL);
    pthread_join(thread_id, NULL);  // 等待线程结束
    printf("Thread has finished execution.\n");
    return 0;
}
```

## 编译方法

### 方法1：使用 `-pthread` 选项（推荐）

```bash
gcc example.c -o example -pthread
```

`-pthread` 选项的作用：
- 定义 `_REENTRANT` 宏（启用可重入代码）
- 链接 `libpthread` 库
- 影响编译器对线程局部存储的处理

### 方法2：显式链接 `-lpthread`

```bash
gcc example.c -o example -lpthread
```

> ⚠️ **注意**：`-lpthread` 仅链接库，不设置编译期宏。某些场景下可能不够，建议优先使用 `-pthread`。

### 方法3：CMake 配置

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

add_executable(MyProject main.c)

# 方法A：使用 CMake 内置模块（推荐）
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(MyProject Threads::Threads)

# 方法B：直接链接（简单项目）
# target_link_libraries(MyProject pthread)
```

### 方法4：Makefile 配置

```makefile
CC = gcc
CFLAGS = -Wall -pthread
LDFLAGS = -pthread

TARGET = myapp
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o)

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(OBJS) -o $@ $(LDFLAGS)

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: all clean
```

## 静态链接注意事项

```bash
# 静态链接（不推荐用于多线程程序）
gcc example.c -o example -static -pthread
```

> ⚠️ **警告**：静态链接 Pthread 可能导致运行时问题，因为某些功能（如线程局部存储）依赖动态链接器。仅在特殊场景（如制作独立可执行文件）下使用。

## 常见问题

| 错误 | 原因 | 解决 |
|------|------|------|
| `undefined reference to 'pthread_create'` | 未链接 Pthread 库 | 添加 `-pthread` 或 `-lpthread` |
| `pthread.h: No such file` | 缺少开发头文件 | 安装 `libc6-dev` 或 `glibc-devel` |
| `cannot find -lpthread` | 缺少静态库 | 安装 `libpthread-stubs0-dev` |

## glibc 2.34+ 的变化

从 glibc 2.34 开始，Pthread 函数被合并到主 `libc.so` 中：

```bash
# 旧版本（glibc < 2.34）
ldd ./example
# linux-vdso.so.1 => ...
# libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0
# libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6

# 新版本（glibc >= 2.34）
ldd ./example
# linux-vdso.so.1 => ...
# libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
# 注意：不再单独显示 libpthread
```

> 即使如此，编译时仍建议使用 `-pthread` 标志以确保兼容性。
