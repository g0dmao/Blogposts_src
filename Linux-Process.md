---
title: Linux-Process
layout: post
tags:
  - Linux
  - 内核
  - 进程
categories:
  - Linux
  - 内核编程
abbrlink: 43741
date: 2025-09-10 14:53:13
ai: ChatGPT
ai_ver: "5"
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

!!! info ""
    AI 辅助创作


在 Linux 编程中，**进程（Process）** 是操作系统分配资源和调度的基本单位。每个进程都有自己独立的内存空间、代码段、数据段和执行上下文。多个进程之间常常需要协作，这就涉及到 **进程间通信（IPC, Inter-Process Communication）**。本篇博客会系统介绍进程的基础操作，并对常见的 IPC 方式进行详细解释与示例，带你从零开始掌握。

---

## 1. 进程与 PID

每个进程都会有一个唯一的 **进程号（PID）**，可以通过 `getpid()` 获取当前进程号，通过 `getppid()` 获取父进程号。这在调试和进程管理时非常重要。

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("当前进程PID=%d, 父进程PID=%d\n", getpid(), getppid());
    return 0;
}
```

⚠️ 注意：如果父进程先退出，子进程会被 `init` 进程接管，`ppid` 会变为 1。

## 2. 创建进程 — `fork()`

`fork()` 是创建新进程的主要方式。它会复制当前进程（父进程）的大部分内容，生成一个几乎一模一样的子进程。

```c
pid_t pid = fork();
```

- **在父进程中，`fork()` 返回子进程的 PID。**
    
- **在子进程中，`fork()` 返回 0。**
    
- 如果失败，返回 -1。
    

示例：

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("[子进程] pid=%d, ppid=%d\n", getpid(), getppid());
    } else if (pid > 0) {
        printf("[父进程] pid=%d, 子进程pid=%d\n", getpid(), pid);
    } else {
        perror("fork 失败");
    }
    return 0;
}
```

!!! caution
    - 子进程不会继承父进程的所有资源，例如文件锁。
    
    - `fork()` 之后，父子进程执行顺序不确定，需要通过 `wait()` 或其他机制控制。


## 3. 程序替换 — `exec()` 系列

子进程常常需要运行一个新的程序，这就需要 `exec()` 系列函数（如 `execlp`, `execvp` 等）。
`exec` 系列函数的作用是：**用新的程序替换当前进程的代码段**，而进程的 PID、文件描述符（除非设置了 `FD_CLOEXEC`）等保持不变。它不会创建新进程，而是在当前进程内加载并运行新程序。
### 常见函数

`exec` 系列一共有 6 个常用函数，主要区别在于 **参数传递方式** 和 **是否依赖 PATH 环境变量**。

|函数|参数传递方式|是否搜索 PATH|说明|
|---|---|---|---|
|`execl(path, arg0, arg1, ..., NULL)`|列表（逐个传参）|否|最常见，参数以不定长列表形式传递，最后必须以 `NULL` 结尾。|
|`execv(path, argv[])`|数组|否|参数通过字符串数组传递，方便动态构造参数。|
|`execlp(file, arg0, arg1, ..., NULL)`|列表|是|与 `execl` 类似，但会在 `PATH` 中搜索可执行文件。|
|`execvp(file, argv[])`|数组|是|与 `execv` 类似，但会在 `PATH` 中搜索可执行文件。|
|`execle(path, arg0, arg1, ..., NULL, envp[])`|列表|否|可以自定义环境变量，最后参数需提供 `envp`。|
|`execve(path, argv[], envp[])`|数组|否|最底层的系统调用，其它 `exec` 函数最终都调用它。|

说明：
- `PATH`：系统环境变量
- `path`：要执行的程序路径（可以是绝对路径，也可以是相对路径）。
- `arg`：程序要输入的参数
### 使用示例

1. **execl：用绝对路径调用程序**

```c
#include <unistd.h>
#include <stdio.h> 

int main() {     
printf("Before exec\n");     
execl("/bin/ls", "ls", "-l", NULL);     
// 只有 exec 调用失败时才会继续执行    
perror("execl failed");     
return 1; }
```

2. **execvp：在 PATH 中搜索命令**
    
```c
#include <unistd.h>
#include <stdio.h>

int main() {
    char *argv[] = {"ls", "-a", NULL};
    execvp("ls", argv);
    perror("execvp failed");
    return 1;
}
```

3. **execle：传入自定义环境变量**
    
```c
#include <unistd.h>
#include <stdio.h>

int main() {
    char *envp[] = {"MYVAR=HelloExec", NULL};
    execl("/usr/bin/env", "env", NULL, envp);
    perror("execle failed");
    return 1;
}
```

### 注意事项

- `exec` 系列 **不会返回**，除非调用失败。
    
- 若调用成功，原进程的代码和数据段会被新程序替换。
    
- **文件描述符默认继承**，可以用于在父进程中设置好管道/重定向，让子进程在 exec 后继续使用。
    
