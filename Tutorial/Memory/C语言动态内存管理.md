# C语言动态内存管理

在C语言编程中，内存管理是项核心技能，直接关系到程序的性能和稳定性。动态内存管理作为C语言内存管理的重要组成部分，允许程序在运行时根据需要分配和释放内存，而不是在编译时确定内存布局。这对于处理不确定大小的数据结构（如字符串、列表）以及实现高效内存利用的应用至关重要。

## 内存管理概述

在深入动态内存管理之前，我们首先需要了解C语言中的内存分配类型。

### 内存分配类型

C语言程序使用三种主要的内存分配方式：

-   静态内存分配：由编译器在编译时分配内存，通常用于全局变量和静态变量。这种分配方式在程序运行期间保持不变。
-   栈内存分配：在运行时由编译器自动管理，用于局部变量。当函数被调用时，栈为局部变量分配内存；当函数返回时，这些内存自动释放。
-   堆内存分配：即动态内存分配，由程序员在运行时通过显式调用内存管理函数来控制内存的分配和释放。这种灵活性使得程序能够根据实际需求管理内存。

### 动态内存管理的重要性

动态内存管理在以下场景中特别重要：

-   当程序需要处理的数据量在编译时未知
-   当程序需要根据运行时条件分配资源
-   当需要优化内存使用，避免预分配大量可能不会使用的内存
-   在处理复杂数据结构（如链表、树）时

### 标准库中的动态内存管理函数

C语言标准库提供了几个用于动态内存管理的函数，这些函数在<stdlib.h>头文件中声明。下面我们详细介绍这些函数。

#### malloc函数

malloc 是最基本的内存分配函数，用于从堆中分配指定大小的内存。函数原型：

```c
void *malloc(size_t size);
```

参数：

-   size：要分配的内存字节数返回值：

-   成功时，返回指向分配内存的指针

-   失败时，返回NULL

使用示例：

```c
int *array = malloc(5 * sizeof(int)); //分配5个整数的空司
if (array == NULL) {
    // 内存分配失败，进行错误处理
    fprintf(stderr, "内存分配失败\n");
    exit(EXIT_FAILURE);
}
```

注意事项：

-   用于分配的内存内容是未初始化的，包含不可预测的垃圾值
-   用于需要使用sizeof运算符来计算正确的内存大小
-   用于必须检查malloc的返回值，避免悬空指针

#### calloc函数

calloc函数用千分配多个元素的内存，并将所有位初始化为零。

函数原型：

```c
void *calloc(size_t nmemb, size_t size);
```

参数：

-   nmemb：要分配的元素数量
-   size：每个元素的大小（以字节为单位）

返回值：

-   用于成功时，返回指向分配内存的指针
-   用于失败时，返回NULL

使用示例：

```c
double *array = calloc(10, sizeof(double));
if (array == NULL) {
    // 内存分配失败，进行错误处理
    fprintf(stderr, "内存分配失败\n");
    exit(EXIT_FAILURE);
}
```

注意事项：

-   calloc分配的内存自动初始化为零，而malloc不进行初始化
-   内存总大小由nmemb * size计算得出
-   同样需要检查返回值

#### realloc函数

realloc函数用于调整之前分配的内存块的大小。函数原型：

```c
void *realloc(void *ptr, size_t size);
```

参数：

-   用于ptr：指向先前分配的内存块的指针
-   用于size：新的内存大小（以字节为单位）返回值：
-   用于成功时，返回指向新分配内存的指针（可能与原指针相同或不同）
-   用于失败时，返回NULL, 原内存保持不变

使用示例：

```c
char *buffer = malloc(100); //分配100字节
//后来发现需要更多空间
buffer = realloc(buffer, 200); //尝试将内存扩展到2的字节
if (buffer == NULL) {
    // 重新分配失败，进行错误处理
    fprintf(stderr, "内存分配失败\n");
    exit(EXIT_FAILURE);
}
```

注意事项：

-   用于如果新的大小大于原来的大小，realloc会尝试扩展内存块
-   用于如果新的大小小于原来的大小，realloc会尝试收缩内存块
-   用于如果无法按原地扩展内存，realloc可能会在新位置分配内存，并将旧内存的内容复制到新位置，然后释放旧内存
-   用于如果ptr是NULL, realloc的行为类似于malloc
-   用于如果size是0, realloc的行为类似于free
-   由于返回的指针可能与原来的指针不同，必须将返回值赋值给指针变量
-   在realloc后，如果失败，原始指针仍然有效，内存大小保持不变

