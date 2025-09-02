# CMake命令

## cmake_minimum_required

指定使用的 cmake 的最低版本

>   非必须，如果不加可能会有警告

## project

定义工程名称，并可指定工程的版本、工程描述、web主页地址、支持的语言（默认情况支持所有语言），如果不需要这些都是可以忽略的，只需要指定出工程名字即可。

```
# PROJECT 指令的语法是：
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
       [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
       [DESCRIPTION <project-description-string>]
       [HOMEPAGE_URL <url-string>]
       [LANGUAGES <language-name>...])
```

这个命令不是强制性的，但最好都加上。它会引入两个变量 `<PROJECT-NAME>_BINARY_DIR` 和 `<PROJECT-NAME>_SOURCE_DIR`，同时，cmake 自动定义了两个等价的变量 PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR。

`PROJECT_SOURCE_DIR`：这个变量表示的是 CMake 运行 project()指令的那个目录，通常是项目的顶层目录。值得注意的是，CMake 支持在项目中嵌套 project() 指令，而PROJECT_SOURCE_DIR 只会被设置为最近一次执行 project() 指令的目录。

`CMAKE_CURRENT_SOURCE_DIR`：这个变量表示的是正在处理的 CMakeLists.txt文件的所在目录。无论当前正在处理哪个子目录下的 CMakeLists.txt，CMAKE_CURRENT_SOURCE_DIR总是表示这个子目录。当你返回到上一级目录处理 CMakeLists.txt 时，其值会自动还原。

如果你在编写 CMakeLists.txt 时，需要引用项目的顶层目录，那么应该使用 PROJECT_SOURCE_DIR；如果你需要引用正在处理的 CMakeLists.txt文件的所在目录，那么应该使用 CMAKE_CURRENT_SOURCE_DIR。

## add_executable

add_executable用于在当前的 CMakeLists.txt 文件中定义一个可执行文件的构建规则。它告诉 CMake 如何将源文件编译成可执行文件。

通常，add_executable 被用于指定一个或多个源文件，用于构建一个可执行程序。这些源文件通常包含程序的主要逻辑和功能实现。如果需要，可以在调用 add_executable 之后，使用 target_link_libraries 命令来指定链接到可执行文件的库。

基本语法：

```
add_executable(可执行程序名 源文件名称)
```

-   可执行程序名和project中的项目名没有任何关系
-   源文件名可以是一个也可以是多个，如有多个可用空格或;间隔

```
# 样式1
add_executable(app add.c div.c main.c mult.c sub.c)
# 样式2
add_executable(app add.c;div.c;main.c;mult.c;sub.c)
```

## add_subdirectory

add_subdirectory是CMake 的一个命令，用于在当前的 CMakeLists.txt 文件中添加另一个子目录的 CMakeLists.txt 文件，从而将其包含到当前项目的构建中。

>   add_subdirectory指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

当调用 add_subdirectory 时，CMake 会进入指定的子目录，并在那个子目录的 CMakeLists.txt 文件中执行相应的命令，包括添加源文件、设置编译选项、链接库等。这样，子目录中的项目就会被纳入到父项目的构建过程中。

子目录可以访问根目录中定义的变量和目标，但 `CMake` 会为子目录创建一个新的作用域。这意味着在子目录中定义的变量和目标默认不会影响到父目录或其他子目录。

通常，add_subdirectory 会在父项目的 CMakeLists.txt 文件中用于组织大型项目，将其拆分为多个子目录，每个子目录负责管理特定部分的代码或功能。这样可以提高项目的可维护性和可读性，同时也可以使得构建系统更加模块化和灵活。

基本语法：

```
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```

-   source_dir：指定源文件目录

-   binary_dir：指定二进制文件目录

-   EXCLUDE_FROM_ALL：将这个目录从编译过程中排除

## link_directories

link_directories用于指定编译器在链接时查找库文件的目录。当您使用CMake构建项目时，通常需要链接一些库文件，这些库文件可能位于您的系统中的不同位置。使用link_directories命令可以告诉CMake在编译链接时去哪里搜索这些库文件。

基本语法：

```
link_directories(directory1 directory2 ...)
```

参数含义：

-   directory1，directory2…：是您希望编译器在链接时搜索库文件的目录列表。

示例：

例如，假设您的项目依赖于一个名为libexample.a的库文件，而这个库文件位于/path/to/example/lib目录下。您可以使用link_directories命令告诉CMake在链接时搜索这个目录：

```
link_directories(/path/to/example/lib)
```

然后，在您的CMakeLists.txt文件中，您可能会使用target_link_libraries命令将这个库文件链接到您的可执行文件中：

```
target_link_libraries(your_executable_name example)
```

这样，CMake在构建时就会知道在哪里找到名为libexample.a的库文件，并将其链接到您的可执行文件中。

## link_libraries

使用静态库的方法为：

```
include_directories(${PROJECT_SOURCE_DIR}/inc)  # 包含头文件
link_directories(${PROJECT_SOURCE_DIR}/lib)     # 包含静态库路径
link_libraries(libinfo.a)                       # 链接静态库
add_executable(main main.cpp)                   # 生成可执行文件
```

