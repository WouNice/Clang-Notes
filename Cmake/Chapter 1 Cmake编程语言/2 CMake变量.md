# CMake变量

## 设置变量

在cmake中可以使用set关键词设置变量值，SET 指令的语法是：

```
# [] 中的参数为可选项, 如不需要可以不写
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```

并使用`${VAR}`来引用这一个变量。

参数含义：

-   variable：是要设置的变量名。

-   value：是要给变量赋予的值。您可以将一个或多个值分配给变量。

-   CACHE：可选的CACHE参数用于创建一个缓存变量，这种变量的值可以由用户在命令行或GUI中设置。

-   type：指定了缓存变量的类型，可以是STRING、FILEPATH、PATH、BOOL等。

-   docstring：是对变量的描述，用于显示在用户界面上。

-   FORCE：表示如果该变量已经存在，是否强制覆盖其值。

示例：

创建一个普通变量并给它赋值：

```
set(my_variable "Hello, World!")
```

创建一个列表变量：

```
set(my_list_var "item1" "item2" "item3")
```

>   一个在list内部是一个由分号;分割的一组字符串。例如，set(var a b c d e)命令将会创建一个list:a;b;c;d;e，但是最终打印变量值的时候得到的是abcde。

修改现有变量的值：

```
set(my_variable "New value" CACHE STRING "Description" FORCE)
```

创建一个缓存变量：

```
set(my_cached_variable "Default value" CACHE STRING "Description")
```

## 删除变量

```
unset(my_variable)
```

## 追加

有时候项目中的源文件并不一定都在同一个目录中，但是这些源文件最终却需要一起进行编译来生成最终的可执行文件或者库文件。如果我们通过file命令对各个目录下的源文件进行搜索，最后还需要做一个字符串拼接的操作，关于字符串拼接可以使用set命令也可以使用list命令。

### 使用set拼接

如果使用set进行字符串拼接，对应的命令格式如下：

```
set(变量名1 ${变量名1} ${变量名2} ...)
```

关于上面的命令其实就是将从第二个参数开始往后所有的字符串进行拼接，最后将结果存储到第一个参数中，如果第一个参数中原来有数据会对原数据就行覆盖。

### 使用list拼接

如果使用list进行字符串拼接，对应的命令格式如下：

```
list(APPEND <list> [<element> ...])
```

list命令的功能比set要强大，字符串拼接只是它的其中一个功能，所以需要在它第一个参数的位置指定出我们要做的操作，APPEND表示进行数据追加，后边的参数和set就一样了。

```
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
list(APPEND SRC_1 ${SRC_1} ${SRC_2}
```

## 字符串移除

```
list(REMOVE_ITEM <list> <value> [<value> ...])
```

示例：

```
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
# 移除前日志
message(STATUS "message: ${SRC_1}")
# 移除 main.cpp
list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
```

>   通过 file 命令搜索源文件的时候得到的是文件的绝对路径（在list中每个文件对应的路径都是一个item，并且都是绝对路径），那么在移除的时候也要将该文件的绝对路径指定出来才可以，否是移除操作不会成功
>

## 变量作用域

-   根节点CMakeLists.txt中的变量全局有效
-   父节点CMakeLists.txt中的变量可以在子节点中使用
-   子节点CMakeLists.txt中的变量只能在当前节点中使用

## 常用内置变量

列举一些常见的CMake内置变量，需要注意的是内置变量都是CMAKE开头

1.  CMAKE_SOURCE_DIR: CMakeLists.txt所在的顶级源代码目录的路径。
2.  CMAKE_BINARY_DIR: 构建目录的路径，即执行cmake命令时生成的Makefile或其他构建系统文件所在的目录。
3.  CMAKE_CURRENT_SOURCE_DIR: 当前处理的CMakeLists.txt所在的目录的路径。
4.  CMAKE_CURRENT_BINARY_DIR: 当前处理的CMakeLists.txt生成的目标文件所在的目录的路径。
5.  CMAKE_CURRENT_LIST_FILE: 当前正在处理的CMakeLists.txt的完整路径和文件名。
6.  CMAKE_CURRENT_LIST_DIR: 当前正在处理的CMakeLists.txt所在的目录的路径。
7.  CMAKE_MODULE_PATH: 用于指定额外的CMake模块的路径。
8.  CMAKE_INCLUDE_PATH: 用于指定额外的包含文件的路径。
9.  CMAKE_LIBRARY_PATH: 用于指定额外的库文件的路径。
10.  CMAKE_SYSTEM: 当前操作系统的名称。
11.  CMAKE_SYSTEM_NAME: 当前操作系统的名称，与CMAKE_SYSTEM相同。
12.  CMAKE_SYSTEM_VERSION: 当前操作系统的版本号。
13.  CMAKE_C_COMPILER: C编译器的路径。
14.  CMAKE_CXX_COMPILER: C++编译器的路径。
15.  CMAKE_BUILD_TYPE: 构建类型，如Debug、Release等。
16.  CMAKE_INSTALL_PREFIX: 安装目录的路径。

另外有一些内置变量是没有值的，比如`CMAKE_BUILD_TYPE`。
