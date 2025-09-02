# C语言指针

## 指针的基本概念

指针是C语言中最强大也最复杂的特性之一。简单来说，**指针是一个变量，其值为另一个变量的内存地址**。通过指针，我们可以间接访问和操作存储在特定内存位置的数据。

### 为什么需要指针？

1.  **直接内存操作**：允许程序直接访问和操作内存
2.  **高效传递数据**：避免复制大量数据
3.  **动态内存管理**：在运行时分配和释放内存
4.  **实现复杂数据结构**：如链表、树、图等
5.  **支持回调机制**：通过函数指针实现

### 指针的基本术语

-   **指针变量**：存储内存地址的变量
-   **指针所指向的对象**：指针变量中地址所对应的内存位置存储的数据
-   **解引用**：通过指针访问它所指向的对象
-   **指针类型**：决定了指针解引用时访问的内存大小和解释方式

## 指针的内存模型

为了理解指针，必须先了解计算机内存的基本原理。

### 内存模型

计算机内存可以想象为一系列连续编号的字节（每个字节8位）。每个字节都有一个唯一的地址，从0开始递增。

```
地址:   0x1000    0x1001    0x1002    0x1003    0x1004    ...

内容:   [0x42]    [0x65]    [0x6C]    [0x6C]    [0x6F]    ...
```

### 变量在内存中的表示

假设有一个整型变量：

```
int num = 12345;  // 假设在内存地址0x1000处
```

在内存中的表示（假设int为4字节）：

```
地址:   0x1000    0x1001    0x1002    0x1003
内容:   [0x39]    [0x30]    [0x00]    [0x00]  // 12345的二进制表示，考虑小端序
```

### 指针在内存中的表示

指针变量本身也占用内存空间，存储的是地址值：

```
int num = 12345;  // 地址: 0x1000
int *p = &num;    // 假设p的地址是0x2000
```

内存中的结构：

```
地址:   0x1000    0x1001    0x1002    0x1003
内容:   [0x39]    [0x30]    [0x00]    [0x00]  // num的值

地址:   0x2000    0x2001    0x2002    0x2003    0x2004    0x2005    0x2006    0x2007
内容:   [0x00]    [0x10]    [0x00]    [0x00]    [0x00]    [0x00]    [0x00]    [0x00]  // p的值 (0x1000)
```

## 指针的类型和声明

### 指针类型

指针的类型决定了：

1.  指针所指向的数据类型
2.  指针解引用时访问的内存大小
3.  指针进行算术运算时增减的步长

### 指针声明语法

```
类型 *指针名；
```

例如：

```
int *p;       // 指向整型的指针
char *str;    // 指向字符的指针
double *dp;   // 指向双精度浮点数的指针
void *vp;     // 无类型指针，可以指向任何类型
```

### 指针声明的变体

```
int *p1, *p2, *p3;  // 三个指向整型的指针
int* p1, p2, p3;    // 一个指向整型的指针p1，两个整型变量p2和p3
int * p1;           // 等同于 int *p1;
```

### void指针的特殊性

void指针是一种"通用"指针，可以指向任何类型的数据，但在解引用之前必须进行类型转换：

```
void *vp = &num;
int *ip = (int*)vp;  // 在C中需要显式转换
printf("%d", *ip);   // 正确
printf("%d", *vp);   // 错误，不能直接解引用void指针
```

## 指针的基本操作

### 取地址操作（&）

```
int num = 10;
int *p = &num;  // p现在保存了num的地址
```

### 解引用操作（\*）

```
int num = 10;
int *p = &num;
*p = 20;       // 通过p修改num的值
printf("%d", num);  // 输出20
```

### 指针赋值

```
int x = 10, y = 20;
int *p1 = &x;
int *p2 = &y;
p1 = p2;       // p1现在指向y
```

### 指针比较

