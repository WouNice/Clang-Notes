# GDB 命令速查手册

## 一、程序启动

### 启动方式

| 命令 | 说明 |
| :--- | :--- |
| `gdb <program>` | 加载可执行程序 |
| `gdb -tui <program>` | TUI 模式启动 |
| `gdb --args <program> <args>` | 带参数启动 |
| `gdb attach <pid>` | 附加到进程 |
| `gdb -p <pid>` | 附加到进程 |
| `gdb <program> core` | 调试 core dump |
| `gdb <program> -c <core>` | 调试 core dump |

### 远程调试

| 命令 | 说明 |
| :--- | :--- |
| `target remote <host>:<port>` | 连接远程调试服务器 |
| `target remote <ip>:1234` | 连接 gdbserver |
| `gdbserver :1234 <program>` | 启动调试服务器 |
| `gdbserver :1234 --attach <pid>` | attach 模式 |

## 二、运行控制

### 基本控制

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `run [args]` | `r` | 运行程序（可带参数） |
| `start` | | 运行到 main() 暂停 |
| `continue` | `c` | 继续执行 |
| `continue <次数>` | | 忽略断点指定次数 |
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
| `until [line]` | `u` | 执行到当前循环结束/指定行 |
| `return [value]` | | 从当前函数返回 |
| `jump <line>` | | 跳转到指定行（慎用） |
| `jump *<address>` | | 跳转到指定地址 |

## 三、断点管理

### 设置断点

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `break <function>` | `b` | 在函数入口设置断点 |
| `break <line>` | `b` | 在当前文件指定行设置断点 |
| `break <file>:<line>` | `b` | 在指定文件的指定行设置断点 |
| `break <file>:<function>` | `b` | 在指定文件的函数入口设置断点 |
| `break *<address>` | `b` | 在指定内存地址设置断点 |
| `break +<offset>` / `break -<offset>` | `b` | 在当前位置偏移处设置断点 |
| `break ... if <condition>` | `b` | 设置条件断点 |
| `tbreak <location>` | `tb` | 设置临时断点（命中后自动删除） |
| `rbreak <regex>` | | 为匹配正则表达式的所有函数设置断点 |

### 管理断点

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `info breakpoints` | `i b` | 查看所有断点 |
| `delete [n]` | `d` | 删除指定/所有断点 |
| `disable [n]` | `dis` | 禁用断点 |
| `enable [n]` | `ena` | 启用断点 |
| `enable once [n]` | | 启用一次后自动禁用 |
| `enable delete [n]` | | 启用一次后自动删除 |
| `clear [location]` | | 删除指定位置断点 |
| `ignore <n> <count>` | | 忽略断点指定次数 |
| `condition <n> <expr>` | | 为断点添加条件 |
| `condition <n>` | | 删除断点条件 |
| `commands <n>` | | 设置断点触发时自动执行的命令 |
| `save breakpoints <file>` | | 保存断点到文件 |
| `source <file>` | | 从文件加载断点 |

### 断点示例

```bash
(gdb) break main                   # 函数断点
(gdb) break hello.c:15             # 行号断点
(gdb) break *0x4005f6              # 地址断点
(gdb) break 20 if i == 5           # 条件断点
(gdb) tbreak 30                    # 临时断点
(gdb) ignore 1 5                   # 忽略前 5 次
```

## 四、观察点

### 设置观察点

| 命令 | 说明 |
| :--- | :--- |
| `watch <variable>` | 当变量值改变时暂停 |
| `rwatch <variable>` | 当变量被读取时暂停（仅硬件） |
| `awatch <variable>` | 当变量被读或写时暂停（仅硬件） |
| `info watchpoints` | 查看所有观察点 |
| `delete <n>` | 删除观察点 |
| `set can-use-hw-watchpoints 0` | 强制使用软件观察点 |

### 观察点示例

```bash
(gdb) watch global_var             # 监控全局变量
(gdb) watch *(int *)0x601050       # 监控内存地址
(gdb) watch -l local_var           # 监控局部变量
(gdb) watch array[10]              # 监控数组元素
```

## 五、捕获点

### 设置捕获点

