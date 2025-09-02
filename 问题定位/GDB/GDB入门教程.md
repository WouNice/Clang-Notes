#  GDB入门教程

GDB（GNU Debugger）是一个强大的调试工具，一个图形化的调试器，在本地调试程序时非常有用。

## 启动 GDB

```apl
gdb <程序名>           # 加载可执行程序（必须带调试信息 -g 编译）
gdb -tui <程序名>      # 启用 TUI 模式（带代码窗口）
gdb --args <程序名> 参数1 参数2  # 传入命令行参数到被测程序
//gdb --args ./my_program arg1 arg2 arg3
```

## 运行程序

```apl
run (r) [参数]         # 运行程序，可带参数
start                  # 运行到 main() 处暂停
kill                   # 终止正在运行的程序
```

## 断点（Breakpoints）

```apl
break (b) <行号>           # 在某行设置断点
break (b) <函数名>         # 在某个函数入口设置断点
break (b) <文件名>:<行号>  # 在某个文件的特定行设置断点
tbreak <行号>              # 临时断点，命中一次后自动删除
clear <行号>               # 删除某个行的断点
delete (d) [编号]          # 删除某个断点，delete 直接删除所有断点
disable [编号]             # 禁用断点（不会触发）
enable [编号]              # 启用断点
info breakpoints (i b)     # 查看所有断点
```

## 单步调试

```apl
next (n)            # 单步执行，不进入函数内部
step (s)            # 单步执行，进入函数内部
finish              # 执行完当前函数，返回调用处
until (u) <行号>    # 运行到指定行
stepi (si)          # 单步执行一条机器指令
nexti (ni)          # 执行下一条指令，不进入函数
ENTER（回车）        #重复上一条命令。
```

## 继续执行

```apl
continue (c)        # 继续执行程序，直到遇到断点
jump <行号>         # 直接跳转到某行继续执行（慎用）
signal <信号>       # 发送信号给程序，如 signal SIGKILL
jump *<地址>       # 跳到指定内存地址执行
return <值>        # 让当前函数立即返回某个值
```

## 变量 & 内存查看

```apl
print (p) 变量名          # 打印变量的值
print *指针变量           # 解引用指针查看内容
whatis 变量名            # 查看变量类型
ptype 变量名             # 查看变量结构体信息
info locals             # 查看当前函数的所有局部变量
info args               # 查看当前函数的参数
x/<格式> <地址>         # 直接查看内存，如 x/10xw $rsp
display 变量名          # 每次停止时自动打印变量
undisplay 变量名        # 取消自动打印
```

示例：

```apl
p x        # 查看变量 x
p *p       # 查看指针 p 指向的值
p array[3] # 查看数组第 3 个元素
x/10xb &array # 查看数组内存布局（10 个字节，以十六进制显示）
```

在 GDB 中，x 命令用于查看内存内容，其基本格式为：

```apl
x/nfu address
```

-   `n`：表示要查看的内存单元数量。
-   `f`：表示显示格式（例如，x 表示十六进制，d 表示十进制，c 表示字符，s 表示字符串）。
-   `u`：表示内存单元的大小（例如，b 表示字节，h 表示半字（2 字节），w 表示字（4 字节），g 表示巨字（8 字节））。
-   `address`：表示要查看的内存地址。

## 栈和调用信息

```apl
ulimit -c unlimited     # 允许生成 core dump 文件
gdb <程序> core         # 载入 core 文件进行分析
backtrace (bt)      # 显示当前调用栈（函数调用路径）
frame (f) <编号>    # 切换到指定栈帧
info frame          # 查看当前帧信息
info registers      # 显示所有寄存器信息
```

开启OS上的“核心转储”，当程序崩溃时你会得到一个核心（core）文件。这个核心文件就像是对程序的解剖，便于你了解崩溃时发生了什么，以及由什么原因导致。

## 线程调试

```apl
info threads        # 列出所有线程
thread <编号>       # 切换到某个线程
break <行号> thread <线程号>  # 在指定线程的某行设断点
```

## 检查内存

```apl
x/<数量><格式> <地址>  # 查看内存
x/10xw 0x8048000       # 查看 10 个 word（4 字节）的十六进制值
x/20cb 变量名          # 查看变量的 20 个字节，字符格式
info proc mappings     # 显示进程内存映射
```