```
if (p1 == p2) {  // 比较两个指针是否指向同一个内存位置
    printf("指向同一个地址");
}

if (*p1 == *p2) {  // 比较两个指针指向的值是否相等
    printf("指向的值相等");
}
```

### 空指针

```
int *p = NULL;  // NULL通常定义为(void*)0
if (p == NULL) {
    printf("这是一个空指针");
}
```

## 指针和数组的关系

在C语言中，数组名称实际上是指向数组第一个元素的常量指针。

### 数组名称作为指针

```
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;  // 等价于 p = &arr[0]

printf("%d", *p);      // 输出10
printf("%d", *(p+1));  // 输出20
```

### 指针访问数组元素

```
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;

// 以下表达式等价
printf("%d", arr[2]);
printf("%d", *(arr+2));
printf("%d", *(p+2));
printf("%d", p[2]);
```

### 数组名称和指针的区别

虽然数组名称可以当作指针使用，但它们有关键区别：

1.  **数组名是常量指针**，不能被修改：

```
arr++;  // 错误，数组名不能被修改
p++;    // 正确，p是变量
```

1.  **sizeof运算符的行为不同**：

```
sizeof(arr);  // 返回整个数组的大小：5 * sizeof(int)
sizeof(p);    // 只返回指针本身的大小，通常是4或8字节
```

### 多维数组与指针

```
int matrix[3][4];  // 3行4列的二维数组

// 访问元素
matrix[1][2] = 10;

// 使用指针访问
*(*(matrix+1)+2) = 10;  // 等价于上面的语句

// 指向行的指针
int (*row)[4] = matrix;  // 指向有4个int元素的数组的指针
```

## 指针和字符串

在C中，字符串是以空字符('\0')结尾的字符数组。字符串与指针有密切关系。

### 字符串声明和初始化

```
char str1[] = "Hello";            // 字符数组
char *str2 = "World";             // 字符指针，指向常量字符串
char *str3 = (char*)malloc(10);   // 动态分配的字符串
strcpy(str3, "Dynamic");
```

### 字符串操作

```
// 字符串拷贝
char dest[20];
char *src = "Source";
strcpy(dest, src);

// 字符串连接
strcat(dest, " String");

// 字符串长度
int len = strlen(dest);

// 字符串比较
if (strcmp(str1, str2) == 0) {
    printf("相等");
}
```

### 字符串与指针操作

```
char str[] = "Hello";
char *p = str;

// 遍历字符串
while (*p != '\0') {
    printf("%c", *p);
    p++;
}

// 字符串复制
char *src = "Source";
char *dest = (char*)malloc(strlen(src) + 1);
char *p_dest = dest;
while (*src) {
    *p_dest++ = *src++;
}
*p_dest = '\0';
```

## 指针与函数

指针与函数的结合是C语言中最强大的特性之一。

### 指针作为函数参数

```
// 通过指针修改变量值
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main() {
    int x = 5, y = 10;
    swap(&x, &y);
    // 现在x=10, y=5
    return 0;
}
```

### 指针作为函数返回值

```
int* findMax(int arr[], int size) {
    int *max_ptr = &arr[0];
    for (int i = 1; i < size; i++) {
        if (arr[i] > *max_ptr) {
            max_ptr = &arr[i];
        }
    }
    return max_ptr;
}

int main() {
    int numbers[] = {5, 12, 7, 9, 3};
    int *max = findMax(numbers, 5);
    printf("最大值: %d\n", *max);
    return0;
}
```

### 注意事项

返回指向局部变量的指针是危险的：

```
int* createInt() {
    int x = 10;
    return &x;  // 危险！x在函数返回后将不再有效
}
```

正确的做法是使用动态内存分配：

```
int* createInt() {
    int *p = (int*)malloc(sizeof(int));
    *p = 10;
    return p;  // 返回指向堆内存的指针，需要调用者释放
}
```

## 指针算术运算

指针可以进行加减运算，但增减的单位是指针类型的大小。