| 命令 | 说明 |
| :--- | :--- |
| `catch throw` | C++ 抛出异常时暂停 |
| `catch catch` | C++ 捕获异常时暂停 |
| `catch fork` | 调用 fork 时暂停 |
| `catch vfork` | 调用 vfork 时暂停 |
| `catch exec` | 调用 exec 时暂停 |
| `catch syscall <name>` | 调用指定系统调用时暂停 |
| `tcatch <event>` | 临时捕获点 |

## 六、变量查看

### 打印变量

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `print <variable>` | `p` | 打印变量值 |
| `print/<format> <variable>` | `p` | 按指定格式打印 |
| `display <variable>` | | 每次停止时自动打印 |
| `undisplay <n>` | | 取消自动打印 |
| `info display` | | 查看自动打印列表 |
| `whatis <variable>` | | 查看变量类型 |
| `ptype <variable>` | | 查看详细类型信息 |
| `info locals` | | 查看所有局部变量 |
| `info args` | | 查看函数参数 |
| `info variables [regex]` | | 查看匹配的全局变量 |

### 打印格式

| 格式 | 说明 |
| :--- | :--- |
| `x` | 十六进制 |
| `d` | 十进制 |
| `u` | 无符号十进制 |
| `o` | 八进制 |
| `t` | 二进制 |
| `a` | 地址 |
| `c` | 字符 |
| `f` | 浮点数 |
| `s` | 字符串 |

### 打印设置

```bash
(gdb) set print pretty on          # 美化结构体输出
(gdb) set print array-indexes on   # 显示数组下标
(gdb) set print elements 0         # 不限制字符串长度
(gdb) set print null-stop on       # 不显示 \000
```

### 打印数组

```bash
(gdb) print *array@10              # 前 10 个元素
(gdb) print array[5]@10            # 从第 5 个开始打印 10 个
```

### 打印结构体

```bash
(gdb) p *(struct mystruct *)ptr    # 查看结构体
```

## 七、内存查看

### x 命令格式

```bash
x/<数量><格式><单位> <地址>
```

| 单位 | 说明 |
| :--- | :--- |
| `b` | 字节（1 字节） |
| `h` | 半字（2 字节） |
| `w` | 字（4 字节） |
| `g` | 巨字（8 字节） |

### 内存查看示例

```bash
(gdb) x/10xw $rsp                  # 栈顶 10 个 4 字节（十六进制）
(gdb) x/20cb array                 # 数组前 20 字节（字符）
(gdb) x/s ptr                      # 指针指向的字符串
(gdb) x/5i $pc                     # 当前位置 5 条汇编指令
```

## 八、栈帧操作

### 查看调用栈

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `backtrace [n]` | `bt` | 显示调用栈 |
| `backtrace full [n]` | `bt full` | 显示调用栈及局部变量 |
| `backtrace <N>` | `bt <N>` | 显示前 N 个栈帧 |
| `backtrace -<N>` | `bt -<N>` | 显示后 N 个栈帧 |

### 切换栈帧

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `frame <n>` | `f` | 切换到指定栈帧 |
| `up [n]` | | 向上一层栈帧 |
| `up <N>` | | 向上 N 层 |
| `down [n]` | | 向下一层栈帧 |
| `down <N>` | | 向下 N 层 |
| `info frame` | | 查看当前栈帧信息 |
| `info args` | | 查看当前函数参数 |
| `info locals` | | 查看当前函数局部变量 |

## 九、线程调试

### 线程管理

| 命令 | 说明 |
| :--- | :--- |
| `info threads` | 查看所有线程 |
| `thread <n>` | 切换到指定线程 |
| `thread apply <n> <cmd>` | 对指定线程执行命令 |
| `thread apply all <cmd>` | 对所有线程执行命令 |
| `break <loc> thread <n>` | 在指定线程设置断点 |
| `set scheduler-locking on` | 只运行当前线程 |
| `set scheduler-locking off` | 所有线程同步运行 |
| `set scheduler-locking step` | step 时只运行当前线程 |
| `set non-stop on/off` | 设置非停止模式 |

## 十、进程调试

### 进程管理