#### free函数

free函数用于将之前分配的内存块返回给堆，供未来分配使用。函数原型：

```c
void free(void *ptr);
```

参数：

-   ptr：指向要释放的内存块的指针

使用示例：

```c
int *array = malloc(S * sizeof(int));
// 使用array...
free(array); //释放内存
array = NULL; //避免悬空指针
```

注意事项：

-   用于释放内存后，指针变为悬空指针，不能再通过该指针访问内存，否则会导致未定义行为
-   用于释放已经释放过的内存会导致未定义行为
-   用于释放未分配的内存（例如，指向堆外的指针）也会导致未定义行为
-   用于好的实践是在释放内存后将指针设置为NULL, 以避免误用已释放的内存

## 动态内存管理的实现原理

为了更好地理解这些函数的工作原理，我们需要了解它们在底层的实现机制。

### 堆内存管理

堆是操作系统为进程提供的—个内存区域，用千动态内存分配。当程序调用malloc等函数时，实际上是在请求操作系统从堆中分配内存。堆内存管理通常涉及以下步骤：

1.  内存池：操作系统维护一个可用内存池，当进程请求内存时，从该池中分配。
2.  内存块管理：每个内存块包含以下信息：
    -   大小：内存块的大小
    -   状态：表示内存块是已分配还是未分配
    -   用户数据：应用程序存储的实际数据
3.  分配策略：操作系统使用各种算法来管理内存分配，常见的策略包括：
    -   首次适应(First Fit) ：从内存池的开头开始查找第一个足够大的空闲块
    -   最佳适应(Best Fit ) ：找到最适合请求大小的空闲块
    -   最差适应(Worst Fit) ：找到最大的空闲块

4.  内存碎片：当内存池中存在多个小的空闲块，无法满足较大的内存请求时，就会出现内存碎片问题。

### 分配过程详解

当调用malloc函数时，内存分配过程通常包括以下步骤：

1.  检查缓存：许多C运行时库在操作系统提供的堆之上实现了自己的内存分配器，这些分配器通常维护一个空闲内存块的缓存。
2.  确定内存块大小：分配器需要确定要分配的内存块的大小，这可能大于请求的大小，以包括必要的元数据。
3.  分割内存块：如果可用内存块大于请求的大小，分配器可能会将该块分割成一个满足清求的块和一个剩余的空闲块。
4.  更新数据结构：分配器维护数据结构来跟踪哪些内存块已分配，哪些未分配。

### 释放过程详解

当调用free函数时，内存释放过程通常包括以下步骤：

1.  验证指针：分配器验证要释放的指针是否有效，即它是否指向之前分配的内存块。
2.  合井空闲块：如果释放的内存块相邻的内存块也是空闲的，分配器可能会将它们合并成一个更大的空闲块。
3.  更新数据结构：分配器更新其数据结构，标记该内存块为可用。

## 动态内存管理中的常见陷阱

动态内存管理虽然强大，但也容易出错。以下是开发者在使用动态内存管理时常见的陷阱：

### 内存泄漏

内存泄漏是指程序分配了内存但不再使用它，却没有释放它。随着时间的推移，内存泄漏会导致程序使用越来越多的内存，最终可能导致程序变漫甚至崩溃。示例：

```c
void leak_memory() {
    int *ptr = malloc(100); //分配内存
    // 使用ptr...
    // 忘记释放内存
}

int main() {
    leak_memory();
    // ptr指向的内存仍然存在，庙ain函数无法访问它
    return 0;
}
```

预防措施：

-   使用free及时释放不再使用的内存
-   使用指针跟踪分配的内存，确保每个malloc都有对应的free
-   在错误处理路径中也释放内存
-   考虑使用RAII(Resource Acquisition Is Initialization)技术，在对象生命周期结束时自动释放资源

### 悬空指针

悬空指针是指向已释放内存的指针。通过悬空指针访问内存会导致未定义行为，可能引起程序 崩溃或不正确的结果。示例：

