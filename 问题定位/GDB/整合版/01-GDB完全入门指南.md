# GDB 完全入门指南

## 一、GDB 简介

### 什么是 GDB

GDB（GNU Debugger）是 GNU 项目开发的源代码级调试器，是 Linux 环境下最强大的 C/C++ 调试工具。

**主要功能**:
- 启动程序，指定任何可能影响程序行为的内容
- 在指定条件下停止程序（断点、观察点、捕获点）
- 检查程序停止时的状态
- 修改程序状态，试验 bug 修复

### 调试模型

| 类型 | 说明 | 适用场景 |
| :--- | :--- | :--- |
| 本地调试 | 调试器和被调试程序在同一主机 | 开发环境 |
| 远程调试 | 调试器和被调试程序在不同主机 | 嵌入式设备 |
| Core Dump | 分析程序崩溃后的内存状态 | 线上问题 |

## 二、编译与启动

### 编译带调试信息的程序

使用 `-g` 选项编译程序以嵌入调试信息：

```bash
# C 程序
gcc -g program.c -o program

# C++ 程序
g++ -g program.cpp -o program

# 推荐：关闭优化以获得最佳调试体验
gcc -g -O0 program.c -o program
```

> **注意**: `-O2` 等优化选项可能导致代码行号与执行顺序不一致，调试时建议使用 `-O0`。

### 启动 GDB 的方式

#### 方式 1：直接启动

```bash
# 基本启动
gdb ./program

# TUI 模式（带代码窗口）
gdb -tui ./program

# 传入命令行参数
gdb --args ./program arg1 arg2
```

#### 方式 2：附加到运行中的进程

```bash
# 绑定到运行中的进程
gdb attach <pid>
gdb -p <pid>
```

> **权限要求**: attach 需要足够权限，否则会出现 `ptrace: Operation not permitted` 错误。

#### 方式 3：调试 Core Dump

```bash
# 调试 core dump 文件
gdb ./program core
gdb ./program -c core_file
```

#### 方式 4：远程调试

**目标设备**（被调试端）：
```bash
gdbserver :1234 ./program          # 在端口 1234 启动调试服务
gdbserver :1234 --attach <pid>     # attach 到运行中的进程
```

**主机**（调试端）：
```bash
gdb ./program
(gdb) target remote <target_ip>:1234
```

## 三、基本调试流程

### 典型调试会话

```bash
$ gdb ./helloworld
Reading symbols from helloworld...done.

(gdb) break main                 # 在 main 函数设置断点
(gdb) run                        # 运行程序

Breakpoint 1, main () at helloworld.c:5
5       int result = 0;

(gdb) next                       # 单步执行（不进入函数）
(gdb) step                       # 单步执行（进入函数）
(gdb) print result               # 打印变量值
(gdb) continue                   # 继续执行到下一断点
(gdb) quit                       # 退出 GDB
```

### 调试示例

**示例代码**:
```c
#include <stdio.h>

int main(int argc, char **argv) {
    int i;
    int result = 0;

    if (1 >= argc) {
        printf("Hello World.\n");
    }
    printf("Hello World %s!\n", argv[1]);

    for (i = 1; i <= 100; i++) {
        result += i;
    }

    printf("result = %d\n", result);
    return 0;
}
```

**调试过程**:
```bash
# 编译
$ gcc -g helloworld.c -o helloworld

# 启动调试
$ gdb helloworld

# 设置断点并运行
(gdb) break 9                      # 在第 9 行设置断点
(gdb) run China                    # 带参数运行

Breakpoint 1, main (argc=2, argv=0x7fffffffdca8) at helloworld.c:9
9           printf("Hello World %s!\n", argv[1]);

# 查看变量
(gdb) print argc                   # 参数个数
$1 = 2
(gdb) print argv[1]                # 第一个参数
$2 = 0x7fffffffe0a8 "China"

# 继续执行
(gdb) next
Hello World China!

# 设置条件断点
(gdb) break 17 if i == 50
(gdb) continue

Breakpoint 2, main () at helloworld.c:17
17              result += i;

(gdb) print i                      # i = 50
(gdb) print result                 # result = 1225

(gdb) continue                     # 继续执行直到结束
result = 5050
[Inferior 1 exited normally]

(gdb) quit
```

## 四、断点管理

### 设置断点

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `break <函数名>` | `b` | 在函数入口设置断点 |
| `break <行号>` | `b` | 在当前文件指定行设置断点 |
| `break <文件>:<行号>` | `b` | 在指定文件的指定行设置断点 |
| `break *<地址>` | `b` | 在指定内存地址设置断点 |
| `break ... if <条件>` | `b` | 设置条件断点 |
| `tbreak <位置>` | `tb` | 设置临时断点（命中后自动删除） |

### 管理断点

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `info breakpoints` | `i b` | 查看所有断点 |
| `delete <编号>` | `d` | 删除指定断点 |
| `disable <编号>` | `dis` | 禁用断点 |
| `enable <编号>` | `ena` | 启用断点 |
| `clear` | | 删除当前行断点 |
| `ignore <编号> <次数>` | | 忽略断点指定次数 |

### 断点示例

```bash
(gdb) break main                   # 函数断点
(gdb) break hello.c:15             # 行号断点
(gdb) break *0x4005f6              # 地址断点
(gdb) break 20 if i == 5           # 条件断点
(gdb) tbreak 30                    # 临时断点
(gdb) ignore 1 5                   # 忽略前 5 次
```

## 五、变量查看

