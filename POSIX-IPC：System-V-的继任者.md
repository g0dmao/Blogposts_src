---
title: POSIX IPC：System V 的继任者
layout: post
tags:
  - Linux
  - IPC
  - POSIX
categories:
  - Linux
  - 内核编程
ai: Gemini
ai_ver: 2.5 Flash
abbrlink: 26506
date: 2025-09-15 22:24:58
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---
- [1. POSIX IPC：System V 的继任者](#1-posix-ipcsystem-v-的继任者)
  - [1.1. POSIX 消息队列](#11-posix-消息队列)
    - [1.1.1. 消息队列属性](#111-消息队列属性)
  - [1.2. POSIX 信号量](#12-posix-信号量)
  - [1.3. POSIX 共享内存](#13-posix-共享内存)


## 1. POSIX IPC：System V 的继任者

**POSIX IPC**（Portable Operating System Interface）是一套新的 IPC 标准，旨在解决 **System V IPC** 的一些局限性。它提供了一套更统一、更现代的 API，使用**文件名**作为标识符，而不是 System V 的**键值**，这使得 IPC 资源的管理更加直观。

### 1.1. POSIX 消息队列

与 System V 消息队列类似，但 API 更简洁。

- **特点**：

    - **基于文件**：通过 `/dev/mqueue` 目录下的文件名来标识。

    - **优先级**：支持消息优先级，高优先级的消息会被优先处理。

    - **非阻塞模式**：可以设置消息队列为非阻塞模式，防止 `mq_send` 或 `mq_receive` 阻塞进程。

- **使用**：

    - **`mq_open()`**：创建或打开一个消息队列。

    - **`mq_send()`**：发送消息。

    - **`mq_receive()`**：接收消息。

    - **`mq_close()`**：关闭消息队列。

    - **`mq_unlink()`**：删除消息队列。

#### 1.1.1. 消息队列属性

在POSIX消息队列中，**消息队列属性**（`mq_attr`）是用来设置和获取消息队列的特性和参数的结构体。它的主要作用是控制消息队列的行为，例如最大消息数、消息大小等。`mq_attr`结构体包含了以下几个字段：
```c
struct mq_attr {
    long    mq_flags;      // 消息队列的标志
    long    mq_maxmsg;     // 消息队列中最多可以容纳的消息数量
    long    mq_msgsize;    // 队列中每条消息的最大大小
    long    mq_curmsgs;    // 当前队列中的消息数
};
```

1. `mq_flags`

	这个字段用于设置消息队列的标志。常见的标志有：
	
	- **`O_NONBLOCK`**：以非阻塞模式打开消息队列。这意味着如果消息队列为空，接收操作（`mq_receive()`）会立即返回，而不是阻塞等待。
	
	- **`O_RDWR`**：允许对消息队列进行读写操作。
	
	如果没有设置这些标志，消息队列将会默认阻塞模式操作。

2. `mq_maxmsg`

	这个字段定义了消息队列可以容纳的最大消息数量。如果消息队列已经存满，且没有空间容纳新的消息，后续的消息发送（`mq_send()`）将会被阻塞，直到队列中有足够的空间。这个属性通常设置为你希望消息队列中能存储的消息个数。

3. `mq_msgsize`

	此字段定义每条消息的最大字节数。在创建消息队列时，必须确保消息发送和接收操作的消息大小不会超过这个值。如果消息超过了这个大小，发送操作将返回错误。

4. `mq_curmsgs`

	这个字段是一个只读属性，用于获取当前队列中的消息数量。它是一个动态更新的值，每次读取时会返回当前消息队列中实际存储的消息数。这个字段对应用程序来说很有用，尤其是在需要了解队列当前状态的场景。

这个例子展示了两个不相关的进程如何通过 POSIX 消息队列进行通信。
发送方(sender.c)：
```c
#include <stdio.h>
#include <stdlib.h>          /* 动态内存管理及其他辅助功能如exit、abort */
#include <string.h>
#include <fcntl.h>           /* 提供 O_CREAT, O_RDWR 用于文件控制 */
#include <sys/stat.h>        /* 文件属性操作 如chmod、umask */
#include <mqueue.h>          /* 提供POSIX消息队列支持 */
#include <unistd.h>          /* 提供POSIX接口 */

#define MQ_NAME "/my_mq"

int main() {
    mqd_t mqd;
    struct mq_attr attr;
    char buffer[256];
    const char *msg = "Hello from sender!";

    // 设置消息队列属性
    attr.mq_flags = 0;
    attr.mq_maxmsg = 10;
    attr.mq_msgsize = 256;
    attr.mq_curmsgs = 0;

    // 创建或打开消息队列
    mqd = mq_open(MQ_NAME, O_CREAT | O_WRONLY, 0666, &attr);
    if (mqd == (mqd_t)-1) {
        perror("mq_open");
        exit(EXIT_FAILURE);
    }

    printf("Sender: Sending message...\n");

    // 发送消息，优先级为1
    if (mq_send(mqd, msg, strlen(msg) + 1, 1) == -1) {
        perror("mq_send");
        exit(EXIT_FAILURE);
    }

    // 关闭消息队列描述符
    mq_close(mqd);

    printf("Sender: Message sent and mq closed.\n");
    return 0;
}
```

接收方(receiver.c)：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <mqueue.h>
#include <unistd.h>

#define MQ_NAME "/my_mq"

int main() {
    mqd_t mqd;
    struct mq_attr attr;
    char buffer[256 + 1];
    unsigned int prio;

    // 打开消息队列
    mqd = mq_open(MQ_NAME, O_RDONLY);
    if (mqd == (mqd_t)-1) {
        perror("mq_open");
        exit(EXIT_FAILURE);
    }

    // 获取消息队列属性
    mq_getattr(mqd, &attr);

    printf("Receiver: Waiting for message...\n");

    // 接收消息
    if (mq_receive(mqd, buffer, attr.mq_msgsize, &prio) == -1) {
        perror("mq_receive");
        exit(EXIT_FAILURE);
    }

    printf("Receiver: Received message: %s (Priority: %u)\n", buffer, prio);

    // 关闭和删除消息队列
    mq_close(mqd);
    mq_unlink(MQ_NAME);

    return 0;
}
```

**编译和运行**： `gcc sender.c -o sender -lrt` `gcc receiver.c -o receiver -lrt` （需要链接实时库 `-lrt`） 先运行 `./sender`，再运行 `./receiver`。

### 1.2. POSIX 信号量

用于进程间的同步，功能与 System V 信号量类似，但提供了更简单的接口。

- **特点**：

    - **有名信号量**：通过一个文件名标识，可以用于非亲缘进程。

    - **无名信号量**：常用于线程间的同步，存放在共享内存中，只能用于有亲缘关系的进程。

- **使用**：

    - **`sem_open()`**：创建或打开有名信号量。

    - **`sem_wait()`**：原子性地减少信号量计数器，如果为0则阻塞。

    - **`sem_post()`**：原子性地增加信号量计数器。

    - **`sem_close()`**：关闭信号量。

    - **`sem_unlink()`**：删除信号量。

这个例子展示了如何使用 POSIX 信号量来同步两个进程对一个共享资源的访问。

共享资源访问程序 (sem_client.c)：
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>           /* For O_CREAT, O_RDWR */
#include <sys/stat.h>        /* For mode constants */
#include <semaphore.h>
#include <unistd.h>

#define SEM_NAME "/my_semaphore"

int main() {
    sem_t *sem;
    int value;

    // 打开信号量
    sem = sem_open(SEM_NAME, 0); // O_CREAT is not needed here
    if (sem == SEM_FAILED) {
        perror("sem_open");
        exit(EXIT_FAILURE);
    }

    printf("Client: Waiting to get semaphore...\n");

    // P 操作：等待信号量，如果值为0则阻塞
    if (sem_wait(sem) == -1) {
        perror("sem_wait");
        exit(EXIT_FAILURE);
    }

    printf("Client: Got the semaphore. Accessing shared resource...\n");
    sleep(2); // 模拟访问共享资源

    printf("Client: Finished. Releasing semaphore...\n");

    // V 操作：释放信号量，计数器加1
    if (sem_post(sem) == -1) {
        perror("sem_post");
        exit(EXIT_FAILURE);
    }

    // 关闭信号量
    sem_close(sem);

    return 0;
}
```

信号量控制程序 (sem_main.c)：`
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>
#include <unistd.h>
#include <sys/wait.h>

#define SEM_NAME "/my_semaphore"

int main() {
    sem_t *sem;
    pid_t pid;

    // 创建并初始化信号量，初始值为1
    sem = sem_open(SEM_NAME, O_CREAT, 0666, 1);
    if (sem == SEM_FAILED) {
        perror("sem_open");
        exit(EXIT_FAILURE);
    }

    printf("Main: Semaphore created and initialized to 1.\n");

    // 创建一个子进程
    pid = fork();
    if (pid == 0) {
        // 子进程运行另一个程序来访问信号量
        execlp("./sem_client", "sem_client", NULL);
        perror("execlp"); // 如果执行失败
        exit(EXIT_FAILURE);
    } else if (pid > 0) {
        // 父进程也尝试访问信号量
        sleep(1); // 确保子进程先运行
        printf("Main: Waiting to get semaphore...\n");
        sem_wait(sem);
        printf("Main: Got the semaphore. Accessing shared resource...\n");
        sleep(2);
        printf("Main: Finished. Releasing semaphore...\n");
        sem_post(sem);

        wait(NULL); // 等待子进程结束

        // 删除信号量
        sem_unlink(SEM_NAME);
        printf("Main: Child process finished. Semaphore unlinked.\n");
    }

    // 关闭信号量
    sem_close(sem);

    return 0;
}
```

**编译和运行**： `gcc sem_client.c -o sem_client -lrt` `gcc sem_main.c -o sem_main -lrt` （同样需要链接实时库 `-lrt`） 运行 `./sem_main`。



### 1.3. POSIX 共享内存

和 System V 共享内存一样，都是最快的 IPC 方式，但 POSIX 版本使用了文件描述符。

- **特点**：

    - **基于文件**：通过 `shm_open` 创建或打开一个共享内存对象，返回一个文件描述符。

    - **内存映射**：通过 `mmap()` 将文件描述符对应的内存映射到进程的地址空间。

- **使用**：

    - **`shm_open()`**：创建或打开共享内存对象。

    - **`ftruncate()`**：调整共享内存对象的大小。

    - **`mmap()`**：将共享内存映射到进程地址空间。

    - **`munmap()`**：解除映射。

    - **`shm_unlink()`**：删除共享内存对象。

这个例子展示了两个进程如何通过 POSIX 共享内存来共享一块内存区域，并用一个信号量来同步。

写入方 (shm_writer.c)
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/mman.h>
#include <semaphore.h>
#include <unistd.h>

#define SHM_NAME "/my_shm"
#define SEM_NAME "/my_shm_sem"
#define SHM_SIZE 4096

int main() {
    int shm_fd;
    void *ptr;
    sem_t *sem;
    const char *msg = "Hello from shm writer!";

    // 创建/打开共享内存对象
    shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    // 设置共享内存大小
    ftruncate(shm_fd, SHM_SIZE);

    // 将共享内存映射到进程地址空间
    ptr = mmap(0, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(EXIT_FAILURE);
    }

    // 创建/打开信号量，用于同步
    sem = sem_open(SEM_NAME, O_CREAT, 0666, 0); // 初始值为0，表示不可访问
    if (sem == SEM_FAILED) {
        perror("sem_open");
        exit(EXIT_FAILURE);
    }

    printf("Writer: Writing to shared memory...\n");
    // 写入数据
    sprintf(ptr, "%s", msg);

    // V 操作：释放信号量，通知读者可以读取了
    sem_post(sem);
    printf("Writer: Data written and semaphore posted.\n");

    // 关闭
    sem_close(sem);
    munmap(ptr, SHM_SIZE);
    close(shm_fd);

    return 0;
}
```

读取方 (shm_reader.c)：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/mman.h>
#include <semaphore.h>
#include <unistd.h>

#define SHM_NAME "/my_shm"
#define SEM_NAME "/my_shm_sem"
#define SHM_SIZE 4096

int main() {
    int shm_fd;
    void *ptr;
    sem_t *sem;

    // 打开共享内存对象
    shm_fd = shm_open(SHM_NAME, O_RDONLY, 0666);
    if (shm_fd == -1) {
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    // 将共享内存映射到进程地址空间
    ptr = mmap(0, SHM_SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(EXIT_FAILURE);
    }

    // 打开信号量
    sem = sem_open(SEM_NAME, 0);
    if (sem == SEM_FAILED) {
        perror("sem_open");
        exit(EXIT_FAILURE);
    }

    printf("Reader: Waiting for writer...\n");
    // P 操作：等待信号量，直到有数据可用
    sem_wait(sem);

    printf("Reader: Data received: %s\n", (char *)ptr);

    // 关闭和清理
    sem_close(sem);
    sem_unlink(SEM_NAME); // 删除信号量
    munmap(ptr, SHM_SIZE);
    close(shm_fd);
    shm_unlink(SHM_NAME); // 删除共享内存对象

    return 0;
}
```

**编译和运行**： `gcc shm_writer.c -o shm_writer -lrt` `gcc shm_reader.c -o shm_reader -lrt` （同样需要链接实时库 `-lrt`） 先运行 `./shm_writer`，再运行 `./shm_reader`。



