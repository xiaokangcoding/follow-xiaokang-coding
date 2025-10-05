# 全网最全、最详细的 Linux 进程间通信方式讲解，你不容错过！

大家好，我是小康。

>📘 原文首发于公众号「跟着小康学编程」，微信搜索 跟着小康学编程 即可关注。
>👉 仓库地址：https://github.com/xiaokangcoding/follow-xiaokang-coding

### 引言

不少小伙伴在学习 Linux 或后端开发的时候，都会问我：“进程间通信有这么多方式，到底该怎么理解？消息队列、管道、共享内存……一堆概念，怎么看都头大！”

其实，Linux 进程间通信（IPC, Inter-Process Communication）并没有想象中那么复杂。只要理解原理，再配合图文示例和代码实践，每一种通信方式都可以像积木一样拼接使用。掌握了这些，你不仅能在项目中自如地管理进程，还能在面试中轻松应对相关考题。

在正式开始之前，我们先回顾一下基础概念：操作系统中运行着许多进程，简单来说，进程就是一个程序的运行实例。比如，你打开了一个浏览器、一个音乐播放器和一个聊天软件，这三个程序就是三个独立的进程。可是问题来了，进程和进程之间如果要“说话”，要互相交换信息，它们该怎么办呢？这就是 IPC 要解决的问题，也是我们这篇文章的重点。

为了帮助大家入门，我专门整理了这篇 **全网最全、最详细的 Linux 进程间通信 讲解**。文章内容结合 思维导图、图解和代码示例，保证一看就懂，学完就能在实际开发中使用。

> 💡 学习建议：边看边动手实践，自己写示例、观察进程行为，是理解 IPC 的最佳方式！
> 
> 如果在学习过程中遇到问题，或者想获取更多 Linux、C/C++、Go 后端开发等技术干货，欢迎关注我的公众号「跟着小康学编程」。有问题也可以添加我的个人微信交流讨论。
> 
> <table>
> <tr>
> <td align="center">
> <img src="https://github.com/xiaokangcoding/follow-xiaokang-coding/raw/main/images/qrcode-wechat-official.png" width="200">
> <br>
> <em>公众号「跟着学小康编程」</em>
> </td>
> <td align="center">
> <img src="https://github.com/xiaokangcoding/follow-xiaokang-coding/raw/main/images/qrcode-personal-wechat.png" width="200">
> <br>
> <em>个人微信（备注：加群）</em>
> </td>
> </tr>
> </table>


#### Linux 所有的进程间通信方式：

