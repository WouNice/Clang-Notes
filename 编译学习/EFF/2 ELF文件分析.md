# EFF文件分析

## gcc怎么生成elf文件

在使用GCC生成ELF（Executable and Linkable Format）文件时，你需要使用GCC的编译和链接选项。

以下是生成ELF文件的基本步骤：

1.  使用GCC编译源代码文件，生成目标文件（.o）。
2.  使用GCC链接目标文件，生成ELF可执行文件。

假设你有一个名为`test.c`的源代码文件，你可以按照以下步骤使用GCC来生成ELF文件：

test.c：

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

生成ELF文件：

```v
gcc -c test.c -o test.o
gcc test.o -o test.elf
```
运行ELF文件：

```v
# ./test.elf
Hello, World!
```

解释：

-   第一条命令使用`-c`选项来编译`test.c`，生成目标文件`test.o`。
-   第二条命令链接`test.o`，生成名为`test.elf`的ELF可执行文件。

## 分析工具

在Linux下，分析ELF（Executable and Linkable Format）文件的主要工具有很多种。下面是一些常用的ELF文件分析工具。

### readelf

功能：显示 ELF 文件的详细信息，如头部信息、段表、符号表、重定位表等。

常用场景：用于快速查看 ELF 文件的元数据和结构。

示例：

readelf -h显示可执行文件的头部信息。

```v
# readelf -h ./test.elf
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x401040
  Start of program headers:          64 (bytes into file)
  Start of section headers:          15392 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         32
  Section header string table index: 31
```

readelf -S：查看test可执行文件各个section的信息

```v
# readelf -S test.elf
There are 32 section headers, starting at offset 0x3c20:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000400318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000400338  00000338
       0000000000000020  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000400358  00000358
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000040037c  0000037c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000004003a0  000003a0
       000000000000001c  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           00000000004003c0  000003c0
       0000000000000090  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000400450  00000450
       000000000000007e  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           00000000004004ce  000004ce
       000000000000000c  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          00000000004004e0  000004e0
       0000000000000030  0000000000000000   A       7     1     8
  [10] .rela.dyn         RELA             0000000000400510  00000510
       0000000000000060  0000000000000018   A       6     0     8
  [11] .rela.plt         RELA             0000000000400570  00000570
       0000000000000018  0000000000000018  AI       6    23     8
  [12] .init             PROGBITS         0000000000401000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [13] .plt              PROGBITS         0000000000401020  00001020
       0000000000000020  0000000000000010  AX       0     0     16
  [14] .text             PROGBITS         0000000000401040  00001040
       00000000000000fb  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         000000000040113c  0000113c
       000000000000000d  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         0000000000402000  00002000
       000000000000001e  0000000000000000   A       0     0     8
  [17] .eh_frame_hdr     PROGBITS         0000000000402020  00002020
       000000000000002c  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000402050  00002050
       000000000000008c  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       0000000000403e00  00002e00
       0000000000000008  0000000000000008  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       0000000000403e08  00002e08
       0000000000000008  0000000000000008  WA       0     0     8
  [21] .dynamic          DYNAMIC          0000000000403e10  00002e10
       00000000000001d0  0000000000000010  WA       7     0     8
  [22] .got              PROGBITS         0000000000403fe0  00002fe0
       0000000000000020  0000000000000008  WA       0     0     8
  [23] .got.plt          PROGBITS         0000000000404000  00003000
       0000000000000020  0000000000000008  WA       0     0     8
  [24] .data             PROGBITS         0000000000404020  00003020
       0000000000000004  0000000000000000  WA       0     0     1
  [25] .bss              NOBITS           0000000000404024  00003024
       0000000000000004  0000000000000000  WA       0     0     1
  [26] .comment          PROGBITS         0000000000000000  00003024
       000000000000002e  0000000000000001  MS       0     0     1
  [27] .annobin.notes    STRTAB           0000000000000000  00003052
       000000000000013e  0000000000000001  MS       0     0     1
  [28] .gnu.build.a[...] NOTE             0000000000406028  00003190
       0000000000000144  0000000000000000           0     0     4
  [29] .symtab           SYMTAB           0000000000000000  000032d8
       0000000000000600  0000000000000018          30    46     8
  [30] .strtab           STRTAB           0000000000000000  000038d8
       000000000000020c  0000000000000000           0     0     1
  [31] .shstrtab         STRTAB           0000000000000000  00003ae4
       000000000000013b  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```