### 指针加减整数

```
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;  // p指向arr[0]

p = p + 1;     // p指向arr[1]
p += 2;        // p指向arr[3]
p--;           // p指向arr[2]
```

### 指针相减

两个指针相减的结果是它们之间的元素数量：

```
int arr[5] = {10, 20, 30, 40, 50};
int *p1 = &arr[1];
int *p2 = &arr[4];

int diff = p2 - p1;  // diff = 3，表示从p1到p2有3个元素的距离
```

### 不同类型指针的步长

```
char *cp = (char*)0x1000;
int *ip = (int*)0x1000;

cp++;  // 现在cp = 0x1001 (增加1字节)
ip++;  // 现在ip = 0x1004 (增加4字节，假设int为4字节)
```

## 多级指针

指针可以指向另一个指针，形成多级指针结构。

### 二级指针

```
int num = 10;
int *p = &num;   // 一级指针
int **pp = &p;   // 二级指针

printf("%d", **pp);  // 输出10
```

内存表示：

```
num: [10]       地址: 0x1000
p:   [0x1000]   地址: 0x2000
pp:  [0x2000]   地址: 0x3000
```

### 多级指针的应用

1.  二维数组的动态分配：

```
int **matrix = (int**)malloc(rows * sizeof(int*));
for (int i = 0; i < rows; i++) {
    matrix[i] = (int*)malloc(cols * sizeof(int));
}
```

1.  通过函数修改指针：

```
void allocateBuffer(char **buffer, int size) {
    *buffer = (char*)malloc(size);
}

int main() {
    char *myBuffer = NULL;
    allocateBuffer(&myBuffer, 100);
    strcpy(myBuffer, "Hello");
    free(myBuffer);
    return 0;
}
```

### 三级及以上指针

```
int ***ppp;  // 三级指针
int ****pppp; // 四级指针
```

虽然理论上可以使用任意级别的指针，但实际应用中很少使用三级以上的指针，因为结构会变得难以理解和维护。

## 指针数组与数组指针

### 指针数组

指针数组是一个数组，每个元素都是指针：

```
int *arr[5];  // 包含5个int指针的数组

int a=1, b=2, c=3, d=4, e=5;
arr[0] = &a;
arr[1] = &b;
// ...

printf("%d", *arr[1]);  // 输出2
```

指针数组常用于管理字符串：

```
char *names[] = {"John", "Alice", "Bob", "Carol"};
printf("%s", names[1]);  // 输出"Alice"
```

### 数组指针

数组指针是指向数组的指针：

```
int (*p)[5];  // 指向包含5个int元素的数组的指针

int arr[5] = {1, 2, 3, 4, 5};
p = &arr;  // p指向整个数组

printf("%d", (*p)[2]);  // 输出3
```

### 区分指针数组和数组指针

-   `int *arr[5]` - 指针数组：一个有5个元素的数组，每个元素是int指针
-   `int (*arr)[5]` - 数组指针：一个指针，指向有5个int元素的数组

### 多维数组与数组指针

```
int matrix[3][4];

// 使用数组指针访问
int (*p)[4] = matrix;  // p是指向4个int元素的数组的指针
printf("%d", p[1][2]);  // 访问matrix[1][2]
```

## 函数指针

函数指针是指向函数的指针变量，可以用来调用函数或者作为参数传递。

### 定义函数指针

```
// 声明一个函数指针类型
typedef int (*Operation)(int, int);

// 定义函数
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }

// 使用函数指针
Operation op = add;
int result = op(5, 3);  // 调用add函数，结果为8

op = subtract;
result = op(5, 3);  // 调用subtract函数，结果为2
```

### 函数指针作为参数

```
int processNumbers(int a, int b, int (*func)(int, int)) {
    return func(a, b);
}

int main() {
    int result1 = processNumbers(5, 3, add);      // 结果为8
    int result2 = processNumbers(5, 3, subtract); // 结果为2
    return 0;
}
```

