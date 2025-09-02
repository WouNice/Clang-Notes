# ELF文件格式

ELF（Executable and Linkable Format）文件是一种用于二进制文件、可执行文件、目标代码、共享库和core转存的格式文件，是UNIX系统实验室作为应用程序二进制接口（ABI）而开发和发布的，也是Linux的主要可执行文件格式。

-   可执行文件（.out）：包含可执行的机器代码和数据，可直接运行，其代码和数据都有固定的地址或基地址偏移，系统可以根据这些地址加载程序

-   可重定位文件（.o）：机器代码和数据地址相对，需重定位才能运行，通常用于编译过程。
-   共享目标文件（.so）：动态链接库，包含可共享代码和数据，可在运行时被多个进程共享。数据是在链接时被链接器（ld）和运行时动态链接器（ld.so.1、libc.so.1、ld-linux.so.1）使用
-   内核转储（core dump）：存放当前进程的执行上下文；程序崩溃或异常终止时生成，包含内存状态和寄存器信息，用于调试。

## ELF文件格式视图

ELF文件格式提供了2种视图：链接视图和执行视图

链接视图就是在链接时用到的视图，而执行视图则是在执行时用到的视图，链接视图以节（section）为单位，执行视图以段（segment）为单位。

## ELF构成

ELF文件由以下部分组成：

-   ELF头（ELF Header）：
    -   定义全局属性信息，比如幻数、目标体系结构、节头表地址偏移等；
    -   描述体系结构和操作系统等基本信息，指出section header table和program header table在文件的位置
-   程序头表（Program Header Table）：从运行的角度来看ELF文件的，主要给出了各个segment的信息和它们的属性，在汇编和链接过程中没用。程序表头需要加载器将文件中的节接在到虚拟内存中，对于可链接文件来说，程序表头可能为空；
-   节区（Section Table）：ELF文件中的数据和代码以节区的形式存储，不同类型的节区包含了不同的信息，如代码区、数据区、符号表区等；
-   节头表（Section Header Table）：包含对节（section）的描述，记录了ELF文件中各个节的起始偏移、大小、标志等信息；保存了所有的section的信息，这是从编译和链接的角度来看ELF文件的

实际上，一个文件中不一定包含全部内容，而且它们的位置也未必如同所示这样安排，只有ELF头的位置是固定的，其余各部分的位置、大小等信息由ELF头中的各项值来决定。

>   注意：.o文件没有程序头表，ELF文件并不一定有程序头表。

ELF文件格式如下图，位于ELF Header和Section Header Table 之间的都是段（Section）。一个典型的ELF文件包含下面几个段：

![](assets/img-w-q.png)

-   `text`：已编译程序的指令代码段。
-   `rodata`：ro 代表 read only，即只读数据（譬如常数 const）。
-   `data`：已初始化的 C 程序全局变量和静态局部变量。
-   `bss`：未初始化的 C 程序全局变量和静态局部变量。
-   `debug`：调试符号表，调试器用此段的信息帮助调试。

>   段（Segment）和节（Section）有什么区别？
>
>   节是ELF文件的基本单位，包含了程序的代码，数据等信息。Linu系统为了高效加载ELF文件，将多个节划分为一个组（段），段是节的集合，Linux系统通过段加载代码（节）和数据（节）。

### ELF头

描述ELF文件的主要特性（`/usr/include/elf.h`）

ELF头是整个ELF文件的起始部分，位置固定，包含了识别和解释文件内容的关键信息。

Linux的文件头定义了两种标准：一种是32位机器的，一种是64位机器的。它们的内容结构基本上是一样的，只是存储宽度不一样而已。

