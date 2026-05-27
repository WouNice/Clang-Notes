# Linux多进程编程

## 进程的创建

>   这里vfolk和clone不常用，场景太局限了，一笔掠过

### 手动启动或者系统创建

通过命令行直接运行程序或者使用系统的启动脚本自动启动程序，表象上一种创建方式，其实底层本质上还是通过fork、exec去创建进程。不算是一种创建方式。

```c
# 命令行直接运行
./program
# 或者指定完整路径
/usr/bin/program

# 后台运行
nohup ./program 2<&1 > /dev/null &

# 通过脚本启动
bash start_script.sh
```

### `fork()` - 标准进程创建

**特点**：

-   完整复制父进程的地址空间（采用写时复制COW技术）
-   子进程继承父进程的文件描述符、信号处理等资源
-   子进程获得新的PID和独立的内存空间

**使用场景**：

-   需要与父进程共享环境的子进程
-   并行任务处理
-   最常用的进程创建方式

**fork()函数原型**

```v
NAME
    fork - 创建一个子进程

SYNOPSIS
    #include <sys/types.h>
    #include <unistd.h>

    pid_t fork(void);

DESCRIPTION
    fork() 通过复制调用进程来创建一个新进程。新进程称为子进程。调用进程称为父进程。
RETURN VALUE
    成功返回pid号，其中-1表示失败，0表示是子进程。
```

**fork demo**

```c
#include "stdio.h"
#include "stdlib.h"
#include "unistd.h"
#include "sys/wait.h"

int main() {
    pid_t pid = fork();
    if (pid < 0) {
        perror("fork");
        exit(1);
    }
    if (pid == 0) {
        printf("child process: pid = %d, ppid = %d\n", getpid(), getppid());
        sleep(5);
    } else {
        printf("parent process: pid = %d, ppid = %d\n", getpid(), getppid());
        //等待子进程结束,也可以用wait，waitpid可以指定等待的子进程，而wait是等待任意一个子进程结束
        waitpid(pid, NULL, 0);
        //wait(NULL);
    }
    return 0;
}
```

### `exec`函数族 - 程序替换

**特点**

-   完全替换当前进程的代码段
-   保持进程ID不变
-   有多个变体处理不同参数传递方式

**使用场景**：

-   执行外部程序
-   改变当前进程映像
-   配合`fork()`实现"fork-exec"模式

**函数原型**：

```c
execl("/path/to/prog", "prog", "arg1", NULL);  // 参数列表
execv("/path/to/prog", argv);                 // 参数数组
execle("/path/to/prog", "prog", NULL, envp);  // 自定义环境
```

**exec demo**

```c
#include "stdio.h"
#include "stdlib.h"
#include "unistd.h"
#include "sys/wait.h"

int main() {
    pid_t pid = fork();
    if (pid < 0) {
        // 错误处理
        perror("fork failed");
        exit(1);
    } else if (pid == 0) {
        // 子进程
        execl("/bin/ls", "ls", "-l", NULL);
        // 如果exec执行成功，不会执行到这里
        // 如果执行到这里，说明exec失败
        perror("exec failed");
        exit(1);
    } else {
        // 父进程
        wait(NULL); // 等待子进程结束
    }
    return 0;
}
```

### `vfork()` - 轻量级进程创建

**特点**：

-   子进程共享父进程地址空间
-   子进程优先运行，父进程阻塞
-   子进程必须立即调用`exec()`或`_exit()`

**使用场景**：

-   内存受限系统
-   需要立即执行新程序的场景

**vfork demo**

```c
#include "stdio.h"
#include "stdlib.h"
#include "unistd.h"
#include "sys/wait.h"

int main() {
    pid_t pid = vfork();
    if (pid == 0) {
        execl("/bin/ls", "ls", "-l", NULL);
        _exit(127); // 仅exec失败时执行
    } else if (pid > 0) {
        int status;
        waitpid(pid, &status, 0);
    }
    return 0;
}
```

### `clone` - 灵活的进程/线程创建

**特点**：

-   可以精确控制与父进程共享的资源
-   可以创建进程或线程
-   通过flags参数控制共享特性

**使用场景**：