```c
int *dangerous() {
    int *ptr = malloc(100); //分配内存
    // 使用ptr...
    free(ptr);//释放内存
    return ptr; //返回指向已坚放内存的指针
}

int main() {
    int *ptr = dangerous();
    // 访问ptr会导致未定义行为
    return 0;
}
```

预防措施：

-   在释放内存后将指针设置为NULL
-   使用智能指针或封装内存管理的类来自动管理内存生命周期
-   在使用指针之前检查它是否为NULL

### 重复释放

重复释放同一块内存会导致未定义行为，可能引起程序崩溃或内存损坏。示例：

```c
void double_free() {
    int *ptr = malloc(100); //分配内存
    // 使用ptr...
    free(ptr); //释放内存
    free(ptr); //重凄释放同一内存
}
```

预防措施：

-   在释放内存后将指针设置为NULL
-   使用引用计数来跟踪有多少指针指向同
-   确保每个内存块只释放一次

### 访问越界内存

访问越界内存是指访问内存块之外的内存。这通常发生在数组索引超出范围或指针偏移量过大时。示例：

```c
void out_of_bounds() {
    int *array = malloc(5 * sizeof(int)); //分配5个整数的空间
    // 使用数组．．
    array[5] = 42;
    // ．．．
    free(array);
}
```

 预防措施：

-   跟踪内存块的大小
-   在访问内存之前检查索引是否在有效范围内
-   使用边界检查来验证内存访问

### 不检查malloc的返回值

如果malloc无法分配请求的内存，它将返回NULL。如果不检查返回值，程序可能会尝试使用NULL指针，导致未定义行为。示例：

```c
void no_check() {
    int *ptr = malloc(1000000000); //尝试分配大量内存
    // 不检查malloc的返回值
    *ptr = 42; //如果malloc返NULL,这将导致未定义行为
}
```

 预防措施：

-   始终检查malloc、calloc和realloc的返回值
-   在内存分配失败时提供适当的错误处理

### 内存碎片

内存碎片是指当内存池中存在多个小的空闲块，无法满足较大的内存请求时的清况。这会导致内存使用效率低下，甚至可能导致内存不足错误，尽管总可用内存足够。示例：

```c
void memory_fragmentation() {
    while (1) {
        int *ptr1 = malloc(100); //分配小块内存
        int *ptr2 = malloc(100000); //分配大块内存
        free(ptr2); //释放大块内存但可能无法与后续的大内存请求合并
        // 重复这个过程, 会导致内存碎片
    }
}
```

预防措施：

-   尽量分配和释放内存块的顺序一致
-   尽量减少频繁分配和释放小内存块
-   考虑使用内存池技术来减少内存碎片

## 动态内存管理的最佳实践

为了编写高效、安全的动态内存管理代码，以下是一些最佳实践：

### 使用一致的内存管理策略

在代码中使用一致的内存管理策略，这样可以减少错误并使代码更易于维护。示例：

```c
// 策略：所有内存分配都使用malloc，释放都使用free
// 确保每个malloc都有对应的free
// 在错误处理路径中也释放内存
int main() {
    int *array = malloc(5 * sizeof(int));
    if (array == NULL) {
        // 处理错淉
        return EXIT_FAILURE;
    }
    // 使用 array...
    free(array);
    return 0;
}
```

### 使用智能指针或封装类

C语言没有内置的智能指针，但可以使用一些技术来模拟它们，例如封装内存管理的结构或函数。示例：

```c
#include <malloc.h>

typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

DynamicArray *da_create() {
    DynamicArray *da = malloc(sizeof(DynamicArray));
    if (da == NULL) {
        return NULL;
    }
    da->size = 0;
    da->capacity = 1;
    da->data = malloc(da->capacity * sizeof(int));
    if (da->data == NULL) {
        free(da);
        return NULL;
    }
    return da;
}

void da_destroy(DynamicArray *da) {
    if (da != NULL) {
        free(da->data);
        free(da);
    }
}

// 使用封装的DynamicArray结构
int main() {
    DynamicArray *da = da_create();
    if (da == NULL) { // 处理错误
        return EXIT_FAILURE;
    }
    // 使用da...
    da_destroy(da);
    return 0;
}
```

### 使用边界检查和错误处理