```c
#define EI_NIDENT 16
typedef struct {
    /*ELF的一些标识信息，固定值*/
    unsigned char e_ident[EI_NIDENT];
    /*目标文件类型：1-可重定位文件，2-可执行文件，3-共享目标文件等*/
    Elf32_Half e_type;
    /*文件的目标体系结构类型：3-intel 80386*/
    Elf32_Half e_machine;
    /*目标文件版本：1-当前版本*/
    Elf32_Word e_version;
    /*程序入口的虚拟地址，如果没有入口，可为0*/
    Elf32_Addr e_entry;
    /*程序头表(segment header table)的偏移量，如果没有，可为0*/
    Elf32_Off e_phoff;
    /*节区头表(section header table)的偏移量，没有可为0*/
    Elf32_Off e_shoff;
    /*与文件相关的，特定于处理器的标志*/
    Elf32_Word e_flags;
    /*ELF头部的大小，单位字节*/
    Elf32_Half e_ehsize;
    /*程序头表每个表项的大小，单位字节*/
    Elf32_Half e_phentsize;
    /*程序头表表项的个数*/
    Elf32_Half e_phnum;
    /*节区头表每个表项的大小，单位字节*/
    Elf32_Half e_shentsize;
    /*节区头表表项的数目*/
    Elf32_Half e_shnum;
    /*某些节区中包含固定大小的项目，如符号表。对于这类节区，此成员给出每个表项的长度字节数。*/
    Elf32_Half e_shstrndx;
} Elf32_Ehdr;

typedef struct {
    unsigned char e_ident[EI_NIDENT];
    Elf64_Half e_type;
    Elf64_Half e_machine;
    Elf64_Word e_version;
    Elf64_Addr e_entry;
    Elf64_Off e_phoff;
    Elf64_Off e_shoff;
    Elf64_Word e_flags;
    Elf64_Half e_ehsize;
    Elf64_Half e_phentsize;
    Elf64_Half e_phnum;
    Elf64_Half e_shentsize;
    Elf64_Half e_shnum;
    Elf64_Half e_shstrndx;
} Elf64_Ehdr;
```

查看ELF文件头信息：

```shell
# read -h ELF文件
```

ELF文件头字段解析：

-   Magic：魔数，ELF文件识别信息。
-   Class：文件类型，32位或64位。
-   Data：编码方式，大端或小端。
-   Version：ELF版本。
-   OS/ABI：操作系统信息。
-   ABI：ABI版本。
-   Type：文件类型：可重定位文件（REL）、可执行文件（DYN）、共享对象文件（DYN），核心转储文件（CORE）。
-   Machine：机器类型，表示ELF文件的平台属性，如x86、AArch64等。
-   Entry point address：程序入口地址。
-   Start of program headers: 程序头表偏移量。
-   Start of section headers: 节头表偏移量。
-   Flags：标志。
-   Size of this header：ELF头大小。
-   Size of program headers：程序头表条目大小。
-   Number of program header：程序头表有多少条目。
-   Size of section headers：节头表条目大小。
-   Number of section headers：节头表有多少条目。

### 程序表头

程序头表（Program Header Table）用于描述文件中的段（Segment）信息，指导操作系统加载程序。

-   程序头表由多个程序头组成，每个头对应一个段，包含该段的详细信息：段的类型、在文件中的偏移地址、映射到内存的虚拟地址、大小及权限等。
-   一个段可以包含一个或多个节，但是不同的段可能会有重合，即一个节在不同段里
-   程序头表是一个结构体数组，每一个元素类型都是Elf64_Phdr（64位编译器）、Elf32_Phdr（32位编译器）描述了一个段或者其他系统在准备程序执行时所需要的信息，其中 ELF 头中的 e_phentsize 和 e_phnum 指定了该数组每个元素的大小以及元素个数，一个目标文件的段包含一个或者多个节。可以说程序表头就是专门为ELF文件运行时中的段所准备的

```c
typedef struct
{
  Elf64_Word    p_type;            /* Segment type */  该字段为段的类型
  Elf64_Word    p_flags;        /* Segment flags  该字段给出了与段相关的标记*/
  Elf64_Off    p_offset;        /* Segment file offset 该字段给出了从文件开始到该段开头的第一个字节的偏移*/
  Elf64_Addr    p_vaddr;        /* Segment virtual address 该字段给出了该段第一个字节在内存中的虚拟地址*/
  Elf64_Addr    p_paddr;        /* Segment physical address 该字段仅用于物理地址寻址相关的系统中*/
  Elf64_Xword    p_filesz;        /* Segment size in file  该字段给出了文件镜像中该段的大小，可能为 0*/
  Elf64_Xword    p_memsz;        /* Segment size in memory 该字段给出了内存镜像中该段的大小，可能为 0*/
  Elf64_Xword    p_align;        /* Segment alignment 可加载的程序的段的 p_vaddr 以及 p_offset 的大小必须是 page 的整数倍*/
} Elf64_Phdr;
```

查看程序头表信息：

```v
# readelf -l ELF文件
```

程序头表字段解析：

-   TYPE：段类型，常见类型如下：
    -   PT_PHDR：程序头表本身在文件中的位置和大小。
    -   PT_LOAD：表示一个可加载的段，这种段包含了程序的实际代码和数据，需要被加载到内存中以便执行。
    -   PT_DYNAMIC：指向动态链接信息，包含了动态链接器所需的各种表和字符串，如动态库路径、依赖库列表、符号表等。
    -   PT_INTERP：指定了程序解释器的路径，这通常是动态链接器的路径。
    -   PT_NOTE：附加信息。
