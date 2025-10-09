###  引言

大家好，我是小康 ，今天我们来学习一下 Linux 系统编程相关的知识。Linux 系统编程是连接高级语言和硬件的桥梁，它对深入理解计算机系统至关重要。无论你是打算构建高性能服务器还是开发嵌入式设备，掌握 Linux 系统编程是 C 和 C++ 开发者的基本技能。

本文旨在为初学者提供一个清晰的 Linux 系统编程入门指南，带你步入 Linux 系统编程的世界，从基本概念到实用技能，一步步建立起您的知识体系。

>想系统学习更多 C++ 知识？欢迎关注我的公众号「**跟着小康编程**」，我会持续更新 C、 C++、Linux、后端开发等高质量技术文章。也可以加我的个人微信，一起进群讨论学习！
>
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

## 基本概念

**什么是系统编程？**

系统编程，指的是开发那些直接与计算机硬件或操作系统进行交互的程序。这些程序负责管理和控制计算机系统的资源，包括但不限于进程、内存、文件系统和设备驱动。确保为应用程序提供一个稳定、高效的运行环境。

**系统编程与应用编程的主要区别**：

- **目的性**：系统编程旨在为计算机或操作系统本身提供功能和服务，而应用编程是为了满足最终用户的特定需求。
- **交互对象**：系统编程直接与硬件或操作系统交互，而应用编程与操作系统或其他应用交互。
- **复杂性**：由于系统编程需要管理和控制计算机的底层资源，因此通常比应用编程更为复杂。
- **开发工具**：系统编程通常使用低级语言，如 C 或汇编，因为这些语言提供了直接访问硬件的能力。而应用编程可能使用更高级的语言，如 Python 或 Java，以提高开发效率。


## Linux系统编程核心技术概览

在电脑的世界中，操作系统起到桥梁的作用，连接用户与计算机硬件。其中，Linux 由于其开源、稳定和安全的特点，成为了许多工程师的首选。为了更深入地理解它，我们首先需要了解其系统架构的神秘面纱。