- 需要保证参数列表最后有 `NULL`，否则会引发不可预期错误。
    

## 4. 等待子进程 — `wait()` / `waitpid()`

**父进程需要回收子进程退出时的资源，否则子进程会成为 僵尸进程（Zombie）。**

这两个函数用于 **父进程等待子进程结束** 并**回收资源**，避免僵尸进程。

### 1. wait()

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

pid_t wait(int *status);
```

- **参数**
    
    - `status`：指向整数的指针，用于存储子进程退出状态。如果不关心状态，可以传 `NULL`。
        
- **返回值**
    
    - 成功：返回结束的子进程 PID
        
    - 失败：返回 -1（例如没有子进程存在时）
        
- **使用示例**

```c
#include <stdio.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) { // 子进程
        printf("Child running...\n");
        return 42;
    } else { // 父进程
        int status;
        pid_t cpid = wait(&status); // 等待子进程
        if (WIFEXITED(status)) {   // 判断是否正常退出
            printf("Child %d exited with code %d\n", cpid, WEXITSTATUS(status));
        }
    }
    return 0;
}
```

!!! caution   
    - `wait()` 会阻塞父进程，直到任意一个子进程退出。
        
    - 如果父进程没有子进程，返回 -1 并设置 `errno = ECHILD`。
        

### 2. waitpid()

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

pid_t waitpid(pid_t pid, int *status, int options);
```

- **参数**
    - `pid`：
        
        - `>0` ：等待指定 PID 的子进程
            
        - -1 ：等待任意子进程（功能类似 `wait()`）
            
        - 0 ：等待与调用进程同组的任意子进程
            
        - <-1：等待进程组 ID = |pid| 的任意子进程
            
    - `status`：指向整数的指针，用于保存退出状态
        
    - `options`：控制函数行为，常用值：
        
        - `0`：阻塞等待（默认行为）
            
        - `WNOHANG`：非阻塞，如果没有子进程退出立即返回 0
            
        - `WUNTRACED`：返回被暂停的子进程（收到 SIGSTOP）
            
        - `WCONTINUED`：返回继续运行的子进程（收到 SIGCONT）
            
- **返回值**
    
    - 成功：返回子进程 PID
        
    - 非阻塞且没有子进程退出：返回 0
        
    - 失败：返回 -1，并设置 `errno`
        
- **使用示例**
    

```c
#include <stdio.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) { // 子进程
        sleep(2);
        return 100;
    } else { // 父进程
        int status;
        pid_t cpid;
        while ((cpid = waitpid(pid, &status, WNOHANG)) == 0) {
            printf("Child still running...\n");
            sleep(1);
        }
        if (WIFEXITED(status)) {
            printf("Child %d exited with code %d\n", cpid, WEXITSTATUS(status));
        }
    }
    return 0;
}
```

!!! caution 
    - `waitpid()` 可以实现 **非阻塞等待**，适合父进程同时管理多个子进程。
        
    - `WIFEXITED(status)` 判断子进程是否正常退出。
        
    - `WIFSIGNALED(status)` 判断子进程是否被信号终止。
        
    - 使用 `waitpid()` 可以精准回收指定子进程，避免僵尸进程积累。
        

!!! note "tips"
    - 父进程只需要等待任意子进程 → 用 `wait()` 即可。
    
    - 父进程需要精确控制或非阻塞 → 用 `waitpid()`。
    
    - 调用前一定要理解 **阻塞 vs 非阻塞**，避免父进程被卡住。

## 5. 进程退出 — `exit()` 与 `_exit()`

- `exit()`：会刷新缓冲区，执行清理函数。
    
- `_exit()`：立即退出，不做清理，通常在子进程 `fork()` 后的错误处理中使用。
    

⚠️ 区别：`exit()` 适合普通退出，`_exit()` 适合在 `fork()` 后不希望影响父进程的环境时使用。

!!! caution
    - 子进程执行退出函数后，子进程结束并变为僵尸🧟，内核保留子进程在进程表中的部分信息（PID、退出码等）占用少量系统资源，但不会再运行。需要父进程调用wait释放子进程资源，净化僵尸，释放灵魂👻


## 6. ps 命令简述

### `ps` 命令的常见用法

#### 1. 查看当前终端中的进程

这是最简单的用法，不加任何选项，`ps` 命令只会显示当前终端会话中运行的进程。

```bash
ps
```

运行后你可能会看到类似下面的输出：

```bash
PID TTY          TIME CMD
1234 pts/0    00:00:00 bash
5678 pts/0    00:00:00 ps
```

- **PID**：进程的唯一标识符（Process ID）。
    
- **TTY**：进程运行所在的终端。
    
- **TIME**：进程使用的 CPU 时间。
    
- **CMD**：启动进程的命令。
    

#### 2. 查看所有进程

如果你想查看系统中所有用户的所有进程，可以使用 `-e` 选项（代表所有进程）或 `-a` 选项（代表所有进程，除了会话引导进程）。

```bash
ps -e
```