### objdump

功能：显示二进制文件的反汇编、段信息、符号表等。它还可以显示目标文件的不同表示形式，如机器码、汇编指令或源代码。

常用场景：用于调试、性能分析或理解程序如何编译和链接。

示例：显示目标文件的反汇编代码及其对应的源码。

```shell
# objdump -dS ./test.elf

./test.elf:     file format elf64-x86-64


Disassembly of section .init:

0000000000401000 <_init>:
  401000:       f3 0f 1e fa             endbr64
  401004:       48 83 ec 08             sub    $0x8,%rsp
  401008:       48 8b 05 e1 2f 00 00    mov    0x2fe1(%rip),%rax        # 403ff0 <__gmon_start__>
  40100f:       48 85 c0                test   %rax,%rax
  401012:       74 02                   je     401016 <_init+0x16>
  401014:       ff d0                   callq  *%rax
  401016:       48 83 c4 08             add    $0x8,%rsp
  40101a:       c3                      retq

Disassembly of section .plt:

0000000000401020 <.plt>:
  401020:       ff 35 e2 2f 00 00       pushq  0x2fe2(%rip)        # 404008 <_GLOBAL_OFFSET_TABLE_+0x8>
  401026:       ff 25 e4 2f 00 00       jmpq   *0x2fe4(%rip)        # 404010 <_GLOBAL_OFFSET_TABLE_+0x10>
  40102c:       0f 1f 40 00             nopl   0x0(%rax)

0000000000401030 <puts@plt>:
  401030:       ff 25 e2 2f 00 00       jmpq   *0x2fe2(%rip)        # 404018 <puts@GLIBC_2.2.5>
  401036:       68 00 00 00 00          pushq  $0x0
  40103b:       e9 e0 ff ff ff          jmpq   401020 <.plt>

Disassembly of section .text:

0000000000401040 <_start>:
  401040:       f3 0f 1e fa             endbr64
  401044:       31 ed                   xor    %ebp,%ebp
  401046:       49 89 d1                mov    %rdx,%r9
  401049:       5e                      pop    %rsi
  40104a:       48 89 e2                mov    %rsp,%rdx
  40104d:       48 83 e4 f0             and    $0xfffffffffffffff0,%rsp
  401051:       50                      push   %rax
  401052:       54                      push   %rsp
  401053:       45 31 c0                xor    %r8d,%r8d
  401056:       31 c9                   xor    %ecx,%ecx
  401058:       48 c7 c7 26 11 40 00    mov    $0x401126,%rdi
  40105f:       ff 15 7b 2f 00 00       callq  *0x2f7b(%rip)        # 403fe0 <__libc_start_main@GLIBC_2.34>
  401065:       f4                      hlt
  401066:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  40106d:       00 00 00

0000000000401070 <_dl_relocate_static_pie>:
  401070:       f3 0f 1e fa             endbr64
  401074:       c3                      retq
  401075:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  40107c:       00 00 00
  40107f:       90                      nop

0000000000401080 <deregister_tm_clones>:
  401080:       48 8d 3d a1 2f 00 00    lea    0x2fa1(%rip),%rdi        # 404028 <__TMC_END__>
  401087:       48 8d 05 9a 2f 00 00    lea    0x2f9a(%rip),%rax        # 404028 <__TMC_END__>
  40108e:       48 39 f8                cmp    %rdi,%rax
  401091:       74 15                   je     4010a8 <deregister_tm_clones+0x28>
  401093:       48 8b 05 4e 2f 00 00    mov    0x2f4e(%rip),%rax        # 403fe8 <_ITM_deregisterTMCloneTable>
  40109a:       48 85 c0                test   %rax,%rax
  40109d:       74 09                   je     4010a8 <deregister_tm_clones+0x28>
  40109f:       ff e0                   jmpq   *%rax
  4010a1:       0f 1f 80 00 00 00 00    nopl   0x0(%rax)
  4010a8:       c3                      retq
  4010a9:       0f 1f 80 00 00 00 00    nopl   0x0(%rax)

00000000004010b0 <register_tm_clones>:
  4010b0:       48 8d 3d 71 2f 00 00    lea    0x2f71(%rip),%rdi        # 404028 <__TMC_END__>
  4010b7:       48 8d 35 6a 2f 00 00    lea    0x2f6a(%rip),%rsi        # 404028 <__TMC_END__>
  4010be:       48 29 fe                sub    %rdi,%rsi
  4010c1:       48 89 f0                mov    %rsi,%rax
  4010c4:       48 c1 ee 3f             shr    $0x3f,%rsi
  4010c8:       48 c1 f8 03             sar    $0x3,%rax
  4010cc:       48 01 c6                add    %rax,%rsi
  4010cf:       48 d1 fe                sar    %rsi
  4010d2:       74 14                   je     4010e8 <register_tm_clones+0x38>
  4010d4:       48 8b 05 1d 2f 00 00    mov    0x2f1d(%rip),%rax        # 403ff8 <_ITM_registerTMCloneTable>
  4010db:       48 85 c0                test   %rax,%rax
  4010de:       74 08                   je     4010e8 <register_tm_clones+0x38>
  4010e0:       ff e0                   jmpq   *%rax
  4010e2:       66 0f 1f 44 00 00       nopw   0x0(%rax,%rax,1)
  4010e8:       c3                      retq
  4010e9:       0f 1f 80 00 00 00 00    nopl   0x0(%rax)

00000000004010f0 <__do_global_dtors_aux>:
  4010f0:       f3 0f 1e fa             endbr64
  4010f4:       80 3d 29 2f 00 00 00    cmpb   $0x0,0x2f29(%rip)        # 404024 <completed.0>
  4010fb:       75 13                   jne    401110 <__do_global_dtors_aux+0x20>
  4010fd:       55                      push   %rbp
  4010fe:       48 89 e5                mov    %rsp,%rbp
  401101:       e8 7a ff ff ff          callq  401080 <deregister_tm_clones>
  401106:       c6 05 17 2f 00 00 01    movb   $0x1,0x2f17(%rip)        # 404024 <completed.0>
  40110d:       5d                      pop    %rbp
  40110e:       c3                      retq
  40110f:       90                      nop
  401110:       c3                      retq
  401111:       66 66 2e 0f 1f 84 00    data16 nopw %cs:0x0(%rax,%rax,1)
  401118:       00 00 00 00
  40111c:       0f 1f 40 00             nopl   0x0(%rax)

0000000000401120 <frame_dummy>:
  401120:       f3 0f 1e fa             endbr64
  401124:       eb 8a                   jmp    4010b0 <register_tm_clones>

0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  40112f:       e8 fc fe ff ff          callq  401030 <puts@plt>
  401134:       b8 00 00 00 00          mov    $0x0,%eax
  401139:       5d                      pop    %rbp
  40113a:       c3                      retq

Disassembly of section .fini:

000000000040113c <_fini>:
  40113c:       f3 0f 1e fa             endbr64
  401140:       48 83 ec 08             sub    $0x8,%rsp
  401144:       48 83 c4 08             add    $0x8,%rsp
  401148:       c3                      retq
```