![](https://files.mdnice.com/user/48364/6ada5cc1-b0f3-47cb-be10-e0ed99484a64.png)


### 1.管道 （Pipe）

管道是 Linux 中用于进程间通信的一种机制。它们分为两种类型：**匿名管道**和**有名管道**。

#### 匿名管道: 

  **概念**：匿名管道是一种在有亲缘关系的进程间（如父子进程）进行单向数据传输的通信机制，存在于内存中，通常用于临时通信。如果需要双向通信，则一般需要两个管道。

  **简单图解：**

  ![](https://files.mdnice.com/user/48364/c2616d1a-1b1e-431d-901d-037100ccf1ff.png)

  **使用场景**：适用于有亲缘关系的进程间的简单数据传输，例如:父子进程。

  **简单示例：**

```c
  #include <unistd.h>
  int main() {
    int pipefd[2];
    pipe(pipefd); // 创建匿名管道
    if (fork() == 0) { // 子进程
        close(pipefd[1]); // 关闭写端
        //读取数据
        read(pipefd[0],buf,5);
        // ... 
    } else { // 父进程
        close(pipefd[0]); // 关闭读端
        // 写入数据
        write(pipefd[1],"hello",5);
        // ... 
    }
  }
```

#### 有名管道:

**概念：** **有名管道（FIFO，First-In-First-Out）** 是一种特殊类型的文件，用于在不相关的进程之间实现通信。与匿名管道不同，有名管道在文件系统中具有一个实际的路径名。这允许任何具有适当权限的进程打开和使用它，而不仅限于有亲缘关系的进程。

**简单图解：**

![](https://files.mdnice.com/user/48364/2b35ebf8-b8b3-4ad4-bf5c-7f6fe2ebb3bf.png)

**简单说明**：

有名管道是 Linux 中一种特殊的文件，它允许不同的进程通过读写这个文件来相互通信。

**使用场景**：用于本机任何两个进程间的通信，特别是当这些进程没有血缘关系时。

**简单示例：**

```c
// server.c
int main() {
    const char *fifoPath = "/tmp/my_fifo";
    mkfifo(fifoPath, 0666); // 创建有名管道
    char buf[1024];
    int fd;
    // 永久循环，持续监听有名管道
    while (1) {
        fd = open(fifoPath, O_RDONLY); // 打开管道进行读取
        read(fd, buf, sizeof(buf));

        // 打印接收到的消息
        printf("Received: %s\n", buf);
        close(fd);
    }
    return 0;
}

// client.c
int main() {
    const char *fifoPath = "/tmp/my_fifo";
    char buf[1024];
    int fd;
    printf("Enter message: ");    // 获取要发送的消息
    fgets(buf, sizeof(buf), stdin);
    fd = open(fifoPath, O_WRONLY); // 打开管道进行写入
    write(fd, buf, strlen(buf) + 1);
    close(fd);
    return 0
}
```

#### 2.信号 (Signals)

**概念**：
在 Linux 中，信号是一种用于进程间通信（IPC）的机制，允许操作系统或一个进程向另一个进程发送简单的消息。信号主要用于传递关于系统事件的通知，例如中断请求、程序异常、或其他重要事件。每个信号代表了一个特定类型的事件，并且进程可以根据收到的信号执行相应的动作。

信号是异步的，意味着它们可以在任何时间点被发送到进程，通常与进程的正常控制流无关。信号的使用为进程提供了一种处理外部事件和错误的方式。

可以使用命令 `kill -l` 来查看 Linux 系统支持的信号有哪些？

```
~$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
...

```

**使用场景**：

- **异常处理**：当程序遇到运行时错误，比如除以零、非法内存访问等，操作系统会向该进程发送一个适当的信号，如SIGFPE（浮点异常）、SIGSEGV（段错误）。默认情况下：都会使程序终止。
- **外部中断**：用户可以通过特定的键盘输入（最常见的是Ctrl+C）来中断正在终端上运行的进程。这会生成 SIGINT 信号，通常导致程序终止。
- **进程控制**：如使用 kill 命令发送信号来终止或暂停某个进程。
- **定时器和超时**： 程序可以设置定时器，当定时器到期时，会收到 SIGALRM 信号。这常用于限制某些操作的执行时间，确保它们不会占用过多时间。
- **子进程状态变化**：当一个子进程结束或停止时，它的父进程会收到 SIGCHLD 信号。这使得父进程可以监控其子进程的状态变化（从运行到正常退出）。

**简单示例：**
```c
void signal_handler(int signal_num) {
    printf("Received signal: %d\n", signal_num);
}

int main() {
    signal(SIGINT, signal_handler);  // 注册信号处理函数
    // 无限循环，等待信号
    while (1) {
        sleep(1); // 暂停一秒
    }
    return 0;
}
```
在这个例子中，程序设置了一个信号处理函数来处理 SIGINT 信号（通常由 Ctrl+C 产生）。当收到该信号时，signal_handler 函数会被调用。

**以下是对上述代码执行流程的简单图解说明，方便大家理解**：

![](https://files.mdnice.com/user/48364/2a44f276-f4e7-4ecf-963a-a899b3bccaa3.png)

#### 3.文件(Files)

**概念**：

文件在 Linux 系统中是一种基本的持久化存储机制，可用于**进程间通信**。多个进程可以通过对同一个文件的读取和写入来共享信息。

**简单图解：**

![](https://files.mdnice.com/user/48364/a67586c2-09be-475d-9739-b23bd353ea0d.png)

**使用场景：**

- **数据交换：**

  进程之间可以通过读写同一文件来交换数据。例如，一个进程写入结果数据，另一个进程读取这些数据进行进一步处理。

- **持久化存储：**

  文件用于保存需要在应用程序重启后依然保留的数据，例如用户数据、应用状态等。


**简单示例：**

```c
// 写进程: 向文件中写入数据
int main() {
    const char *file = "/tmp/ipc_file";
    int fd = open(file, O_RDWR | O_CREAT, 0666);
    write(fd, "Hello from Process A", 20);  // 向文件写入数据
    close(fd);     // 关闭文件
    return 0;
}
// 读进程: 从文件中读取数据

int main() {
    const char *file = "/tmp/ipc_file";
    int fd = open(file, O_RDWR | O_CREAT, 0666);
    char buf[50];
    read(fd, buf, 20);  // 从文件中读取数据
    close(fd);     // 关闭文件
    return 0;
}
```

>  **注意：** 
>  如果存在多个写进程同时操作同一个文件，那么会引发数据竞态和一致性问题。为了解决这个问题，可以使用文件锁或其他同步机制来协调对文件的访问，确保数据的完整性和一致性。

**Linux 文件锁:**

**文件锁是什么？**

通俗来说，文件锁是 Linux 系统中用来控制多个进程同时访问同一个文件的一种机制。它的作用就像是门上的锁，确保在同一时间内只有一个进程能够写入文件，从而避免多个进程同时写入数据导致的混乱。

**文件锁有什么用？**

- **防止数据覆盖**：
当一个进程正在写文件时，文件锁可以防止其他进程同时写入，从而避免数据被覆盖。

- **保证写操作的完整性**：

  通过锁定文件，确保每次只有一个进程能够执行写操作，这有助于保持写入数据的完整性。

**实现文件锁:**

在 Linux 中，可以使用 fcntl 或 flock 系统调用来实现文件锁。

**示例代码** 

使用 fcntl 实现文件锁，从而保证多个进程在操作同一文件时不会相互干扰，维护数据的一致性和完整性。以下是一个具体的示例：

```c
int main() {
    const char *file = "/tmp/ipc_file";
    int fd = open(file, O_RDWR | O_CREAT, 0666);
    // 设置文件锁
    struct flock fl;
    fl.l_type = F_WRLCK;  // 设置写锁
    fl.l_whence = SEEK_SET;
    fl.l_start = 0;
    fl.l_len = 0;  // 锁定整个文件
    if (fcntl(fd, F_SETLKW, &fl) == -1) {
        perror("Error locking file");
        return -1;
    }
    write(fd, "Hello from Process A", 20); // 执行写操作
    // 释放锁
    fl.l_type = F_UNLCK;
    fcntl(fd, F_SETLK, &fl);
    close(fd);
    return 0;
}

```

#### 4.信号量(Semaphores)

**概念**:
信号量是一种在进程间或同一进程的不同线程间提供同步的机制。它是一个计数器，用于控制对共享资源的访问。当计数器值大于0时，表示资源可用；当值为0时，表示资源被占用。进程在访问共享资源前必须减少（wait）信号量，访问后必须增加（post）信号量。

信号量有两种，一种是 POSIX 信号量，另一种是 System V 信号量。由于 POSIX 信号量提供了更简洁、更易于理解和使用的 API，并且在现代操作系统中得到了广泛支持和优化，所以这里我重点讲解 POSIX 信号量。

  **简单图解：**

  ![](https://files.mdnice.com/user/48364/40719381-d64e-4adc-8e59-3360144bcfaa.png)

  **分类：**

 **匿名信号量**

  **概念:**

 匿名信号量是内存中的信号量，不与任何文件系统的名称关联。它们通常用于单一进程内不同线程间的同步，或在具有共同祖先的进程之间进行同步。

 **特点：**
 - **作用域**：限于创建它的进程内部或其子进程之间。
 - **生命周期**：与创建它们的进程的生命周期相同，进程终止时信号量也会消失。

**使用场景**：

- **互斥访问**：在多线程程序中，确保同一时刻只有一个线程可以访问某个共享资源。
- **同步操作**：协调多个线程的执行顺序，一个线程在另一个线程完成其任务之后再开始执行。如：线程池中的任务队列没任务时，线程必须等待，而当有有线程向队列添加任务时，需要唤醒其他线程来进行消费任务。

 **有名信号量**

 **概念:**  有名信号量在文件系统中具有一个唯一的名称，允许不同的独立进程通过这个名称访问同一个信号量，实现进程间同步。

 **特点：**
 - **作用域**：可以跨不同的进程使用。它们在文件系统中具有一个全局唯一的名称，任何知道这个名称的进程都可以访问同一个信号量。
 - **生命周期**：生命周期可以超过创建它们的进程。即使创建它们的进程已经结束，只要有名信号量的名称存在于文件系统中，它们就继续存在。

**使用场景**：

- **进程间互斥：** 多个独立进程共享资源，如文件或内存映射区域，需要互斥访问以避免冲突。
- **同步操作**：协调多个进程的执行顺序，一个进程在另一个进程完成其任务之后再开始执行。如：在生产者消费者模型中，只要当生产者向队列添加数据，队列不为空的时候，消费者才能消费数据，否则只能等待。
  

**来看一个进程互斥的例子：**

下面是使用 有名信号量 来实现两个独立进程对同一个日志文件进行互斥访问的简化示例。

写入进程 1 (process1.c)
```c
int main() {
    FILE *logFile = fopen("logfile.txt", "a");  // 打开日志文件

    // 打开或创建有名信号量
    sem_t *sem = sem_open("/log_semaphore", O_CREAT, 0644, 1);

    // 获取信号量
    sem_wait(sem);

    // 写入日志
    fprintf(logFile, "Log message from Process 1\n");
    fflush(logFile);

    // 释放信号量
    sem_post(sem);

    // 关闭信号量和文件
    sem_close(sem);
    fclose(logFile);

    return 0;
}
```
写入进程 2 (process2.c)

```c
int main() {
    FILE *logFile = fopen("logfile.txt", "a");  // 打开日志文件

    // 打开或创建有名信号量
    sem_t *sem = sem_open("/log_semaphore", O_CREAT, 0644, 1);

    // 获取信号量
    sem_wait(sem);

    // 写入日志
    fprintf(logFile, "Log message from Process 2\n");
    fflush(logFile);

    // 释放信号量
    sem_post(sem);

    // 关闭信号量和文件
    sem_close(sem);
    fclose(logFile);

    return 0;
}
```

**匿名信号量和有名信号量 API 接口区别：**

![](https://files.mdnice.com/user/48364/0d25667d-32f0-4bcc-8751-99a524f6bcd7.png)


#### 5.共享内存(Shared Memory)

**概念**：
在 Linux 中，共享内存是进程间通信（IPC）的一种形式。当多个进程需要访问相同的数据时，使用共享内存是一种高效的方式。它允许两个或多个进程访问同一个物理内存区域，这使得数据传输不需要通过内核空间，从而提高了通信效率。

在讲解共享内存前，我们需要了解内存映射技术？

**内存映射技术（Memory Mapping）** 是一种将文件或设备的数据映射到进程内存地址空间的技术，它允许进程直接对这部分内存进行读写操作，就像访问普通内存一样。这种技术不仅可以用于文件I/O操作，提高文件访问效率，而且是实现共享内存的基础。

在 Linux 系统中，内存映射可以通过 **mmap** 系统调用来实现。**mmap** 允许将文件映射到进程的地址空间，也可以用来创建匿名映射（即不基于任何文件的共享内存区域）。

在 Linux 中，共享内存可以分为如下几类。

**a、匿名共享内存**

**工作原理**：

匿名共享内存不与任何具体的文件系统文件直接关联，其内容仅在内存中存在。这意味着当所有使用它的进程都结束时，该内存区域的数据就会消失。这种特性使得匿名共享内存非常适合于那些需要临时共享数据但又不需要将数据持久存储到磁盘的场景。

**简单图解：**

![](https://files.mdnice.com/user/48364/b7a96581-d62c-4168-ad98-704aeee22f9e.png)

**注意**：在 Linux 中，**匿名共享内存主要被设计用于有亲缘关系的进程间通信，如父子进程间**。这是因为匿名共享内存的引用（例如，通过 mmap 创建时返回的内存地址）不会自动出现在其他进程中，而是需要通过某种进程间通信的方式（如Unix域套接字）传递给相关的进程。而通过 Unix 域套接字来实现又稍显复杂，所以我们一般推荐匿名共享内存适用于有亲缘关系的进程间通信。

**创建和使用**：

在 Linux 系统中，匿名共享内存通常是通过 mmap()函数创建的，调用时需指定MAP_ANONYMOUS标志。此外，还需要设置 PROT_READ 和 PROT_WRITE 权限，以确保内存区域可读写。创建时也可以选择 MAP_SHARED 标志，以便在多个进程间共享这块内存。

**示例代码片段如下：**
```c
#include <sys/mman.h>
void* shared_memory = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOU
```
在这里，size是希望映射的内存区域大小，mmap()调用成功后，返回指向共享内存区域的指针。

**使用场景**：

**大量数据交换** ：当两个或多个进程需要交换大量数据时，使用共享内存比传统的进程间通信方法（如管道或消息队列）更有效率。

**而谈到共享内存，又不得不探讨下关于共享内存的同步问题？**

在使用共享内存时，由于多个进程可以直接并且同时访问同一个物理内存区域，不加以适当控制就可能引起数据竞态和一致性问题。

**数据竞态**：当多个进程尝试同时修改共享内存中的同一数据项时，最终结果可能依赖于各进程操作的具体顺序，可能导致不符合预期的结果。

**一致性问题**：在没有合适同步机制的情况下，一个进程可能在另一个进程写入数据的同时读取共享内存，导致获取到不完整或不一致的数据。

**解决策略：使用信号量**

信号量是一种常用的同步机制，用于控制对共享资源的并发访问。通过增加（释放资源）或减少（占用资源）信号量的值，可以有效地控制对共享内存区域的访问，防止数据竞态和确保数据一致性。

**使用信号量来解决匿名共享内存同步问题的简单示例**：

```c
int main() {
    // 创建有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);

    // 创建匿名共享内存
    int *counter = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    *counter = 0;  // 初始化计数器

    pid_t pid = fork();
    if (pid == 0) {
        // 子进程
        for (int i = 0; i < 10; ++i) {
            sem_wait(sem); // 等待信号量
            (*counter)++;   // 递增计数器
            printf("Child process increments counter to %d\n", *counter);
            sem_post(sem);  // 释放信号量
            sleep(1);       // 模拟工作负载
        }
        exit(0);
    } else {
        // 父进程
        for (int i = 0; i < 10; ++i) {
            sem_wait(sem); // 等待信号量
            printf("Parent process reads counter as %d\n", *counter);
            sem_post(sem); // 释放信号量
            sleep(1);      // 模拟工作负载
        }

        // 等待子进程结束
        wait(NULL);
        sem_close(sem);
        sem_unlink("/mysemaphore");
        munmap(counter, sizeof(int)); // 释放共享内存
    }

    return 0;
}
```

**b、基于文件的共享内存**

**工作原理:**

基于文件的共享内存通过将磁盘上的实际文件映射到一个或多个进程的地址空间中来实现。当文件被映射到内存后，进程就可以像访问普通内存一样直接读写文件内容，操作系统负责同步内存修改回磁盘文件。这种机制既提高了数据访问的效率，也实现了数据的持久化存储。

相比匿名共享内存只能适合有亲缘关系的进程，**基于文件的共享内存特别适合于实现非亲缘关系进程间的数据共享**。

**简单图解：**

![](https://files.mdnice.com/user/48364/52700546-60f3-4d5d-9c39-6928affae520.png)

**创建和使用:**

要创建基于文件的共享内存，首先需要打开（或创建）一个文件，然后使用 mmap()将文件映射到内存中。与匿名共享内存不同，这里需要提供**文件描述符**而不是 MAP_ANONYMOUS 标志。

**示例代码片段如下：**

```c
#include <sys/mman.h>
#include <fcntl.h>

size_t size = 4096; // 共享内存区域大小

int fd = open("shared_file", O_RDWR | O_CREAT, 0666);
ftruncate(fd, size); // 设置文件大小
void* shared_memory = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
```

在这里，shared_file 是被映射的文件名，size 是文件的预期大小。通过 ftruncate() 调整文件大小以匹配共享内存的需求。mmap()成功后返回指向共享内存区域的指针。

**使用场景：**

**大量数据交换** ：基于文件的共享内存同样适用于多个进程需要进行大量数据交换的场景。与匿名共享内存不同的是，这些数据可以持久化存储到磁盘上。

在使用基于文件的共享内存时，同样需要解决多个进程共享数据的同步问题，以保证数据的一致性和完整性。


**解决方案**：

- **信号量**： 
  信号量可以理解是一个计数器，用来控制同时访问共享资源（如共享内存）的进程数量。如果信号量计数大于0，表示资源可用，进程可以访问资源并将计数减1；如果信号量计数为0，表示资源不可用，进程必须等待。当资源使用完毕后，进程会增加信号量计数，表示资源再次可用。

- **文件锁**：
  文件锁允许进程对共享内存所基于的文件加锁，防止其他进程同时访问。如果一个进程要写入共享内存，它可以加一个排他锁，这时其他进程既不能读也不能写；如果只需要读取，进程可以加一个共享锁，这样其他进程也可以加共享锁来读取数据，但不能写入。在 Linux 中，文件锁的实现主要依赖于两个系统调用：fcntl 和 flock。而关于 fcntl 和 flock 的讲解，我在前文也有提到过。
  

**简单来说**：

- 使用信号量是为了确保在同一时间只有限定数量的进程可以操作共享内存。
- 使用文件锁是为了防止在某个进程读写共享内存时，其他进程进行干扰。


下面来看一个使用**有名信号量解决基于文件的共享内存同步问题的示例**，这个简单的示例演示了两个进程：一个进程向共享内存写入数据，另一个进程从共享内存读取数据。这两个进程使用同一个有名信号量来同步对共享内存区域的访问。

**示例代码：**

首先，确保你有一个名为 shared_file 的文件和一个名为 /mysemaphore 的信号量。

**写入进程**：

```c
int main() {
    const char* filename = "shared_file";
    const size_t size = 4096;

    // 打开共享文件
    int fd = open(filename, O_RDWR | O_CREAT, 0666);

    // 映射文件到内存
    void* addr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

    // 打开有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);

    // 等待信号量，写入数据
    sem_wait(sem);
    strcpy((char*)addr, "Hello from writer process!");
    sem_post(sem);

    // 清理
    munmap(addr, size);
    close(fd);
    sem_close(sem);

    return 0;
}
```
**读取进程：**

```c
int main() {
    const char* filename = "shared_file";
    const size_t size = 4096;

    // 打开共享文件
    int fd = open(filename, O_RDONLY);

    // 映射文件到内存
    void* addr = mmap(NULL, size, PROT_READ, MAP_SHARED, fd, 0);

    // 打开有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);

    // 等待信号量，读取数据
    sem_wait(sem);
    printf("Read from shared memory: %s\n", (char*)addr);
    sem_post(sem);

    // 清理
    munmap(addr, size);
    close(fd);
    sem_close(sem);

    return 0;
}
```

**说明**：上面的信号量初始值为 1 ，实际上信号量在这里充当的就是互斥锁。

**c、Posix 共享内存**

POSIX 共享内存提供了一种高效的方式，允许多个进程通过共享内存区域进行通信。与基于文件的共享内存相比，POSIX 共享内存不需要直接映射磁盘上的文件，而是通过创建命名的共享内存对象来实现进程间的数据共享。这些对象虽然在逻辑上类似于文件（因为可以通过shm_open创建和打开），但实质上直接存在于内存中，提供了更快的数据访问速度。

**Posix 共享内存接口**：
```c
shm_open()        // 创建或打开一个共享内存对象
shm_unlink()      // 删除一个共享内存对象的名称
ftruncate()       // 调整共享内存对象的大小
mmap()            // 将共享内存对象映射到调用进程的地址空间
munmap()          // 解除共享内存对象的映射
```

以下是一个 使用 POSIX 共享内存 进行 两个不相关进程间 通信的简化示例。一个进程写数据，另一个进程读取数据。

**示例演示**：

写入进程 (writer.c)
```c
#define SHM_NAME "/example_shm"     // 共享内存名称
#define SHM_SIZE 4096               // 共享内存大小
#define SEM_NAME "/example_sem"     // 信号量名称

int main() {
    // 创建或打开共享内存对象
    int shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    ftruncate(shm_fd, SHM_SIZE);

    // 映射共享内存到进程地址空间
    void *ptr = mmap(0, SHM_SIZE, PROT_WRITE | PROT_READ, MAP_SHARED, shm_fd, 0);

    // 创建或打开有名信号量，初始值为 1
    sem_t *sem = sem_open(SEM_NAME, O_CREAT, 0666, 1);

    // 等待信号量，开始写入数据
    sem_wait(sem);
    const char *message = "Hello from writer process!";
    sprintf(ptr, "%s", message);
    printf("Writer wrote to shared memory: %s\n", message);
    sem_post(sem);  // 释放信号量

    // 清理资源
    munmap(ptr, SHM_SIZE);
    close(shm_fd);
    sem_close(sem);

    return 0;
}


```
读取进程 (reader.c)
```c
#define SHM_NAME "/example_shm"     // 共享内存名称
#define SHM_SIZE 4096               // 共享内存大小
#define SEM_NAME "/example_sem"     // 信号量名称

int main() {
    // 打开已经存在的共享内存对象
    int shm_fd = shm_open(SHM_NAME, O_RDONLY, 0666);

    // 映射共享内存到进程地址空间
    void *ptr = mmap(0, SHM_SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);

    // 打开有名信号量
    sem_t *sem = sem_open(SEM_NAME, 0);

    // 等待信号量，开始读取数据
    sem_wait(sem);
    printf("Reader read from shared memory: %s\n", (char*)ptr);
    sem_post(sem);  // 释放信号量

    // 清理资源
    munmap(ptr, SHM_SIZE);
    close(shm_fd);
    sem_close(sem);

    return 0;
}
```


**d、System V共享内存**

System V共享内存是一种传统的进程间通信（IPC）机制，它允许多个进程通过共享内存区域进行通信。与POSIX共享内存不同，System V共享内存使用IPC键值key_t来标识和管理共享内存段，而不是通过命名的方式。这种机制提供了一套底层控制共享内存的API，允许进行更细粒度的操作，如权限控制、共享内存状态的查询和管理等。

**System V共享内存接口**：
```c
shmget()         // 创建或获取共享内存段的标识符
shmat()          // 将共享内存段附加到进程的地址空间
shmdt()          // 分离共享内存段和进程的地址空间
shmctl()         // 对共享内存段执行控制操作
```


**示例演示**：

以下是一个使用 System V 共享内存 和 System V 信号量 实现两个不相关进程的同步通信的示例。一个进程写入共享内存，另一个进程读取共享内存，并通过信号量进行同步控制。

信号量初始化 (semaphore_init.c)
```c
#define FILE_PATH "sharedfile"  // ftok生成key所用的文件路径

int main() {
    // 生成信号量键值
    key_t sem_key = ftok(FILE_PATH, 'B');

    // 创建信号量集，包含一个信号量
    int sem_id = semget(sem_key, 1, 0666 | IPC_CREAT);

    // 初始化信号量的值为 1
    semctl(sem_id, 0, SETVAL, 1);

    printf("Semaphore initialized.\n");

    return 0;
}
```

写入进程 (writer.c)
```c
#define SHM_SIZE 1024  // 共享内存大小
#define FILE_PATH "sharedfile"  // ftok生成key所用的文件路径

int main() {
    // 生成共享内存和信号量的键值
    key_t shm_key = ftok(FILE_PATH, 'A');
    key_t sem_key = ftok(FILE_PATH, 'B');

    // 创建共享内存段
    int shm_id = shmget(shm_key, SHM_SIZE, 0666 | IPC_CREAT);

    // 将共享内存段附加到进程的地址空间
    char *shm_ptr = (char*) shmat(shm_id, NULL, 0);

    // 创建信号量集，只有一个信号量
    int sem_id = semget(sem_key, 1, 0666 | IPC_CREAT);

    // 等待信号量
    struct sembuf sem_op = {0, -1, 0};  // P 操作
    semop(sem_id, &sem_op, 1);

    // 向共享内存写入数据
    strcpy(shm_ptr, "Hello from writer process!");
    printf("Writer wrote to shared memory: %s\n", shm_ptr);

    // 释放信号量
    sem_op.sem_op = 1;  // V 操作
    semop(sem_id, &sem_op, 1);

    // 分离共享内存段
    shmdt(shm_ptr);

    return 0;
}
```

读取进程 (reader.c)
```c
#define SHM_SIZE 1024  // 共享内存大小
#define FILE_PATH "sharedfile"  // ftok生成key所用的文件路径

int main() {
    // 生成共享内存和信号量的键值
    key_t shm_key = ftok(FILE_PATH, 'A');
    key_t sem_key = ftok(FILE_PATH, 'B');

    // 获取共享内存段
    int shm_id = shmget(shm_key, SHM_SIZE, 0666);

    // 将共享内存段附加到进程的地址空间
    char *shm_ptr = (char*) shmat(shm_id, NULL, 0);

    // 获取信号量集
    int sem_id = semget(sem_key, 1, 0666);

    // 等待信号量
    struct sembuf sem_op = {0, -1, 0};  // P 操作
    semop(sem_id, &sem_op, 1);

    // 读取共享内存中的数据
    printf("Reader read from shared memory: %s\n", shm_ptr);

    // 释放信号量
    sem_op.sem_op = 1;  // V 操作
    semop(sem_id, &sem_op, 1);

    // 分离共享内存段
    shmdt(shm_ptr);

    return 0;
}
```

**方便大家学习，我又将这几种共享内存作了总结**：

| 类型 | 区别 | 适用场景 | 使用方式 |
| ---- | ---- | -------- | -------- |


| 匿名共享内存 | 无需文件支持，进程通过内存页直接共享数据，通常通过 `mmap()`<br/> 进行内存映射 | 需要快速创建和销毁共享内存，且不需要持久化 | `mmap()`<br/>, `shm_open()`<br/>, `ftruncate()` |
| ------------ | ------------------------------------------------------------ | ------------------------------------------ | ----------------------------------------------- |


| 基于文件的共享内存 | 使用文件作为媒介，进程通过文件系统共享内存，适合需要持久化的场景 | 共享数据需要持久化或跨设备使用 | `mmap()`<br/>, `open()`<br/> 结合文件操作 |
| ------------------ | ------------------------------------------------------------ | ------------------------------ | ----------------------------------------- |


| POSIX 共享内存 | POSIX 标准接口，更现代，支持更细粒度的控制和权限管理 | 在 POSIX 兼容系统中，适合现代应用程序，提供更好的并发控制 | `shm_open()`<br/>, `mmap()` |
| -------------- | ---------------------------------------------------- | --------------------------------------------------------- | --------------------------- |


| System V 共享内存 | 传统 System V 接口，支持较大规模共享内存段，但接口老旧 | 老系统或需要兼容旧版代码，适用于大规模数据共享 | `shmget()`<br/>, `shmat()` |
| ----------------- | ------------------------------------------------------ | ---------------------------------------------- | -------------------------- |




#### 6.消息队列 (Message Queues)

**概念**：

消息队列是一种允许一个或多个进程向其写入消息，并由一个或多个进程读取消息的 IPC 机制。每条消息都由一个消息队列标识符（ID）识别， 且可以携带一个特定的类型。消息队列允许不同进程非阻塞地发送和接收记录或数据块，这些记录可以是不同类型和大小的。

**消息队列图解：**

![](https://files.mdnice.com/user/48364/b3ab33df-59d9-4b30-9539-9b3a0dd019ce.png)


**使用场景**：

- **进程间通信：**
  在涉及多个运行进程的应用中，消息队列提供了一种高效的方式来传递信息。它允许进程之间无需直接相互连接就能交换数据，从而简化了通信过程。

- **异步数据处理：**
  消息队列使进程能够异步处理信息。一个进程（即生产者）可以发送任务或数据至队列，并继续其他操作，而另一进程（即消费者）可以在准备就绪时从队列中取出并处理这些数据。这种模式有效地分离了数据的生成和消费过程，提高了应用的效率和响应速度。实际的应用比如：日志记录，某些系统可能有一个专门的进程负责记录日志，其他进程可以将日志消息发送到消息队列，由该专门进程异步地写入日志文件。


**以下是使用 System V 消息队列的一个简单示例:**

```c
struct message {
    long mtype;
    char mtext[100];
};

// 发送消息至消息队列
int main() {
    key_t key = ftok("queuefile", 65);  // 生成唯一键
    int msgid = msgget(key, 0666 | IPC_CREAT); // 创建消息队列
    struct message msg;
    msg.mtype = 1; // 设置消息类型
    sprintf(msg.mtext, "Hello World"); // 消息内容
    msgsnd(msgid, &msg, sizeof(msg.mtext), 0); // 发送消息
    printf("Sent message: %s\n", msg.mtext);
    return 0;
}

// 从消息队列中获取消息
int main() {
    key_t key = ftok("queuefile", 65);
    int msgid = msgget(key, 0666 | IPC_CREAT);
    struct message msg;
    msgrcv(msgid, &msg, sizeof(msg.mtext), 1, 0); // 接收消息
    printf("Received message: %s\n", msg.mtext);
    msgctl(msgid, IPC_RMID, NULL); // 销毁消息队列
    return 0;
}
```

#### 7.套接字 (Sockets)

**概念**：

套接字是一种在不同进程间进行数据交换的通信机制。在 Linux 中，套接字可以用于同一台机器上的进程间通信（IPC）或不同机器上的网络通信。套接字支持多种通信协议，最常见的是TCP（可靠的、连接导向的协议）和UDP（无连接的、不可靠的协议）。

**简单图解：**

![](https://files.mdnice.com/user/48364/0b9d8bec-8add-40e9-82f7-f55f10933114.png)


**使用场景：**

**网络通信**：
同一台主机或不同主机上的进程之间通过网络套接字进行数据交换。

**简单示例：** - 使用 TCP 套接字进行通信
```c
int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[1024] = {0};

    // 创建套接字
    server_fd = socket(AF_INET, SOCK_STREAM, 0);

    // 定义套接字地址
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);

    // 绑定套接字到地址和端口
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));

    // 监听套接字
    listen(server_fd, 3);

    while (1) {
        // 接受客户端连接
        new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen);

        // 读取客户端数据
        read(new_socket, buffer, 1024);
        printf("Message from client: %s\n", buffer);

        // 关闭客户端连接
        close(new_socket);
    }

    // 关闭服务器套接字
    close(server_fd);

    return 0;
}

```

客户端 (client.c)
```c
int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;

    // 创建套接字
    sock = socket(AF_INET, SOCK_STREAM, 0);

    // 定义服务器地址
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(8080);

    // 连接到服务器
    connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr));

    // 发送数据到服务器
    char *message = "Hello from the client!";
    send(sock, message, strlen(message), 0);

    // 关闭套接字
    close(sock);

    return 0;
}

```

#### 8.域套接字 (Unix Domain Sockets)

**概念**：

域套接字（Unix Domain Sockets）是一种在同一台机器上的进程间进行数据通信的机制。相对于网络套接字，它们提供了更高效的本地通信方式，**因为数据不需要经过网络协议栈**。域套接字支持流（类似TCP）和数据报（类似UDP）两种模式。

**特别说明**：在域套接字通信中，**“不经过网络协议栈”** 指的是数据传输不需要IP层的路由、不需要TCP/UDP等传输层协议的封包与解包处理，也不需要网络接口层的参与。这一点与网络套接字不同，后者用于跨网络的通信，需要经过完整的网络协议栈处理，包括数据的封装、传输、路由和解封装等。

**简单图解：**

![](https://files.mdnice.com/user/48364/d6fd5224-7a31-4ee9-8ed4-e9f4797b0862.png)


**使用场景：**

- **本地进程间通信**：

  当需要在同一台机器上的不同进程间高效地交换数据时。

- **替代管道和消息队列**：

  当需要比管道和消息队列更复杂的双向通信时。


**简单示例：** - 使用 Unix 域套接字进行通信

服务器端 (server.c)

```c
int main() {
    int server_fd, client_socket;
    struct sockaddr_un address;

    // 创建套接字
    server_fd = socket(AF_UNIX, SOCK_STREAM, 0);

    // 设置地址
    address.sun_family = AF_UNIX;
    strcpy(address.sun_path, "/tmp/unix_socket");

    // 绑定并监听
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 5);

    while (1) {
        // 接受客户端连接
        client_socket = accept(server_fd, NULL, NULL);

        // 读取客户端发送的数据
        char buffer[100];
        read(client_socket, buffer, sizeof(buffer));
        printf("Received: %s\n", buffer);

        // 关闭客户端套接字
        close(client_socket);
    }

    // 关闭服务器套接字并删除 socket 文件
    close(server_fd);
    unlink("/tmp/unix_socket");

    return 0;
}

```

客户端 (client.c)

```c
int main() {
    int sock;
    struct sockaddr_un address;

    // 创建套接字
    sock = socket(AF_UNIX, SOCK_STREAM, 0);

    // 设置地址
    address.sun_family = AF_UNIX;
    strcpy(address.sun_path, "/tmp/unix_socket");

    // 连接服务器
    connect(sock, (struct sockaddr *)&address, sizeof(address));

    // 发送数据到服务器
    char *message = "Hello from the client!";
    write(sock, message, strlen(message));

    // 关闭套接字
    close(sock);

    return 0;
}

```

**注意事项**：

- Unix 域套接字的地址是文件系统中的路径，而不是IP地址和端口。
- Unix 域套接字通常用于同一台机器上的进程间通信，而不适用于网络通信。
- 使用 Unix 域套接字时，需要确保套接字文件的路径是可访问的，并在通信完成后清理套接字文件。

> **温馨提示**：上面的代码示例只是一个基础引导，帮助大家快速理解每种进程间通信方式的基本用法。如果你想更深入地学习，并获取完整的代码示例，欢迎关注我的公众号 「**跟着小康学编程**」。在公众号后台回复 “process”，你就能获取完整的代码资源哦！

> **想更快找到我？** 直接微信搜索 「**跟着小康学编程**」，或者扫描下方二维码关注，和一群爱学习的编程小伙伴一起成长吧！

### 总结

Linux 的进程间通信方式多种多样，从最简单的管道到复杂的套接字，每种方式都有其独特的应用场景。通过这篇文章的学习，希望你已经对这些通信方式有了系统的了解。无论是小型应用程序，还是处理复杂的进程间数据传输，找到合适的 IPC 方式就能轻松解决问题。

如果你觉得已经掌握了，那恭喜你！如果还没完全弄明白，没关系，欢迎在评论区留言，我们一起讨论，确保你都能搞懂！

### 最后

如果你觉得这篇文章对你有帮助，记得给我点个赞 👍，并分享给有需要的小伙伴吧！

- **订阅我的公众号**：我会定期更新更多Linux和C++相关的编程干货，分享进阶技巧和实战经验，帮你每天都能学到有用的知识。
- **更多精彩内容等你来发现**：点击关注，第一时间收到最新的技术文章推送，在编程高手的道路上，我们携手同行！
- **互动与反馈**：如果有任何问题或建议，欢迎留言，我非常期待与你的交流！

#### 关注我能学到什么？

我不仅提供高质量的编程知识，还会结合实际项目和常见面试问题，带给你全面的学习路径。如果你正在学习编程、准备面试，或者在工作中想要提升技能，这里绝对是你提升自我的好帮手。

#### 如何关注公众号:

非常简单！扫描下方二维码即可一键关注。

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

另外我还建了个**技术交流群**，里面都是真正在写代码的同行，不聊虚的，只聊技术。有问题大家一起讨论，比一个人闷头学效率高多了。
  
![](https://files.mdnice.com/user/48364/4ebc72e9-e4bb-447a-9a92-8367a178df6d.png)

记住，技术这条路，一个人走容易迷路，一群人走才能走得更远。