## 观察点（Watchpoints）

```apl
watch 变量名         # 当变量值发生变化时暂停
rwatch 变量名        # 当变量被读时暂停
awatch 变量名        # 当变量被读或写时暂停
info watchpoints     # 查看所有观察点
```

## 进程附加调试

```apl
gdb -p <pid>        # 附加到某个正在运行的进程
attach <pid>        # 运行 GDB 后，附加到进程
detach
```

有几种方法可以找到 PID：

ps 命令 (Linux/macOS)：`ps aux | grep <process_name>`是一个广泛使用的命令。

pidof <process_name> (Linux)：如果你只需要 PID 并且知道确切的进程名称，这个命令更简单。

-   示例：`pidof my_server`

pgrep <pattern> (Linux/macOS)：对进程名称进行更灵活的模式匹配。

-   示例：`pgrep my_server`

top 或 htop (Linux/macOS)：这些交互式进程监视器显示正在运行的进程的实时列表，包括它们的 PID。

**重要注意事项和潜在问题：**

-   权限：通常，您需要是运行要附加到的进程的同一用户，或者是 root 用户（使用 sudo gdb attachment <PID>）。如果您没有足够的权限，gdb 将拒绝附加。

-   调试符号：为了有效调试，强烈建议您附加到的进程使用调试符号进行编译（编译期间使用 -g 标志）。如果进程是在没有调试符号的情况下编译的，您仍然可以附加，但信息会很有限。您可能会看到汇编代码而不是源代码，并且变量名称可能不可用。

-   对正在运行的进程的影响：附加 gdb 会暂停进程。暂停期间，进程不会执行其预期的工作。请注意影响，尤其是对于时间敏感的进程或生产系统。附加到生产进程时应极其谨慎，并且通常仅在受控的非关键情况下进行。通常最好在开发或暂存环境中进行调试。

-   共享库：如果进程使用共享库，gdb 可能还需要加载这些库的符号。有时 gdb 会自动执行此操作，但如果您在调试共享库中的代码时遇到问题，则可能需要在 gdb 中使用 sharedlibrary 之类的命令。

-   在退出时终止进程（取决于 gdb 设置以及您启动 gdb 的方式）。

## 断点 & 检查点

```apl
checkpoint          # 创建一个程序状态的检查点
restart <编号>      # 恢复到某个检查点
info checkpoints    # 列出所有检查点
```

## 脚本执行

```apl
source <文件>       # 运行 GDB 脚本文件
set logging on      # 开启日志记录
set logging off     # 关闭日志记录
```

## 修改变量值

```apl
set variable 变量名 = 值  # 直接修改变量值
set var x = 10            # 将 x 赋值为 10
set *(ptr) = 42           # 修改指针指向的值
set $寄存器名 = 值  # 修改 CPU 寄存器的值
set $rax = 0x1234  # 设置 rax 寄存器为 0x1234
```

## 逆向调试

```apl
(gdb) record  // 开始记录
(gdb) run     // 运行程序
// ... 程序执行 ...
(gdb) stop    // 停止记录
(gdb) reverse-continue  // 反向执行
(gdb) reverse-step      // 反向单步执行
reverse-next             //反向单步执行程序，跳过函数调用。
reverse-finish           //在函数内部反向执行，直到函数返回。
set exec-direction mode  //设置执行方向，
forward                  //为正向执行，
reverse                  //为反向执行。
info record      //查看记录状态，包括是否正在记录、记录了多少指令等。
record save <filename>    //将记录保存到文件中。
record restore <filename> //从文件中恢复记录。
record discard            //丢弃当前记录。
info record         // 查看记录状态
```

注意事项

-   `record功能会占用大量内存，长时间记录可能会导致 GDB 崩溃。`
-   回放速度可能比正常执行要慢。
-   建议在调试复杂问题时使用record功能，并结合其他 GDB 命令（如断点、监视点等）一起使用，以提高调试效率。

## 退出 GDB

```apl
quit (q)           # 退出 GDB
```

  上述只是GDB功能的一部分，但在日常使用过程中已经够用了。如果需要其他功能请自行用man查询手册，或者直接问AI，更加方便快捷。