ELF文件无法被当做普通文本文件打开，如果希望直接查看一个 ELF文件包含的指令和数据，需要使用反汇编的方法，可使用`objdump -D`，示例：

```v
# objdump -D ./test.elf

./test.elf:     file format elf64-x86-64


Disassembly of section .interp:

0000000000400318 <.interp>:
  400318:	2f                   	(bad)
  400319:	6c                   	insb   (%dx),%es:(%rdi)
  40031a:	69 62 36 34 2f 6c 64 	imul   $0x646c2f34,0x36(%rdx),%esp
  400321:	2d 6c 69 6e 75       	sub    $0x756e696c,%eax
  400326:	78 2d                	js     400355 <__abi_tag-0x27>
  400328:	78 38                	js     400362 <__abi_tag-0x1a>
  40032a:	36 2d 36 34 2e 73    	ss sub $0x732e3436,%eax
  400330:	6f                   	outsl  %ds:(%rsi),(%dx)
  400331:	2e 32 00             	xor    %cs:(%rax),%al

Disassembly of section .note.gnu.property:
...
```

 使用`objdump -S`将其反汇编并且将其C语言源代码混合显示出来：

```v
# objdump -S ./test.elf

./test.elf:     file format elf64-x86-64


Disassembly of section .init:

0000000000401000 <_init>:
  401000:	f3 0f 1e fa          	endbr64
  401004:	48 83 ec 08          	sub    $0x8,%rsp
  401008:	48 8b 05 e1 2f 00 00 	mov    0x2fe1(%rip),%rax        # 403ff0 <__gmon_start__>
  40100f:	48 85 c0             	test   %rax,%rax
  401012:	74 02                	je     401016 <_init+0x16>
  401014:	ff d0                	callq  *%rax
  401016:	48 83 c4 08          	add    $0x8,%rsp
  40101a:	c3                   	retq

Disassembly of section .plt:
...
```

