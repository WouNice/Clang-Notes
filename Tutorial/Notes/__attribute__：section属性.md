# `__attribute__`：section属性

GNU C 增加一个 `__atttribute__` 关键字用来声明一个函数、变量或类型的特殊属性。声明这个特殊属性有什么用呢？主要用途就是指导编译器在编译程序时进行特定方面的优化或代码检查。比如，我们可以通过使用属性声明指定某个变量的数据边界对齐方式。

`__atttribute__` 的使用非常简单，当我们定义一个函数、变量或类型时，直接在它们名字旁边添加下面的属性声明即可：

```
__atttribute__((ATTRIBUTE))
```

这里需要注意的是：`__atttribute__` 后面是两对小括号，不能图方便只写一对，否则编译可能通不过。括号里面的 ATTRIBUTE 代表的就是要声明的属性。现在 `__atttribute__` 支持十几种属性：

-   section
-   aligned
-   packed
-   format
-   weak
-   alias
-   noinline
-   always_inline
-   ……

在这些属性中，aligned 和 packed 用来显式指定一个变量的存储边界对齐方式。一般来讲，我们定义一个变量，编译器会根据变量类型，按照默认的规则来给这个变量分配大小、按照默认的边界对齐方式分配一个地址。而使用 `__atttribute__` 这个属性声明，就相当于告诉编译器：按照我们指定的边界地址对齐去给这个变量分配存储空间。

```c
char c2 __attribute__((aligned(8)) = 4;
int global_val __attribute__((section(".data")));
```

有些属性可能还有自己的参数。比如 aligned(8) 表示这个变量按8字节地址对齐，参数也要使用小括号括起来。如果属性的参数是一个字符串，小括号里的参数还要用双引号引起来。

当然，我们也可以对一个变量同时添加多个属性说明。在定义时，各个属性之间用逗号隔开就可以了。

```c
char c2 __attribute__((packed,aligned(4)));
char c2 __attribute__((packed,aligned(4))) = 4;
__attribute__((packed,aligned(4))) char c2 = 4;
```

在上面的示例中，我们对一个变量添加2个属性声明，这两个属性都放在 `__atttribute__` (()) 的2对小括号里面，属性之间用逗号隔开。这里还有一个细节，就是属性声明要紧挨着变量，上面的三种定义方式都是没有问题的，但下面的定义方式在编译的时候可能就通不过。

```c
char c2 = 4 __attribute__((packed,aligned(4)));
```

## 属性声明：section

使用 `__atttribute__` 来声明一个 section 属性，主要用途是在程序编译时，将一个函数或变量放到指定的段，即 section 中。

## `U-boot` 启动过程中的镜像自拷贝分析

有了 section 这个属性，我们接下来就可以试着分析，`U-boot` 在启动过程中，是如何将自身代码加载的 RAM 中的。

搞嵌入式的都知道 `U-boot`，`U-boot` 的用途主要是加载 Linux 内核镜像到内存、给内核传递启动参数、然后引导 Linux 操作系统启动。

`U-boot` 一般存储在 Nor flash 或 NAND Flash 上。无论从 Nor Flash 还是从 Nand Flash 启动，`U-boot` 其本身在启动过程中，也会从 Flash 存储介质上加载自身代码到内存，然后进行重定位，跳到内存 RAM 中去执行。这个功能一般叫做“自举”，是不是感觉很牛 X？`U-boot` 是怎么完成自拷贝的，或者说它是怎样将自身代码从 Flash 拷贝到内存 RAM 中的。

在拷贝自身代码的过程中，一个主要的疑问就是，`U-boot` 是如何识别自身代码的？是如何知道从哪里拷贝代码的？是如何知道拷贝到哪里停止的？这个时候我们不得不说起 `U-boot` 源码中的一个零长度数组。

```c
char __image_copy_start[0] __attribute__((section(".__image_copy_start")));
char __image_copy_end[0] __attribute__((section(".__image_copy_end")));
```