### 函数指针数组

```
int (*operations[4])(int, int) = {add, subtract, multiply, divide};
int result = operations[0](5, 3);  // 调用add函数
```

### 回调函数

函数指针常用于实现回调机制：

```
// 排序函数
void bubbleSort(int arr[], int n, int (*compare)(int, int)) {
    for (int i = 0; i < n-1; i++) {
        for (int j = 0; j < n-i-1; j++) {
            if (compare(arr[j], arr[j+1]) > 0) {
                // 交换元素
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

// 比较函数
int ascending(int a, int b) { return a - b; }
int descending(int a, int b) { return b - a; }

// 使用
int arr[] = {5, 2, 8, 1, 9};
bubbleSort(arr, 5, ascending);  // 升序排序
bubbleSort(arr, 5, descending); // 降序排序
```

## 动态内存分配

C语言提供了一组函数用于动态内存管理，这些函数与指针密切相关。

### malloc

分配指定字节数的内存：

```
int *p = (int*)malloc(5 * sizeof(int));  // 分配5个int的空间
if (p != NULL) {
    for (int i = 0; i < 5; i++) {
        p[i] = i + 1;
    }
}
```

### calloc

分配指定数量的元素，并初始化为0：

```
int *p = (int*)calloc(5, sizeof(int));  // 分配5个int的空间，并初始化为0
```

### realloc

调整已分配内存的大小：

```
int *p = (int*)malloc(5 * sizeof(int));
// ...使用p...

// 扩展内存到10个int
p = (int*)realloc(p, 10 * sizeof(int));
```

### free

释放动态分配的内存：

```
free(p);    // 释放内存
p = NULL;   // 避免悬挂指针
```

### 内存泄漏

如果分配了内存但没有释放，就会发生内存泄漏：

```
void leakMemory() {
    int *p = (int*)malloc(sizeof(int));
    *p = 10;
    // 函数结束时没有调用free(p)，导致内存泄漏
}
```

### 动态二维数组

```
// 分配3行4列的二维数组
int **matrix = (int**)malloc(3 * sizeof(int*));
for (int i = 0; i < 3; i++) {
    matrix[i] = (int*)malloc(4 * sizeof(int));
}

// 使用
matrix[1][2] = 10;

// 释放
for (int i = 0; i < 3; i++) {
    free(matrix[i]);
}
free(matrix);
```

## 指针的高级应用

### 实现链表

```
typedef struct Node {
    int data;
    struct Node *next;
} Node;

// 创建节点
Node* createNode(int data) {
    Node *newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = NULL;
    return newNode;
}

// 在链表尾部添加节点
void appendNode(Node **head, int data) {
    Node *newNode = createNode(data);

    if (*head == NULL) {
        *head = newNode;
        return;
    }

    Node *current = *head;
    while (current->next != NULL) {
        current = current->next;
    }
    current->next = newNode;
}

// 打印链表
void printList(Node *head) {
    Node *current = head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

// 释放链表
void freeList(Node *head) {
    Node *current = head;
    Node *next;

    while (current != NULL) {
        next = current->next;
        free(current);
        current = next;
    }
}
```

### 实现二叉树

```
typedef struct TreeNode {
    int data;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

// 创建节点
TreeNode* createNode(int data) {
    TreeNode *newNode = (TreeNode*)malloc(sizeof(TreeNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// 插入节点
TreeNode* insertNode(TreeNode *root, int data) {
    if (root == NULL) {
        return createNode(data);
    }

    if (data < root->data) {
        root->left = insertNode(root->left, data);
    } else {
        root->right = insertNode(root->right, data);
    }

    return root;
}

// 中序遍历
void inorderTraversal(TreeNode *root) {
    if (root != NULL) {
        inorderTraversal(root->left);
        printf("%d ", root->data);
        inorderTraversal(root->right);
    }
}

// 释放树
void freeTree(TreeNode *root) {
    if (root != NULL) {
        freeTree(root->left);
        freeTree(root->right);
        free(root);
    }
}
```

