# GCC编译工具链

GCC 编译工具链（toolchain）是指以GCC编译器为核心的一整套工具，用于把源代码转化成可执行应用程序。它主要包含以下三部分内容：

1.  gcc-core：即GCC编译器，用于完成预处理和编译过程，例如把C代码转换成汇编代码。
2.  Binutils：除GCC编译器外的一系列小工具包括了链接器ld，汇编器as、目标文件格式查看器readelf等。
3.  glibc：包含了主要的C语言标准函数库，C语言中常常使用的打印函数printf、malloc 函数就在glibc库中。

在很多场合下会直接用GCC 编译器来指代整套GCC 编译工具链。

>   GCC和gcc是两个概念，GCC是工具链的集合，里面除了gcc/g++还包含了ccl，cclplus等组件。gcc/g++只是GCC工具链的一个子集。

## Binutils 工具集

Binutils（bin utility）：是GNU 二进制工具集，通常跟GCC编译器一起打包安装到系统。

网站地址为：[Binutils - GNU Project](https://www.gnu.org/software/binutils/)

在进行程序开发的时候通常不会直接调用这些工具，而是在使用GCC编译指令的时候由 GCC 编译器间接调用。下面是其中一些常用的工具：

-   addr2line：用来将程序地址转换成其所对应的程序源文件及所对应的代码行，也可以得到所对应的函数。该工具将帮助调试器在调试的过程中定位对应的源代码位置
-   as：汇编器，把汇编语言代码转换为机器码（目标文件）。
-   ld：链接器，把编译生成的多个目标文件组织成最终的可执行程序文件。
-   ar：主要用于创建静态库
-   ldd：可以用于查看一个可执行程序依赖的共享库
-   readelf：可用于查看目标文件或可执行程序文件的信息。
-   nm：可用于查看目标文件中出现的符号。
-   objcopy：可用于目标文件格式转换，如.bin 转换成.elf、.elf 转换成.bin 等。
-   objdump：可用于查看目标文件的信息，最主要的作用是反汇编。
-   size：可用于查看目标文件不同部分的尺寸和总尺寸，例如代码段大小、数据段大小、使用的静态内存、总大小等。

## glibc 库

glibc 库是 GNU 组织为 GNU 系统以及 Linux 系统编写的 C 语言标准库，因为绝大部分C程序都依赖该函数库，该文件甚至会直接影响到系统的正常运行，例如常用的文件操作函数read、write、open，打印函数printf、动态内存申请函数malloc 等。

glibc 官网地址为：[The GNU C Library](https://www.gnu.org/software/libc/)

## GCC常用编译器

| GCC 编译器命令 | 含义                  |
| -------------- | --------------------- |
| cc             | 指的是 C 语言编译器   |
| gcc            | 指的是 C 语言编译器   |
| cpp            | 指的是预处理编译器    |
| g++            | 指的是 C++ 语言编译器 |

一个有趣的事实就是，就本质而言，gcc和g++并不是编译器，也不是编译器的集合，它们只是一种驱动器，根据参数中要编译的文件的类型，调用对应的GUN编译器而已。

由于编译器是可以更换的，所以 gcc 不仅仅可以编译 C 文件。所以，更准确的说法是：gcc 调用了 C compiler，而 g++ 调用了 C++ compiler。
### gcc 和 g++ 的主要区别

-   对于 *.c 和 *.cpp 文件，gcc 分别当做 c 和 cpp 文件编译（c 和 cpp 的语法强度是不一样的）

-   对于 *.c 和 *.cpp 文件，g++ 则统一当做 cpp 文件编译

-   使用 g++ 编译文件时，g++ 会自动链接标准库 STL，而 gcc 不会自动链接 STL

-   gcc 在编译 C 文件时，可使用的预定义宏是比较少的

-   gcc 在编译 cpp 文件时、g++ 在编译 c 文件和 cpp 文件时（这时候 gcc 和 g++ 调用的都是 cpp 文件的编译器），会加入一些额外的宏。

-   在用 gcc 编译 c++ 文件时，为了能够使用 STL，需要加参数 -lstdc++，但这并不代表 gcc -lstdc++ 和 g++ 等价，它们的区别不仅仅是这个。

## 常用指令

### `ld` 指令

ld（`ld-linux.so`）默认搜索 `/lib` 和 `/usr/lib` 这两个目录。

### `ldd`指令

`ldd` 指令，全称 `list dynamic dependencies`，用于检查动态库依赖关系。

```apl
$ ldd test

linux-vdso.so.1 (0x00007fff065e4000)
liba.so => not found
libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fa1d384e000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa1d345d000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fa1d30bf000)
/lib64/ld-linux-x86-64.so.2 (0x00007fa1d3dd9000)
libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fa1d2ea7000
```

参数解释：

-   linux-vdso.so.1为一个虚拟的库 Virtual Dynamic Shared Object，所以没有路径，包含一些内核api，比如 syscall() 这个函数就是在linux-vdso.so.1里头。内核把包含某 .so 的内存页在程序启动的时候映射入其内存空间，对应的程序就可以当普通的 .so 来使用里面的函数。

-   /lib64/ld-linux-x86-64.so.2为链接器，是程序中第一个被加载的，通过 ldopen 来动态加载剩下的库。

-   libstdc++.so.6和libc.so.6应该是c++和c的标准库。

-   libgcc_s.so.1是gcc运行时库，提供一些gcc支持的api。

### `readelf`指令

readelf，用于查看elf文件。

### `LD_DEBUG`指令

`LD_DEBUG=libs`，对链接库加载过程进行debug，会打印出 `error while loading shared libraries`的搜索过程。

```apl
LD_DEBUG=libs ./test
```

