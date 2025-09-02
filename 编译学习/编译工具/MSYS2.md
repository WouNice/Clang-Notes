# MSYS2

msys2官网：https://www.msys2.org/

## 配置MSYS2

配置MSYS2镜像需参考：[msys2镜像](https://mirror.tuna.tsinghua.edu.cn/help/msys2/)

更换镜像源后，执行以下命令来更新包管理器和基础系统：

```shell
pacman -Sy && pacman -Su
# 或者
pacman -Syu
```

## MSYS2子环境

MSYS2 提供了多个子环境，每个子环境都有其特定的用途和特点。以下是这些子环境的详细对比：

-   MSYS：

    -   基础环境：MSYS 是 MSYS2 的基础环境，包含了各种 Linux 命令行工具，如pacman包管理器。
    -   依赖性：在这个子环境中编译的程序依赖于 MSYS2 的动态库。
    -   适用场景：适合需要使用 Linux 命令行工具的场景，但编译出的程序在分发时需要携带这些依赖库，否则无法在其他 Windows 系统上运行。
    -   推荐度：一般不建议使用，除非需要完整的 Linux 环境，可以考虑使用 WSL 或虚拟机。

-   MINGW64：

    -   64位环境：MINGW64 是基于 MinGW 的 64 位子环境，编译的程序不依赖于 MSYS2 的动态库，只依赖于 Windows 自带的 C 语言库

        msvcrt。

    -   通用性：较为通用，适合大多数 Windows 应用程序的开发。

    -   适用场景：适用于需要开发原生 Windows 软件的场景，尤其是那些不需要大量依赖于 POSIX API 的项目。

    -   推荐度：推荐使用，特别是在需要与 Windows 工具链集成的情况下。

-   UCRT64：

    -   较新的 C 语言库：UCRT64 与 MINGW64 类似，但依赖于较新的 C 语言库 ucrt，这是目前微软 Visual Studio 使用的库。
    -   兼容性：ucrt 库在 Windows 10 和 11 中是自带的，但在 Windows 7 和 XP 中可能需要手动安装。
    -   未来趋势：预计未来会逐渐替代 MINGW64，成为主流的 64 位编译环境。
    -   推荐度：强烈推荐使用，特别是对于新项目和需要较高兼容性的开发。

-   CLANG64：

    -   LLVM 工具链：CLANG64 使用 LLVM 工具链而非 GCC 工具链，所有配套环境都是基于 LLVM 的。
    -   特点：gcc.exe 实际上是 clang.exe 的重命名，适合那些对 LLVM 工具链有特定需求的项目。
    -   适用场景：适用于需要使用 LLVM 工具链进行编译的项目，例如那些对编译器优化有较高要求的项目。
    -   推荐度：如果项目对 LLVM 工具链有特定需求，可以考虑使用。

-   MINGW32 和 CLANG32：

    -   32位环境：这两个子环境分别使用 32 位的 MinGW 和 Clang 工具链。
    -   适用场景：除非有特殊需求，否则通常不建议使用 32 位版本，因为现代操作系统和硬件大多支持 64 位。
    -   推荐度：一般不推荐使用，除非项目明确要求 32 位编译。

### 安装 MINGW64 工具链

```shell
pacman -S mingw-w64-x86_64-toolchain
```

这将安装包括 GCC、GDB、Binutils 等在内的完整工具链。

### 安装 UCRT64 工具链

```shell
$ pacman -S mingw-w64-ucrt-x86_64-gcc
```

## 环境变量配置

安装完成后，设置以下环境变量：

```shell
MSYS2_HOME=D:\msys64
MSYS2_PATH_TYPE=inherit
MSYSTEM=MINGW64
```

MSYS2有三种启动方式，分别是：

-   MSYS2 MSYS：`set MSYSTEM=MSYS`
-   MSYS2 MinGW 32bit：`set MSYSTEM=MINGW32`
-   MSYS2 MinGW 64bit：`set MSYSTEM=MINGW64`

在系统PATH中添加MSYS2的路径：

```shell
%MSYS2_HOME%\bin
%MSYS2_HOME%\usr\bin
```

并在MSYS2的shell窗口中编辑 `~/.bashrc` 文件，加入以下配置：

```shell
export PATH=$PATH:/mingw64/bin/
```