使用动态库的方法为：

```
include_directories(${PROJECT_SOURCE_DIR}/inc) # 包含头文件路径
link_directories(${PROJECT_SOURCE_DIR}/lib)    # 包含动态库路径
add_executable(main main.cpp)                  # 生成可执行文件
target_link_libraries(main libinfo.so)         # 链接动态库
```

可以看到两者的区别是：

-   静态链接：先链接库，在生成可执行文件
-   动态链接：先生成可执行文件，在链接库

这是因为静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动时，静态库就已经被加载到内存中了。而动态库不会在链接阶段被打包到可执行文件中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存。因此，在 `cmake` 中指定要链接的动态库的时候，应该将链接命令写到生成可执行文件之后。

此外：动态库的链接具有传递性，如果动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法。

## add_library

add_library 是 CMake 构建系统中用于定义一个新的库目标的指令，将指定的源文件编译成目标库。它允许您创建静态库、共享库或模块库，以供后续链接到其他目标中。库目标可以在后续的 target_link_libraries 指令中被链接到其他目标中。

基本语法：

```
add_library(target_name
            [STATIC | SHARED | MODULE]
            [source1 [source2 [...]]]
            [...])
```

参数含义：

-   target_name：新库目标的名称，可以是任意有效的 CMake 目标名称。

-   STATIC | SHARED | MODULE：指定要创建的库的类型。可选的类型包括：

    -   STATIC：静态库，会在链接时静态地嵌入到可执行文件中。
    -   SHARED：共享库（动态库），会在运行时动态加载。
    -   MODULE：模块库，类似于共享库，但用于加载时插件。与 SHARED 库不同，它们不链接到项目中的任何目标，不过可以进行动态加载。
    -   OBJECT：可将给定 add_library 的列表中的源码编译到目标文件，不将它们归档到静态库中，也不能将它们链接到共享对象中。如果需要一次性创建静态库和动态库，那么使用对象库尤其有用。
    -   如果未提供类型，CMake 将会通过 `BUILD_SHARED_LIBS` 的值来选择构建 STATIC 还是 SHARED 类型的库，默认为 STATIC。

-   source1, source2, …：用于构建库的源文件列表。

## add_definitions

add_definitions用于向CMake 生成的构建系统中添加编译器定义。当这条指令在CMakeLists.txt文件中被调用时，它会为之后定义的目标（例如，通过add_executable或add_library 创建的目标）添加预处理器定义。

基本语法：

```
add_definitions(-D宏名称)
```

参数含义：

-   -D宏名称：会告知 CMake 在后续的编译步骤中添加 -D 前缀的编译器标志，用于定义预处理器宏。一般情况下，你可以重复使用 add_definitions 来添加多个定义。

示例：

```
add_definitions(-DUSE_FEATURE_X)
add_definitions(-DMY_ANOTHER_DEFINE)
```

以上命令会添加 USE_FEATURE_X 和 MY_ANOTHER_DEFINE 宏到编译器的命令行中，使它们在整个项目的编译过程中都被定义。如果在源代码中使用了#ifdef USE_FEATURE_X，那么它将被编译器识别，并且相关的代码会被包含在编译过程中。

注意事项：

add_definitions 对整个项目目录及其子目录有效，影响所有的目标（即使在 add_definitions 之后才定义的目标）。

建议使用target_compile_definitions代替add_definitions，因为 target_compile_definitions 允许你更精细地控制哪些目标需要被添加定义，而不会影响全局。

>   虽然 add_definitions 是一个有用的指令，但是其全局性可能会导致一些不易调试的问题。对于大型项目和需要更细致管理的情况，建议使用 target_compile_definitions 来为特定目标设置特定的编译器定义，以保持项目的清晰和模块化。

## target_compile_definitions

target_compile_definitions 用于为指定的目标添加编译器定义，而不是全局应用。这更有利于项目的模块化管理，因为它避免了对全局 CMake 环境的污染。

使用 target_compile_definitions：

```
target_compile_definitions(target_name PRIVATE -DDEFINITION)
```

在这个例子中，target_name 应该被替换为实际目标的名称（例如可执行文件或库的名称），DEFINITION将只会添加到那个特定目标的编译定义中。

## target_compile_definitions

## target_compile_options

## target_include_directories

为指定目标（target）添加搜索路径，指定目标是指通过如add_executable()，add_library()这样的命令生成的，并且决不能是alias target（引用目标，别名目标）。

语法格式：