![](https://files.mdnice.com/user/48364/b73e3e41-fd20-4c43-ae24-74dfcdc84f41.png)

### Linux 系统架构解析

#### 用户空间和内核空间的布局

![](https://files.mdnice.com/user/48364/22aad1ee-5f7a-4571-8e54-ac104a2c6fd6.png)

**各个内核组件说明**:

- **系统调用 （Syscalls）**：

  当应用程序需要访问硬件资源时，它们使用系统调用来与内核通信。

- **进程管理**：

  负责处理进程创建、调度和终止。确保系统中的进程公平、有效地获得 CPU 时间，并管理进程间的通信和同步。
- **内存管理**：

  管理物理内存，提供虚拟内存和分页功能。确保每个进程都有它自己的地址空间，同时保护进程间的内存不被非法访问。

- **文件系统**：

  提供文件和目录的创建、读取、写入和删除功能。它抽象了物理存储设备，为用户和应用程序提供了一个统一的文件访问接口。

- **虚拟文件系统（VFS）**：

  用户和应用程序不直接与各种文件系统交互。而是通过 VFS（虚拟文件系统）进行操作。VFS为各种不同的文件系统（如EXT4, FAT, NFS等）提供一个统一的接口。这样，无论底层使用的是哪种文件系统，用户和应用的文件访问方式都保持一致，实现在 Linux 中的无缝集成。

- **网络协议栈**：

  负责处理计算机之间的通信，使设备能够在网络上发送和接收数据。它包含了多层协议，如 **TCP/IP**，使计算机能够连接到互联网和其他网络，并与其他计算机进行数据交换。

- **设备驱动**：

  设备驱动是一种特殊的软件程序，它允许 Linux 内核和计算机的硬件组件进行交互。这些硬件组件可以是任何物理设备，如显卡、声卡、网络适配器、硬盘或其他输入/输出设备。设备驱动为硬件设备提供了一个抽象层，使得内核和应用程序不需要知道硬件的具体细节，就能与其进行通信和控制。**简而言之，设备驱动是硬件和操作系统之间通信的桥梁。**

#### 用户空间 (User Space)
所有的应用程序，如浏览器、文档编辑器或音乐播放器都运行在这个空间。

- **安全性**：用户空间的程序运行在受限的环境中，它们只能访问分配给它们的资源，不能直接访问硬件或其他程序的数据。
- **稳定性**：如果一个应用程序崩溃，它不会影响其他应用程序或系统的核心功能。

#### 内核空间 (Kernel Space)

内核空间是操作系统的核心。

- **权限**：内核可以直接访问硬件，并有权执行任何命令。
- **安全性**：虽然内核拥有广泛的权限，但只有那些已知且经过严格测试和验证的代码才被允许在内核空间执行。
- **稳定性**：如果内核遇到问题，整个系统可能会崩溃。


### 系统调用与库函数

在 **Linux** 编程中，我们经常听到“系统调用”和“库函数”这两个词，但你知道它们之间的区别吗？接下来就让我们来详细了解一下。

#### 什么是系统调用？

系统调用是一个程序向操作系统发出的请求。当应用程序需要访问某些资源（如磁盘、网络或其他硬件设备）或执行某些特定的操作（如创建进程或线程）时，它通常会通过系统调用来完成。

**工作原理**

- **模式切换**：应用程序在用户空间运行，而操作系统内核在内核空间运行。系统调用涉及从用户空间切换到内核空间。
- **参数传递**：程序将参数传递给系统调用，通常通过特定的寄存器。
- **执行**：内核根据传递的参数执行相应的操作。
- **返回结果**：操作完成后，内核将结果返回给应用程序，并将控制权返回给应用程序。

**常见的系统调用函数：**

```c
read() 和 write()：分别用于读取和写入文件。
open() 和 close()：打开和关闭文件。
fork()：创建一个新的进程。
wait()：等待进程结束。
exec()：执行一个新程序。
```

这只是系统调用的冰山一角。**Linux** 提供了上百个系统调用，每个都有其特定的功能。

#### 什么是库函数？

库函数是预编写的代码，存储在库文件中，供程序员使用。它们通过系统调用和操作系统的内核通信。例如，printf（） 是 C 语言的一个库函数，它内部使用 write（） 系统调用来和内核进行交互。


### 文件 IO 

文件IO（输入/输出）是计算机程序与文件系统交互的基本方式，允许程序读取和写入文件。要深入理解和使用文件IO，首先需要了解一些关键概念和操作。


![](https://files.mdnice.com/user/48364/331e7ff1-b08f-48d3-8d12-9b7124b7e9b9.png)

#### 文件描述符是什么？

文件描述符「 fd 」是一个整数，它代表了一个打开的文件。在 Linux 中，每次我们打开或创建一个文件时，系统都会返回一个文件描述符。而应用程序正是通过这个文件描述符「 fd 」来进行文件的读写的。

**特殊的文件描述符**:

- 标准输入**「stdin」** 是 0
- 标准输出**「stdout」** 是 1
- 标准错误 **「stderr」** 是 2

#### 常见的文件操作

当应用程序要与文件交互时，最基本的操作包括打开、读取、写入和关闭文件。这可以通过以下函数来实现。

```c
打开文件：open()
读取文件：read()
写入文件：write()
关闭文件：close()

# demo
int fd = open("example.txt", O_RDWR | O_CREAT);
write(fd, "Hello, File!", 12);
close(fd);
```

#### 文件位置与移动

有时，我们可能需要移动到文件的特定位置进行读写。使用 lseek（） 可以实现这一点。举个例子：
```c
/* 
假设我们有一个名为 "data.txt" 的文件，内容为：Hello World!
 现在我们有一个简单需求：我们想将文件中的"World"替换为"Linux"，但不想重写整个文件。 
*/

# demo 展示：

char buffer[6];  // 存放从文件中读取的数据

int fd = open("data.txt", O_RDWR);  # 以读写模式打开文件
lseek(fd, 6, SEEK_SET);  // 使用 lseek() 移动到"World"的开头位置
read(fd, buffer, 5);     // 读取5个字符（"World"的长度）

if (strcmp(buffer, "World") == 0) {
    // 重新定位文件指针以替换"World",这里需要重新定位的原因是：上面 read 操作使得文件指针已经指向文件末尾了，因此需要重新定位。
    lseek(fd, 6, SEEK_SET);
    write(fd, "Linux", 5);
}
close(fd) ; 
```

#### 高级文件 I/O

有时，简单的读写操作无法满足我们的需求，尤其当我们追求高效率或特殊功能时。为了更优雅、高效地处理文件数据，我们引入了一些高级文件 I/O 技巧。

**分散读取和集中写入**：

```c
#include <sys/uio.h>
// 读取操作
ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
// 写入操作
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);

# iovec 结构的定义如下：
struct iovec {
    void  *iov_base;  
    size_t iov_len; 
};

iov_base 是指向缓冲区起始地址的指针。
iov_len 是缓冲区的大小。
```

这两个函数主要用于多缓冲区的输入/输出操作，允许您在单次系统调用中，从文件读取到多个缓冲区或从多个缓冲区写入文件。

它们的主要目的是提高效率，因为常规的读/写函数每次只能在一个缓冲区进行操作。

**内存映射文件I/O**

内存映射文件 I/O 允许程序员将文件的一部分直接映射到进程的内存中。这样，程序可以通过直接访问这块内存来访问文件的内容，而不是使用传统的 read 、write 系统调用。这可以提高效率，特别是对于大文件的访问。

```c
#include <sys/mman.h>

// 相关函数声明
void* mmap(void* addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void* addr, size_t length);

// demo 举例:

int fd = open("example.txt", O_RDWR);
// 获取文件的大小
struct stat sb;
if (fstat(fd, &sb) == -1) {
    perror("fstat");
}
char *mapped = mmap(NULL, sb.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

// 后续的所有对文件的操作就可以通过 mapped 指针来进行。
// 例如：将第一个字符改为 'J'）
mapped[0] = 'J';
...
```

使用 mmap ，你可以直接在内存中访问文件内容，如同访问数组或其他数据结构一样。

**同步文件操作**

当您向文件写入数据时，操作系统可能会缓存这些数据，而不是立即写入磁盘,这样可以提高效率。 但在某些情况下，您可能需要确保数据确实已经写入磁盘。这就是同步文件操作的用处。

```c
#include <unistd.h>
#include <sys/mman.h>

int msync(void *addr, size_t length, int flags);
int fsync(int fd);
int fdatasync(int fd);
void sync(void);
```

- msync  用于同步内存映射（通过 mmap 函数创建）文件的内容。它将内存中的更改写回到映射的文件中。
- fsync 函数用于将指定文件描述符（fd）关联的文件的所有修改（包括数据和元数据）同步到磁盘
- fdatasync 函数类似于 fsync，但它只同步文件的数据部分，而不同步元数据。
- sync 同步整个文件系统的所有修改的数据到磁盘，包括所有打开的文件。



#### 文件锁定
**什么是文件锁定？**

文件锁定是一个在多个进程或线程之间协调对共享资源访问的机制。在这里，这个"共享资源"指的是文件。简单说，**文件锁**就是确保当一个进程正在使用一个文件时，其他进程不能修改它。

**为什么需要文件锁定？**

考虑这样一个场景：两个程序同时写入一个文件。不锁定文件可能会导致数据混乱。例如，一个进程可能会覆盖另一个进程的更改。所以，文件锁定是确保数据完整性的关键。

**文件锁的两种模式**：

- **共享锁（Shared Locks）**：也被称为读锁。当一个进程持有共享锁时，其他进程可以获得该文件的共享锁以进行读取，但不能获得独占锁进行写入。

- **独占锁（Exclusive Locks）**：也被称为写锁。当一个进程持有独占锁时，其他进程不能获得该文件的任何类型的锁。这意味着其他进程不可以读取或写入该文件。

**如何实现文件锁定？**

在 Linux  编程中，文件锁定可以使用以下函数实现：

`fcntl()` : 允许对文件中的特定部分进行锁定。

`flock()` ：提供了一个简化的锁定机制，直接锁定整个文件。

#### 重定向

**什么是重定向？**

**重定向**，顾名思义，指的是改变数据流的方向。在 **Linux** 系统编程中，程序通常与三种标准I/O 流进行交互：标准输入（stdin）、标准输出（stdout）、和标准错误输出（stderr）。

- 标准输入（stdin）         : 来自键盘的输入。
- 标准输出（stdout）        : 显示到屏幕上。
- 标准错误输出（stderr)     : 也显示到屏幕上。

**重定向的核心是将这些标准的 I/O 流改变到其他地方，如文件或其他程序。**

例如，当我们在命令行中执行命令并将结果保存到文件中，或者从文件中获取命令的输入而不是从键盘中获取，我们都是在使用重定向。

```
# 将 ls -l 命令的输出（即当前目录的详细列表）重定向到 filelist.txt 文件中
ls -l > filelist.txt   
```

重定向不仅局限于命令行界面，它在程序中也很有用，允许我们动态地更改程序的输入和输出来源，为构建更复杂、灵活的应用程序提供了基础。

在 **Linux** 系统编程中，实现重定向的一个核心函数是 **dup2** 函数。 

```c
#include <unistd.h>

int dup2(int oldfd, int newfd);
/*
其中：
oldfd 是原始文件描述符。
newfd 是要复制到的目标文件描述符。
*/

# demo 举例:

int main() {
    // 打开一个文件用于写入
    int file_fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (file_fd < 0) {
      // 错误处理
      ...
    }

    // 使用 dup2 将标准输出重定向到文件
    if (dup2(file_fd, STDOUT_FILENO) < 0) {
     // 错误处理
      ...
    }

    // 现在，所有标准输出都会被写入文件
    printf("This will be written to the file 'output.txt'\n");
    close(file_fd);
    return 0;
}

```

### Linux 进程

你有没有想过，当你在 **Linux** 操作系统上运行一个程序时，都发生了哪些神奇的事情？接下来，我们将一步一步地深入探讨 Linux 进程的世界。

![](https://files.mdnice.com/user/48364/bf801081-3c72-49dc-bcaf-24bff1d7cbbf.png)


#### 进程究竟是什么？

每当你启动一个程序，**Linux** 系统都会创建一个新的进程。这个进程有它自己的内存地址、系统资源和状态。简而言之，进程是程序的一个运行实例。

#### 进程的创建和终止

`fork()`：当调用 fork 函数时，它会创建一个新的子进程。这个子进程几乎是父进程的复制品，包括父进程的内存、程序计数器等。

`wait() & waitpid()`：这些函数允许父进程等待子进程的结束，并收集子进程的退出状态。防止出现僵尸进程。

`exec() 系列函数`：**exec** 系列函数 允许一个进程运行另一个程序，它实际上替换了当前进程的内容。

#### 进程的状态转换图

![](https://files.mdnice.com/user/48364/fb46aaf7-c81a-4c7d-9c6d-582e782eacd1.png)

**五态简要说明**:

- **新建状态**: 这是进程刚被创建时的状态。在这个状态下，操作系统为进程分配了一个唯一的进程标识符（PID）和必要的资源。但进程还没有开始执行任何代码。新建状态通常非常短暂，用户很难观察到，因为进程很快就会转移到 **「就绪状态」**。

- **就绪状态** : 进程已准备好运行并等待操作系统的调度器分配 CPU 时间片。在这个状态下，进程已经加载了所有必要的代码和数据到内存中，且已准备好执行。
- **运行状态** : 进程正在 CPU 上执行。一个进程只有在运行状态时才能执行其指令。
- **阻塞状态** : 进程不能执行，因为它在等待一些事件发生，例如 I/O 操作的完成、信号的接收等。在此状态下，即使 CPU 空闲，进程也不能执行。
- **终止状态** : 进程已完成执行或被终止。在这个状态下，进程的资源通常被回收，进程退出。

#### 进程间通信

在 Linux 的世界里，进程是操作系统进行资源分配的基本单位。但是，进程并不是孤立的存在。当你的应用分成多个独立运行的进程时，这些进程之间如何有效地交换信息呢？这正是通过进程间通信的方式来实现的。

**Linux 提供了以下几种进程间通信的方式**：

1.管道 （Pipe）

管道是 Linux 中用于进程间通信的一种机制。它们分为两种类型：**匿名管道**和**有名管道**。

**匿名管道**   : 

  **概念**：匿名管道是一种在有亲缘关系的进程间（如父子进程）进行单向数据传输的通信机制，存在于内存中，通常用于临时通信。如果需要双向通信，则一般需要两个管道。
  
  **简单图解：**
  
  ![](https://files.mdnice.com/user/48364/c2616d1a-1b1e-431d-901d-037100ccf1ff.png)

  **使用场景**：适用于有亲缘关系的进程间的简单数据传输。
  
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
  
**有名管道**   ：

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

**2.信号 (Signals)**

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

**3.文件(Files)**

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
  如果存在多个写进程同时操作同一个文件，那么会引发数据竞态和一致性问题。为了解决这个问题，可以使用文件锁或其他同步机制来协调对文件的访问，确保数据的完整性和一致性。

**文件锁的作用:**

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

**4.信号量(Semaphores)**

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
```c
// 假设日志文件已经打开
FILE* logFile;

void writeToLog(const char* message) {
    sem_t* sem = sem_open("/log_semaphore", O_CREAT, 0644, 1);

    sem_wait(sem);  // 获取信号量
    fprintf(logFile, "%s\n", message);  // 写入日志
    fflush(logFile);
    sem_post(sem);  // 释放信号量

    sem_close(sem);
}

int main() {
    // ... 进程的其它操作 ...
    writeToLog("Log message from Process");
    return 0;
}

```

**匿名信号量和有名信号量 API 接口区别：**

![](https://files.mdnice.com/user/48364/0d25667d-32f0-4bcc-8751-99a524f6bcd7.png)


**5.共享内存(Shared Memory)**

**概念**：
在 Linux 中，共享内存是进程间通信（IPC）的一种形式。当多个进程需要访问相同的数据时，使用共享内存是一种高效的方式。它允许两个或多个进程访问同一个物理内存区域，这使得数据传输不需要通过内核空间，从而提高了通信效率。

在讲解共享内存前，我们需要了解内存映射技术？

**内存映射技术（Memory Mapping）** 是一种将文件或设备的数据映射到进程内存地址空间的技术，它允许进程直接对这部分内存进行读写操作，就像访问普通内存一样。这种技术不仅可以用于文件I/O操作，提高文件访问效率，而且是实现共享内存的基础。

在 Linux 系统中，内存映射可以通过 **mmap** 系统调用来实现。**mmap** 允许将文件映射到进程的地址空间，也可以用来创建匿名映射（即不基于任何文件的共享内存区域）。

在 Linux 中，共享内存可以分为如下几类。

**匿名共享内存**

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
    // 创建或打开有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);
    if (sem == SEM_FAILED) {
        // 错误处理，退出程序
        perror("sem_open failed");
        exit(EXIT_FAILURE);
    }

    // 创建匿名共享内存
    void* shared_memory = mmap(NULL, sizeof(int), PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (shared_memory == MAP_FAILED) {
      // 错误处理，退出程序
    }
    int* counter = (int*)shared_memory;
    *counter = 0; // 初始化计数器

    pid_t pid = fork();
    if (pid == 0) {
        // 子进程
        for (int i = 0; i < 10; ++i) {
            sem_wait(sem); // 等待信号量
            (*counter)++;
            printf("Child process increments counter to %d\n", *counter);
            sem_post(sem); // 释放信号量
            sleep(1); // 暂停一段时间，模拟工作负载
        }
        exit(0);
    } else if (pid > 0) {
        // 父进程
        for (int i = 0; i < 10; ++i) {
            sem_wait(sem);
            printf("Parent process reads counter as %d\n", *counter);
            sem_post(sem);
            sleep(1);
        }
    } else {
        // fork失败
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    // 清理资源
    if (pid > 0) { // 父进程等待子进程完成
        wait(NULL);
        sem_close(sem);
        sem_unlink("/mysemaphore");
        munmap(shared_memory, sizeof(int));
    }
    return 0;
}
```

**基于文件的共享内存**

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
    // 打开文件
    int fd = open(filename, O_RDWR);
    if (fd < 0) {
        // 错误处理并退出
        perror("open");
        exit(EXIT_FAILURE);
    }
    // 映射文件
    void* addr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED) {
        // 错误处理并退出
    }
    // 打开有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);
    if (sem == SEM_FAILED) {
        // 错误处理并退出
    }
    // 等待信号量，开始写入数据
    sem_wait(sem);
    strcpy((char*)addr, "Hello, Shared Memory!");
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
    // 打开文件
    int fd = open(filename, O_RDONLY);
    if (fd < 0) {
        // 错误处理并退出
        perror("open");
        exit(EXIT_FAILURE);
    }
    // 映射文件
    void* addr = mmap(NULL, size, PROT_READ, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED) {
       // 错误处理并退出
    }
    // 打开有名信号量
    sem_t *sem = sem_open("/mysemaphore", O_CREAT, 0666, 1);
    if (sem == SEM_FAILED) {
        // 错误处理并退出
    }
    // 等待信号量，开始读取数据
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

**Posix 共享内存**

POSIX 共享内存提供了一种高效的方式，允许多个进程通过共享内存区域进行通信。与基于文件的共享内存相比，POSIX 共享内存不需要直接映射磁盘上的文件，而是通过创建命名的共享内存对象来实现进程间的数据共享。这些对象虽然在逻辑上类似于文件（因为可以通过shm_open创建和打开），但实质上直接存在于内存中，提供了更快的数据访问速度。

**Posix 共享内存接口**：
```c
shm_open()        // 创建或打开一个共享内存对象
shm_unlink()      // 删除一个共享内存对象的名称
ftruncate()       // 调整共享内存对象的大小
mmap()            // 将共享内存对象映射到调用进程的地址空间
munmap()          // 解除共享内存对象的映射
```

**示例演示**：

```c
#define SHM_NAME "/example_shm"
#define SHM_SIZE 4096
int main() {
    int shm_fd;
    void *ptr;
    // 创建共享内存对象
    shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        // 错误处理并退出
        perror("shm_open");
        return 1;
    }
    // 设置共享内存大小
    if (ftruncate(shm_fd, SHM_SIZE) == -1) {
       // 错误处理并退出
    }
    // 映射共享内存
    ptr = mmap(0, SHM_SIZE, PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        // 错误处理并退出
    }
    // 写入数据到共享内存
    const char *message = "Hello, POSIX Shared Memory!";
    sprintf(ptr, "%s", message);
    printf("Data written to shared memory: %s\n", message);
    // 解除映射
    munmap(ptr, SHM_SIZE);
    // 关闭共享内存对象
    close(shm_fd);
    return 0;
}
```
**System V共享内存**

System V共享内存是一种传统的进程间通信（IPC）机制，它允许多个进程通过共享内存区域进行通信。与POSIX共享内存不同，System V共享内存使用IPC键值key_t来标识和管理共享内存段，而不是通过命名的方式。这种机制提供了一套底层控制共享内存的API，允许进行更细粒度的操作，如权限控制、共享内存状态的查询和管理等。

**System V共享内存接口**：
```c
shmget()         // 创建或获取共享内存段的标识符
shmat()          // 将共享内存段附加到进程的地址空间
shmdt()          // 分离共享内存段和进程的地址空间
shmctl()         // 对共享内存段执行控制操作
```

**示例演示**：

```c
#include <sys/ipc.h>
#include <sys/shm.h>
int main() {
    key_t key = ftok("somefile", 65); // 创建IPC键
    int shm_id;
    void *ptr;
    // 创建共享内存段
    shm_id = shmget(key, 1024, 0666|IPC_CREAT);
    if (shm_id < 0) {
        perror("shmget");
        return -1;
    }
    // 将共享内存段附加到进程的地址空间
    ptr = shmat(shm_id, (void*)0, 0);
    if (ptr == (void*) -1) {
        perror("shmat");
        return -1;
    }
    // 在共享内存上操作，例如写入数据
    // 示例：写入一个字符串
    strcpy(ptr, "Hello, System V Shared Memory!");
    // 分离共享内存段
    if (shmdt(ptr) < 0) {
        perror("shmdt");
        return -1;
    }
    // 删除共享内存段
    shmctl(shm_id, IPC_RMID, NULL);
    return 0;
}

```

**6.消息队列 (Message Queues)**

**概念**：

消息队列是一种允许一个或多个进程向其写入消息，并由一个或多个进程读取消息的 IPC 机制。每条消息都由一个消息队列标识符（ID）识别， 且可以携带一个特定的类型。消息队列允许不同进程非阻塞地发送和接收记录或数据块，这些记录可以是不同类型和大小的。

**消息队列图解：**

![](https://files.mdnice.com/user/48364/b3ab33df-59d9-4b30-9539-9b3a0dd019ce.png)


**使用场景**：

- **进程间通信：**
  在涉及多个运行进程的应用中，消息队列提供了一种高效的方式来传递信息。它允许进程之间无需直接相互连接就能交换数据，从而简化了通信过程。

- **异步数据处理：**
  消息队列使进程能够异步处理信息。一个进程（即生产者）可以发送任务或数据至队列，并继续其他操作，而另一进程（即消费者）可以在准备就绪时从队列中取出并处理这些数据。这种模式有效地分离了数据的生成和消费过程，提高了应用的效率和响应速度。实际的应用比如：日志记录，某些系统可能有一个专门的进程负责记录日志，其他进程可以将日志消息发送到消息队列，由该专门进程异步地写入日志文件。


**以下是使用 System V IPC 消息队列的一个简单示例:**

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

**7.套接字 (Sockets)**

**概念**：

套接字是一种在不同进程间进行数据交换的通信机制。在 Linux 中，套接字可以用于同一台机器上的进程间通信（IPC）或不同机器上的网络通信。套接字支持多种通信协议，最常见的是TCP（可靠的、连接导向的协议）和UDP（无连接的、不可靠的协议）。

**简单图解：**

![](https://files.mdnice.com/user/48364/0b9d8bec-8add-40e9-82f7-f55f10933114.png)


**使用场景：**

**网络通信**：
同一台主机或不同主机上的进程之间通过网络套接字进行数据交换。

**简单示例：** - 使用 TCP 套接字进行通信
```c
// 服务器端（监听和接收数据）:
int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[1024] = {0};
    server_fd = socket(AF_INET, SOCK_STREAM, 0);  // 创建套接字
    // 定义套接字地址
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);
    // 绑定套接字
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 3);    // 监听套接字
    while(1) {
        // 接受连接
        printf("Waiting for a connection...\n");
        new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen);
        // 读取数据
        read(new_socket, buffer, 1024);
        printf("Message: %s\n", buffer);
        // 可以在这里处理收到的消息或执行其他任务
        close(new_socket);  // 关闭这次连接的套接字
    }

    // 关闭监听的套接字
    // 注意：由于 while(1)，这行代码不会执行，除非在循环中加入退出条件
    close(server_fd);
    return 0;
}

// 客户端进程（发送数据）:
int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    sock = socket(AF_INET, SOCK_STREAM, 0); // 创建套接字
    serv_addr.sin_family = AF_INET; // 定义套接字地址
    serv_addr.sin_port = htons(8080);
    connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr));    // 连接到服务器
    
    // 发送数据
    char *message = "Hello from the client!";
    send(sock, message, strlen(message), 0);
    close(sock);
    return 0;
}
```

**8.域套接字 (Unix Domain Sockets)**

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

```c
//服务器端（监听和接收数据）:
int main() {
    int server_fd, client_socket;
    struct sockaddr_un address;
    server_fd = socket(AF_UNIX, SOCK_STREAM, 0);  // 创建套接字
    address.sun_family = AF_UNIX;     // 设置套接字地址
    strcpy(address.sun_path, "/tmp/unix_socket");
    // 绑定和监听
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 5);
    while(1){
        // 接受连接
        client_socket = accept(server_fd, NULL, NULL);

        // 处理数据
        char buffer[100];
        read(client_socket, buffer, 100);
        printf("Received: %s\n", buffer);
        // 进行其他的业务处理
        // ...
        
        close(client_socket);
    }
    // 清理
    close(server_fd);
    unlink("/tmp/unix_socket");
    return 0;
}