### 通用数据结构

使用void指针实现通用数据结构：

```
typedef struct GenericNode {
    void *data;
    struct GenericNode *next;
} GenericNode;

// 创建节点
GenericNode* createNode(void *data, size_t dataSize) {
    GenericNode *newNode = (GenericNode*)malloc(sizeof(GenericNode));
    newNode->data = malloc(dataSize);
    memcpy(newNode->data, data, dataSize);
    newNode->next = NULL;
    return newNode;
}
```

## 常见错误与陷阱

### 未初始化指针

```
int *p;       // 未初始化，包含随机值
*p = 10;      // 危险！可能导致段错误
```

正确做法：

```
int *p = NULL;
if (p != NULL) {
    *p = 10;  // 安全检查
}

// 或者
int num;
int *p = &num;
*p = 10;      // 安全
```

### 内存泄漏

```
void memoryLeak() {
    int *p = (int*)malloc(sizeof(int));
    // 没有调用free(p)
}
```

正确做法：

```
void noLeak() {
    int *p = (int*)malloc(sizeof(int));
    // 使用p
    free(p);
    p = NULL;  // 避免悬挂指针
}
```

### 使用已释放的内存

```
int *p = (int*)malloc(sizeof(int));
free(p);
*p = 10;     // 危险！p已经成为野指针
```

### 指针越界访问

```
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;
*(p+10) = 100;  // 危险！超出数组边界
```

### 返回局部变量的地址

```
int* badFunction() {
    int x = 10;
    return &x;  // 危险！x在函数结束后不再有效
}
```

正确做法：

```
int* goodFunction() {
    int *p = (int*)malloc(sizeof(int));
    *p = 10;
    return p;  // 返回堆内存的指针，调用者负责释放
}
```

### 指针类型错误

```
int num = 10;
char *cp = (char*)&num;
*cp = 'A';    // 只修改了num的第一个字节
```

### 字符串常量修改

```
char *str = "Hello";
str[0] = 'h';  // 危险！尝试修改字符串常量
```

正确做法：

```
char str[] = "Hello";
str[0] = 'h';  // 安全，str是数组
```

## 指针使用的最佳实践

### 总是初始化指针

```
int *p = NULL;  // 初始化为NULL
int num = 10;
int *q = &num;  // 初始化为有效地址
```

### 检查动态内存分配的结果

```
int *p = (int*)malloc(size);
if (p == NULL) {
    // 处理内存分配失败
    fprintf(stderr, "Memory allocation failed\n");
    return -1;
}
```

### 使用后释放动态内存

```
p = (int*)malloc(size);
// 使用p
free(p);
p = NULL;  // 避免悬挂指针
```

### 避免指针运算导致的越界访问

```
for (int i = 0; i < size; i++) {
    // 而不是直接 p++
    process(p + i);
}
```

### 使用const限定符保护数据

```
void printString(const char *str) {
    // str指向的内容不能被修改
    printf("%s", str);
}
```

### 避免过度使用指针

如果简单变量就能解决问题，就不要使用指针：

```
// 不好的做法
void addOne(int *num) {
    (*num)++;
}

// 对于简单计算，可以这样写
int addOne(int num) {
    return num + 1;
}
```

### 使用typedef简化指针定义

```
typedef int* IntPtr;

IntPtr p1, p2;  // 两个指向int的指针
```

## 实际案例分析

### 案例1: 字符串处理函数

实现一个函数来删除字符串中的所有空格：

```
char* removeSpaces(const char *str) {
    if (str == NULL) returnNULL;

    int len = strlen(str);
    char *result = (char*)malloc(len + 1);
    if (result == NULL) returnNULL;

    int j = 0;
    for (int i = 0; i < len; i++) {
        if (str[i] != ' ') {
            result[j++] = str[i];
        }
    }
    result[j] = '\0';

    return result;
}

// 使用
char *original = "Hello World!";
char *processed = removeSpaces(original);
printf("%s\n", processed);  // 输出"HelloWorld!"
free(processed);
```