### ldd

功能：列出可执行文件所依赖的共享库。它显示程序在运行时需要加载的库文件。

常用场景：用于检查二进制文件的库依赖关系，确保所有依赖项都可用。

示例：显示可执行文件的库依赖关系。

```v
# ldd ./test.elf
        linux-vdso.so.1 (0x00007fff64587000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f2647a00000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f2647d1e000)
```

### nm

功能：列出目标文件中的符号表。符号是变量、函数等名称的引用，nm 显示这些符号的名称和地址。

常用场景：用于查找二进制文件中的特定符号或函数。

示例：列出目标文件中的符号表。

```v
# nm ./test.elf
0000000000403e10 d _DYNAMIC
0000000000404000 d _GLOBAL_OFFSET_TABLE_
0000000000402000 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
00000000004020d8 r __FRAME_END__
0000000000402020 r __GNU_EH_FRAME_HDR
0000000000404028 D __TMC_END__
000000000040037c r __abi_tag
0000000000404024 B __bss_start
0000000000404020 D __data_start
00000000004010f0 t __do_global_dtors_aux
0000000000403e08 d __do_global_dtors_aux_fini_array_entry
0000000000402008 R __dso_handle
0000000000403e00 d __frame_dummy_init_array_entry
                 w __gmon_start__
                 U __libc_start_main@GLIBC_2.34
0000000000401070 T _dl_relocate_static_pie
0000000000404024 D _edata
0000000000404028 B _end
000000000040113c T _fini
0000000000401000 T _init
0000000000401040 T _start
0000000000404024 b completed.0
0000000000404020 W data_start
0000000000401080 t deregister_tm_clones
0000000000401120 t frame_dummy
0000000000401126 T main
                 U puts@GLIBC_2.2.5
00000000004010b0 t register_tm_clones
```

### strings

功能：从二进制文件中提取可打印的字符串。它搜索文件中的 ASCII 字符串，这对于查看文件内容或查找关键字符串很有用。

常用场景：用于快速浏览二进制文件中的文本信息。

示例：从二进制文件中提取字符串并搜索关键词。

```v
# strings ./test.elf | grep "keyword"
```

### gdb

功能：GNU 调试器，用于调试程序。它提供了丰富的调试功能，如设置断点、单步执行、查看变量值等。

常用场景：用于调试和分析程序的行为，包括运行时错误、性能问题等。

示例：启动 GDB 调试器并加载可执行文件。

```v
gdb ./test.elf
```

### dwarfdump

功能：查看 ELF 文件中的 DWARF 调试信息。DWARF 是一种用于存储调试信息的格式，包括源文件路径、变量位置、数据类型等。

常用场景：用于深入调试分析，特别是当需要理解源代码和编译后代码之间的映射关系时。

示例：显示 ELF 文件中的 DWARF 调试信息。

```v
dwarfdump ./test.elf
```

### size

功能：显示二进制文件各个部分的大小，包括代码段、数据段、bss 段等。它帮助开发者了解程序在内存中的占用情况。

常用场景：用于评估程序的内存使用或查找潜在的优化点。

示例：显示二进制文件的大小信息。

```v
# size ./test.elf
   text    data     bss     dec     hex filename
   1143     548       4    1695     69f ./test.elf
```

### c++filt