// 客户端（发送数据）:
int main() {
    int sock;
    struct sockaddr_un address;
    sock = socket(AF_UNIX, SOCK_STREAM, 0); // 创建套接字
    address.sun_family = AF_UNIX;           // 设置套接字地址
    strcpy(address.sun_path, "/tmp/unix_socket");
    // 连接到服务器
    connect(sock, (struct sockaddr *)&address, sizeof(address));
    // 发送数据
    char *message = "Hello from the client!";
    write(sock, message, strlen(message));
    // 清理
    close(sock);
    return 0;
}
```

**注意事项**：

- Unix 域套接字的地址是文件系统中的路径，而不是IP地址和端口。
- Unix 域套接字通常用于同一台机器上的进程间通信，而不适用于网络通信。
- 使用 Unix 域套接字时，需要确保套接字文件的路径是可访问的，并在通信完成后清理套接字文件。


### Linux 线程

![](https://files.mdnice.com/user/48364/1bfac283-f8c1-43eb-9c6b-530b7098a5b0.png)


#### 什么是线程？

线程，有时被称为“轻量级进程”，是程序执行流的最小单位。它允许多任务在单个进程内部并发执行。

#### 线程与进程的区别：

- **进程**: 拥有独立的地址空间和资源。
- **线程**: 共享其所在进程的资源，但有自己的堆栈空间。

#### 创建你的第一个线程

在 Linux 下，我们使用 POSIX Threads （简称 Pthreads）库来操作线程。以下是一个简单的例子，创建并运行两个线程：

```c
#include <stdio.h>
#include <pthread.h>

