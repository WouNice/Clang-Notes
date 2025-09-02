# 30分钟入门GDB

GDB全称GDB Debugger。GDB具备各种调试功能，使用GDB的调试人员可以查看及修改程序的内部变量值。它是Linux C++开发者赖以生存的神器。本篇文章将简要介绍GDB常用功能，希望对于初学者能起到快速入门的作用。

## 绑定进程

```
gdb ./a.out       # 绑定尚未运行的程序
gdb attach <pid>  # 绑定正在运行的进程
```

## 查看代码

### 查看程序代码

```
> dir <source_directory>    # 添加cpp原文件目录
> l                         # 默认显示暂停处代码
> l   <function>            # 显示函数代码
> l   <file:function>       # 显示文件中某函数代码
> l   <file:line>           # 显示文件中指定行数代码
```

### 查看汇编指令

```
> disass                    # 显示当前汇编指令
> display/i $pc             # 每次回到gdb命令行时，显示当前汇编指令
```

## 断点

### 断点设置

```
> b <file:line>                    # 设置文件中某行断点
> b <file:function>                # 设置文件中某函数断点
> b <namespace::class::function>   # 设置某类的成员函数断点
> b <location> <thread-id>         # 设置某个线程在某处的断点
> b <location>  if <condition>     # 设置某处的条件断点
```

### 断点查看

```
> info  b                          # 查询所有断点
```

### 断点删除/启用/禁用

```
> d <break-id>                      # 删除某断点
> disable   <break-id>              # 禁用某断点
> enable    <break-id>              # 启用某断点
```

### 断点设置自动执行命令

```
> command <break-id>
> p <var>                         # 运行到断点处时自动打印变量<var>
> end
```

## 线程调试

```
pstack <pid>      # 可事先dump某个进程下所有线程的thread id和backtrace，方便gdb调试
> info  threads               # 查看当前所有线程信息
> bt                          # 查看当前线程的backtrace
> bt full                     # 查看当前线程更详细的backtrace(每个栈帧上的参数)
> thread <thread-id>          # 切换到某一个线程
> set scheduler-locking on    # 多线程环境下，只有当前被调试线程会执行
> set scheduler-locking off   # 多线程环境下，除当前被调试线程之外的其他线程也在同步执行
> set scheduler-locking step  # 多线程环境下，对当前被调试线程用step调试时，其他线程不会执行；使用next调试时，其他线程也许会执行
```

## 运行控制

```
> r arg1 arg2 ...             # 重新开始运行二进制
> stop                        # 暂停运行
> c                           # 继续执行
> n                           # 单步执行，遇到函数则跳过
> s                           # 单步执行，遇到函数则跳入函数体
> finish                      # 运行直到跳出当前函数
> until  line                 # 运行直到到达指定行
> call command                # 运行C++命令
> shell                       # 进入shell模式，回到linux终端
> exit                        # 退出shell模式，回到gdb命令行

> set $var=XXX                # 设置gdb变量
> set var=XXX                 # 设置程序中变量
```

## 结束调试

>   detach   # 如果通过gdb attach调试，detach之后，原进程将继续执行 quit 退出gdb

## 调试core

一般情况下，当设置了`ulimit -c unlimited`之后，当程序遇到异常时，会自动转储core文件，方便开发者查看分析现场。

但是，如果想对一个正常运行的进程进行转储, 可使用`gcore`命令：

```
gcore <pid>             # 将进程<pid>转储到core文件中
gdb -c core ./a.out     # 调试core文件
> bt full               # 查看异常的backtrace
```

## 代码窗口

gdb命令行中输入CTRL + X或者CTRL + A，即可调出代码窗口，再按一次退出代码窗口。

注意：在tui模式下，无法使用方向键获取上一条或下一条命令，可使用ctrl+p和ctrl+n替代

## 打印变量

### 打印普通变量

```
> p <var>                     # 打印变量<var>
```

### 打印protobuf message

```
> p <var>.DebugString()       # 使用DebugString()将proto对象内部结构打印出来
```

### 打印内存地址

```
# n：为正整数，表示需要打印的内存单元个数
#
# f：打印格式，如下
# - x: 十六进制
# - d: 十进制
# - u: 十六进制
# - o: 八进制
# - t: 二进制
# - a: 十六进制
# - c: 字符格式
# - f: 浮点数
#
# u: 内存单元大小，如下：
# - b: 单字节
# - h: 双字节
# - w: 四字节
# - g: 八字节
#
# addr: 要打印的内存地址
> x/<n/f/u>  <addr>   # 打印内存地址
```

### 打印长字符串

gdb会限制打印字符串的最大长度。使用下列命令可修改限制：

```
> show print elements  # 显示字符串最大打印长度
> set print elements 0 # 取消字符串最大打印长度
```

### 打印CPU寄存器的值

```
> i r                   # 打印所有寄存器的值
> i r es                # 打印寄存器es的值
```

## 开启日志

gdb默认不开启日志。可使用如下命令开启：

```
> set logging on       # 设置gdb日志开启，gdb会在当前目录下生成gdb.txt记录gdb命令行所有输出结果，方便回溯历史。
```