### 案例2: 实现简单的内存池

```
#include <stdio.h>
#include <stdlib.h>

// 定义内存块结构体
typedefstruct MemoryBlock {
    struct MemoryBlock *next;
} MemoryBlock;

// 定义内存池结构体
typedefstruct MemoryPool {
    MemoryBlock *freeList;
    size_t blockSize;
    size_t blockCount;
    void *pool;
} MemoryPool;

// 初始化内存池
void initMemoryPool(MemoryPool *pool, size_t blockSize, size_t blockCount) {
    pool->blockSize = blockSize;
    pool->blockCount = blockCount;
    // 分配大块内存
    pool->pool = malloc(blockSize * blockCount);
    if (pool->pool == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return;
    }
    // 初始化空闲列表
    pool->freeList = (MemoryBlock *)pool->pool;
    MemoryBlock *current = pool->freeList;
    for (size_t i = 0; i < blockCount - 1; i++) {
        current->next = (MemoryBlock *)((char *)current + blockSize);
        current = current->next;
    }
    current->next = NULL;
}

// 从内存池分配内存
void *allocateFromPool(MemoryPool *pool) {
    if (pool->freeList == NULL) {
        fprintf(stderr, "Memory pool is full\n");
        returnNULL;
    }
    MemoryBlock *allocated = pool->freeList;
    pool->freeList = pool->freeList->next;
    return (void *)allocated;
}

// 将内存块释放回内存池
void freeToPool(MemoryPool *pool, void *block) {
    MemoryBlock *released = (MemoryBlock *)block;
    released->next = pool->freeList;
    pool->freeList = released;
}

// 销毁内存池
void destroyMemoryPool(MemoryPool *pool) {
    free(pool->pool);
    pool->freeList = NULL;
    pool->blockSize = 0;
    pool->blockCount = 0;
    pool->pool = NULL;
}

int main() {
    MemoryPool pool;
    size_t blockSize = 1024;
    size_t blockCount = 10;
    // 初始化内存池
    initMemoryPool(&pool, blockSize, blockCount);
    // 分配内存
    void *block1 = allocateFromPool(&pool);
    void *block2 = allocateFromPool(&pool);
    if (block1 != NULL && block2 != NULL) {
        printf("Allocated two blocks from memory pool\n");
    }
    // 释放内存
    freeToPool(&pool, block1);
    freeToPool(&pool, block2);
    printf("Freed two blocks back to memory pool\n");
    // 销毁内存池
    destroyMemoryPool(&pool);
    return0;
}
```

此内存池用于管理固定大小的内存块，基本思路是预先分配一大块内存，然后把它分割成多个固定大小的内存块，每次分配时从空闲列表里取出一个可用的内存块，释放时将该内存块放回空闲列表。

代码说明：

1.  **`MemoryBlock` 结构体**：用于表示内存块，其中 `next` 指针用于构建空闲列表。
2.  **`MemoryPool` 结构体**：表示内存池，包含空闲列表指针 `freeList`、每个内存块的大小 `blockSize`、内存块数量 `blockCount` 以及指向大块内存的指针 `pool`。
3.  **`initMemoryPool` 函数**：初始化内存池，分配大块内存并初始化空闲列表。
4.  **`allocateFromPool` 函数**：从内存池分配一个内存块，若空闲列表为空则输出错误信息。
5.  **`freeToPool` 函数**：将一个内存块释放回内存池，插入到空闲列表头部。
6.  **`destroyMemoryPool` 函数**：销毁内存池，释放大块内存并重置相关指针和计数器。
7.  **`main` 函数**：演示了如何使用内存池进行内存分配和释放。