功能：解码 C++ 编译器生成的混淆符号名，将其转换回原始的可读形式。这对于理解 C++ 程序的调试信息很有用。

常用场景：当使用如 objdump 或 readelf 等工具查看 ELF 文件的符号信息时，可以使用 c++filt 来解码混淆的符号名。

示例：解码 objdump 输出的混淆 C++ 符号名。

```shell
# objdump -tC ./test.elf | c++filt

./test.elf:     file format elf64-x86-64

SYMBOL TABLE:
0000000000400318 l    d  .interp        0000000000000000              .interp
0000000000400338 l    d  .note.gnu.property     0000000000000000              .note.gnu.property
0000000000400358 l    d  .note.gnu.build-id     0000000000000000              .note.gnu.build-id
000000000040037c l    d  .note.ABI-tag  0000000000000000              .note.ABI-tag
00000000004003a0 l    d  .gnu.hash      0000000000000000              .gnu.hash
00000000004003c0 l    d  .dynsym        0000000000000000              .dynsym
0000000000400450 l    d  .dynstr        0000000000000000              .dynstr
00000000004004ce l    d  .gnu.version   0000000000000000              .gnu.version
00000000004004e0 l    d  .gnu.version_r 0000000000000000              .gnu.version_r
0000000000400510 l    d  .rela.dyn      0000000000000000              .rela.dyn
0000000000400570 l    d  .rela.plt      0000000000000000              .rela.plt
0000000000401000 l    d  .init  0000000000000000              .init
0000000000401020 l    d  .plt   0000000000000000              .plt
0000000000401040 l    d  .text  0000000000000000              .text
000000000040113c l    d  .fini  0000000000000000              .fini
0000000000402000 l    d  .rodata        0000000000000000              .rodata
0000000000402020 l    d  .eh_frame_hdr  0000000000000000              .eh_frame_hdr
0000000000402050 l    d  .eh_frame      0000000000000000              .eh_frame
0000000000403e00 l    d  .init_array    0000000000000000              .init_array
0000000000403e08 l    d  .fini_array    0000000000000000              .fini_array
0000000000403e10 l    d  .dynamic       0000000000000000              .dynamic
0000000000403fe0 l    d  .got   0000000000000000              .got
0000000000404000 l    d  .got.plt       0000000000000000              .got.plt
0000000000404020 l    d  .data  0000000000000000              .data
0000000000404024 l    d  .bss   0000000000000000              .bss
0000000000000000 l    d  .comment       0000000000000000              .comment
0000000000000000 l    d  .annobin.notes 0000000000000000              .annobin.notes
0000000000406028 l    d  .gnu.build.attributes  0000000000000000              .gnu.build.attributes
0000000000000000 l    df *ABS*  0000000000000000              /usr/lib/gcc/x86_64-redhat-linux/11/../../../../lib64/crt1.o
000000000040037c l     O .note.ABI-tag  0000000000000020              __abi_tag
0000000000000000 l    df *ABS*  0000000000000000              crtstuff.c
0000000000401080 l     F .text  0000000000000000              deregister_tm_clones
00000000004010b0 l     F .text  0000000000000000              register_tm_clones
00000000004010f0 l     F .text  0000000000000000              __do_global_dtors_aux
0000000000404024 l     O .bss   0000000000000001              completed.0
0000000000403e08 l     O .fini_array    0000000000000000              __do_global_dtors_aux_fini_array_entry
0000000000401120 l     F .text  0000000000000000              frame_dummy
0000000000403e00 l     O .init_array    0000000000000000              __frame_dummy_init_array_entry
0000000000000000 l    df *ABS*  0000000000000000              test.c
0000000000000000 l    df *ABS*  0000000000000000              crtstuff.c
00000000004020d8 l     O .eh_frame      0000000000000000              __FRAME_END__
0000000000000000 l    df *ABS*  0000000000000000
0000000000403e10 l     O .dynamic       0000000000000000              _DYNAMIC
0000000000402020 l       .eh_frame_hdr  0000000000000000              __GNU_EH_FRAME_HDR
0000000000404000 l     O .got.plt       0000000000000000              _GLOBAL_OFFSET_TABLE_
0000000000000000       F *UND*  0000000000000000              __libc_start_main@GLIBC_2.34
0000000000000000  w      *UND*  0000000000000000              _ITM_deregisterTMCloneTable
0000000000404020  w      .data  0000000000000000              data_start
0000000000000000       F *UND*  0000000000000000              puts@GLIBC_2.2.5
0000000000404024 g       .data  0000000000000000              _edata
000000000040113c g     F .fini  0000000000000000              .hidden _fini
0000000000404020 g       .data  0000000000000000              __data_start
0000000000000000  w      *UND*  0000000000000000              __gmon_start__
0000000000402008 g     O .rodata        0000000000000000              .hidden __dso_handle
0000000000402000 g     O .rodata        0000000000000004              _IO_stdin_used
0000000000404028 g       .bss   0000000000000000              _end
0000000000401070 g     F .text  0000000000000005              .hidden _dl_relocate_static_pie
0000000000401040 g     F .text  0000000000000026              _start
0000000000404024 g       .bss   0000000000000000              __bss_start
0000000000401126 g     F .text  0000000000000015              main
0000000000404028 g     O .data  0000000000000000              .hidden __TMC_END__
0000000000000000  w      *UND*  0000000000000000              _ITM_registerTMCloneTable
0000000000401000 g     F .init  0000000000000000              .hidden _init
```

