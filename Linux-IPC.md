---
title: 深入浅出 Linux IPC：进程间通信的艺术
layout: post
tags:
  - Linux
  - IPC
categories:
  - Linux
  - 内核编程
ai: Gemini
ai_ver: 2.5 Flash
abbrlink: 46571
date: 2025-09-12 17:28:41
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

- [1. 为什么需要 IPC？](#1-为什么需要-ipc)
- [2. 常见的 Linux IPC 机制](#2-常见的-linux-ipc-机制)
  - [2.1. 管道（Pipes）](#21-管道pipes)
  - [2.2. FIFO（命名管道）](#22-fifo命名管道)
  - [2.3. 信号](#23-信号)
    - [2.3.1. 信号的分类](#231-信号的分类)
    - [2.3.2. 信号与 IPC 的关系](#232-信号与-ipc-的关系)
    - [2.3.3. 信号的三种处理方式](#233-信号的三种处理方式)
    - [2.3.4. 如何使用信号？](#234-如何使用信号)
      - [2.3.4.1. 发送信号：`kill()` 函数](#2341-发送信号kill-函数)
      - [2.3.4.2. 注册信号处理函数：`signal()` 和 `sigaction()`](#2342-注册信号处理函数signal-和-sigaction)
  - [2.4. 信号集（Signal Set）和阻塞](#24-信号集signal-set和阻塞)
    - [2.4.1. 信号集操作函数](#241-信号集操作函数)
    - [2.4.2. 阻塞信号：`sigprocmask()`](#242-阻塞信号sigprocmask)
  - [2.5. System V IPC (System V IPC)\[^1\]](#25-system-v-ipc-system-v-ipc)
    - [2.5.1. 消息队列（Message Queues）](#251-消息队列message-queues)
    - [2.5.2. 信号量（Semaphores）](#252-信号量semaphores)
    - [2.5.3. 共享内存（Shared Memory）](#253-共享内存shared-memory)
  - [2.6. IPC 总结与选择](#26-ipc-总结与选择)


在 Linux 世界中，进程是独立的执行单元，拥有自己的地址空间。但很多时候，为了完成一个复杂的任务，不同的进程需要协同工作，交换数据。这时，我们就需要**进程间通信（IPC, Inter-Process Communication）**。IPC 就像是进程之间的一座桥梁，让它们能够相互“交谈”，共享信息。

本文将带你深入了解 Linux 中常见的 IPC 机制，并以**使用为导向**，结合代码示例，让你能够快速掌握这些“通信”技术。

## 1. 为什么需要 IPC？

想象一个场景：你正在开发一个 Web 服务器。一个主进程负责监听网络请求，但处理这些请求非常耗时。如果主进程自己处理，服务器就会变得很慢，无法响应新的请求。一个更好的设计是，主进程每接收到一个请求，就创建一个新的子进程或将请求发送给一个工作进程池来处理。这样，主进程可以立即回去监听新的连接，而工作进程则专注于处理任务。

在这个例子中，主进程需要将请求数据传递给工作进程。这就是 IPC 发挥作用的地方。

## 2. 常见的 Linux IPC 机制

Linux 提供了多种 IPC 机制，每种都有其独特的优缺点和适用场景。我们可以将它们分为两大类：**基于文件**和**基于内存**。

### 2.1. 管道（Pipes）

管道可能是最简单、最古老的 IPC 形式。它就像一个单向的“水管”，一端用于写入，另一端用于读取。

- **特点**：

    - **单向通信**：数据只能从一端流向另一端。

    - **父子进程通信**：管道通常用于有亲缘关系的进程之间，比如父进程和子进程。

    - **半双工**：虽然是单向，但如果创建两个管道，就可以实现双向通信。

- **使用**：

    - **`pipe()` 函数**：这是创建管道的核心函数。

    ```c
    #include <unistd.h>
    int pipe(int pipefd[2]);
    ```

    `pipefd` 是一个包含两个文件描述符的数组，`pipefd[0]` 用于读取，`pipefd[1]` 用于写入。

- **代码示例**：一个简单的父子进程通信。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char buf[20];
    const char *msg = "Hello from parent!";

    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();
    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid > 0) { // 父进程
        close(pipefd[0]); // 关闭读取端
        write(pipefd[1], msg, strlen(msg) + 1);
        close(pipefd[1]); // 关闭写入端，如果不关，子进程读端收不到EOF信号，则会一直读导致程序阻塞
        wait(NULL);
    } else { // 子进程
        close(pipefd[1]); // 关闭写入端，如果不关
        read(pipefd[0], buf, sizeof(buf));
        printf("Child received: %s\n", buf);
        close(pipefd[0]); // 关闭读取端
    }

    return 0;
}
```

### 2.2. FIFO（命名管道）

管道只能用于有亲缘关系的进程，那如果两个毫不相关的进程想通信怎么办？答案就是 **FIFO (First-In, First-Out)**，也叫**命名管道**。

- **特点**：

    - **文件系统路径**：它在文件系统中有一个路径名，不同于匿名管道。

    - **非亲缘进程通信**：任意两个进程都可以通过这个路径名打开并通信。

    - **单向**：和管道一样，FIFO 也是单向的，需要两个 FIFO 来实现双向通信。

- **使用**：

    - **`mkfifo()` 函数**或 **`mkfifo` 命令**：用来创建 FIFO。


    ```c
    #include <sys/types.h>
    #include <sys/stat.h>
    int mkfifo(const char *pathname, mode_t mode);
    ```

    - **`open()`, `read()`, `write()` 函数**：像操作普通文件一样来操作 FIFO。

- **代码示例**：

    - **`writer.c`**：

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    #define FIFO_NAME "my_fifo"

    int main() {
        int fd;
        char *msg = "Hello from writer!";

        mkfifo(FIFO_NAME, 0666);
        fd = open(FIFO_NAME, O_WRONLY);
        write(fd, msg, strlen(msg) + 1);
        close(fd);
        unlink(FIFO_NAME); // 删除命名管道文件
        return 0;
    }
    ```

    - **`reader.c`**：

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    #define FIFO_NAME "my_fifo"

    int main() {
        int fd;
        char buf[20];

        mkfifo(FIFO_NAME, 0666); // 保证文件存在
        fd = open(FIFO_NAME, O_RDONLY);
        read(fd, buf, sizeof(buf));
        printf("Reader received: %s\n", buf);
        close(fd);
        return 0;
    }
    ```

    你可以先运行 `writer.c`，再运行 `reader.c`。



### 2.3. 信号

**信号（Signal）** 是一种更轻量级、更异步的进程间通信和事件通知机制。它就像一个“软中断”，用来通知进程发生了某个事件。

想象一下，你正在专注地工作，突然有人拍了你一下肩膀。你停下手中的活，转头看看发生了什么事，然后根据情况做出反应（比如，对方是同事，你可能和他聊两句；对方是领导，你可能马上站起来）。

在 Linux 中，**信号**就是那个“拍肩膀”的动作。当一个进程收到一个信号时，它会暂停当前执行的任务，转而去处理这个信号，处理完后再恢复执行。

- **发送者**：可以是内核（比如你按下 `Ctrl+C`，内核会发送 `SIGINT` 信号给前台进程）、也可以是其他进程（使用 `kill()` 函数）。

- **接收者**：任何一个进程都可以接收信号。


#### 2.3.1. 信号的分类

信号有很多种，每种都有其特定的用途。常见的信号及其作用如下：

|信号名称|默认行为|解释|
|---|---|---|
|**`SIGHUP` (1)**|终止进程|当终端关闭时发送给关联的进程。|
|**`SIGINT` (2)**|终止进程|来自键盘中断，通常是 `Ctrl+C`。|
|**`SIGQUIT` (3)**|终止并生成核心转储文件|来自键盘退出，通常是 `Ctrl+\\`。|
|**`SIGKILL` (9)**|强制终止进程|无法被捕获、阻塞或忽略，强制杀死进程。|
|**`SIGTERM` (15)**|终止进程|友好的终止请求，可以被捕获。`kill` 命令默认发送此信号。|
|**`SIGCHLD`**|忽略|子进程终止或停止时发送给父进程。|
|**`SIGSTOP`**|停止进程|无法被捕获、忽略，暂停进程。|
|**`SIGCONT`**|继续进程|使停止的进程继续运行。|

#### 2.3.2. 信号与 IPC 的关系

- **异步通知**：信号是典型的异步 IPC 机制，它不像管道或共享内存那样传递数据，而是**传递事件信息**。

- **轻量级**：相比于其他 IPC，信号的开销非常小。

- **同步**：信号也可以用于同步目的，例如 `SIGCHLD` 信号常用于父进程等待子进程结束。


理解信号，特别是信号集和阻塞的概念，对于编写健壮的多进程或多线程程序至关重要。它能让你更好地控制程序对外部事件的响应。


#### 2.3.3. 信号的三种处理方式

当进程收到一个信号时，它可以有三种处理方式：

1. **执行默认动作（Default）**：大多数信号都有一个预定义的默认行为。例如，`SIGINT` 的默认行为就是终止进程。

2. **忽略信号（Ignore）**：有些信号可以被忽略，即进程收到信号后不做任何处理。`SIGCHLD` 信号的默认行为就是忽略。

3. **捕获信号（Catch）**：这是最灵活的方式。进程可以为某个信号注册一个**信号处理函数（Signal Handler）**。当信号到来时，进程会执行这个函数来处理信号，而不是执行默认动作。

!!! caution
    **注意**：`SIGKILL` 和 `SIGSTOP` 这两个信号是**不能被捕获、忽略或阻塞**的。它们是系统管理员强制终止或停止进程的“最后手段”。



#### 2.3.4. 如何使用信号？

##### 2.3.4.1. 发送信号：`kill()` 函数

你可以使用 `kill()` 函数向另一个进程发送信号。

```c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```

- `pid`：目标进程的 ID。

- `sig`：要发送的信号编号。


##### 2.3.4.2. 注册信号处理函数：`signal()` 和 `sigaction()`

- **`signal()` 函数**：这是最简单的注册方式，但它在不同系统上的行为可能不一致，不推荐在新代码中使用。

- **`sigaction()` 函数**：这是 POSIX 标准推荐的方式，更强大，更可靠。


```c
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

- `signum`：要捕获的信号编号。

- `act`：指向 `struct sigaction` 结构体，该结构体定义了新的信号处理行为。

- `oldact`：可选，用于保存旧的信号处理行为。


**`struct sigaction` 结构体**：
```c
struct sigaction {
    void (*sa_handler)(int);  // 信号处理函数
    sigset_t sa_mask;         // 信号集，在信号处理函数执行期间需要阻塞的信号
    int sa_flags;             // 标志位
};
```

### 2.4. 信号集（Signal Set）和阻塞

当你在处理一个信号时，你可能不希望被其他信号打断。**信号集（`sigset_t`）** 就是用来管理一组信号的。通过操作信号集，你可以**阻塞（Block）** 某些信号，让它们在进程处理完当前任务后才被传递。

#### 2.4.1. 信号集操作函数

- **`sigemptyset()`**：初始化一个空的信号集。

- **`sigaddset()`**：向信号集中添加一个信号。

- **`sigdelset()`**：从信号集中删除一个信号。

- **`sigismember()`**：检查一个信号是否在信号集中。


#### 2.4.2. 阻塞信号：`sigprocmask()`

`sigprocmask()` 函数用来设置进程的**信号阻塞掩码（Signal Mask）**。

```c
#include <signal.h>
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```

- `how`：指定如何修改信号阻塞掩码，有以下几种：

    - `SIG_BLOCK`：将 `set` 中的信号添加到阻塞掩码中。

    - `SIG_UNBLOCK`：将 `set` 中的信号从阻塞掩码中移除。

    - `SIG_SETMASK`：将阻塞掩码设置为 `set`。

- `set`：包含要阻塞或解除阻塞的信号集。

- `oldset`：可选，用于保存旧的阻塞掩码。


**示例**：在处理关键代码段时临时阻塞 `SIGINT`。

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void my_handler(int sig) {
    printf("Caught signal %d\n", sig);
}

int main() {
    sigset_t block_mask, old_mask;

    // 1. 设置要阻塞的信号集
    sigemptyset(&block_mask);
    sigaddset(&block_mask, SIGINT);

    // 2. 阻塞 SIGINT 信号
    sigprocmask(SIG_BLOCK, &block_mask, &old_mask);

    printf("SIGINT is blocked. Press Ctrl+C...\n");
    sleep(10); // 在这10秒内，Ctrl+C不会终止进程

    printf("Unblocking SIGINT...\n");
    // 3. 解除阻塞，恢复旧的信号掩码
    sigprocmask(SIG_SETMASK, &old_mask, NULL);

    printf("SIGINT is unblocked. Press Ctrl+C again.\n");

    // 4. 注册一个信号处理函数
    struct sigaction sa;
    sa.sa_handler = my_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sigaction(SIGINT, &sa, NULL);

    while(1) {
        printf("Running...\n");
        sleep(1);
    }

    return 0;
}
```

运行这段代码，你会看到在阻塞期间，`Ctrl+C` 无法终止进程。当解除阻塞后，`Ctrl+C` 才能触发信号处理函数。


### 2.5. System V IPC (System V IPC)[^1]

System V IPC 是 Linux 系统中一组更高级、更强大的 IPC 机制，包括消息队列、信号量和共享内存。它们都是**基于内核**的，需要一个唯一的键值（key）来标识。

#### 2.5.1. 消息队列（Message Queues）

消息队列就像一个链表，允许进程向其中添加消息或从中读取消息。

- **特点**：

    - **异步通信**：发送进程可以发送消息后立即返回，不需要等待接收进程。

    - **带类型**：消息可以带有类型，接收进程可以只接收特定类型的消息。

    - **存储在内核中**：即使发送进程结束，消息依然保留在队列中，直到被读取。

- **使用**：

    - **`ftok()`**：将文件路径和整数转换为一个唯一的 IPC 键值。

    - **`msgget()`**：创建或获取一个消息队列。

    - **`msgsnd()`**：发送消息。

    - **`msgrcv()`**：接收消息。

    - **`msgctl()`**：控制消息队列，如删除。


#### 2.5.2. 信号量（Semaphores）

信号量主要用于**同步**，控制对共享资源的访问。它本身不传递数据，而是作为一种“计数器”。

- **特点**：

    - **互斥和同步**：常用于实现互斥锁，确保同一时间只有一个进程访问共享资源。

    - **原子操作**：信号量的操作（P/V操作）是原子的，不会被中断。

- **使用**：

    - **`semget()`**：创建或获取一组信号量。

    - **`semop()`**：对信号量进行操作，如加/减计数。

    - **`semctl()`**：控制信号量。


#### 2.5.3. 共享内存（Shared Memory）

共享内存是最高效的 IPC 方式。它允许两个或多个进程共享同一块物理内存。

- **特点**：

    - **最高效**：一旦映射到进程的地址空间，读写操作就像访问普通内存一样，不需要内核的参与。

    - **需要同步**：由于多个进程同时访问，需要用**信号量**等机制来同步访问，防止数据竞争。

- **使用**：

    - **`shmget()`**：创建或获取一个共享内存段。

    - **`shmat()`**：将共享内存段附加到进程的地址空间。

    - **`shmdt()`**：将共享内存段从进程的地址空间分离。

    - **`shmctl()`**：控制共享内存，如删除。

### 2.6. IPC 总结与选择

|机制|适用场景|优点|缺点|
|---|---|---|---|
|**管道**|有亲缘关系的进程|简单，易于使用|单向，仅限于亲缘进程|
|**FIFO**|无亲缘关系的进程|可以在文件系统中命名，灵活|单向，需要同步，读写时有阻塞|
|**消息队列**|异步通信，少量数据|消息带类型，无需同步|效率较低，有大小限制|
|**信号量**|进程间同步，互斥|用于控制访问，防止竞争|不传递数据|
|**共享内存**|大量数据传输|最高效，读写速度快|必须配合其他同步机制使用|

**如何选择？**

- 如果是父子进程之间少量数据的通信，**管道**是最佳选择。

- 如果是两个不相关的进程，且数据量不大，**消息队列**是一个不错的方案。

- 如果需要传输大量数据，且对性能要求极高，**共享内存**是首选，但**必须**结合**信号量**或其他同步机制。

- 如果你只想解决资源访问的同步问题，**信号量**是专门为此设计的。


了解这些 IPC 机制，就如同掌握了进程之间“沟通”的多种语言。在开发时，选择合适的“语言”能让你的程序更加健壮、高效。现在，你可以尝试用这些机制来解决你遇到的实际问题了！

[^1]: 作了解，重点使用POSIX IPC。