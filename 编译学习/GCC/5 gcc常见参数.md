# gcc常见参数

```python
-o  # 输出到指定文件。如果不指定，默认输出到a.out
-E  # 仅进行预处理，不进行编译、汇编和链接
-S  # 将代码转换为文件格式为xxx.s的汇编语言文件，但不进行汇编
-c  # 仅进行编译和汇编，不进行链接操作，常用于编译不包含main程序的子程序代码
-v  # 打印gcc编译时的详细步骤信息
```

编译和路径参数

```python
-l[basic library] # 编译时指定要使用的基础库，样例：-lpthread，针对Posix线程共享库进行编译
-L[shared-library path]      # 共享库的路径添加到搜索的范围，路径为包含xxx.dll/xxx.so/xxx.dlyb文件的目录
-I[include header-file path]          # 将头文件的路径添加到搜索的范围，路径为包含xxx.h/xxx.hpp文件的目录
-shared # 生成共享库，库文件格式为xxx.dll/xxx.so/xxx.dlyb格式的文件
-static # 生成静态库，库文件格式为xxx.a格式的文件
-Wl # 告诉编译器将后面的参数传递给链接器
-Wl,-Bstatic # -Bstatic选项用于对指定的库静态连接
-Wl,-Bdynamic # -Bdynamic搜索共享库（默认）
-Wa,option # 此选项传递option给汇编程序;如果option中间有逗号,就将option分成多个选项,然后传递给会汇编程序
-Wl,option # 此选项传递option给连接程序;如果option中间有逗号,就将option分成多个选项,然后传递给会连接程序
```

预处理参数

```python
# 使用形式：-D[FLAG] 或-D[FLAG]=VALUE
-Dmacro # 在命令行里定义宏，相当于C语言中的" # define macro"
-Umacro # 相当于C语言中的" # undef macro"
-undef # 取消对任何非标准宏的定义
```

警告与报错参数

```python
-Wall # 发出gcc提供的所有有用的报警信息
-Werror # 将警告升级为编译报错
-Wextra / -W # 启用-Wall未启用的额外警告位，对合法但值得怀疑的代码发出警告  例如 -Wsign-compare
-pendantic / -Wpendantic # 发出ISO C和ISO C++标准列出的所有警告，用于语法检查，-pedantic-erros的用法也类似
-fsyntax-only # 仅做语法检查
```

调试参数

```python
-g # 产生带有调试信息的目标代码
-gstabs # 此选项以stabs格式声称调试信息,但是不包括gdb调试信息
-gstabs+ # 此选项以stabs格式声称调试信息,并且包含仅供gdb使用的额外调试信息
-ggdb # 生成gdb专用的调试信息
-glevel # 请求生成调试信息，同时用level指出需要多少信息，默认的level值是2
```

编码配置参数

```python
-fno-exceptions # 屏蔽掉C++的异常，常用于于嵌入式或无法接受异常的系统
-fno-rtti # 禁用RTTI，常用于嵌入式或游戏开发
-fno-asm # 不要识别asm,inline或typeof作为关键字，以便代码可以使用这些词作为标识符。您可以使用关键字__asm__,__inline__来__typeof__ 代替。-ansi暗示-fno-asm
-fPIC / -fpic # 让编译器的代码和位置无关，让代码逻辑不使用绝对地址，只用相对地址，方便文件加载
-nostdinc # 使编译器不再系统默认的头文件目录里面找头文件, 一般和 -I 联合使用,明确限定头文件的位置
-nostdin C++ # 规定不在g++指定的标准路经中搜索,但仍在其他路径中搜索,.此选项在创建libg++库使用
```

