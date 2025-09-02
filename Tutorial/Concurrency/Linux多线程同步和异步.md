# Linux多线程同步和异步基础教程

## 简介

在现代计算环境中，多核处理器已经成为标准配置。为了充分利用这些硬件资源，软件开发人员需要采用多线程和异步编程技术来设计高效、响应迅速的应用程序。

## 多线程基础

在Linux系统中，多线程编程主要通过POSIX线程库（也称为pthreads）来实现。多线程允许在一个进程中并发执行多个控制流，从而可以同时处理多个任务，提高程序的执行效率和响应速度。下面我们将详细介绍Linux下多线程的基础知识。

### 线程的概念

线程是比进程更小的执行单位，它与同一进程中的其他线程共享进程的地址空间和资源（如文件描述符、环境变量）。这意味着线程之间的通信比进程间通信更为简单快捷，但同时也意味着它们需要小心处理对共享资源的访问，以避免数据不一致的问题。

### 创建线程

要创建一个新的线程，你需要调用pthread_create函数。这个函数的基本原型如下：

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
```

-   thread：用于存储新线程的标识符。
-   attr：线程属性，通常传递NULL表示使用默认属性。
-   start_routine：新线程开始执行的函数指针。
-   arg：传递给start_routine函数的参数。

### 线程的终止

线程可以通过以下几种方式终止：

-   自然结束：当线程执行完start_routine函数后，线程自然结束。
-   显式退出：线程调用pthread_exit函数，可以带有一个退出状态码。
-   外部取消：其他线程调用pthread_cancel函数取消目标线程。

### 线程的调度

线程的调度策略可以通过pthread_attr_setinheritsched和pthread_attr_setschedpolicy函数设置。Linux支持以下几种调度策略：

-   SCHED_FIFO：先到先服务的调度策略。
-   SCHED_RR：轮转调度策略。
-   SCHED_OTHER：标准调度策略，适用于大多数用户程序。

### 线程取消与同步

线程可以通过pthread_cancel函数被取消。而pthread_join或pthread_cond_wait等函数用于同步线程之间的执行。

### 示例代码

以下是一个简单的多线程程序示例，演示了如何创建两个线程并让它们各自打印一条消息：

```c
#include <stdio.h>
#include <pthread.h>

void *printHello(void *threadid) {
    long tid = (long) threadid;
    printf("Hello World! Thread ID is %ld\n", tid);
    pthread_exit(NULL);
}

int main() {
    pthread_t thread1, thread2;
    int rc;
    long t;

    rc = pthread_create(&thread1, NULL, printHello, (void *) 1L);
    if (rc) {
        printf("ERROR; return code from pthread_create() is %d\n", rc);
        exit(-1);
    }

    rc = pthread_create(&thread2, NULL, printHello, (void *) 2L);
    if (rc) {
        printf("ERROR; return code from pthread_create() is %d\n", rc);
        exit(-1);
    }

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    printf("Main thread exiting.\n");
    return 0;
}
```

在上面的示例中，我们创建了两个线程，每个线程都会打印一条带有其线程ID的消息。pthread_join函数用于等待线程完成。

## 线程同步

线程同步是指在多线程环境中，确保线程以正确的顺序执行或避免对共享资源的并发访问导致的数据不一致性问题。

Linux提供了多种同步原语，包括但不限于：

### 互斥锁（Mutexes）

互斥锁（Mutex）是多线程编程中一种重要的同步机制，用于保护对共享资源的并发访问。在操作系统中，互斥锁提供了对临界区（critical section）的原子性锁定，确保任何时刻只有一个线程可以访问临界区内的共享资源。

#### 互斥锁的原理

互斥锁内部通常由操作系统内核管理，它维护了一个锁状态和一个所有者线程ID（如果有的话）。当一个线程尝试获取互斥锁时，内核会检查锁是否已经被另一个线程持有。如果锁是可用的，那么内核会将其标记为已锁定，并记录下当前线程作为锁的所有者。如果锁已经被另一个线程持有，请求锁的线程会被阻塞，直到锁被释放。

**互斥锁遵循以下原则：**

-   排他性：任何时刻最多只有一个线程可以拥有互斥锁。

-   持有和等待：一个已经持有互斥锁的线程不能再次获取同一个锁，否则会导致死锁。

-   有限等待：等待互斥锁的线程不会无限期地等待，除非锁一直被持有。

#### 编程互斥锁

在Linux中，互斥锁通过POSIX线程库（pthreads）提供，主要涉及以下几个函数：

初始化互斥锁：pthread_mutex_init用于初始化互斥锁对象。你可以选择使用默认属性，或者指定特定的属性。

```c
pthread_mutex_t mutex;


