# typeof 关键字

ANSI C 定义了 sizeof 关键字，用来获取一个变量或数据类型在内存中所占的存储字节数。GNU C 扩展了一个关键字 typeof，用来获取一个变量或表达式的类型。

通过使用 typeof，我们可以获取一个变量或表达式的类型。所以 typeof 的参数有两种形式：表达式或类型。

```
int i ;
typeof(i) j = 20;

typeof(int *) a;

int f();
typeof(f()) k;
```

在上面的代码中，因为变量 i 的类型为 int，所以 typeof(i) 就等于 int，typeof(i) j =20 就相当于 int j = 20，`typeof(int *) a`; 相当于 `int* a;`，`f()` 函数的返回值类型是`int`，所以 `typeof(f()) k`; 就相当于 `int k;`。

## typeof 使用示例

根据上面 typeof 的用法，我们编写一个程序，来学习一下 typeof 的使用。

```c
int main(void) {
    int i = 2;
    typeof(i) k = 6;
    int *p = &k;
    typeof(p) q = &i;
    printf("k = %d\n", k);
    printf("*p= %d\n", *p);
    printf("i = %d\n", i);
    printf("*q= %d\n", *q);
    return 0;
}
```

运行结果为：

```
k  = 6
*p = 6
i  = 2
*q = 2
```

通过运行结果可知，通过 typeof 获取一个变量的类型 int 后，可以使用该类型再定义一个变量。这跟我们直接使用 int 定义一个变量，效果是一样的。

## typeof 的其它使用方法

除了使用 typeof 获取基本数据类型，还有其它一些高级的用法：

```
typeof (int *) y;     // 把 y 定义为指向 int 类型的指针，相当于int *y;
typeof (int)  *y;     //定义一个执行 int 类型的指针变量 y
typeof (*x) y;        //定义一个指针 x 所指向类型 的指针变量y
typeof (int) y[4];    //相当于定义一个：int y[4]
typeof (*x) y[4];     //把 y 定义为指针 x 指向的数据类型的数组
typeof (typeof (char *)[4]) y;//相当于定义字符指针数组：char *y[4];
typeof(int x[4]) y;  //相当于定义：int y[4]
```

## typeof 在内核中的应用

关键字 typeof 在 Linux 内核中被广泛使用，主要用在宏定义中，用来获取宏参数类型。比如内核中，min/max 宏的定义：

```
#define min(x, y) ({                \
    typeof(x) _min1 = (x);          \
    typeof(y) _min2 = (y);          \
    (void) (&_min1 == &_min2);      \
    _min1 < _min2 ? _min1 : _min2; })
#define max(x, y) ({                \
    typeof(x) _max1 = (x);          \
    typeof(y) _max2 = (y);          \
    (void) (&_max1 == &_max2);      \
    _max1 > _max2 ? _max1 : _max2; })
```

## 