```
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]  <INTERFACE|PUBLIC|PRIVATE> [items1...]  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

### AFTER或BEFORE

可以选择让添加的路径位于搜索列表的开头或结尾。缺省时，默认是AFTER。

### INTERFACE，PUBLIC，PRIVATE

指定接下来的参数item（即路径）的作用域：

-   INTERFACE target对应的头文件才能使用，会指定target的属性INTERFACE_INCLUDE_DIRECTORIES
-   PUBLIC target对应头文件和源文件都能使用，会指定target的属性INCLUDE_DIRECTORIES 和INTERFACE_INCLUDE_DIRECTORIES
-   PRIVATE target对应源文件使用，会指定target的属性INCLUDE_DIRECTORIES

注意：

-   所谓使用是指添加头文件搜索路径（item）。
-   target的属性可以通过set_property()修改。

例如，单独为目标projectA添加搜索路径include1。

```
# 注意当前CMakeLists.txt与include1路径的相对位置关系
target_include_directories(projectA ./include1)
add_executable(projectA main.cpp)
```

### SYSTEM

如果指定SYSTEM，在一些平台上，编译器会将路径作系统包含目录路径，可能对包含的头文件在依赖计算时的警告或者忽略，有一些影响。如果SYSTEM和PUBLIC或INTERFACE同时指定，target的属性INTERFACE_SYSTEM_INCLUDE_DIRECTORIES将填充指定目录。

### include_directories与target_include_directories区别

include_directories 会为当前CMakeLists.txt的所有目标，以及之后添加的所有子目录的目标添加头文件搜索路径。因此，慎用target_include_directories，因为会影响全局target。

target_include_directories 只会为指定目标包含头文件搜索路径。如果想为不同目标设置不同的搜索路径，那么用target_include_directories更合适。

## target_link_libraries

target_link_libraries作用是将目标与它所依赖的库文件或其他目标进行链接，最终生成可执行文件或库。当构建目标时，CMake 将自动检测所链接的库文件或目标的位置，并将它们与目标进行关联。

通过使用 target_link_libraries，您可以方便地管理项目中目标之间的依赖关系，使得构建过程更加清晰和可维护。

基本语法：

```
target_link_libraries(target_name [item1 [item2 [...]]]  [...])
```

参数含义：

-   target_name：要链接的目标的名称，可以是可执行文件、静态库、动态库等。

-   item1, item2, …：要链接的内容，可以是以下类型之一：

    -   库名称：链接一个外部库，比如 pthread、m 等。
    -   目标名称：链接另一个 CMake目标，可以是同一 CMakeLists.txt 文件中定义的目标，也可以是其它 CMake 项目中定义的目标。
    -   文件路径：链接一个特定的文件路径，比如一个静态库文件或动态库文件。

示例：

```
# 指定可执行文件 target_name 需要链接的库
target_link_libraries(target_name library1 library2)

# 指定目标 target_name 需要链接的另一个 CMake 目标
target_link_libraries(target_name other_target)

# 指定目标 target_name 需要链接的库文件路径
target_link_libraries(target_name /path/to/library/liblibrary.a)
```

## target_sources

target_sources用于向已经通过 add_executable 或 add_library 等命令创建的目标添加额外的源文件。它允许你在项目的不同部分为目标指定额外的源文件，而无需在原始 add_executable 或 add_library 调用中包含所有源文件，这有助于增强 CMake 列表文件的模块化。

基本语法：

```
target_sources(<target> PRIVATE|PUBLIC|INTERFACE <source>...)
```

参数含义：

-   < target > ：是之前已经定义的目标的名称（如可执行程序或库名）。
-   < source >：是一个或多个要添加的源文件。
-   PRIVATE|PUBLIC|INTERFACE：关键字用来定义源文件的范围和提供给消费者的接口。

关键字说明：

-   PRIVATE：添加的源文件只在构建< target > 时使用，不会在链接此目标的其他目标（如可执行文件或库）中使用。

-   PUBLIC：源文件既用于目标自身的构建，也提供给链接了此目标的消费者使用。

-   INTERFACE：源文件不用于目标自身的构建，但是会提供给那些链接此目标的消费者。

示例：

假设我们有一个叫做 MyLib 的库，我们想为它添加额外的源文件：

```
add_library(MyLib STATIC src/MyLib.cpp)
# as a good practice, specify the source files relative to the current CMakeList.txt directory
target_sources(MyLib PRIVATE src/AdditionalFile1.cpp src/AdditionalFile2.cpp)
```

在此示例中，我们首先创建了一个名为 MyLib 的静态库，它只有一个源文件 src/MyLib.cpp。然后，我们使用 target_sources 为 MyLib 添加了两个额外的私有源文件src/AdditionalFile1.cpp 和 src/AdditionalFile2.cpp。这些私有源文件将只用于构建 MyLib，不会影响链接了MyLib 的其他目标。

target_sources 命令通常在与目标相关的 CMakeLists.txt文件中使用，它有助于你的项目保持组织和模块化。

注意事项：

1.  使用 target_sources 命令时，应该确保 < target > 已经被定义，否则 CMake 将会报错。
2.  < source > 文件应该是相对于当前 CMakeLists.txt 文件的路径或绝对路径。
3.  target_sources不能用来移除以前添加的源文件，它只能用来添加新的源文件。
4.  对于多平台或有条件的源文件配置，你可以结合 if 语句和 target_sources 命令来根据特定条件选择性地包含源文件。