// 使用默认属性初始化
pthread_mutex_init(&mutex, NULL);

// 使用自定义属性初始化
pthread_mutexattr_t attr;
pthread_mutexattr_init(&attr);
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE); // 设置为递归锁
pthread_mutex_init(&mutex, &attr);
```

获取互斥锁：pthread_mutex_lock用于获取互斥锁。如果锁已经被另一个线程持有，当前线程将阻塞，直到锁被释放。

```c
pthread_mutex_lock(&mutex);
```

尝试获取互斥锁：pthread_mutex_trylock尝试获取互斥锁，但不会阻塞。如果锁不可用，函数将立即返回，并设置错误码EBUSY。

```c
if (pthread_mutex_trylock(&mutex) != 0) {
    // 锁已被占用
}
```

释放互斥锁：pthread_mutex_unlock用于释放互斥锁。只有持有锁的线程才能释放锁。

```c
pthread_mutex_unlock(&mutex);
```

销毁互斥锁：pthread_mutex_destroy用于销毁互斥锁对象。在程序结束前应该调用此函数释放互斥锁所占用的资源。

```c
pthread_mutex_destroy(&mutex);
```

#### 示例代码

下面是一个使用互斥锁保护共享数据的简单示例：

```c
#include <stdio.h>
#include <pthread.h>

pthread_mutex_t data_mutex = PTHREAD_MUTEX_INITIALIZER;
int shared_data = 0;

void *increment_data(void *arg) {
    int i;
    for (i = 0; i < 100000; i++) {
        pthread_mutex_lock(&data_mutex);
        shared_data++;
        pthread_mutex_unlock(&data_mutex);
    }
    return NULL;
}