// Thread 1 function
void* func1(void* arg) {
    printf("Hello from thread 1!\n");
    return NULL;
}

// Thread 2 function
void* func2(void* arg) {
    printf("Hello from thread 2!\n");
    return NULL;
}

int main() {
    pthread_t thread1, thread2;

    pthread_create(&thread1, NULL, func1, NULL);    // Create thread 1
    pthread_create(&thread2, NULL, func2, NULL);    // Create thread 2

    pthread_join(thread1, NULL); // Wait for thread 1 to finish
    pthread_join(thread2, NULL); // Wait for thread 2 to finish
    return 0;
}

```

#### 线程同步：何时使用？

当两个或多个线程想要访问同一个资源时，问题就来了！如何确保资源的安全访问？有以下三种线程同步的方式。

- **互斥锁**: 一个线程在使用资源时，锁住它，其他线程等待。一般用在临界区的保护。
- **条件变量**: 线程等待直到某个条件满足。一般和互斥锁搭配使用来实现线程同步 。
- **信号量**: 一种高级的同步方式，可以控制资源的访问数量。

 **信号量更为通用**，因为它不仅可以用作互斥锁，还可以用来同步线程，例如 ：确保线程按特定的顺序执行或控制对有限资源的访问。

**确保线程按特定的顺序执行**：

在某些场景下，您可能希望线程以特定的顺序执行。例如，线程 A 必须在线程 B 之前执行。这可以通过使用信号量来实现。

**控制对有限资源的访问**：

信号量也可用于控制对有限资源的访问。例如，数据库连接池，其中只有一定数量的连接可供线程使用，可以使用信号量来确保只有固定数量的线程可以同时访问这些资源。

**确保线程按特定的顺序执行的示例代码**：

```c
sem_t semA;
// 线程A
void* threadA(void* arg) {
    printf("Thread A is running\n");
    sem_post(&semA); // 释放信号量A
    return NULL;
}