| 命令 | 说明 |
| :--- | :--- |
| `info inferiors` | 查看所有进程 |
| `inferior <n>` | 切换到指定进程 |
| `attach <pid>` | 附加到进程 |
| `detach` | 分离进程 |
| `detach inferior <n>` | 分离指定进程 |
| `kill inferior <n>` | 杀死指定进程 |
| `set follow-fork-mode parent/child` | 设置 fork 后跟踪的进程 |
| `set detach-on-fork on/off` | 是否分离未跟踪的进程 |

## 十一、TUI 命令

### TUI 控制

| 命令 | 说明 |
| :--- | :--- |
| `tui enable` | 启用 TUI |
| `tui disable` | 禁用 TUI |
| `layout src` | 源代码窗口 |
| `layout asm` | 汇编窗口 |
| `layout split` | 源码 + 汇编 |
| `layout regs` | 寄存器 + 源码/汇编 |
| `winheight <win> [+/-]<n>` | 调整窗口高度 |
| `focus <win>` | 切换焦点窗口 |
| `refresh` | 刷新屏幕 |

### TUI 快捷键

| 按键 | 功能 |
| :--- | :--- |
| `Ctrl+X A` | 切换 TUI 模式 |
| `Ctrl+L` | 刷新屏幕 |
| `Ctrl+P` / `Ctrl+N` | 上一条/下一条命令 |
| `Ctrl+B` / `Ctrl+F` | 光标左移/右移 |
| `↑` / `↓` | 滚动源代码 |
| `←` / `→` | 左右移动源码 |

## 十二、其他命令

### 代码查看

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `list [line]` | `l` | 显示源代码 |
| `list <function>` | `l` | 显示函数代码 |
| `list <file>:<line>` | `l` | 显示指定文件行 |
| `dir <directory>` | | 添加源码目录 |
| `show directories` | | 显示源码目录 |

### 汇编调试

| 命令 | 缩写 | 说明 |
| :--- | :--- | :--- |
| `disassemble [function]` | `disas` | 反汇编 |
| `disassemble /m <function>` | | 反汇编 + 源代码 |
| `disassemble /r <function>` | | 反汇编 + 原始字节 |

### 寄存器查看

| 命令 | 说明 |
| :--- | :--- |
| `info registers` (`i r`) | 查看所有寄存器 |
| `info registers <reg>` | 查看指定寄存器 |
| `print $<reg>` | 打印寄存器值 |

### 常用寄存器（x86-64）

| 寄存器 | 用途 |
| :--- | :--- |
| `rax` | 返回值 |
| `rbx` | 被调用者保存 |
| `rcx` | 第 4 个参数 |
| `rdx` | 第 3 个参数 |
| `rsi` | 第 2 个参数 |
| `rdi` | 第 1 个参数 |
| `rbp` | 栈帧基址 |
| `rsp` | 栈顶指针 |
| `rip` | 指令指针 |

### 日志与配置

| 命令 | 说明 |
| :--- | :--- |
| `set logging on` | 开启日志 |
| `set logging file <file>` | 设置日志文件 |
| `set logging overwrite on` | 覆盖模式 |
| `show logging` | 查看日志设置 |
| `source <file>` | 加载脚本文件 |

### 帮助命令

| 命令 | 说明 |
| :--- | :--- |
| `help [command]` | 查看帮助 |
| `apropos <keyword>` | 搜索相关命令 |
| `info <topic>` | 查看信息 |
| `show <setting>` | 查看设置 |
| `set <setting> <value>` | 修改设置 |

## 附录：命令缩写速查

| 完整命令 | 缩写 |
| :--- | :--- |
| `break` | `b` |
| `continue` | `c` |
| `delete` | `d` |
| `display` | `disp` |
| `finish` | `fin` |
| `info breakpoints` | `i b` |
| `next` | `n` |
| `print` | `p` |
| `run` | `r` |
| `step` | `s` |
| `until` | `u` |
| `backtrace` | `bt` |
| `frame` | `f` |
| `info` | `i` |
| `list` | `l` |
| `nexti` | `ni` |
| `stepi` | `si` |
| `quit` | `q` |
| `disassemble` | `disas` |