在访问内存之前进行边界检查，以防止越界访问。示例：

```c
#include <malloc.h>

void safe_access(int *array, size_t size, size_t index, int value) {
    if (array == NULL || index >= size) {
        // 处理错误
        return;
    }
    array[index] = value;
}

int main() {
    int *array = malloc(5 * sizeof(int));
    if (array == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    safe_access(array, 5, 5, 42); // index >= size，会返回i而不修改数组
    free(array);
    return 0;
}
```

### 使用valgrind等工具检测内存错误

Valgrind是一个内存调试工具，可以帮助检测内存泄漏、悬空指针、越界访问等问题。使用示例：

```c
# 编译程序
gcc -g -o my_program my_program.c
# 使用VaLgrind运行程序
valgrind --leak-check=full ./my_program
```

### 使用内存池技术

内存池技术是将多个小内存分配合并为一个大内存分配，以减少内存碎片和提高性能。示例：

```c
#include <stdint.h>
#include <malloc.h>

typedef struct {
    void *buffer;
    size_t buffer_size;
    size_t used_size;
} MemoryPool;

MemoryPool *mp_create(size_t size) {
    MemoryPool *mp = malloc(sizeof(MemoryPool));
    if (mp == NULL) {
        return NULL;
    }
    mp->buffer = malloc(size);
    if (mp->buffer == NULL) {
        free(mp);
        return NULL;
    }
    mp->buffer_size = size;
    mp->used_size = 0;
    return mp;
}

void *mp_alloc(MemoryPool *mp, size_t size) {
    if (mp == NULL || mp->used_size + size > mp->buffer_size) {
        return NULL;
    }
    void *ptr = (char *) mp->buffer + mp->used_size;
    mp->used_size += size;
    return ptr;
}

void mp_reset(MemoryPool *mp) {
    if (mp != NULL) {
        mp->used_size = 0;
    }
}

void mp_destroy(MemoryPool *mp) {
    if (mp != NULL) {
        free(mp->buffer);
        free(mp);
    }
    // 使用内存池
}

int main() {
    MemoryPool *mp = mp_create(1000);
    if (mp == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    int *array1 = mp_alloc(mp, 10 * sizeof(int));
    double *numbers = mp_alloc(mp, 5 * sizeof(double));
    // 使用分配的内存...

    mp_reset(mp);// 重置内存池，重新使用内存
    int *array2 = mp_alloc(mp, 20 * sizeof(int));
    // 使用新的分配...
    mp_destroy(mp);
    return 0;
}
```

### 避免内存泄漏

确保每个内存分配都有对应的内存释放。示例：

```c
#include <stdlib.h>

int main() {
    int *ptr = malloc(100);
    if (ptr == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    // 使用ptr...
    free(ptr); // 释放内存
    // 在错误处理路径中也释放内存
    int *ptr2 = malloc(200);
    if (ptr2 == NULL) {
        return EXIT_FAILURE;
    }
    //使用ptr2..
    free(ptr2);
    return 0;
}
```

### 使用RAII技术

RAll(ResourceAcquisitionIs Initialization)是一种编程范式，资源在对象构造时获取，在析构时释放。虽然C语言没有内置的RAI支持，但可以通过封装来实现。示例：

```c
#include <malloc.h>

typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

DynamicArray *da_create() {
    DynamicArray *da = malloc(sizeof(DynamicArray));
    if (da == NULL) {
        return NULL;
    }
    da->size = 0;
    da->capacity = 1;
    da->data = malloc(da->capacity * sizeof(int));
    if (da->data == NULL) {
        free(da);
        return NULL;
    }
    return da;
}

void da_destroy(DynamicArray *da) {
    if (da != NULL) {
        free(da->data);
        free(da);
    }
}

// 使用封装的DynamicArray结构
int main() {
    DynamicArray *da = da_create();
    if (da == NULL) { // 处理错误
        return EXIT_FAILURE;
    }
    // 使用da...
    da_destroy(da);
    return 0;
}
```

## 动态内存管理在不同场景中的应用

动态内存管理在各种场景中都有应用，从简单的数组扩展到复杂的内存池管理。下面我们探讨一些常见的应用场景。

### 可变长度数据结构