int main() {
    pthread_t thread1, thread2;

    pthread_create(&thread1, NULL, increment_data, NULL);
    pthread_create(&thread2, NULL, increment_data, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    printf("Shared data: %d\n", shared_data);
    pthread_mutex_destroy(&data_mutex);
    return 0;
}
```

在这个例子中，两个线程并行地增加shared_data的值。data_mutex互斥锁确保了每次只有一个线程可以修改shared_data，从而避免了数据竞争。

### 读写锁（Read-Write Locks）

读写锁（Read-Write Lock），也被称为RW锁，是一种特殊的同步机制，用于管理对共享资源的并发访问。与互斥锁不同，读写锁允许多个读线程同时访问共享资源，但只允许一个写线程在任意时间点访问资源。这种机制非常适合于读操作远多于写操作的场景，因为多个读操作可以并发执行，从而提高系统的整体性能。

#### 读写锁的原理

读写锁维护两种类型的锁：读锁和写锁。读锁允许多个线程同时读取共享资源，只要没有线程持有写锁。写锁是排他的，一旦某个线程获取了写锁，所有的读锁和写锁请求都将被阻止，直到写锁被释放。

读写锁的主要特点包括：

1.  读共享：允许多个读线程同时访问资源。

2.  写独占：在写锁持有的情况下，不允许其他读锁或写锁。

3.  写优先：在某些实现中，当有线程正在等待写锁时，读锁请求可能会被延迟，以减少写线程的等待时间。

4.  公平性：有些读写锁实现会考虑公平性，即按照请求的先后顺序授予锁。

#### 编程读写锁

在Linux中，读写锁同样由POSIX线程库（pthreads）提供。主要涉及以下函数：

初始化读写锁：pthread_rwlock_init用于初始化读写锁对象。你可以选择使用默认属性，或者指定特定的属性。

```c
pthread_rwlock_t rwlock;
pthread_rwlockattr_t attr;

// 使用默认属性初始化
pthread_rwlock_init(&rwlock, NULL);

// 使用自定义属性初始化
pthread_rwlockattr_init(&attr);
pthread_rwlockattr_setkind_np(&attr, PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP);
pthread_rwlock_init(&rwlock, &attr);
```

获取读锁：pthread_rwlock_rdlock用于获取读锁。如果当前有写锁被持有，所有读锁请求将被阻塞。

```c
pthread_rwlock_rdlock(&rwlock);
```

获取写锁：pthread_rwlock_wrlock用于获取写锁。如果当前有任何读锁或写锁被持有，写锁请求将被阻塞。

```c
pthread_rwlock_wrlock(&rwlock);
```

尝试获取读锁：pthread_rwlock_tryrdlock尝试获取读锁，但不会阻塞。如果锁不可用，函数将立即返回，并设置错误码EBUSY。

```c
if (pthread_rwlock_tryrdlock(&rwlock) != 0) {
    // 读锁已被占用
}
```

尝试获取写锁：pthread_rwlock_trywrlock尝试获取写锁，但不会阻塞。

```c
if (pthread_rwlock_trywrlock(&rwlock) != 0) {
    // 写锁已被占用
}
```

释放读锁或写锁：pthread_rwlock_unlock用于释放当前持有的读锁或写锁。

```c
pthread_rwlock_unlock(&rwlock);
```

销毁读写锁：pthread_rwlock_destroy用于销毁读写锁对象。

```c
pthread_rwlock_destroy(&rwlock);
```

#### 示例代码

下面是一个使用读写锁保护共享资源的简单示例：

```c
#include <stdio.h>
#include <pthread.h>

pthread_rwlock_t data_rwlock = PTHREAD_RWLOCK_INITIALIZER;
int shared_data = 0;

void *read_data(void *arg) {

    for (int i = 0; i < 10000; i++) {
        pthread_rwlock_rdlock(&data_rwlock);
        printf("Reading: %d\n", shared_data);
        pthread_rwlock_unlock(&data_rwlock);
    }
    return NULL;
}

void *write_data(void *arg) {
    for (int i = 0; i < 10000; i++) {
        pthread_rwlock_wrlock(&data_rwlock);
        shared_data++;
        printf("Writing: %d\n", shared_data);
        pthread_rwlock_unlock(&data_rwlock);
    }
    return NULL;
}

int main() {
    pthread_t reader_thread[5], writer_thread[2];

    for (int i = 0; i < 5; i++) {
        pthread_create(&reader_thread[i], NULL, read_data, NULL);
    }
    for (int i = 0; i < 2; i++) {
        pthread_create(&writer_thread[i], NULL, write_data, NULL);
    }

    for (int i = 0; i < 5; i++) {
        pthread_join(reader_thread[i], NULL);
    }
    for (int i = 0; i < 2; i++) {
        pthread_join(writer_thread[i], NULL);
    }

    pthread_rwlock_destroy(&data_rwlock);
    printf("shared_data: %d\n", shared_data);
    return 0;
}
```

在这个例子中，多个读线程和写线程并发访问shared_data。读写锁确保了读操作可以并发执行，而写操作则具有排他性，保证了数据的一致性。

#### 总结

读写锁是一种非常有用的同步机制，特别适用于读操作远多于写操作的场景。通过允许多个读线程同时访问共享资源，读写锁可以显著提高多线程程序的并发性能。然而，使用读写锁时也需要考虑到公平性、死锁和性能问题，合理设计和测试是必要的。

### 条件变量（Condition Variables）

条件变量（Condition Variable）是多线程编程中一种重要的同步机制，主要用于解决线程间的协作问题，尤其是当线程需要等待某种条件满足时。条件变量通常与互斥锁（Mutex）一起使用，以确保线程在等待条件变化时不会导致数据竞争或死锁。

#### 条件变量的原理

条件变量允许线程在等待某个条件变真时“暂停”或“挂起”，并且当条件满足时可以唤醒一个或所有等待的线程。在Linux中，条件变量是由POSIX线程库（pthreads）提供的。

使用条件变量的基本步骤如下：

1.  初始化：首先，需要初始化一个条件变量和一个互斥锁。

2.  加锁：在访问或修改共享数据之前，线程需要获取互斥锁。

3.  检查条件：在互斥锁保护下，检查条件是否满足。

4.  等待：如果条件不满足，线程调用条件变量的wait函数，释放互斥锁并进入等待状态，直到接收到信号。

5.  释放锁：当条件满足时，线程调用条件变量的signal或broadcast函数，唤醒一个或所有等待的线程。被唤醒的线程重新获得互斥锁，继续执行。

6.  解锁：在完成对共享数据的操作后，线程需要释放互斥锁。

#### 编程条件变量

在Linux中，条件变量相关的函数包括：

初始化条件变量：pthread_cond_init用于初始化条件变量对象。

```c
pthread_cond_t cond;
pthread_cond_init(&cond, NULL);
```

初始化互斥锁：pthread_mutex_init用于初始化互斥锁对象。

```c
pthread_mutex_t mutex;
pthread_mutex_init(&mutex, NULL);
```

等待条件变量：pthread_cond_wait用于等待条件变量。在调用此函数前，必须先获得互斥锁。pthread_cond_wait会释放互斥锁，并在条件满足时自动重新获取互斥锁。

```c
pthread_cond_wait(&cond, &mutex);
```

尝试等待条件变量：pthread_cond_timedwait类似于pthread_cond_wait，但它允许指定一个绝对的时间点，如果在指定时间内条件未满足，则函数返回。

```c
struct timespec abstime;// 设置abstime为当前时间加上等待时间
// ...
pthread_cond_timedwait(&cond, &mutex, &abstime);
```

发送信号：pthread_cond_signal用于唤醒一个等待的线程。

```c
pthread_cond_signal(&cond);
```

广播信号：pthread_cond_broadcast用于唤醒所有等待的线程。

```c
pthread_cond_broadcast(&cond);
```

销毁条件变量和互斥锁：pthread_cond_destroy和pthread_mutex_destroy用于销毁条件变量和互斥锁。

```c
pthread_cond_destroy(&cond);
pthread_mutex_destroy(&mutex);
```

#### 示例代码

下面是一个使用条件变量和互斥锁的简单示例，展示如何让一个线程等待另一个线程设置的条件：

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int ready = 0;

void *waiting_thread_function(void *arg) {
    pthread_mutex_lock(&mutex);
    while (!ready) {
        pthread_cond_wait(&cond, &mutex);
    }
    printf("Thread is notified and ready is now %d\n", ready);
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void *signaling_thread_function(void *arg) {
    sleep(2); // 模拟一些耗时操作
    pthread_mutex_lock(&mutex);
    ready = 1;
    pthread_cond_signal(&cond); // 唤醒等待的线程
    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() {
    pthread_t waiting_thread, signaling_thread;

    pthread_create(&waiting_thread, NULL, waiting_thread_function, NULL);
    pthread_create(&signaling_thread, NULL, signaling_thread_function, NULL);

    pthread_join(waiting_thread, NULL);
    pthread_join(signaling_thread, NULL);

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);
    return 0;
}
```

在这个例子中，waiting_thread_function在ready变量为0时进入等待状态，而signaling_thread_function在一段时间后将ready设置为1，并通过pthread_cond_signal唤醒等待的线程。

#### 总结

条件变量是多线程编程中非常强大的工具，用于协调线程间的同步。与互斥锁结合使用时，条件变量可以确保线程在等待条件变化时不阻塞其他线程对共享资源的访问，同时避免了死锁的风险。正确使用条件变量和互斥锁对于构建高效且可靠的多线程应用程序至关重要。

## 线程异步

异步处理允许程序在等待长时间运行的操作（如磁盘I/O、网络请求）完成时，继续执行其他任务。Linux提供以下异步机制：

### 异步I/O（AIO）

异步I/O（Asynchronous I/O，简称AIO）是一种高级I/O模型，它允许程序在发起I/O操作后立即返回，而不必等待I/O操作完成。在Linux中，AIO由libaio库支持，提供了一种非阻塞的方式来进行文件I/O操作，这对于提高I/O密集型应用程序的性能非常有效。

#### 异步I/O原理

在传统的同步I/O模型中，线程或进程在发起I/O操作后必须等待直到操作完成才能继续执行。这会导致CPU资源在等待I/O操作完成时闲置。相比之下，AIO允许程序在发出I/O请求后立即返回，然后在I/O操作完成时接收通知。这样，CPU可以在此期间执行其他任务，提高了系统的整体效率。

AIO主要涉及以下概念：

1.  I/O上下文（IO context）：这是AIO操作的容器，它包含一组I/O请求队列。一个进程可以创建多个I/O上下文，每个上下文都有自己的I/O请求队列。
2.  I/O请求（IO request）：每个I/O请求对应一个具体的I/O操作，如读或写。I/O请求可以被加入到I/O上下文的队列中。

3.  事件（Event）：当I/O操作完成时，AIO机制会生成一个事件，该事件可以被进程检测到。

#### 编程异步I/O

在Linux中，AIO的主要函数包括：

初始化I/O上下文：io_setup()用于创建I/O上下文。

```c
io_context_t ctx;
int ret = io_setup(1024, &ctx); // 创建一个可以同时处理1024个请求的I/O上下文
```

提交异步读/写请求：aio_read()和aio_write()用于提交异步读写请求。

```c
struct iocb *iocb;
io_prep_read(iocb, fd, buffer, count, offset); // 准备读操作
io_prep_write(iocb, fd, buffer, count, offset); // 准备写操作
io_submit(ctx, 1, &iocb); // 提交I/O请求
```

获取I/O事件：io_getevents()用于从I/O上下文中获取已完成的I/O事件。

```c
struct io_event event;
int num_events = io_getevents(ctx, 1, 1, &event, NULL); // 获取最多1个已完成的事件
```

取消I/O请求：io_cancel()用于取消正在进行的I/O请求。

```c
io_cancel(ctx, iocb, &res); // 取消由iocb标识的I/O请求
```

销毁I/O上下文：io_destroy()用于释放I/O上下文。

```c
io_destroy(ctx);
```

**示例代码**

下面是一个使用AIO进行异步读操作的简单示例：

```c
static ssize_t aio_read(struct io_iocb *iocb, int fd, void *buf, size_t nbyte, off_t offset) {
    struct iocb *req;
    struct io_event event;
    int ret;

    req = iocb;
    memset(req, 0, sizeof(*req));
    io_prep_pread(req, fd, buf, nbyte, offset);
    io_submit(ctx, 1, &req);

    do {
        ret = io_getevents(ctx, 1, 1, &event, NULL);
    } while (ret == -1 && errno == EINTR);

    return event.res;
}

int main() {
    int fd;
    io_context_t ctx;
    char buffer[1024];
    ssize_t bytes_read;

    fd = open("testfile.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    if (io_setup(1024, &ctx) == -1) {
        perror("io_setup");
        close(fd);
        exit(EXIT_FAILURE);
    }

    bytes_read = aio_read(NULL, fd, buffer, sizeof(buffer), 0);
    if (bytes_read == -1) {
        perror("aio_read");
        close(fd);
        io_destroy(ctx);
        exit(EXIT_FAILURE);
    }

    printf("Read %zd bytes: %s\n", bytes_read, buffer);

    close(fd);
    io_destroy(ctx);
    return 0;
}
```

在上面的例子中，aio_read函数封装了AIO读操作的过程。主函数中，首先打开一个文件，然后创建一个I/O上下文，接着调用aio_read进行异步读操作。读操作完成后，数据被读入缓冲区，并输出到屏幕。

**总结**

异步I/O是提高I/O密集型应用程序性能的有效手段。通过使用AIO，应用程序可以在等待I/O操作的同时执行其他任务，从而最大化系统资源的利用率。然而，AIO的编程模型相对复杂，需要仔细设计和调试，以确保正确处理各种异常情况。

### 信号处理

在Linux系统中，信号（Signals）是一种进程间通信（IPC）机制，也是操作系统用来通知进程发生某些事件的标准方式。信号可以由软件产生（例如，由系统调用引发），也可以由硬件触发（比如用户按下中断键）。信号提供了处理错误、终止进程以及实现一些特定操作的机制。

#### 信号的基本原理

1.信号的产生：

-   硬件条件：如键盘中断（Ctrl+C）。

-   软件条件：如除数为零的算术运算。

-   用户请求：使用kill或raise系统调用发送信号给进程。

-   系统调用：如alarm设置定时器信号。

2.信号的类型：

-   Linux支持多种信号，每种信号都有一个固定的编号，如SIGINT（2）、SIGTERM（15）、SIGKILL（9）等。

-   信号编号范围通常是1到31，加上一些实时信号（大于31）。

3.信号的处理：

-   默认动作：忽略、终止进程或生成核心转储。

-   自定义处理：通过信号处理器来捕获并处理信号。

-   忽略信号：使用signal或sigaction函数设置信号处理方式为忽略。

4.信号的传递与阻塞：

-   信号传递到进程时，如果该信号没有被阻塞，且进程没有处于不可中断的睡眠状态，那么信号会被交付。

-   进程可以通过sigprocmask系统调用来阻塞信号，阻止信号的传递直到解除阻塞。

5.信号的排队：

-   如果信号在另一个相同类型的信号被处理之前到达，则第二个信号会被排队，直到第一个信号处理完毕。

-   实时信号有优先级，可以抢占非实时信号。

#### 信号编程

在C语言中，信号通常通过signal或更推荐的sigaction函数来处理。

**使用signal函数**

signal函数用于安装信号处理器，但它的功能有限且不安全，因为它不保证信号处理函数的原子性。

```c
void signal_handler(int sig) {
    // 处理信号的代码
}

int main() {
    signal(SIGINT, signal_handler); // 设置SIGINT信号的处理器
    // ...
}
```

**使用sigaction函数**

sigaction函数提供了更强大的信号处理能力，可以设置信号掩码、指定信号处理方式（SIG_IGN、SIG_DFL或自定义函数）等。

```c
void signal_handler(int sig) {
    printf("Received signal %d\n", sig);
}

int main() {
    struct sigaction sa;

    sa.sa_handler = signal_handler; // 设置信号处理器
    sigemptyset(&sa.sa_mask);       // 清空信号集
    sa.sa_flags = 0;                // 默认标志

    sigaction(SIGINT, &sa, NULL);   // 安装信号处理器

    while (1) {
        // 主循环或其他代码
    }
}
```

**需要注意的点**

-   信号处理函数应该避免使用全局变量，因为它们可能在信号处理过程中被其他线程修改，导致未定义行为。
-   不要在信号处理函数中调用可能导致阻塞的函数，如fread、printf等，这可能导致死锁。
-   SIGKILL和SIGSTOP信号不能被捕捉或忽略。
-   信号的传递是非确定性的，即信号可能在任何时刻到达，因此编写信号安全的代码非常重要。

信号是Linux系统中一个强大但复杂的特性，合理地使用信号可以显著增强程序的健壮性和响应性。但是，不当的使用也会引入难以预料的问题，所以在设计信号处理逻辑时要格外小心。

### 事件循环

在Linux环境下，事件循环（Event Loop）是构建高效并发网络服务器和异步I/O应用的核心机制。事件循环能够帮助应用程序有效地处理多个同时发生的事件，而不需要为每个事件创建独立的线程或进程，从而避免了上下文切换的开销。事件循环的主要工作原理是监控一组文件描述符的I/O事件，并在这些事件发生时调用相应的回调函数。

#### 事件循环的基本原理

1.  事件源注册：应用程序向事件循环注册感兴趣的事件源（通常是文件描述符），并关联一个回调函数。事件源可以是网络套接字、定时器、管道等。
2.  事件轮询：事件循环通过轮询的方式检查所有注册的事件源的状态。这通常通过系统调用如select(), poll(), 或更高效的epoll()来实现。
3.  事件分发：当检测到某个事件源有活动时，事件循环会调用与该事件源相关的回调函数，执行相应的事件处理代码。
4.  循环迭代：事件循环持续运行，不断检查事件源状态，直到满足退出条件，如收到特定信号或显式调用退出函数。

#### 事件循环的编程模型

在C语言中，使用select(), poll(), epoll()等函数可以实现事件循环。

**使用select()**

select系统调用是用于同时监视多个文件描述符的读写状态，直到其中一个或多个文件描述符准备好进行读写操作，或者超时为止。这使得程序可以在等待I/O操作时处理其他事情，而不是阻塞在一个文件描述符上。

```c
int main() {
    fd_set readfds;
    struct timeval timeout;
    int max_fd = 0;
    int ret;

    FD_ZERO(&readfds); // 清空文件描述符集合
    FD_SET(STDIN_FILENO, &readfds); // 添加标准输入
    FD_SET(server_socket, &readfds); // 添加服务器套接字

    max_fd = (server_socket > STDIN_FILENO) ? server_socket : STDIN_FILENO;

    timeout.tv_sec = 5; // 设置超时时间为5秒
    timeout.tv_usec = 0;

    ret = select(max_fd + 1, &readfds, NULL, NULL, &timeout);
    if (ret == -1) {
        perror("select error");
        exit(EXIT_FAILURE);
    } else if (ret == 0) {
        printf("select timed out\n");
    } else {
        if (FD_ISSET(server_socket, &readfds)) {
            // 服务器套接字有活动，处理新连接或数据
        }
        if (FD_ISSET(STDIN_FILENO, &readfds)) {
            // 标准输入有活动，读取用户输入
        }
    }
}
```

**使用poll()**

poll()函数与select()类似，用于实现多路复用I/O，即同时监控多个文件描述符的I/O事件。但使用结构体数组pollfd来管理文件描述符。

poll允许程序等待一组文件描述符中的任何一个或多个变为可读、可写或出现异常状态。当poll调用返回时，它会告诉程序哪些文件描述符已经准备好进行I/O操作。

**poll系统调用**

poll的函数原型如下：

```
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

-   fds：一个pollfd结构体数组的指针，其中包含了要监控的文件描述符及其对应的事件。

-   nfds：fds数组中元素的数量。

-   timeout：等待事件发生的超时时间（以毫秒为单位）。如果为0，则poll将立即返回；如果为负数，则poll将无限期地等待事件。

pollfd结构体定义如下：

```
struct pollfd {
    int   fd;         /* 文件描述符 */
    short events;     /* 请求的事件 */
    short revents;    /* 返回的实际发生的事件 */
};
```

events字段可以设置以下事件：

-   POLLIN：表示文件描述符可以进行读操作。
-   POLLPRI：表示紧急数据可以进行读操作。
-   POLLOUT：表示文件描述符可以进行写操作。
-   POLLERR：表示文件描述符发生了错误。
-   POLLHUP：表示文件描述符出现了挂断状态。
-   POLLNVAL：表示文件描述符无效。

revents字段返回实际发生的事件。

**使用示例**

下面是一个使用poll的简单示例，这个示例创建了一个TCP服务器，监听端口上的连接请求：

```c
#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int listen_sock, conn_sock;
    struct sockaddr_in serv_addr;
    socklen_t addr_len;
    struct pollfd fds[2]; // 监听套接字和客户端连接套接字
    int num_fds = 0;
    char buffer[BUFFER_SIZE];

    // 创建监听套接字
    listen_sock = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_sock < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 设置地址和端口
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(PORT);

    // 绑定地址和端口
    if (bind(listen_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("Bind failed");
        exit(EXIT_FAILURE);
    }

    // 开始监听
    if (listen(listen_sock, 5) < 0) {
        perror("Listen failed");
        exit(EXIT_FAILURE);
    }

    // 初始化pollfd结构体
    fds[num_fds].fd = listen_sock;
    fds[num_fds].events = POLLIN;
    num_fds++;

    while (1) {
        // 等待事件
        if (poll(fds, num_fds, -1) < 0) {
            perror("Poll failed");
            exit(EXIT_FAILURE);
        }

        // 检查监听套接字是否有事件
        if (fds[0].revents & POLLIN) {
            addr_len = sizeof(struct sockaddr_in);
            conn_sock = accept(listen_sock, (struct sockaddr *)&serv_addr, &addr_len);
            if (conn_sock < 0) {
                perror("Accept failed");
                continue;
            }

            // 新的连接，添加到pollfd数组
            if (num_fds >= sizeof(fds)/sizeof(struct pollfd)) {
                perror("Too many connections");
                close(conn_sock);
                continue;
            }

            fds[num_fds].fd = conn_sock;
            fds[num_fds].events = POLLIN;
            num_fds++;
        }

        // 检查客户端连接套接字是否有事件
        for (int i = 1; i < num_fds; i++) {
            if (fds[i].revents & POLLIN) {
                memset(buffer, 0, BUFFER_SIZE);
                if (recv(fds[i].fd, buffer, BUFFER_SIZE-1, 0) <= 0) {
                    // 连接关闭或错误
                    close(fds[i].fd);
                    memmove(&fds[i], &fds[--num_fds], sizeof(struct pollfd));
                } else {
                    // 收到数据
                    printf("Received data: %s\n", buffer);
                }
            }
        }
    }

    close(listen_sock);
    return 0;
}
```

在这个示例中，我们使用poll来监控监听套接字和客户端连接套接字。当监听套接字上有事件时，我们接受新的连接，并将新的连接套接字添加到pollfd数组中。之后，poll将监控这些连接套接字，当有数据可读时，我们读取数据并处理。如果连接关闭或出现错误，我们则关闭该连接，并从数组中移除它。

#### 使用epoll()

epoll是Linux内核提供的一种高效的I/O多路复用技术，用于监控多个文件描述符的I/O事件。epoll通过系统调用的形式提供给用户空间程序使用，而不是作为一个命令行工具。epoll相比于传统的select和poll，提供了更高的性能和扩展性，尤其是在处理大量文件描述符时。

epoll系统调用

epoll主要通过以下三个系统调用来实现其功能：

1.epoll_create(size_t size)：创建一个epoll事件文件描述符，返回一个文件描述符用于后续的epoll操作。size参数在新版本的Linux内核中已经被忽略，但仍然保留以保持向后兼容。

2.epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)：用于注册、修改或删除epoll事件。

-   epfd：epoll文件描述符。

-   op：操作类型，可以是EPOLL_CTL_ADD、EPOLL_CTL_MOD或EPOLL_CTL_DEL。

-   fd：要监控的文件描述符。

-   event：一个epoll_event结构体，包含了要监控的事件类型和相关联的数据。

3.epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)：等待epoll事件的发生，将就绪的事件填入events数组。

-   epfd：epoll文件描述符。

-   events：一个epoll_event数组，用于返回就绪的事件。

-   maxevents：events数组的最大长度。

-   timeout：等待事件的超时时间（毫秒），如果为-1，则无限期等待。

```c
#define MAX_EVENTS 10
#define PORT 8080

int main() {
    int listen_sock, conn_sock, epoll_fd;
    struct sockaddr_in serv_addr;
    socklen_t addr_len;
    struct epoll_event ev, events[MAX_EVENTS];
    int num_events;

    // 创建监听套接字
    listen_sock = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_sock < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 绑定地址和端口
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(PORT);
    if (bind(listen_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("Bind failed");
        exit(EXIT_FAILURE);
    }

    // 监听连接
    if (listen(listen_sock, 5) < 0) {
        perror("Listen failed");
        exit(EXIT_FAILURE);
    }

    // 创建epoll文件描述符
    epoll_fd = epoll_create1(0);
    if (epoll_fd < 0) {
        perror("Epoll create failed");
        exit(EXIT_FAILURE);
    }

    // 注册监听套接字
    ev.events = EPOLLIN;
    ev.data.fd = listen_sock;
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_sock, &ev) < 0) {
        perror("Epoll add failed");
        exit(EXIT_FAILURE);
    }

    while (1) {
        num_events = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (num_events < 0) {
            perror("Epoll wait failed");
            break;
        }

        for (int i = 0; i < num_events; i++) {
            if (events[i].data.fd == listen_sock) {
                // 接受新连接
                addr_len = sizeof(struct sockaddr_in);
                conn_sock = accept(listen_sock, (struct sockaddr*)&serv_addr, &addr_len);
                if (conn_sock < 0) {
                    perror("Accept failed");
                    continue;
                }

                // 注册新连接的文件描述符
                ev.events = EPOLLIN | EPOLLET;
                ev.data.fd = conn_sock;
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, conn_sock, &ev);
            } else {
                // 处理数据
                char buffer[1024] = {0};
                int bytes_received = recv(events[i].data.fd, buffer, 1024, 0);
                if (bytes_received <= 0) {
                    // 对方关闭连接
                    close(events[i].data.fd);
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, events[i].data.fd, NULL);
                } else {
                    printf("Received data: %s\n", buffer);
                }
            }
        }
    }

    // 清理资源
    close(listen_sock);
    close(epoll_fd);
    return 0;
}
```

在这个示例中，我们创建了一个监听套接字，然后使用epoll来监控这个套接字上的读事件。当有新的连接请求时，epoll_wait会返回，我们可以接受连接并将新连接的文件描述符注册到epoll中，以便后续接收数据。当数据到达时，epoll_wait同样会返回，我们可以处理数据。如果对方关闭了连接，我们也会收到通知，并可以清理资源。

#### 事件驱动框架

在实际开发中，通常不会直接使用上述原生API，而是使用更高层次的事件驱动框架，如libev、libevent、Node.js的V8引擎中的libuv等。这些框架封装了底层细节，提供了更易用的接口和更高级的功能，如错误处理、异步I/O、定时器等。

例如，在Node.js中，事件循环是其核心架构的一部分，使用libuv实现：

```c
const net = require('net');

const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    console.log(`Received data: ${data}`);
    socket.write('Echo: ' + data);
  });
});

server.listen(3000);
```

在这个例子中，每当客户端连接到服务器，就会创建一个新的套接字，这个套接字被添加到事件循环中。当数据到达时，事件循环调用data事件的监听器来处理数据。

事件循环是现代网络应用和操作系统中非常重要的概念，掌握其原理和使用方法对于构建高性能和高并发的应用程序至关重要。