// 线程B
void* threadB(void* arg) {
    sem_wait(&semA); // 等待信号量A
    printf("Thread B is running\n");
    return NULL;
}

int main() {
    pthread_t tA, tB;
    // 初始化信号量
    sem_init(&semA, 0, 0);
    // 创建线程
    pthread_create(&tA, NULL, threadA, NULL);
    pthread_create(&tB, NULL, threadB, NULL);
    // 等待线程结束
    pthread_join(tA, NULL);
    pthread_join(tB, NULL);
    // 清理资源
    sem_destroy(&semA);
    return 0;
}
```

#### 线程的优点与缺点:

**优点**:
- 线程之间的切换成本比进程之间的切换成本低。
- 线程间的通信速度比进程间的通信速度快，因为线程共享同一地址空间。
- 利用多线程可以很容易地在单进程应用中实现并发。

**缺点**:
- 因为线程共享同一地址空间，一个线程的错误可能会破坏其他线程的数据或状态。
- 需要复杂的同步操作来避免竞争条件。

#### 常见问题与挑战

**死锁**:

死锁发生在两个或多个线程永久地等待对方释放锁的情况。它通常发生在多个线程需要多个锁时，如果不按相同的顺序获取锁，就可能陷入互相等待的状态。

**解决方案**：

- 确保所有线程以相同的顺序获取锁。
- 使用层次结构的锁定系统，其中线程必须按特定顺序获取锁。
- 设置超时，以便在等待锁的时间过长时，线程可以放弃等待，尝试其他操作。

**线程安全**：

线程安全是指确保代码可以在多线程环境中安全运行，不会因为多个线程同时访问共享资源而导致数据损坏或不一致。

**解决方案：**

- **使用同步机制**，如互斥锁或信号量，来控制对共享资源的访问。
- **编写无状态的代码，或者确保状态信息不在多个线程间共享。** 无状态的代码指的是不保存任何与特定实例相关的数据（状态）的代码。在多线程环境中，这意味着代码不依赖于或不修改任何外部状态，如全局变量或类的成员变量。
- **使用不可变对象**，这些对象一旦创建就不会更改，因此可以安全地在多个线程间共享。不可变对象是指一旦被创建就不能被修改的对象(如字符串)，这些对象的状态在创建后是固定的，因此在多线程环境中安全。

> **总结**：编写无状态的代码和使用不可变对象都是避免多线程环境中的数据冲突和竞争条件的策略。无状态代码避免了共享数据，而不可变对象则确保了即使数据被共享，它们也不会被修改，从而保证线程安全。

####  进一步探索

**线程池**: 

线程池通过重用一组预先创建的线程来处理任务，减少了线程创建和销毁的开销。

**应用**：线程池广泛用于网络服务器应用，特别是在需要处理大量短暂任务的场景中。


**高级同步原语**: 

**读写锁（Read-Write Locks）**

读写锁是一种特殊类型的锁，它允许多个线程同时读取共享资源，但写入操作需要独占访问。这意味着只要没有线程正在写入共享资源，多个线程可以同时读取资源而不会被阻塞。

**应用场景**：适用于读操作远多于写操作的情况，比如缓存系统。

**优点**：提高了在读多写少场景下的并发性能。

**实现**：在 POSIX 线程库中，通过 pthread_rwlock_t 类型提供。

**简单示例**：
```c
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;
void* reader(void* arg) {
    pthread_rwlock_rdlock(&rwlock);
    printf("Reader is reading...\n");
    sleep(1); // 模拟读取操作
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}
void* writer(void* arg) {
    pthread_rwlock_wrlock(&rwlock);
    printf("Writer is writing...\n");
    sleep(1); // 模拟写入操作
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}
int main() {
    pthread_t t1, t2;
    pthread_rwlock_init(&rwlock, NULL);
    pthread_create(&t1, NULL, reader, NULL);
    sleep(2); // 确保读者先运行
    pthread_create(&t2, NULL, writer, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_rwlock_destroy(&rwlock);
    return 0;
}
```

**屏障（Barriers）**

屏障用于同步多个线程在程序中的特定点。当线程到达一个屏障时，它会等待，直到所有其他线程也都到达这个屏障。然后所有线程才能继续执行。

**应用场景**：用于并行算法，确保所有线程完成某个阶段的工作后才开始下一个阶段。

**优点**：确保所有线程同步进行，避免数据不一致。

**实现**：在 POSIX 线程库中，通过 pthread_barrier_t 类型提供。

**简单示例**：
```c
#define NUM_THREADS 5
pthread_barrier_t barrier;
void* task(void* arg) {
    printf("Thread %ld waiting at barrier\n", (long)arg);
    pthread_barrier_wait(&barrier);
    printf("Thread %ld passed barrier\n", (long)arg);
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    pthread_barrier_init(&barrier, NULL, NUM_THREADS);
    for (long i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, task, (void*)i);
    }
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    pthread_barrier_destroy(&barrier);
    return 0;
}
```

**说明**：这个程序演示了如何使用屏障来同步多个线程，确保所有线程都到达一个执行点后才一起继续执行。在这个例子中，所有线程都会在打印“等待”信息后等待，直到它们全部到达 pthread_barrier_wait 调用处。只有当所有线程都到达这个点时，它们才会继续执行并打印“通过”信息。

**原子操作（Atomic Operations）**

原子操作是指在多线程环境中，一系列操作作为一个单独的不可中断的单位执行，确保在读取、修改和更新变量时的原子性。这些操作在执行的全过程中不会被线程调度机制中断。

**应用场景**：

非常适合于计数器、标志位更新等简单状态的更新场景，其中对单一变量的读取、修改和更新必须作为一个整体来执行，以避免数据竞争和保证数据一致性。

**优点**：

- **效率**：相比锁机制，原子操作通常更高效，因为它们避免了锁的开销和潜在的上下文切换。
- **简化编程模型**：对于简单的同步需求，原子操作提供了一种简单直接的解决方案，避免了使用锁的复杂性。

**实现**：在 POSIX 线程库中，原子操作并非直接提供，但可以通过 GCC 提供的内建原子操作函数，如__sync_fetch_and_add、__sync_lock_test_and_set等。C++11及更高版本的标准也提供了原子操作的支持，如 std::atomic 类型。

**简单示例**：
```c
// 定义一个全局计数器
volatile int counter = 0;
// 线程函数，用于增加计数器
void* increment_counter(void* arg) {
    int i;
    for (i = 0; i < 10000; ++i) {
        // 使用GCC的内建原子操作函数进行原子增加
        __sync_fetch_and_add(&counter, 1);
    }
    return NULL;
}
int main() {
    pthread_t t1, t2;
    // 创建两个线程，都执行increment_counter函数
    pthread_create(&t1, NULL, increment_counter, NULL);
    pthread_create(&t2, NULL, increment_counter, NULL);
    // 等待线程完成
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    // 打印最终的计数器值
    printf("Final counter value: %d\n", counter);
    return 0;
}