或者使用 `-ef` 组合，这是最常用的组合之一，它提供了更详细的信息：

```bash
ps -ef
```

- **UID**：启动进程的用户 ID。
    
- **PID**：进程 ID。
    
- **PPID**：父进程 ID。
    
- **C**：CPU 利用率。
    
- **STIME**：进程启动时间。
    
- **TTY**：进程所在的终端。
    
- **TIME**：进程使用的 CPU 时间。
    
- **CMD**：启动进程的命令。
    

#### 3. 查看用户进程

如果你只想查看某个特定用户（例如 `root`）的进程，可以使用 `-u` 选项。

```bash
ps -u root
```

#### 4. 通过管道和 `grep` 查找特定进程

在实际工作中，你通常会需要查找某个特定的进程，比如 Nginx 或 MySQL。这时，`ps` 命令通常会配合 `grep` 命令一起使用。`grep` 可以从 `ps` 的输出中过滤出你想要的信息。

例如，查找所有与 Nginx 相关的进程：

```bash
ps -ef | grep nginx
```

这里的 `|` 是管道符，它的作用是将 `ps -ef` 命令的输出作为 `grep` 命令的输入。

### `ps` 命令的常用选项总结

| 选项   | 描述                   |
| ---- | -------------------- |
| -e   | 显示所有进程               |
| -f   | 显示完整格式的列表            |
| -a   | 显示所有进程（不包括会话引导进程）    |
| -u   | 按用户过滤进程              |
| -aux | 最经典的组合，显示所有用户的详细进程信息 |

### 实际应用示例

假设你想查看当前系统中 CPU 或内存占用最高的几个进程，`ps` 命令可以与 `head`、`sort` 等命令结合使用。

**查看 CPU 占用最高的 5 个进程：**

```bash
ps -aux --sort=-%cpu | head -n 6
```

这里的 `--sort=-%cpu` 表示按 CPU 占用率降序排列，`head -n 6` 则取前 6 行（包括标题行）。

### 解析 `ps -aux` 命令的输出

当你运行 `ps -aux` 命令时，你会看到一个包含多个列的表格。每一列都提供了关于进程的重要信息。下面是对这些列的详细解释：

- **USER**：启动该进程的用户。
    
- **PID**：进程的唯一标识符（**P**rocess **ID**）。当你需要杀死一个进程时，通常会用到这个 ID。
    
- **%CPU**：进程在最近一段时间内使用的 **CPU** 占用率。
    
- **%MEM**：进程使用的 **物理内存** 占用率。
    
- **VSZ**：进程使用的虚拟内存大小（**V**irtual **S**ize），单位为千字节（KB）。
    
- **RSS**：进程使用的 **物理内存** 大小（**R**esident **S**et **S**ize），单位为千字节（KB）。
    
- **TTY**：进程运行所在的终端。如果显示 `?`，表示该进程没有关联的终端，通常是系统进程。
    
- **STAT**：进程的当前状态（**STAT**us）。这是一个非常重要的列，常用的状态码有：
    
    - **R**：正在运行（**R**unning）或可运行（**R**unnable）。
        
    - **S**：可中断的睡眠状态（**S**leeping）。
        
    - **D**：不可中断的睡眠状态（**D**isk sleep），通常表示进程正在等待 I/O 操作。
        
    - **Z**：僵尸进程（**Z**ombie），进程已终止，但其父进程还未对其进行善后处理。
        
    - **T**：停止（**S**topped），进程被信号停止。
        
    - **<**：高优先级进程。
        
    - **N**：低优先级进程。
        
- **START**：进程启动的时间。
    
- **TIME**：进程已经消耗的 **CPU 时间** 总量。这与 %CPU 不同，%CPU 是瞬时值，而 TIME 是累积值。
    
- **COMMAND**：启动该进程的完整命令，包括所有参数。
    

### 示例解析

假设你运行 `ps -aux` 看到以下一行输出：

```bash
root      1234  0.0  0.1 23456 1234 ?        S    Sep01  0:05 /usr/sbin/sshd -D
```

- **USER**：`root` 用户。
    
- **PID**：进程 ID 是 `1234`。
    
- **%CPU**：CPU 占用率是 `0.0%`。
    
- **%MEM**：内存占用率是 `0.1%`。
    
- **VSZ**：虚拟内存大小是 `23456 KB`。
    
- **RSS**：物理内存大小是 `1234 KB`。
    
- **TTY**：`?`，表示没有关联的终端。
    
- **STAT**：`S`，表示进程处于可中断的睡眠状态。
    
- **START**：启动时间是 `Sep01`。
    
- **TIME**：总 CPU 时间是 `0:05`（5秒）。
    
- **COMMAND**：完整的命令是 `/usr/sbin/sshd -D`，这是一个 SSH 服务守护进程。
    

通过这些信息，你可以快速了解一个进程是由谁运行的、它消耗了多少资源、处于什么状态，以及它是什么程序。这对于系统监控和故障排除非常有帮助。