-   需要细粒度控制资源共享
-   实现特殊的线程模型
-   容器技术（如Docker使用clone实现namespace隔离）

### 总结

这些进程创建方式各有特点，选择时需要根据具体场景考虑内存使用、性能需求和功能需求。一般情况下，fork() 是最安全和最常用的选择。

-   一般场景：使用 fork() - 最通用、最安全的选择
-   执行新程序：使用 fork() + exec() 组合
-   特殊需求：需要细粒度控制资源共享时使用 clone()
-   避免的做法：不要在 vfork() 子进程中执行复杂操作

## 进程分离和守护进程(Daemon)

进程分离是指一个进程能够脱离其父进程，即使父进程终止后仍能继续运行。当子进程与其父进程分离时，它会成为一个后台进程或守护进程，能够独立继续运行而不受父进程生命周期的影响。

### 进程分离

```c
#include "stdio.h"
#include "stdlib.h"
#include "unistd.h"
#include "sys/wait.h"

int main() {
    pid_t pid = fork();
    if (pid < 0) {
        perror("fork");
        exit(1);
    }
    if (pid == 0) {
        setsid(); //创建一个新的会话和进程组,子进程分离
        sleep(10);
        printf("child process: pid = %d, ppid = %d\n", getpid(), getppid());
    } else {
        printf("parent process: pid = %d, ppid = %d\n", getpid(), getppid());
    }
    return 0;
}
```

### 守护进程

守护进程是在后台运行的一种特殊进程，与终端或用户会话无关。它们通常在系统启动时启动，并持续运行直到系统关闭。规范的守护进程创建过程中需要进行两次 fork，这是防止守护进程受到终端相关的信号影响，这种方式虽然看起来有点复杂，但是能够最大程度地确保守护进程的独立性和稳定性。

```c
#include "stdio.h"
#include "stdlib.h"
#include "unistd.h"
#include "sys/stat.h"
#include "sys/wait.h"
#include "fcntl.h"

int main() {
    // 第一次 fork
    pid_t pid = fork();
    if (pid < 0) {
        exit(1);
    }
    if (pid > 0) {
        // 父进程退出
        exit(0);
    }

    // 创建新会话
    if (setsid() < 0) {
        exit(1);
    }
    // 第二次 fork
    pid = fork();
    if (pid < 0) {
        exit(1);
    }
    if (pid > 0) {
        // 第一子进程退出
        exit(0);
    }

    //第二次 fork 后不需要调用 setsid
    chdir("/");
    umask(0);

    // 关闭0、1、2文件描述符
    close(0);
    close(1);
    close(2);

    // 将标准输入、输出和错误重定向到 /dev/null
    open("/dev/null", O_RDWR);
    dup(0);
    dup(0);
    printf("child process: pid = %d, ppid = %d\n", getpid(), getppid());
    // 在此处添加你的守护进程任务代码
    return 0;
}
```

## 面试常见问题

### 什么是僵尸进程？如何避免僵尸进程？

子进程死亡时父进程没有回收，这样就造成子进程资源无法回收，通常用 ps 可以看到它被显示为defunct，这样就产生了僵尸进程。它将永远保持这样直到父进程回收。

-   父进程主动回收子进程（wait/waitpid）；
-   父进程注册SIGCHLD信号，在回调中进程回收；
-   父进程忽略SIGCHLD、SIGCLD信号，子进程结束后由内核回收；
-   fork两次，父进程fork一个子进程，然后继续工作，子进程fork一个孙进程后退出，那么孙进程被init接管，孙进程结束后，init会回收。

```v
// 方法1: 父进程调用wait/waitpid
while (waitpid(-1, NULL, WNOHANG) > 0);

// 方法2: 父进程注册SIGCHLD信号，在回调中进程回收；
sigaction(SIGCHLD, &sigaction_fd, NULL)

// 方法3: 忽略SIGCHLD信号
signal(SIGCHLD, SIG_IGN);

// 方法4: 双fork技术
if (fork() == 0) {
    if (fork() == 0) {
        // 实际工作进程
    }
    exit(0); // 中间进程立即退出
}
wait(NULL); // 父进程回收中间进程
```

### 如何实现进程热升级？

使用fork配合execl族函数实现热升级。