```

**自旋锁（Spinlocks）**

自旋锁是一种忙等待的锁，当一个线程尝试获取一个已经被其他线程持有的锁时，它会在一个循环中不断检查锁的状态。这意味着线程会一直占用 CPU，直到它能够获取到锁。

**应用场景**：

特别适合锁持有时间非常短的场景，因为它避免了线程从运行态转为等待态的开销，这在多核处理器上尤其有用。

**实现**：在 POSIX 线程库中，自旋锁通过 pthread_spinlock_t 类型提供，相关的操作包括 pthread_spin_lock、pthread_spin_unlock等。自旋锁的使用和管理相对简单，但需要谨慎使用以避免过度占用 CPU 资源。

**简单示例**：
```c
pthread_spinlock_t spinlock;
void* task(void* arg) {
    pthread_spin_lock(&spinlock);
    printf("Thread %ld got the lock\n", (long)arg);
    sleep(1); // 模拟任务执行
    pthread_spin_unlock(&spinlock);
    return NULL;
}
int main() {
    pthread_t t1, t2;
    pthread_spin_init(&spinlock, PTHREAD_PROCESS_PRIVATE);
    pthread_create(&t1, NULL, task, (void*)1L);
    pthread_create(&t2, NULL, task, (void*)2L);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_spin_destroy(&spinlock);
    return 0;
}
```

### 内存管理入门

在前面的讲解中，我们已经学习了进程和线程的基本概念，了解了它们是操作系统进行资源分配和任务调度的基本单位。而无论是进程还是线程，它们的运行都离不开一个关键的系统资源——内存。这自然引出了一个重要的问题：操作系统是如何管理这些内存资源的？这正是我们接下来要讨论的主题— **Linux内存管理**。

#### 内存分配与释放

首先，我们先来看下内存的分配与释放，常见的内存分配方式包含以下两种：

**静态内存分配** ： 是在编译时完成的，通常用于固定大小的数据结构，比如：普通数组。

**动态内存分配** ： 则在运行时进行，允许程序根据需要分配任意大小的内存块，比如：动态数组。

我们一般使用 **malloc和free** 来进行动态内存分配与释放。

来看个动态内存分配的例子：
```
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *array = malloc(10 * sizeof(int)); # 分配内存
    if (array == NULL) {
        perror("malloc failed");
        return EXIT_FAILURE;
    }
    #使用动态内存 array ...

    free(array); # 释放内存
    return 0;
}
```

#### 内存泄露

应用程序如果没有正确的管理内存的分配与回收，就有可能出现内存泄漏，严重点的有可能导致程序异常退出。

**那什么是内存泄露？**

内存泄露是指程序中动态分配的内存没有及时释放，导致这部分内存在程序执行过程中一直占用，无法被再次利用。在长时间运行的程序中，内存泄露可能会导致内存使用不断增加，最终耗尽所有可用内存，影响程序性能甚至引发程序崩溃。

**如何避免内存泄露？**

- **合理设计程序结构**：确保每次 malloc 后都有对应的 free 操作。可以通过使用自动化工具，如 Valgrind 等，来检测程序运行中的内存泄露问题。

- **使用智能指针**：在支持 C++ 等高级语言中，使用智能指针（如 std::unique_ptr, std::shared_ptr 等）可以帮助管理动态内存的生命周期，智能指针会在适当的时候自动释放内存。

- **及时释放内存**：在不需要动态分配的内存后，应立即释放。尤其是在异常处理、错误处理的代码路径中，也不要忘记释放内存。

- **规范化资源管理**：使用 RAII（Resource Acquisition Is Initialization）原则管理资源，**确保资源的获取即是初始化，随着对象的销毁资源被释放**。

#### 虚拟内存管理

**虚拟内存概念：**

虚拟内存是计算机系统内存管理的一种技术。它使得应用程序认为它拥有很大连续的、可用的内存空间，即使这些内存可能被分散存储在物理内存和磁盘上。

**虚拟内存的主要好处是**：
- 它提供了比实际物理内存更大的地址空间。
- 保证每个程序在内存中有一个连续的地址空间。
- 允许系统运行大于物理内存的程序。
- 通过内存隔离，提高了程序间的安全性。

操作系统通过使用硬盘上的一块称为“交换空间”的区域来实现这一点，它作为物理内存的一个扩展。当系统的物理 RAM 不足时，它可以将当前不活跃的内存页面移动到磁盘上，从而为需要更多内存的进程腾出空间。

**分页机制**

分页是虚拟内存管理中最常用的技术之一。它将虚拟内存和物理内存分成大小相等的块，这些块在虚拟内存中被称为“页”(pages)，在物理内存中被称为“页框”(page frames)。每个程序都有一个页表，页表将程序的虚拟地址映射到物理内存的页框。

**分页机制如何工作：**

1.当程序试图访问虚拟内存中的地址时，它首先会检查页表。

2.如果找到了对应的物理地址，那么数据的存取操作就会继续。

3.如果没有找到，会触发一个缺页中断，由操作系统处理。

**缺页中断**

缺页中断（Page Fault）是分页系统中的一项关键机制，当一个进程访问的虚拟页不在物理内存中时触发。这时候，操作系统会分配一个物理页框，并将该虚拟页所对应的磁盘数据加载至页框中，并在页表中建立虚拟页和物理页的映射关系。这样，当下一次进程在访问相同虚拟页的时候，就可以直接访问内存中的数据了。

通过以上机制，虚拟内存管理提供了高效灵活的内存使用方式，允许操作系统优化内存分配，同时也给应用程序提供了简单的内存管理模型。

### 文件系统：探索 Linux 中的数据管理

前面我们探讨了 Linux 系统中的内存管理，包括内存分配与释放、内存泄漏和虚拟内存等概念，这些都是操作系统保证程序正常运行的基础。内存管理使得多个应用能够高效、安全地共享系统的物理内存资源，同时还提供了数据的临时存储能力。然而，内存只能提供临时存储，当系统断电或重启时，内存中的数据就会丢失。这就引出了我们下一个重要话题：**文件系统**。

在谈文件系统之前，我们先来了解下虚拟文件系统。

#### 虚拟文件系统（VFS）

**什么是VFS？**

Linux内核中的虚拟文件系统（VFS）是一个关键的抽象层，它为各种不同的文件系统提供了一个统一的操作接口。这意味着，不管数据实际上存储在哪个文件系统中（比如EXT4、XFS等），VFS都能提供一致的访问方式。

**VFS的作用**

**兼容性**：使得不同的文件系统都能在 Linux 上工作。

**统一性**：它为应用程序提供了一个标准的文件操作接口，简化了文件访问和管理。

接下来让我们来看下文件系统。

**Linux 的文件系统是什么？**

Linux 文件系统是 Linux 操作系统用于存储、管理和访问文件和目录的一套规则和结构。它提供了一个层次化的目录结构，让用户和程序能够以一致的方式组织和访问数据。Linux 文件系统支持多种类型，如 EXT4、XFS 和 Btrfs，每种都有其特定的优势和用途。文件系统管理文件的存储细节，包括文件的创建、读取、写入和删除操作，同时也处理文件的权限和安全性。通过虚拟文件系统（VFS）层，Linux 能够提供一个统一的接口来访问这些不同的文件系统，使得文件操作对用户和应用程序透明。

**文件系统核心组件:**

**超级块（Superblock）**

超级块是文件系统的元数据的一部分，它包含了关于整个文件系统的全局信息，如文件系统的类型、大小、状态、空闲和已用的块和Inode数量等。超级块的主要作用是提供文件系统的关键信息，以便操作系统能够正确地管理和访问文件系统。

**Inode**

Inode 是文件系统中的一个关键数据结构，每个文件和目录都有一个唯一的Inode。它包含了文件的元数据（如文件大小、所有者、权限、时间戳）和指向实际存储文件数据的数据块的指针。Inode 不存储文件名，文件名存储在目录文件中，这些目录文件将文件名映射到 Inode 号。**inode** 号是文件的唯一标识，而不是文件名。

**目录项（Dentry）**

目录项（或Dentry缓存）是内核用来维护文件名与其对应Inode之间映射的结构。目录项缓存是一个重要的性能优化机制，它减少了从文件名到文件内容的查找时间。

**文件数据块**

文件数据块是存储文件实际内容的磁盘空间。Linux文件系统将磁盘空间分割成一系列的块，这些块可以直接被Inode指向，或者通过间接块来存储较大文件的数据。

**文件和目录**

文件和目录是用户与文件系统交互的基本单元。在 Linux 中，一切皆文件：传统的数据文件、目录、设备（如字符设备和块设备）等都通过文件或文件系统的接口来访问。

**下面是文件、目录、inode 、以及数据块之间的映射关系图**：

![](https://files.mdnice.com/user/48364/4936695c-fe7d-4329-a222-0ddc3f49fd6f.png)

我以程序访问磁盘文件为例，来给大家说明下具体的访问过程，方便大家理解上述图示。

**操作系统会执行以下几个步骤**：

- **解析文件路径**：操作系统首先解析完整的文件路径，确定文件在文件系统中的位置。

- **查找目录项**：利用文件路径，操作系统在文件系统的目录结构中查找对应的目录项（Dentry）。目录项将文件名映射到一个唯一的Inode编号。

- **访问Inode**：每个文件都有一个Inode，其中包含该文件的元数据（如所有者、权限）和指向文件实际数据块的指针。操作系统使用目录项提供的 Inode 编号来访问 Inode Table，进而访问对应的 inode。

- **读取数据块**：通过 Inode 中的信息，操作系统找到存储文件数据的磁盘块位置，然后读取这些数据块以获取文件内容。

除此之外，在 Linux 中，还存在两种特殊的引用文件的方式：**硬链接和软链接**

#### 硬链接和软链接

**什么是硬链接？**

硬链接实际上是目标文件的另一个名称。它与原文件共享相同的 **inode** 号，因此，无论通过哪个名称访问，内容都是一致的。

**图示**：

![](https://files.mdnice.com/user/48364/d19f40f3-352b-44f5-a41d-815ccca7b86d.png)

这里，“file1”和“link1”都是硬链接，它们指向同一个inode。这意味着它们共享相同的数据块和文件属性。

**如何创建硬链接？**

**命令**：` ln 源文件 目标文件`

例如，创建一个名为 file1 的文件的硬链接 link1，你可以使用：ln file1 link1。

**特点**：

- 硬链接不能跨文件系统。
- 不能为目录创建硬链接。
- 删除原始文件或硬链接中的任何一个不会影响其他文件，因为它们共享相同的数据块。

**什么是软链接？**

与硬链接不同，软链接是一个独立的文件，它并不包含实际的文件内容，而是指向另一个文件或目录的路径。

**图示**：

![](https://files.mdnice.com/user/48364/c1250f8d-b856-41f0-b62f-6784e85bef42.png)

在这里，“link1”是一个指向“file1”的软链接。与硬链接不同，软链接只是一个指向另一个文件或目录的路径。当我们访问软链接时，系统会自动重定向我们到它所指向的实际文件。

**如何创建软链接？**

**命令**：` ln -s 源文件 目标文件`

例如，为 file1 创建一个软链接 link1，你可以使用：ln -s file1 link1。

**特点**：

- 软链接可以跨文件系统。
- 可以为目录创建软链接。
- 如果删除了目标文件，软链接会变为死链接，无法再访问原始内容。


## 总结

这篇文章主要是为想学习 Linux 系统编程的初学者提供一个学习指南，从基本概念到高级功能，我们不仅揭示了 Linux 系统的核心技术和架构，还探讨了用户空间与内核空间的关键区别，系统调用与库函数的基本理解，以及文件IO的多样化操作。我们学习了进程和线程的基础，理解了它们之间的差异，以及如何有效地使用线程同步技术来编写稳定的多线程程序。此外，我们还涵盖了内存管理的基础知识，从内存分配与释放到虚拟内存管理，最后学习了 Linux 文件系统的基本概念及其核心组件，以及硬链接和软链接的使用和区别。

无论你是刚开始接触 Linux 系统编程的新手，还是希望巩固现有知识的经验开发者，本文都提供了宝贵指南。

通过本文的学习，我希望读者能够：

- 掌握 Linux 系统架构的关键组成部分，包括用户空间和内核空间的区别。
- 理解系统调用和库函数的作用，以及它们在系统编程中的重要性。
- 熟练进行文件IO操作，包括文件描述符的使用，文件位置的移动，以及高级文件I/O技术的应用。
- 了解进程和线程的基本概念，包括它们的创建、终止和状态转换，以及进程间通信的方法。
- 掌握线程同步的技巧，了解线程的优缺点以及在实际编程中的应用。
- 建立内存管理的基本知识框架，包括内存分配释放、虚拟内存管理以及如何避免内存泄露。
- 探索 Linux 文件系统，理解虚拟文件系统（VFS）的概念，以及硬链接和软链接的使用和区别。

## 🚀 跟我学，你能收获啥？

在这里，你不仅能看到干货，还能真正学到能用的技能：

+ **Linux 实战技巧**：服务器调优、常用命令、Shell 脚本，让你像高手一样操作系统。
+ **C/C++ 后台开发**：从基础语法到高性能编程，带你写出稳、快、可维护的服务端代码。
+ **C/C++ 项目实战**：真实项目案例，教你从需求到上线完整流程，掌握开发套路和最佳实践。
+ **常用开发工具**：调试、版本控制、构建工具、性能分析工具，让开发效率大幅提升。
+ **性能优化**：CPU/内存/IO 调优技巧，定位瓶颈，让你的程序跑得更快更稳。
+ **项目架构设计**：微服务、分层架构、模块设计思路，帮你构建可扩展、易维护的系统。
+ **Go 后端开发**：微服务、云原生实战，教你用 Go 搭建高并发、高可用系统。
+ **编程面试干货 & 算法**：核心算法套路、面试高频题解析，让你不再手忙脚乱。
+ **计算机基础梳理**：操作系统、网络、数据结构、并发原理，知识体系清晰明了。
+ **成长路线图**：系统规划你的学习路径，从初学到高级，帮你少走弯路。

内容**深入浅出、实用有趣**，再也不用看书看到睡着。  
无论是面试冲刺，还是技能升级，这里都是你的“技术加油站”。


## 👀 想加入？很简单！
**扫一扫下面二维码**，一键关注公众号，开启你的技术学习之旅！

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

另外，我还建了一个**技术交流群**，里面都是认真写代码的小伙伴，不吹牛、不闲聊，只聊技术。  
有问题？大家一块儿讨论，比一个人闷头学效率高多了！

![](https://files.mdnice.com/user/48364/4ebc72e9-e4bb-447a-9a92-8367a178df6d.png)

技术这条路，一个人走容易迷路，一群人走才有方向。  
跟上节奏，我们一起变强 💪


