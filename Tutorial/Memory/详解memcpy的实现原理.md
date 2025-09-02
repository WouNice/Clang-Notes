# 详解memcpy的实现原理

可能各个平台的实现原理不同，但都大同小异，本文主要介绍`glibc`中的`memcpy`。

## 基本定义

```c
void *memcpy(void *dest, const void *src, size_t n);
```

`memcpy`的功能是将`src`指向的内存区域的前n个字节复制到`dest`指向的内存区域。

使用`memcpy`，需要注意以下几点：

1.  不处理重叠区域（这是`memmove`的职责）
2.  返回目标地址dest
3.  要求调用者确保：
    -   `dest`和`src`都是有效指针
    -   内存区域不重叠
    -   有足够的空间

## 实现策略

`glibc`的`memcpy`实现采用了三层复制策略：

1.  字节复制（Byte Copy）
2.  字复制（Word Copy）
3.  页复制（Page Copy）

面试时候回答出三层复制策略这个关键点就很加分了。

### 核心实现

整体策略大体如下

1.  数据量小于16字节：
    1.  直接使用字节复制
    2.  避免对齐开销
2.  数据量在16字节到16KB之间（不一定是16KB，为了方便起见，统一使用16，你理解原理即可）：
    1.  先对齐
    2.  使用字复制
    3.  处理剩余字节
3.  数据量大于16KB：
    1.  先对齐
    2.  尝试页复制
    3.  字复制处理剩余数据
    4.  字节复制处理尾部

代码详见：

```c
void *MEMCPY(void *dstpp, const void *srcpp, size_t len) {
    unsigned long int dstp = (long int) dstpp;
    unsigned long int srcp = (long int) srcpp;

    /* 保存原始目标地址用于返回 */
    void *orig_dstpp = dstpp;

    /* 对于足够大的数据块，使用优化策略 */
    if (len >= OP_T_THRES) {
        /* 1. 首先对齐目标地址 */
        len -= (-dstp) % OPSIZ;
        BYTE_COPY_FWD(dstp, srcp, (-dstp) % OPSIZ);

        /* 2. 尝试页复制 */
        PAGE_COPY_FWD_MAYBE(dstp, srcp, len, len);

        /* 3. 使用字复制处理剩余数据 */
        WORD_COPY_FWD(dstp, srcp, len, len);
    }

    /* 4. 处理剩余字节 */
    BYTE_COPY_FWD(dstp, srcp, len);

    return orig_dstpp;
}
```

### 详细分析

#### 字节复制

最基础的复制方式，一次复制一个字节：

```c
#define BYTE_COPY_FWD(dst_bp, src_bp, nbytes)             \
  do                        \
    {                       \
      size_t __nbytes = (nbytes);               \
      while (__nbytes > 0)                  \
  {                     \
    byte __x = ((byte *) src_bp)[0];              \
    src_bp += 1;                    \
    __nbytes -= 1;                  \
    ((byte *) dst_bp)[0] = __x;               \
    dst_bp += 1;                    \
  }                     \
    } while (0)
```

使用场景：

-   小数据量复制
-   处理非对齐数据
-   处理剩余字节

#### 字复制

按机器字长进行复制，提高效率：

```c
#define WORD_COPY_FWD(dst_bp, src_bp, nbytes_left, nbytes)          \
  do                        \
    {                       \
      if (src_bp % OPSIZ == 0)                  \
  _wordcopy_fwd_aligned (dst_bp, src_bp, (nbytes) / OPSIZ);       \
      else                      \
  _wordcopy_fwd_dest_aligned (dst_bp, src_bp, (nbytes) / OPSIZ);        \
      src_bp += (nbytes) & -OPSIZ;                \
      dst_bp += (nbytes) & -OPSIZ;                \
      (nbytes_left) = (nbytes) % OPSIZ;               \
    } while (0)
```

优化特点：

-   利用CPU字长进行批量复制
-   要求内存对齐
-   比字节复制更高效

#### 页复制

最高效的复制方式，适用于大块数据：

```c
#define PAGE_COPY_FWD_MAYBE(dstp, srcp, nbytes_left, nbytes)              \
  do {                                                                      \
    if ((nbytes) >= PAGE_COPY_THRESHOLD                                     \
        && PAGE_OFFSET ((dstp) - (srcp)) == 0)                             \
    {                                                                       \
      /* 处理第一个不完整页 */                                              \
      size_t nbytes_before = PAGE_OFFSET (-(dstp));                        \
      if (nbytes_before != 0)                                              \
        {                                                                   \
          WORD_COPY_FWD (dstp, srcp, nbytes_left, nbytes_before);          \
          nbytes -= nbytes_before;                                         \
        }                                                                   \
      /* 执行页复制 */                                                      \
      PAGE_COPY_FWD (dstp, srcp, nbytes_left, nbytes);                    \
    }                                                                       \
  } while (0)

#define PAGE_COPY_THRESHOLD      (16384)

#define PAGE_SIZE    __vm_page_size
#define PAGE_COPY_FWD(dstp, srcp, nbytes_left, nbytes)               \
  ((nbytes_left) = ((nbytes)                       \
          - (__vm_copy (__mach_task_self (),             \
              (vm_address_t) srcp, trunc_page (nbytes),   \
              (vm_address_t) dstp) == KERN_SUCCESS       \
             ? trunc_page (nbytes)                 \
             : 0)))
```
