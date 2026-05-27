# GDB 高级调试技术

## 一、GDB 调试原理

### 调试模型

GDB 调试涉及两个程序：调试器（gdb）和被调试程序。根据运行位置分为：

| 类型 | 说明 | 通信方式 |
| :--- | :--- | :--- |
| 本地调试 | 调试器和被调试程序在同一主机 | 直接进程间通信 |
| 远程调试 | 调试器和被调试程序在不同主机 | RSP 协议（GDB Remote Serial Protocol） |

远程调试架构：
```
主机（运行 GDB） ←→ 目标设备（运行 gdbserver + 被调试程序）
```

### ptrace 系统调用

GDB 通过 `ptrace` 系统调用接管进程执行：

```c
#include <sys/ptrace.h>
long ptrace(enum __ptrace_request request, pid_t pid, void *addr, void *data);
```

**主要请求类型**：

| 请求 | 说明 |
| :--- | :--- |
| `PTRACE_TRACEME` | 标记当前进程为被跟踪目标 |
| `PTRACE_ATTACH` | 附加到指定进程 |
| `PTRACE_DETACH` | 分离调试器 |
| `PTRACE_PEEKDATA` | 读取目标进程内存 |
| `PTRACE_POKEDATA` | 写入目标进程内存 |
| `PTRACE_GETREGS` | 获取寄存器值 |
| `PTRACE_SETREGS` | 设置寄存器值 |
| `PTRACE_CONT` | 继续执行 |
| `PTRACE_SINGLESTEP` | 单步执行 |

### 启动调试流程

```
GDB 进程
    ↓ fork()
子进程
    ↓ ptrace(PTRACE_TRACEME)
    ↓ exec() 加载被调试程序
被调试程序（所有信号被 GDB 接管）
```

**attach 模式**：

```
GDB 进程 → ptrace(PTRACE_ATTACH, pid) → 目标进程暂停 → GDB 接管调试
```

### 断点实现原理

**软件断点**：
1. **设置**：保存原指令，替换为 `int 3`（0xCC）
2. **命中**：执行 `int 3` 触发 `SIGTRAP` 信号
3. **处理**：GDB 接收信号，恢复原始指令，回退 PC
4. **继续**：再次替换为 `int 3`，恢复执行

**硬件断点**：
- 使用 CPU 调试寄存器（x86: DR0-DR3）
- 无需修改代码，性能更好
- 条件断点优先使用硬件实现

## 二、TUI 模式进阶

### 启动与切换

```bash
# 启动时启用 TUI
gdb -tui ./program

# GDB 内切换
(gdb) tui enable                     # 启用 TUI
(gdb) tui disable                    # 禁用 TUI
(gdb) Ctrl+X A                       # 快捷键切换
```

### 布局控制

| 命令 | 说明 |
| :--- | :--- |
| `layout src` | 源代码窗口 |
| `layout asm` | 汇编窗口 |
| `layout split` | 源码 + 汇编 |
| `layout regs` | 寄存器 + 源码/汇编 |
| `winheight <win> [+/-]<n>` | 调整窗口高度 |
| `focus <win>` | 切换焦点窗口 |

**窗口名称**：`src`（源码）、`asm`（汇编）、`regs`（寄存器）、`cmd`（命令）

### TUI 快捷键

| 按键 | 功能 |
| :--- | :--- |
| `Ctrl+L` | 刷新屏幕 |
| `Ctrl+P` / `Ctrl+N` | 上一条/下一条命令 |
| `Ctrl+B` / `Ctrl+F` | 光标左移/右移 |
| `↑` / `↓` | 滚动源代码 |
| `←` / `→` | 左右移动源码 |

## 三、汇编级调试

### 查看汇编代码

```bash
# 反汇编
(gdb) disassemble                    # 当前函数
(gdb) disassemble /m main            # 反汇编 + 源代码
(gdb) disassemble /r main            # 反汇编 + 原始字节
(gdb) x/10i $pc                      # 查看 10 条指令
```

### 汇编级单步执行

```bash
(gdb) stepi                          # 单步执行（进入函数）
(gdb) nexti                          # 单步执行（不进入函数）
(gdb) si                             # stepi 缩写
(gdb) ni                             # nexti 缩写
```

### 汇编级断点

```bash
(gdb) break *main+24                 # 函数偏移处
(gdb) break *0x4005f6                # 指定地址
```

### 无源码调试