## EFF文件分析示例

假设一段代码test.c：

```c
#include <stdio.h>

unsigned char s_muse[] = {0x12, 0x34, 0x56, 0x78, 0x90};

int main() {
    int a = 10;
    printf("a = %d\n", a);
}
```

gcc -c test.c -o test.o

```shell
$ readelf -h test.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          720 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         13
  Section header string table index: 12
```

可以知道架构是X86-64、elf头64字节、节头每个节64字节、elf有13个section、secton name在第12节中，节头表偏移在 720(0x2D0)这个位置上，我们可以从0x2D0开始解析所有节，从0x2D0到结尾都是节头表，总共64*13=832字节，可以通过readelf -S test.o获取section信息

```
$ readelf -S test.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          720 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         13
  Section header string table index: 12
[root@localhost code]#
[root@localhost code]# readelf -S test.o
There are 13 section headers, starting at offset 0x2d0:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       000000000000002a  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000220
       0000000000000030  0000000000000018   I      10     1     8
  [ 3] .data             PROGBITS         0000000000000000  0000006a
       0000000000000005  0000000000000000  WA       0     0     1
  [ 4] .bss              NOBITS           0000000000000000  0000006f
       0000000000000000  0000000000000000  WA       0     0     1
  [ 5] .rodata           PROGBITS         0000000000000000  0000006f
       0000000000000006  0000000000000000   A       0     0     1
  [ 6] .comment          PROGBITS         0000000000000000  00000075
       000000000000002f  0000000000000001  MS       0     0     1
  [ 7] .note.GNU-stack   PROGBITS         0000000000000000  000000a4
       0000000000000000  0000000000000000           0     0     1
  [ 8] .eh_frame         PROGBITS         0000000000000000  000000a8
       0000000000000038  0000000000000000   A       0     0     8
  [ 9] .rela.eh_frame    RELA             0000000000000000  00000250
       0000000000000018  0000000000000018   I      10     8     8
  [10] .symtab           SYMTAB           0000000000000000  000000e0
       0000000000000120  0000000000000018          11     9     8
  [11] .strtab           STRTAB           0000000000000000  00000200
       000000000000001b  0000000000000000           0     0     1
  [12] .shstrtab         STRTAB           0000000000000000  00000268
       0000000000000061  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```

上图的所有信息其实都是在节头表每64个字节的节头表结构体中解析出来的，从节信息中又可以得到对应节的起始偏移、占用大小、一些flag等。

比如代码中定义了一个全局数组{0x12, 0x34, 0x56, 0x78, 0x90}，它是保存在.data节中

可以看到起始位置是0x6a（.data的Offset属性）、占用5个字节（.data的Size属性），首地址8字节对齐，使用16进制格式打开test.o：

![](assets/img-e-q.png)

可以看到0x6a开始的5个字节刚好就是数组数据了

ELF所有信息都可以通过偏移和结构体获得