-   Offset：文件偏移，段在文件中的偏移量。
-   VirtAddr：虚拟地址，段加载至内存后的虚拟地址。
-   PhysAddr：物理地址：段的物理地址。
-   FileSiz：文件大小，段在文件中大小。
-   MemSiz：内存大小，段在内存中大小。
-   Flags：段标识，段属性：只读属性（R），只写属性（W），可执行属性（E）。
-   Align：对齐方式。

### 节头表

节头表（Section Header Table），用于描述文件中各个节（Section）的属性和信息。

每个节都有一个对应的节表头，它包含了节的名称、类型、大小、偏移量等关键数据，为链接器和加载器提供必要的信息。

节头表位于ELF文件的尾部，具体位置在ELF头中的e_shoff项给出了偏移，e_shnum告诉我们节头表中包含的节数，e_shentsize给出了每一节的字节大小（比如64位系统就是64字节）

```c
typedef struct {
    Elf64_Word    sh_name;
    Elf64_Word    sh_type;
    Elf64_Xword    sh_flags;
    Elf64_Addr    sh_addr;
    Elf64_Off    sh_offset;
    Elf64_Xword    sh_size;
    Elf64_Word    sh_link;
    Elf64_Word    sh_info;
    Elf64_Xword    sh_addralign;
    Elf64_Xword    sh_entsize;
} Elf64_Shdr;
```

| 成员         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| sh_name      | 节名称，是节区头字符串表节区中（Section Header String Table Section）的索引，因此该字段实际是一个数值。在字符串表中的具体内容是以 NULL 结尾的字符串。 |
| sh_type      | 根据节的内容和语义进行分类                                   |
| sh_flags     | 每一比特代表不同的标志，描述节是否可写、可执行，需要分配内存等属性。 |
| sh_addr      | 如果节区将出现在进程的内存映像中，此成员给出节区的第一个字节应该在进程镜像中的位置。否则，此字段为 0 |
| sh_offset    | 给出节区的第一个字节与文件开始处之间的偏移。SHT_NOBITS 类型的节区不占用文件的空间，因此其 sh_offset 成员给出的是概念性的偏移。 |
| sh_size      | 此成员给出节区的字节大小。除非节区的类型是 SHT_NOBITS ，否则该节占用文件中的 sh_size 字节。类型为 SHT_NOBITS 的节区长度可能非零，不过却不占用文件中的空间。 |
| sh_link      | 此成员给出节区头部表索引链接，其具体的解释依赖于节区类型     |
| sh_info      | 此成员给出附加信息，其解释依赖于节区类型。                   |
| sh_addralign | 某些节区的地址需要对齐。例如，如果一个节区有一个 doubleword 类型的变量，那么系统必须保证整个节区按双字对齐。也就是说sh_addr%sh_addralign = 0。 目前它仅允许为 0，以及 2 的正整数幂数。 0 和 1 表示没有对齐约束 |
| sh_entsize   | 某些节区中存在具有固定大小的表项的表，如符号表。对于这类节区，该成员给出每个表项的字节大小。反之，此成员取值为 0 |

查看节头表信息：

```v
# readelf -s ELF文件
```

节头表字段解析：

-   Name：节名。
-   Type：节类型
-   Address：虚拟地址。
-   Offset：文件偏移量。
-   Size：节大小，节在文件中的大小。
-   EntSize：节条目大小。
-   Flags：节属性：只读属性（R），只写属性（W），可执行属性（E）。
-   Link：链接，表示与该节相关联的符号表或者字符串表。
-   Info：节信息。
-   Align：对齐方式。

### 节区

一些系统预定的固定section

| sh_name   | sh_type      | 说明                                                |
| --------- | ------------ | --------------------------------------------------- |
| .text     | SHT_PROGBITS | 代码段，包含程序的可执行指令                        |
| .data     | SHT_PROGBITS | 包含初始化的数据，将出现在程序的内存映像中          |
| .bss      | SHT_NOBITS   | 未初始化数据，因为只有符号                          |
| .rodata   | SHT_PROGBITS | 包含只读数据                                        |
| .comment  | SHT_PROGBITS | 包含版本控制信息                                    |
| .dynsym   | SHT_DYNSYM   | 此节区包含了动态链接符号表                          |
| .shstrtab | SHT_STRTAB   | 存放section名，字符串表.Section Header String Table |
| .strtab   | SHT_STRTAB   | 字符串表                                            |
| .symtab   | SHT_SYMTAB   | 符号表                                              |

通过查看ELF文件符号表，可以协助我们排查程序bug：

```v
# readelf -s ELF文件
```
