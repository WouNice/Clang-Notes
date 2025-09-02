# 支持 gdb

CMake和GDB的结合确实能使C或C++的开发工作变得轻松，它们可以共同实现跨平台的项目构建和源代码级别的调试。

## 如何支持gdb调试

让 CMake 支持 gdb 的设置也很容易，只需要指定 `Debug` 模式下开启 `-g` 选项：

```
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

1、在CMakeLists.txt中加入这三行代码。

当CMAKE_BUILD_TYPE变量是Debug 的时候，CMake 会使用变量 CMAKE_CXX_FLAGS_DEBUG 和 CMAKE_C_FLAGS_DEBUG 中的字符串作为编译选项生成 Makefile。

2、在编译程序时加上-g选项的作用是生成带调试信息的可执行文件。

这些调试信息包括变量名、函数名、文件名等信息，可以让调试器（如gdb）在调试程序时更容易跟踪程序的执行过程，并能够查看变量的值、函数的调用栈等信息。

通过使用-g选项编译程序，可以使得在调试器中更方便地定位程序中的问题，提高调试效率。

实例：

1、创建CMake配置文件：

```
# 设置不同构建类型的编译器标志
set(CMAKE_C_FLAGS_DEBUG "-O0 -g -Wall")
set(CMAKE_C_FLAGS_RELEASE "-O3 -Wall")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "-O2 -g -Wall")
set(CMAKE_C_FLAGS_MINSIZEREL "-Os -Wall")
```

2、构建项目：

```
cmake -DCMAKE_BUILD_TYPE=Debug ..
```