### 打印变量

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `print <变量>` | `p` | 打印变量值 |
| `print/<格式> <变量>` | `p` | 按指定格式打印 |
| `display <变量>` | | 每次停止时自动打印 |
| `whatis <变量>` | | 查看变量类型 |
| `ptype <变量>` | | 查看详细类型信息 |
| `info locals` | | 查看所有局部变量 |
| `info args` | | 查看函数参数 |

### 打印格式

| 格式 | 说明 |
| :--- | :--- |
| `x` | 十六进制 |
| `d` | 十进制 |
| `u` | 无符号十进制 |
| `o` | 八进制 |
| `t` | 二进制 |
| `c` | 字符 |
| `f` | 浮点数 |
| `s` | 字符串 |

### 查看内存

```bash
x/<数量><格式><单位> <地址>
```

| 单位 | 说明 |
| :--- | :--- |
| `b` | 字节（1 字节） |
| `h` | 半字（2 字节） |
| `w` | 字（4 字节） |
| `g` | 巨字（8 字节） |

**示例**:
```bash
(gdb) x/10xw $rsp                  # 栈顶 10 个 4 字节（十六进制）
(gdb) x/20cb array                 # 数组前 20 字节（字符）
(gdb) x/s ptr                      # 指针指向的字符串
(gdb) x/5i $pc                     # 当前位置 5 条汇编指令
```

## 六、栈帧操作

### 查看调用栈

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `backtrace` | `bt` | 显示调用栈 |
| `backtrace full` | `bt full` | 显示调用栈及局部变量 |
| `backtrace <N>` | `bt <N>` | 显示前 N 个栈帧 |
| `backtrace -<N>` | `bt -<N>` | 显示后 N 个栈帧 |

### 切换栈帧

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `frame <编号>` | `f` | 切换到指定栈帧 |
| `up [N]` | | 向上一层栈帧 |
| `down [N]` | | 向下一层栈帧 |
| `info frame` | | 查看当前帧信息 |

## 七、执行控制

### 运行控制

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `run [args]` | `r` | 运行程序（可带参数） |
| `start` | | 运行到 main() 暂停 |
| `continue` | `c` | 继续执行 |
| `kill` | | 终止程序 |
| `quit` | `q` | 退出 GDB |

### 单步执行

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `next [n]` | `n` | 单步执行，不进入函数 |
| `step [n]` | `s` | 单步执行，进入函数 |
| `nexti [n]` | `ni` | 单步执行一条汇编指令，不进入函数 |
| `stepi [n]` | `si` | 单步执行一条汇编指令，进入函数 |
| `finish` | | 执行完当前函数 |
| `until [line]` | `u` | 执行到指定行 |
| `return [值]` | | 从当前函数返回 |

## 八、TUI 模式

### 启动与切换

```bash
# 启动时启用 TUI
gdb -tui ./program

# GDB 内切换
(gdb) tui enable                   # 启用 TUI
(gdb) tui disable                  # 禁用 TUI
(gdb) Ctrl+X A                     # 快捷键切换
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

### TUI 快捷键

| 按键 | 功能 |
| :--- | :--- |
| `Ctrl+L` | 刷新屏幕 |
| `Ctrl+P` / `Ctrl+N` | 上一条/下一条命令 |
| `↑` / `↓` | 滚动源代码 |
| `←` / `→` | 左右移动源码 |

## 九、配置文件

### .gdbinit 配置

创建 `~/.gdbinit` 自动加载常用设置：

```bash
# 显示设置
set pagination off                 # 关闭分页
set confirm off                    # 关闭确认提示
set print pretty on                # 美化输出
set print array-indexes on         # 显示数组下标

# 历史记录
set history save on
set history filename ~/.gdb_history
set history size 10000

# 常用别名
define pstack
    bt
    info locals
end
```

### 加载顺序

1. `$HOME/.gdbinit`
2. 命令行选项 (`-ex`, `-x`)
3. `./.gdbinit`
4. `-x` 选项指定的文件

## 十、常见问题

### Q1: 无法 attach 到进程

**错误**: `ptrace: Operation not permitted`

**解决**:
```bash
# 临时修改（当前终端）
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

# 永久修改
# 编辑 /etc/sysctl.d/10-ptrace.conf
# kernel.yama.ptrace_scope = 0
```

### Q2: 变量显示 `<optimized out>`

**原因**: 编译时使用了优化选项

**解决**: 使用 `-O0` 重新编译
```bash
gcc -g -O0 program.c -o program
```

### Q3: TUI 模式下方向键失效

**解决**: 使用 `Ctrl+P` / `Ctrl+N` 代替上下箭头

## 附录：常用命令速查表

### 运行控制

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `run [args]` | `r` | 运行程序 |
| `continue` | `c` | 继续执行 |
| `next [n]` | `n` | 单步跳过 |
| `step [n]` | `s` | 单步进入 |
| `finish` | | 跳出函数 |
| `until [line]` | `u` | 执行到指定行 |
| `kill` | | 终止程序 |
| `quit` | `q` | 退出 GDB |

### 断点管理

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `break <loc>` | `b` | 设置断点 |
| `delete [n]` | `d` | 删除断点 |
| `disable [n]` | | 禁用断点 |
| `enable [n]` | | 启用断点 |
| `info breakpoints` | `i b` | 查看断点 |

### 变量查看

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `print <var>` | `p` | 打印变量 |
| `display <var>` | | 自动显示变量 |
| `info locals` | | 查看局部变量 |
| `info args` | | 查看函数参数 |
| `whatis <var>` | | 查看变量类型 |

### 栈帧操作

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `backtrace [n]` | `bt` | 显示调用栈 |
| `frame <n>` | `f` | 切换栈帧 |
| `up [n]` | | 向上一层 |
| `down [n]` | | 向下一层 |
| `info frame` | | 查看帧信息 |