这两行代码定义在 `U-boot-2016.09` 中的`arch/arm/lib/section.c`文件中。在其它版本中可能路径不同或者没有定义，为了分析这个功能，建议大家可以下载`U-boot-2016.09`这个版本的`U-boot`源码。

这两行代码的作用是分别定义一个零长度数组，并告诉编译器要分别放在 `.image_copy_start` 和 `.image_copy_end` 这两个 section 中。

链接器在链接各个目标文件时，会按照链接脚本里各个 section 的排列顺序，将各个 section 组装成一个可执行文件。`U-boot` 的链接脚本 u-boot.lds 在 `U-boot` 源码的根目录下面。

```v
OUTPUT_FORMAT("elf32-littlearm",
    "elf32-littlearm",
    "elf32-littlearm")
OUTPUT_ARCH(arm)
ENTRY(_start)
SECTIONS
{
 . = 0x00000000;
 . = ALIGN(4);
 .text :
 {
  *(.__image_copy_start)
  *(.vectors)
  arch/arm/cpu/armv7/start.o (.text*)
  *(.text*)
 }
 . = ALIGN(4);
 .data : {
  *(.data*)
 }
    ...
    ...
 . = ALIGN(4);
 .image_copy_end :
 {
  *(.__image_copy_end)
 }
 .end :
 {
  *(.__end)
 }
 _image_binary_end = .;
 . = ALIGN(4096);
 .mmutable : {
  *(.mmutable)
 }
 .bss_start __rel_dyn_start (OVERLAY) : {
  KEEP(*(.__bss_start));
  __bss_base = .;
 }
 .bss __bss_base (OVERLAY) : {
  *(.bss*)
   . = ALIGN(4);
   __bss_limit = .;
 }
 .bss_end __bss_limit (OVERLAY) : {
  KEEP(*(.__bss_end));
 }
}
```

通过链接脚本我们可以看到，`__image_copy_start` 和 `__image_copy_end` 这两个 section，在链接的时候分别放在了代码段 `.text` 的前面、数据段 `.data` 的后面，作为 `U-boot`拷贝自身代码的起始地址和结束地址。而在这两个 section 中，我们除了放2个零长度数组外，并没有再放其它变量。根据前面的学习我们知道，零长度数组是不占用存储空间的，所以上面定义的两个零长度数组，其实就分别代表了 `U-boot` 镜像要拷贝自身镜像的起始地址和结束地址。

无论 `U-boot` 自身镜像是存储在 Nor Flash，还是 Nand Flash 上，我们只要知道了这两个地址，就可以直接调用相关代码拷贝。

接着在 arch/arm/lib/relocate.S 中，ENTRY(relocate_code) 汇编代码主要完成代码拷贝的功能。

```c
ENTRY(relocate_code)
    ldr r1, =__image_copy_start /* r1 <- SRC &__image_copy_start */
    subs    r4, r0, r1      /* r4 <- relocation offset */
    beq relocate_done       /* skip relocation */
    ldr r2, =__image_copy_end   /* r2 <- SRC &__image_copy_end */
copy_loop:
    ldmia   r1!, {r10-r11}      /* copy from source address [r1]    */
    stmia   r0!, {r10-r11}      /* copy to   target address [r0]    */
    cmp r1, r2          /* until source end address [r2]    */
    blo copy_loop
```

在这段汇编代码中，寄存器 R1、R2 分别表示要拷贝镜像的起始地址和结束地址，R0 表示要拷贝到 RAM 中的地址，R4 存放的是源地址和目的地址之间的偏移，在后面重定位过程中会用到这个偏移值。

```c
ldr r1, =__image_copy_start
```

见上面指令，在汇编代码中，ARM的 ldr 指令立即寻址，直接对数组名进行引用，获取要拷贝镜像的首地址，并保存在 R1 寄存器中。数组名本身其实就代表一个地址。通过这种方式，`U-boot` 在嵌入式启动的初始阶段，就完成了自身代码的拷贝工作：从 Flash 上拷贝自身镜像到 RAM 中，然后再进行重定位，最后跳到 RAM 中执行。