当处理可变长度的数据结构时，动态内存管理非常有用。例如，当处理用户输入时，我们可能不知道需要存储多少数据。示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define INITIAL_SIZE 10
#define GROWTH_FACTOR 2

char *read_line() {
    char *buffer = malloc(INITIAL_SIZE);
    if (buffer == NULL) {
        return NULL;
    }
    size_t size = INITIAL_SIZE;
    size_t index = 0;
    while (1) {
        char c = fgetc(stdin);
        if (c == EOF || c == '\n') {
            buffer[index] = '\0';
            break;
        }
        if (index >= size - 1) {
            // 保留一个位置给空终止符
            size *= GROWTH_FACTOR;
            char *new_buffer = realloc(buffer, size);
            if (new_buffer == NULL) {
                free(buffer);
                return NULL;
            }
            buffer = new_buffer;
        }
        buffer[index++] = c;
    }
    return buffer;
}

int main() {
    char *line = read_line();
    if (line == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    printf("读取的行是：%s\n", line);
    free(line);
    return 0;
}
```

### 动态数据结构

动态数据结构（如链表、树）需要根据需要分配和释放节点。示例：一个简单的链表实现

```c
#include <stddef.h>
#include <malloc.h>

typedef struct Node {
    int value;
    struct Node *next;
} Node;

Node *create_node(int value) {
    Node *node = malloc(sizeof(Node));
    if (node == NULL) {
        return NULL;
    }
    node->value = value;
    node->next = NULL;
    return node;
}

void destroy_node(Node *node) {
    if (node != NULL) {
        free(node);
    }
}

void append(Node **head, int value) {
    Node *new_node = create_node(value);
    if (new_node == NULL) {
        return;
    }
    if (*head == NULL) {
        *head = new_node;
        return;
    }
    Node *current = *head;
    while (current->next != NULL) {
        current = current->next;
    }
    current->next = new_node;
}

void destroy_list(Node *head) {
    Node *current = head;
    while (current != NULL) {
        Node *next = current->next;
        destroy_node(current);
        current = next;
    }
}

int main() {
    Node *head = NULL;
    append(&head, 1);
    append(&head, 2);
    append(&head, 3);
    // 遍历链表...
    destroy_list(head);
    return 0;
}
```

### 内存池管理

内存池是一种优化技术，通过将多个小内存分配合并为一个大内存分配来减少内存碎片和提高性能。示例：一个简单的内存池实现

```c
#include <stdint.h>
#include <malloc.h>

typedef struct {
    void *buffer;
    size_t buffer_size;
    size_t used_size;
} MemoryPool;

MemoryPool *mp_create(size_t size) {
    MemoryPool *mp = malloc(sizeof(MemoryPool));
    if (mp == NULL) {
        return NULL;
    }
    mp->buffer = malloc(size);
    if (mp->buffer == NULL) {
        free(mp);
        return NULL;
    }
    mp->buffer_size = size;
    mp->used_size = 0;
    return mp;
}

void *mp_alloc(MemoryPool *mp, size_t size) {
    if (mp == NULL || mp->used_size + size > mp->buffer_size) {
        return NULL;
    }
    void *ptr = (char *) mp->buffer + mp->used_size;
    mp->used_size += size;
    return ptr;
}

void mp_reset(MemoryPool *mp) {
    if (mp != NULL) {
        mp->used_size = 0;
    }
}

void mp_destroy(MemoryPool *mp) {
    if (mp != NULL) {
        free(mp->buffer);
        free(mp);
    }
    // 使用内存池
}

int main() {
    MemoryPool *mp = mp_create(1000);
    if (mp == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    int *array1 = mp_alloc(mp, 10 * sizeof(int));
    double *numbers = mp_alloc(mp, 5 * sizeof(double));
    // 使用分配的内存...

    mp_reset(mp);// 重置内存池，重新使用内存
    int *array2 = mp_alloc(mp, 20 * sizeof(int));
    // 使用新的分配...
    mp_destroy(mp);
    return 0;
}
```

### 多线程环境中的内存管理

在多线程环境中，内存管理需要特别注意，以避免竞态条件和内存泄漏。示例：使用互斥锁保护内存操作

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

typedef struct {
    void *buffer;
    size_t buffer_size;
    size_t used_size;
    pthread_mutex_t mutex;
} ThreadSafeMemoryPool;

ThreadSafeMemoryPool *ts_mp_create(size_t size) {
    ThreadSafeMemoryPool *ts_mp = malloc(sizeof(ThreadSafeMemoryPool));
    if (ts_mp == NULL) {
        return NULL;
    }
    ts_mp->buffer = malloc(size);
    if (ts_mp->buffer == NULL) {
        free(ts_mp);
        return NULL;
    }
    ts_mp->buffer_size = size;
    ts_mp->used_size = 0;
    if (pthread_mutex_init(&ts_mp->mutex, NULL) != 0) {
        free(ts_mp->buffer);
        free(ts_mp);
        return NULL;
    }
    return ts_mp;
}

void *ts_mp_alloc(ThreadSafeMemoryPool *ts_mp, size_t size) {
    if (ts_mp == NULL || size == 0) {
        return NULL;
    }
    pthread_mutex_lock(&ts_mp->mutex);
    if (ts_mp->used_size + size > ts_mp->buffer_size) {
        pthread_mutex_unlock(&ts_mp->mutex);
        return NULL;
    }
    void *ptr = (char *) ts_mp->buffer + ts_mp->used_size;
    ts_mp->used_size += size;
    pthread_mutex_unlock(&ts_mp->mutex);
    return ptr;
}

void ts_mp_reset(ThreadSafeMemoryPool *ts_mp) {
    if (ts_mp != NULL) {
        pthread_mutex_lock(&ts_mp->mutex);
        ts_mp->used_size = 0;
        pthread_mutex_unlock(&ts_mp->mutex);
    }
}

void ts_mp_destroy(ThreadSafeMemoryPool *ts_mp) {
    if (ts_mp != NULL) {
        pthread_mutex_destroy(&ts_mp->mutex);
        free(ts_mp->buffer);
        free(ts_mp);
    }
}

// 在多线程环境中使用线程安全内存池
void *thread_function(void *arg) {
    ThreadSafeMemoryPool *ts_mp = arg;
    if (ts_mp == NULL) {
        return NULL;
    }
    // 从内存池分配内存...
    int *data = ts_mp_alloc(ts_mp, sizeof(int));
    if (data == NULL) {
        // 处理错误
        return NULL;
    }
    *data = 42;
    // ...
    return NULL;
}

int main() {
    ThreadSafeMemoryPool *ts_mp = ts_mp_create(1000);
    if (ts_mp == NULL) {
        //处理错误
        return EXIT_FAILURE;
    }
    pthread_t thread1, thread2;
    pthread_create(&thread1, NULL, thread_function, ts_mp);
    pthread_create(&thread2, NULL, thread_function, ts_mp);
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    ts_mp_destroy(ts_mp);
    return 0;
}
```

## 高级内存管理技术

除了标准库提供的动态内存管理函数外，还有一些高级技术可以进一步优化内存管理。

### 自定义内存分配器

在某些情况下，标准库提供的内存分配器可能无法满足特定应用程序的需求。在这种情况下，可以实现自定义内存分配器。示例：一个简单的自定义内存分配器

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>

// 1MB
#define CHUNK_SIZE (1024 * 1024)

typedef struct {
    void *start;
    void *end;
    size_t size;
} Chunk;

typedef struct {
    Chunk *chunks;
    size_t num_chunks;
    size_t current_chunk;
    void *current_ptr;
} CustomAllocator;

CustomAllocator *ca_create() {
    CustomAllocator *ca = malloc(sizeof(CustomAllocator));
    if (ca == NULL) {
        return NULL;
    }
    ca->chunks = NULL;
    ca->num_chunks = 0;
    ca->current_chunk = 0;
    ca->current_ptr = NULL;
    return ca;
}

void *ca_alloc(CustomAllocator *ca, size_t size) {
    if (ca == NULL || size == 0) {
        return NULL;
    }
    //如果还没有分配任何块，分配第一个块
    if (ca->current_ptr == NULL) {
        Chunk *chunk = malloc(sizeof(Chunk));
        if (chunk == NULL) {
            return NULL;
        }
        chunk->size = CHUNK_SIZE;
        chunk->start = mmap(NULL, chunk->size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOU, -1, 0);
        if (chunk->start == MAP_FAILED) {
            free(chunk);
            return NULL;
        }
        chunk->end = (char *) chunk->start + chunk->size;
        ca->chunks = malloc(sizeof(Chunk));
        if (ca->chunks = NULL) {
            munmap(chunk->start, chunk->size);
            free(chunk);
            return NULL;
        }
        ca->chunks[0] = *chunk;
        ca->num_chunks = 1;
        ca->current_chunk = 0;
        ca->current_ptr = ca->chunks[0].start;
        return ca->current_ptr;
    }
    //如果当前块不够大，分配新块
    if (chunk->start == MAP_FAILED) {
        free(chunk);
        return NULL;
    }
    chunk->end = (char *) chunk->start + chunk->size;
    ca->chunks = realloc(ca->chunks, (ca->num_chunks + 1) * sizeof(Chunk));
    if (ca->chunks = NULL) {
        munmap(chunk->start, chunk->size);
        free(chunk);
        return NULL;
    }
    ca->chunks[ca->num_chunks++] = *chunk;
    free(chunk);
    ca->current_chunk = 0;
    ca->current_ptr = ca->chunks[0].start;

    // 检查当前块是否有足够的空间
    if ((char *) ca->current_ptr + size <= (char *) ca->chunks[ca->current_chunk].end) {
        void *ptr = ca->current_ptr;
        ca->current_ptr = (char *) ptr + size;
        return ptr;
    }
    // 否则，尝试分配新的块
    if (ca->current_chunk < ca->num_chunks - 1) {
        ca->current_chunk++;
        ca->current_ptr = ca->chunks[ca->current_chunk].start;
        return ca_alloc(ca, size); // 递归调用
    } else {
        // 分配新的块
        Chunk *chunk = malloc(sizeof(Chunk));
        if (chunk == NULL) {
            return NULL;
        }
        chunk->size = CHUNK_SIZE;
        chunk->start = MMap(NULL, chunk->size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (chunk->start == MAP_FAILED) {
            free(chunk);
            return NULL;
        }
        chunk->end = (char *) chunk->start + chunk->size;
        ca->chunks = realloc(ca->chunks, (ca->num_chunks + 1) * sizeof(Chunk));
        if (ca->chunks == NULL) {
            munmap(chunk->start, chunk->size);
            free(chunk);
            return NULL;
        }
        ca->chunks[ca->num_chunks++] = *chunk;
        free(chunk);
        ca->current_chunk = ca->num_chunks - 1;
        ca->current_ptr = ca->chunks[ca->current_chunk].start;
        return ca_alloc(ca, size); // 递归调用
    }
}

void ca_free(CustomAllocator *ca, void *ptr) {
    // 在这个简单的实现中，不实现真正的释放功能
    // 可以在更复杂的实现中添加释放逻辑
}

void ca_destroy(CustomAllocator *ca) {
    if (ca != NULL) {
        for (size_t i = 0; i < ca->num_chunks; i++) {
            if (ca->chunks[i].start != NULL) {
                munmap(ca->chunks[i].start, ca->chunks[i].size);
            }
        }
        free(ca->chunks);
        free(ca);
    }
}

// 使用自定义内存分配器
int main() {
    CustomAllocator *ca = ca_create();
    if (ca == NULL) {
        // 处理错误
        return EXIT_FAILURE;
    }
    int *array1 = ca_alloc(ca, 100 * sizeof(int);
    double *numbers = ca_alloc(ca, 50 * sizeof(double));
    // 使用分配的内存...
    ca_destroy(ca);
    return 0;
}
```

### 内存映射

内存映射是一种将文件或匿名内存区域映射到进程地址空间的技术。它可以用于高效地管理大块内存或与文件交互。示例：使用匿名内存映射

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <unistd.h>

// 1MB
#define MAP_SIZE (1024 * 1024)

int main() {
    // 映射匿名内存
    void *map = Mmap(NULL, MAP_SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (map == MAP_FAILED) {
        // 处理错误
        return EXIT_FAILURE;
    }
    // 使用映射的内存..
    int *array = map;
    array[0] = 42;
    // 分离映射
    if (munmap(map, MAP_SIZE) == -1) {
        // 处理错误
        return EXIT_FAILURE;
    }
    return 0;
}
```