```bash
$ gcc program.s -o program           # 不带 -g 编译
$ gdb ./program

(gdb) layout asm                     # 显示汇编
(gdb) layout regs                    # 显示寄存器
(gdb) break main
(gdb) run
(gdb) stepi                          # 单步执行
(gdb) info registers                 # 查看寄存器
```

### 二进制文件分析工具

| 工具 | 用途 |
| :--- | :--- |
| `objdump -d <file>` | 反汇编 |
| `objdump -t <file>` | 符号表 |
| `nm <file>` | 查看符号 |
| `strings <file>` | 查看字符串 |

## 四、逆向调试

逆向调试允许程序反向执行，便于定位问题根因。

### 基本命令

```bash
(gdb) record                         # 开始记录
(gdb) record stop                    # 停止记录
(gdb) reverse-continue               # 反向继续
(gdb) reverse-step                   # 反向单步
(gdb) reverse-next                   # 反向单步（不进入函数）
(gdb) reverse-finish                 # 反向执行到函数开始
(gdb) set exec-direction reverse     # 设置反向执行
(gdb) set exec-direction forward     # 设置正向执行
```

### 记录管理

```bash
(gdb) info record                    # 查看记录状态
(gdb) record save <file>             # 保存记录
(gdb) record restore <file>          # 恢复记录
(gdb) record discard                 # 丢弃记录
```

### 注意事项

| 限制 | 说明 |
| :--- | :--- |
| 内存占用 | 记录会占用大量内存 |
| 稳定性 | 长时间记录可能导致崩溃 |
| 性能 | 回放速度比正常执行慢 |
| 适用场景 | 复杂问题定位、偶现 bug |

## 五、多线程调试

### 常用命令

| 命令 | 说明 |
| :--- | :--- |
| `info threads` | 查看所有线程 |
| `thread <n>` | 切换到线程 n |
| `thread apply all <cmd>` | 对所有线程执行命令 |
| `break <loc> thread <n>` | 对指定线程设置断点 |
| `set scheduler-locking on` | 只运行当前线程 |
| `set scheduler-locking off` | 所有线程同步运行 |

### 调试示例

```cpp
#include <iostream>
#include <thread>

void thread_func(int id) {
    std::cout << "Thread " << id << " running\n";
}

int main() {
    std::thread t1(thread_func, 1);
    std::thread t2(thread_func, 2);
    t1.join();
    t2.join();
    return 0;
}
```

```bash
$ g++ -g -pthread test.cpp -o test
$ gdb ./test

(gdb) break main
(gdb) run

[New Thread 0x7ffff6fd2700 (LWP 44996)]
[New Thread 0x7ffff67d1700 (LWP 44997)]

(gdb) info threads
  Id   Target Id         Frame
  3    Thread 0x7ffff67d1700 (LWP 44997) "test"  nanosleep ()
  2    Thread 0x7ffff6fd2700 (LWP 44996) "test"  nanosleep ()
* 1    Thread 0x7ffff7fe7740 (LWP 44987) "test"  main () at test.cpp:10

(gdb) thread 2                       # 切换到线程 2
(gdb) bt                             # 查看线程 2 的调用栈
(gdb) thread apply all bt            # 查看所有线程的调用栈
```

### 调试技巧

- **线程切换**：使用 `thread <n>` 切换到指定线程查看状态
- **线程锁**：`set scheduler-locking on` 避免其他线程干扰
- **批量操作**：`thread apply all bt` 查看所有线程调用栈
- **线程断点**：`break <loc> thread <n>` 针对特定线程设断点

## 六、多进程调试

### 跟踪模式设置

```bash
(gdb) set follow-fork-mode child     # 跟踪子进程
(gdb) set follow-fork-mode parent    # 跟踪父进程（默认）
(gdb) set detach-on-fork off         # 同时跟踪父子进程
(gdb) info inferiors                 # 查看所有进程
(gdb) inferior 2                     # 切换到进程 2
```

### 多进程调试命令

| 命令 | 说明 |
| :--- | :--- |
| `set follow-fork-mode child/parent` | 设置 fork 后跟踪的进程 |
| `set detach-on-fork on/off` | 是否分离未跟踪的进程 |
| `info inferiors` | 查看所有进程 |
| `inferior <编号>` | 切换到指定进程 |
| `detach inferior <编号>` | 分离指定进程 |
| `kill inferior <编号>` | 杀死指定进程 |

### 使用 attach

