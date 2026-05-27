# Linux 内核中的 container_of 宏

### container_of 宏介绍

container_of宏在 Linux 内核中应用甚广：

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#define  container_of(ptr, type, member) ({    \
     const typeof( ((type *)0)->member ) *__mptr = (ptr); \
     (type *)( (char *)__mptr - offsetof(type, member) );})
```

这个宏到底是干什么的呢？它的主要作用就是：根据结构体某一成员的地址，获取这个结构体的首地址。根据宏定义，我们可以看到，这个宏有三个参数，它们分别是：

-   type：结构体类型
-   member：结构体内的成员
-   ptr：结构体内成员member的地址

也就是说，我们知道了一个结构体的类型，结构体内某一成员的地址，就可以直接获得到这个结构体的首地址。container_of 宏返回的就是这个结构体的首地址。

### container_of 宏使用示例

比如现在，我们定义一个结构体类型student：

```c
struct student {
    int age;
    int num;
    int math;
};

int main(void) {
    struct student stu;
    struct student *p;
    p = container_of(&stu.num, struct student, num);
    return 0;
}
```

在这个程序中，我们定义一个结构体类型 student，然后定义一个结构体变量 stu，我们现在已经知道了结构体成员变量 stu.num 的地址，那我们就可以通过 container_of 宏来获取结构体变量 stu 的首地址。

这个宏在内核中非常重要。我们知道，Linux 内核驱动中，为了抽象，对数据结构体进行了多次封装，往往一个结构体里面嵌套多层结构体。也就是说，内核驱动中不同层次的子系统或模块，使用的是不同封装程度的结构体，这也是 C 语言的面向对象思想。分层、抽象、封装，可以让我们的程序兼容性更好，适配更多的设备，但同时也增加了代码的复杂度。

我们在内核中，经常会遇到这种情况：我们传给某个函数的参数是某个结构体的成员变量，然后在这个函数中，可能还会用到此结构体的其它成员变量，那这个时候怎么办呢？container_of 就是干这个的，通过它，我们可以首先找到结构体的首地址，然后再通过结构体的成员访问就可以访问其它成员变量了。

```c
struct student {
    int age;
    int num;
    int math;
};

int main(void) {
    struct student stu = {20, 1001, 99};
    int *p = &stu.math;
    struct student *stup = NULL;
    stup = container_of(p, struct student, math);
    printf("%p\n", stup);
    printf("age: %d\n", stup->age);
    printf("num: %d\n", stup->num);
    return 0;
}
```

在这个程序中，我们定义一个结构体变量 stu，知道了它的成员变量 math 的地址 &stu.math，我们就可以通过 container_of 宏直接获得 stu 结构体变量的首地址，然后就可以直接访问 stu 结构体的其它成员 stup->age 和 stup->num。

## container_of 宏实现分析

知道了结构体成员的地址，如何去获取结构体的首地址？很简单，直接拿结构体成员的地址，减去该成员在结构体内的偏移，就可以得到该结构体的首地址了。

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#define container_of(ptr, type, member) ({    \
        const typeof( ((type *)0)->member ) *__mptr = (ptr); \
        (type *)( (char *)__mptr - offsetof(type, member) );})
```

从语法角度，我们可以看到，container_of 宏的实现由一个语句表达式构成。语句表达式的值即为最后一个表达式的值：

```c
(type *)((char *)__mptr - offsetof(type,member));
```

最后一句的意义就是，拿结构体某个成员 member 的地址，减去这个成员在结构体 type 中的偏移，结果就是结构体 type 的首地址。因为语句表达式的值等于最后一个表达式的值，所以这个结果也是整个语句表达式的值，container_of 最后就会返回这个地址值给宏的调用者。

那如何计算结构体某个成员在结构体内的偏移呢？内核中定义了 offset 宏来实现这个功能，我们且看它的定义：

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

这个宏有两个参数，一个是结构体类型 TYPE，一个是结构体的成员 MEMBER，它使用的技巧跟我们上面计算0地址常量指针的偏移是一样的：将0强制转换为一个指向 TYPE 的结构体常量指针，然后通过这个常量指针访问成员，获取成员 MEMBER 的地址，其大小在数值上就等于 MEMBER 在结构体 TYPE 中的偏移。

因为结构体的成员数据类型可以是任意数据类型，所以为了让这个宏兼容各种数据类型。我们定义了一个临时指针变量 __mptr，该变量用来存储结构体成员 MEMBER 的地址，即存储 ptr 的值。那如何获取 ptr 指针类型呢，通过下面的方式：

```c
typeof( ((type *)0)->member ) *__mptr = (ptr);
```

我们知道，宏的参数 ptr 代表的是一个结构体成员变量 MEMBER 的地址，所以 ptr 的类型是一个指向 MEMBER 数据类型的指针，当我们使用临时指针变量 `__mptr` 来存储 ptr 的值时，必须确保 `__mptr` 的指针类型是一个指向 MEMBER 类型的指针变量。`typeof( ((type *)0)->member )`表达式使用 typeof 关键字，用来获取结构体成员 member 的数据类型，然后使用该类型，使用 `typeof( ((type *)0)->member ) *__mptr` 这行程序语句，就可以定义一个指向该类型的指针变量了。

还有一个需要注意的细节就是：在语句表达式的最后，因为返回的是结构体的首地址，所以数据类型还必须强制转换一下，转换为 TYPE，即返回一个指向 TYPE 结构体类型的指针，所以你会在最后一个表达之中看到一个强制类型转换(TYPE )。