```bash
# 终端 1：运行程序
$ ./test_process &

# 终端 2：attach 到子进程
$ gdb -p <child_pid>
```

## 七、Core Dump 调试

### 启用 Core Dump

```bash
# 临时设置
$ ulimit -c unlimited

# 永久设置（/etc/security/limits.conf）
* soft core unlimited

# 设置 core 文件名格式
$ echo "core-%e-%p-%t" | sudo tee /proc/sys/kernel/core_pattern
```

### 生成 Core Dump

```bash
# 方法 1：程序崩溃自动生成
$ ./program
段错误 (core dumped)

# 方法 2：使用 gcore 生成运行中进程的 core
$ gcore <pid>
```

### 分析 Core Dump

```bash
$ gdb ./program core
# 或
$ gdb ./program -c core_file

(gdb) bt                             # 查看调用栈
(gdb) bt full                        # 查看完整调用栈
(gdb) info locals                    # 查看局部变量
(gdb) info args                      # 查看函数参数
(gdb) info registers                 # 查看寄存器
(gdb) list                           # 查看源代码
(gdb) frame 2                        # 切换到上层栈帧
```

### 调试技巧

| 目的 | 命令 |
| :--- | :--- |
| 查看崩溃位置 | `bt` |
| 查看局部变量 | `info locals` |
| 查看函数参数 | `info args` |
| 查看寄存器 | `info registers` |
| 生成 core 文件 | `gcore <pid>` |

## 八、Python 扩展

GDB 支持 Python 脚本扩展：

### 执行 Python 代码

```bash
(gdb) python print("Hello from Python!")
```

### 定义 Python 命令

```python
(gdb) python
import gdb

class HelloCommand(gdb.Command):
    def __init__(self):
        super().__init__("hello", gdb.COMMAND_USER)

    def invoke(self, arg, from_tty):
        print(f"Hello, {arg}!")

HelloCommand()
end

(gdb) hello World
Hello, World!
```

### 常用 Python API

```python
# 获取当前帧
frame = gdb.selected_frame()

# 获取变量值
value = gdb.parse_and_eval("variable_name")

# 设置断点
bp = gdb.Breakpoint("main")

# 执行 GDB 命令
gdb.execute("continue")
```

## 九、自定义命令

### 定义命令

```bash
# 定义简单命令
(gdb) define print_stack
>bt
>info locals
>end

# 添加文档
(gdb) document print_stack
>Print stack trace and local variables
>end

# 带参数的命令
(gdb) define dump_memory
>set $addr = $arg0
>set $count = $arg1
>x/$count xw $addr
>end

# 条件命令
(gdb) define check_null
>if $arg0 == 0
 >print "Pointer is NULL!"
 >bt
 >else
 >print "Pointer is valid"
 >end
>end
```

### GDB 脚本

```bash
# debug.gdb - 自动化调试脚本
set pagination off
set confirm off
set print pretty on

break main
run

set $count = 0
while $count < 10
    next
    print i
    set $count = $count + 1
end

continue
quit
```

**执行脚本**:
```bash
# 启动时加载
$ gdb ./program -x debug.gdb

# GDB 内加载
(gdb) source debug.gdb
```

## 十、调试优化代码

### 优化带来的问题

| 问题 | 现象 |
| :--- | :--- |
| 变量优化 | 显示 `<optimized out>` |
| 代码重排 | 行号与执行顺序不一致 |
| 函数内联 | 无法单步进入内联函数 |

### 解决方案

```bash
# 推荐：使用 -Og 优化调试体验
gcc -g -Og program.c -o program

# 或保留部分调试信息
gcc -g -O2 program.c -o program
```

```bash
# GDB 中查看优化信息
(gdb) info line                      # 显示代码地址
(gdb) disassemble /m                 # 反汇编 + 源码对照
```

## 附录：便利变量

### 值的历史

| 变量 | 说明 |
| :--- | :--- |
| `$` | 最后一个值 |
| `$n` | 第 n 个值 |
| `$$` | 倒数第二个值 |
| `$$n` | 倒数第 n 个值 |
| `$_` | x 命令最后显示的地址 |
| `$__` | x 命令最后显示的内容 |
| `$_exitcode` | 程序退出码 |
| `$bpnum` | 最后设置的断点编号 |

### 自定义便利变量

```bash
(gdb) set $i = 0                     # 定义变量
(gdb) set $ptr = (int *)0x601000     # 定义指针
(gdb) print $i
(gdb) print *$ptr
```

