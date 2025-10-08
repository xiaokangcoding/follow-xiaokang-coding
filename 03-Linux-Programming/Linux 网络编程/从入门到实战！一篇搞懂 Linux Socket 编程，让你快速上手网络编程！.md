# 前言：从“进程对话”到“计算机通话”，这一次我们聊聊 Socket！

大家好，我是小康。

在上一篇文章中，我们聊了进程间通信（IPC），那是计算机内部的“自言自语”——不同进程之间如何共享信息。而今天，我们要迈出更大一步：
让你的程序，不仅能和同一台电脑上的其他进程对话，还能和世界上任意一台计算机“打电话”！

没错，这就是 Socket 编程 的世界。
它就像一根看不见的“网络电话线”，让不同主机、不同进程之间可以建立连接、发送消息、收发数据。你每天打开的网页、使用的微信、播放的音乐，其背后都离不开 Socket 通信。

很多同学一看到 “socket 编程” 就会觉得头疼，好像涉及网络协议、端口、字节序这些晦涩难懂的概念。
但其实，只要抓住核心原理，Socket 远没有你想象的复杂。它更像是一个可编程的“网络插座”，你只要知道怎么插上、怎么发、怎么收，就能轻松上手网络通信。

> 💡 学习建议：别只看，动手敲代码！每一个 send、recv、bind 都是理解网络通信的关键。
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

准备好了吗？

让我们从“进程之间的对话”走向“计算机之间的交流”，一起揭开 Linux Socket 编程 的神秘面纱吧！

#### 一、socket 是什么鬼？

别急着写代码，我们先来聊聊，什么是 socket？

大白话来说，socket 就像你家里的插座。你把电器插进去，通电了，它就能工作。socket 也是类似的东西，只不过它连接的不是电，而是网络数据。我们可以用 socket 让两台计算机进行交流，比如让你的电脑和远程服务器握个手，聊聊天。

实际上，**socket 就是 IP 地址和端口号的组合，它用来标识网络中的一个具体通信端点**。就像每个房子都有一个门牌号（IP 地址）和门（端口号），只有两者结合在一起，才能唯一标识网络中的某个应用程序。socket 可以理解为你在网络世界中打电话的“电话机”。你拨号，连接上对方，两个人就可以对话——这就是网络通信的本质！

### IP 地址和端口号是什么？

在继续讲解 socket 编程之前，先简单介绍一下 IP 地址和端口号，帮助你更好理解 socket 的工作原理。

- **IP 地址**：IP 地址就像是互联网中的“门牌号”，它唯一标识了一台设备的位置。无论是你的电脑、手机，还是服务器，它们都需要有一个独特的 IP 地址，这样其他设备才能找到它们的“位置”。

- **端口号**：端口号可以理解为房子中的“房间号”，用来标识设备上正在运行的具体服务或应用程序。比如说，你家有一个门牌号表示位置，但家里的不同房间有不同的功能，端口号就是这样的“房间号”。不同的服务有不同的端口号，比如网页服务通常用 80 端口，而安全的 HTTPS 服务用 443 端口。

结合起来，IP 地址和端口号就像是找到某台设备上具体服务的方式。你去某个地址（IP 地址）拜访朋友，然后敲开他家的某个房间门（端口号）找到他，这就是 IP 地址和端口号的工作方式。

#### 二、socket 编程流程：做个“电话”要几步？

用 socket 编程呢，可以做 TCP 聊天程序，也可以做 UDP 聊天程序。不过，TCP 是大部分场景里最常见的‘香饽饽’，所以今天我们主要来聊聊怎么搞定一个 TCP 服务器。

首先，要让两个程序通过网络对话，你得先准备两个“电话机”：

1. **服务器端**：这是接电话的人，需要等着别人拨号联系。
2. **客户端**：这是打电话的人，主动拨号，建立连接。

来看张图：

![](https://files.mdnice.com/user/71186/2ba1c7b6-dd3a-4326-a120-319f15cae31a.png)

> **小提示**：建立 TCP 连接实际上是通过三次握手完成的，关于三次握手的流程，大家可以自行百度看看，很简单的。

就像打电话聊天一样，socket 通信也有一套步骤。在具体讲这些步骤之前，先来认识一下常用的套接字 API，先混个眼熟，稍后我们会详细讲解它们。

套接字常用 API：

```c
socket()                    : 创建套接字
bind()                      : 绑定套接字到本地地址
listen()                    : 监听网络连接
accept()                    : 接受网络连接
connect()                   : 连接到远程主机
send(), recv()              : 发送和接收数据（面向连接的套接字）
sendto(), recvfrom()        : 发送和接收数据（无连接的套接字）
close() ,shutdown()         : 关闭套接字
getsockopt(), setsockopt()  : 获取和设置套接字选项
```


接下来我们详细来看下 socket 通信的具体步骤,这里我分服务器和客户端两边来讲：

**服务器端的基本步骤**：

1. **创建 socket**：先做个“电话机”，用代码来说就是调用 `socket()` 函数。
2. **绑定地址**：做出来的电话机得有个“号码”，也就是IP和端口号，用 `bind()` 来指定。
3. **监听**：`listen()`，挂在那儿等电话。
4. **接电话**：如果有电话打过来了，就用 `accept()` 接通。
5. **收发数据**：双方可以通过 `send()` 和 `recv()` 函数收发数据，这就相当于双方聊天。
6. **关闭连接**：聊完了，`close()` 挂电话。

**客户端的基本步骤**：

1. **创建 socket**：跟服务器端一样，也得先做个电话机，用 `socket()`。
2. **连接服务器**：用 `connect()` 去拨号，目标是服务器的 IP 和端口号。
3. **收发数据**：跟服务器一样，可以用 `send()` 和 `recv()` 来聊天。
4. **关闭连接**：聊完了，`close()` 挂电话。

再来看个图，更直观点：

![](https://files.mdnice.com/user/71186/2c7e04b1-6a97-4e0a-af63-07f70e788a35.png)

> **注意**：上面说的是实现 TCP 服务器的步骤，如果是实现 UDP 服务器，则不需要建立连接就可以互发数据。


### 三、简单代码示例

不搞那些复杂的花里胡哨，我们用简单的代码示例看看服务端和客户端怎么写。以下是基于 Linux C 实现的一个极简的 socket 通信例子。

**1. 服务端代码流程：**

```c++
// 创建 socket
server_fd = socket(AF_INET, SOCK_STREAM, 0);

// 设置套接字地址
struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;  // 代表 IPV4
server_addr.sin_port = htons(12345); 
server_addr.sin_addr.s_addr = INADDR_ANY; 

// 绑定地址和端口
bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));

// 监听连接
listen(server_fd, 5);

// 接收客户端连接
client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_addr_len);

// 接收和发送数据
read(client_fd, buffer, sizeof(buffer));
write(client_fd, message, strlen(message));

// 关闭连接
close(client_fd);
close(server_fd);
```

**2. 客户端代码流程：**

```c++
// 创建 socket
client_fd = socket(AF_INET, SOCK_STREAM, 0);

// 设置服务器地址
server_addr.sin_family = AF_INET;  // 代表 IPV4
server_addr.sin_port = htons(12345); // 12345 代表服务器应用程序的端口
server_addr.sin_addr.s_addr = inet_addr("192.168.1.1"); // 192.168.1.1 是服务器的地址

// 连接服务器
connect(client_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));

// 发送和接收数据
write(client_fd, message, strlen(message));
read(client_fd, buffer, sizeof(buffer));

// 关闭连接
close(client_fd);
```


### 四、socket 编程的一些细节

1. **TCP vs UDP**：
+ TCP就像打电话，你要先拨号、接通，然后才能聊（有连接的，可靠）。
+ UDP就像寄快递，直接把包裹丢出去，不管对方有没有收到（无连接，不可靠）。
2. **Socket 类型和协议家族**：
+ 想象 socket 类型就像不同的通讯工具，`SOCK_STREAM` 是类似于电话，需要建立连接（TCP），而 `SOCK_DGRAM` 更像是广播（UDP），无需连接，直接发送。
+ 协议家族有很多，比如 `AF_INET`（IPv4）和 `AF_INET6`（IPv6），你可以理解为用不同的“国家区号”来打电话。
3. **端口号选择：系统专用的VIP通道**
+ 端口号有 `0-65535` 之分。`0-1023` 是给系统的“VIP通道”，比如 HTTP 的 80 端口，SSH 的 22 端口，一般人用不了。我们普通开发者用 `1024` 以上的“普通通道”，选一个不冲突的就好。
4. **阻塞与非阻塞**：
+ 代码中的 `accept()` 或 `recv()` 默认是阻塞的，也就是等不到数据不会继续往下走。如果你不想让程序卡住，可以把 socket 设为非阻塞模式。
+ 设置非阻塞模式：`fcntl(socket_fd, F_SETFL, O_NONBLOCK); `
5. **超时设置**：
+ 你知道等待一个没接的电话是什么感觉吗？socket 也可能因为对方不响应而“卡住”。你可以使用 `setsockopt()` 为 socket 设置超时时间，例如只等对方 5 秒，超时了就不再等待。

```c

struct timeval timeout;
timeout.tv_sec = 5;
timeout.tv_usec = 0;
setsockopt(socket_fd, SOL_SOCKET, SO_RCVTIMEO, (const char*)&timeout, sizeof timeout);
```

6. **地址复用**：
+ 假设你每次打电话前都得换新号码，那得多麻烦？通过设置 `SO_REUSEADDR`，你可以在服务器重启后继续使用相同的端口，而不必等待旧连接释放资源。

```c

int opt = 1;
setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```

####  五、socket 编程套接字地址结构和常用接口

在进行 socket 编程时，有一些常用的 API 和工具是你必须熟练掌握的。这里我们来逐个了解这些关键工具，帮助你在编写网络程序时游刃有余。

#### 5.1 套接字地址结构及地址转换 API

先来看看常用的套接字地址结构：

```c++

struct sockaddr {
   sa_family_t     sa_family;      /* Address family */
   char            sa_data[];      /* Socket address */
};

// 套接字地址结构（适用于IPv4网络通信的地址结构）
struct sockaddr_in 
{
    sa_family_t    sin_family; # address family: AF_INET 
    in_port_t      sin_port;   # port in network byte order 
    struct in_addr sin_addr;   # ip address 
};

struct in_addr
{
     uint32_t       s_addr;    # address in network byte order 
};
```

+ `sockaddr` 是一个通用的套接字地址结构，它通常与特定的地址族结构（如 `sockaddr_in`）一起使用。大多数套接字函数，如 `bind()`、`connect()` 和 `accept()`，需要使用指向 `sockaddr` 结构的指针作为参数。
+ `sockaddr_in` 是适用于 IPv4 网络通信的地址结构，专门用于存储 IP 地址和端口信息。

**代码示例**：使用 `sockaddr` 和 `sockaddr_in` 进行服务器地址的绑定。

```c++

struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(12345); // 端口号转换为网络字节序
server_addr.sin_addr.s_addr = INADDR_ANY; // 绑定到本地所有可用地址

// 使用 sockaddr 进行绑定
if (bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
    perror("bind failed");
    close(server_fd);
    exit(EXIT_FAILURE);
}
```

在这个示例中，我们先创建并初始化了 `sockaddr_in` 结构体，然后将其强制转换为通用的 `sockaddr` 结构指针传递给 `bind()` 函数，这样就完成了地址绑定。

#### 5.2 字节序转换 API

在 socket 编程中，我们经常会遇到一个叫做字节序的问题。主机和网络之间的字节顺序可能不同，网络数据采用的是“大端字节序”，而机器有可能采用“小端字节序”，也有可能是“大端字节序”。为了保证数据在网络和主机设备之间传输时没有问题，我们需要在发送数据前做一些“字节序转换”。

**主角们登场！**

- `htons() 和 htonl()`：这两个函数用于将数据从主机字节序（通常是小端字节序）转换为网络字节序（大端字节序）。htons() 处理 16 位数据（如端口号），htonl() 处理 32 位数据（如 IP 地址）。

- `ntohs() 和 ntohl()`：它们的名字刚好是反过来的，作用也相反，是把网络字节序转换回主机字节序，确保接收到的数据能够正确理解。ntohs() 处理 16 位数据(端口)，ntohl(IP 地址) 处理 32 位数据。

**记忆小技巧**：
- htons：host to network short（主机字节序转换为网络字节序，处理短整型数据，16 位）。
- htonl：host to network long（主机字节序转换为网络字节序，处理长整型数据，32 位）。
- ntohs：network to host short（网络字节序转换为主机字节序，处理短整型数据，16 位）。
- ntohl：network to host long（网络字节序转换为主机字节序，处理长整型数据，32 位）。

> 小提示：short 和 long 用于区分 16 位 和 32 位 数据，这样可以帮助你记住哪个函数处理哪种数据类型。

说到这里可能有点绕，咱们看看代码就简单明了啦。

来看看在 C 中如何使用这些 API：
```c
#include <stdio.h>
#include <arpa/inet.h>

int main() {
    // 主机字节序的端口号
    uint16_t port = 12345;
    uint16_t net_port = htons(port); // 将主机字节序转换为网络字节序，net_port 就可以直接在网络上传输
    printf("网络字节序的端口号: %d\n", net_port);

    // 主机字节序的 IP 地址
    uint32_t ip = 0x7F000001; // 127.0.0.1 的 16 进制
    uint32_t net_ip = htonl(ip); // 将主机字节序转换为网络字节序, 这样 net_ip 就可以在网络上传输了
    printf("网络字节序的 IP 地址: %x\n", net_ip);

    // 接收数据时转换回主机字节序
    uint16_t received_port = ntohs(net_port); // 将网络字节序转换为主机字节序
    printf("主机字节序的端口号: %d\n", received_port);

    return 0;
}
```
代码解说:

- 这段代码展示了如何用 htons() 和 htonl() 来把主机字节序转换为网络字节序。比如，我们有个端口号 12345，直接拿去网络上传的话可能会变成乱七八糟的数字，因为各个机器对字节顺序的理解不一样，所以得先用 htons() 把它变成大家都能理解的格式。

- 类似地，对于 IP 地址 127.0.0.1（0x7F000001），我们使用 htonl() 来确保在网络上传输时不会因为字节顺序不同而出错。

- 当数据从网络传回来时，我们需要用 ntohs() 和 ntohl() 把它们转换回主机的字节序，就像我们用 ntohs() 把接收到的端口号转回去，这样我们才能正确地理解这些数据。

#### 5.3 地址转换 API

除了字节序转换，网络编程中还有一个常见的问题是 IP 地址的表示方式。我们经常需要在字符串形式（比如 "192.168.1.1"）和二进制形式之间来回转换。这时候，就有两位“好帮手”可以用：

- inet_pton()：inet_pton() 是把人类看得懂的 IP 地址字符串(如 "192.168.1.1")，翻译成计算机能理解的二进制格式，用于网络通信。

- inet_ntop()：这个函数则是做反向操作，把二进制格式转换为字符串形式，方便我们查看。

来看看地址转换代码怎么写：

```c
#include <iostream>
#include <arpa/inet.h>

int main() {
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(12345); // 将端口号转换为网络字节序

    // 将字符串形式的 IP 地址转换为二进制格式
    inet_pton(AF_INET, "192.168.1.1", &server_addr.sin_addr);
    printf("二进制格式的 IP 地址已设置\n");

    // 将二进制格式的 IP 地址转换为字符串形式
    char ip_str[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &server_addr.sin_addr, ip_str, INET_ADDRSTRLEN);
    printf("IP 地址: %s\n", ip_str); // 输出: 192.168.1.1

    return 0;
}
```
代码解说

- 在这个例子里，我们使用了 inet_pton() 来把字符串形式的 IP 地址 "192.168.1.1" 转换为 sockaddr_in 结构体中的二进制格式。这样做的原因是，计算机更喜欢处理二进制数据，而不是人类可读的字符串。
- 然后，我们用 inet_ntop() 把这个二进制格式的 IP 地址转换回字符串形式，方便输出和查看。这两个函数就像 IP 地址的“翻译官”，帮助我们在不同的表示方式之间来回切换。

另外，inet_addr() 和 inet_aton() 也用于将字符串形式的 IPv4 地址转换为数值格式，不过它们较旧，功能较少，且只支持 IPv4。inet_pton() 和 inet_ntop() 是现代网络编程中推荐使用的函数，因为它们支持 IPv4 和 IPv6，并且更安全。

**小结**：通过这些 API，我们可以轻松地在主机字节序和网络字节序之间进行转换，也可以在字符串和二进制的 IP 地址之间切换。这些函数就像是数据的“翻译官”，让我们在网络编程中处理数据时更加方便、安全，也更不容易出错。

### 六、完整 socket 服务器客户端示例(基于 Linux C 实现的单进程版本)

下面提供一个完整的 socket 服务器程序和客户端程序，基于 Linux C 实现。

**1. 完整的服务器程序：**

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 12345

int main() {
    int server_fd, client_fd;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    char buffer[1024] = {0};

    // 创建 socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == -1) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 绑定地址和端口
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    if (bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 监听连接
    if (listen(server_fd, 5) == -1) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("服务器正在等待连接...
");

    while (1) {
        // 接收客户端连接
        client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_addr_len);
        if (client_fd == -1) {
            perror("accept failed");
            continue;
        }

        // 接收数据
        read(client_fd, buffer, sizeof(buffer));
        printf("收到数据: %s
", buffer);

        // 发送数据
        char *message = "Hello from server!";
        write(client_fd, message, strlen(message));

        // 关闭客户端连接
        close(client_fd);
    }

    // 关闭服务器连接
    close(server_fd);

    return 0;
}
```

**2. 完整的客户端程序：**

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 12345

int main() {
    int client_fd;
    struct sockaddr_in server_addr;
    char buffer[1024] = {0};

    // 创建 socket
    client_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_fd == -1) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 设置服务器地址
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    server_addr.sin_addr.s_addr = inet_addr("192.168.1.1"); // 服务器地址

    // 连接服务器
    if (connect(client_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("connect failed");
        close(client_fd);
        exit(EXIT_FAILURE);
    }

    // 循环发送和接收数据
    while (1) {
        char message[1024];
        printf("请输入要发送的消息 (输入"exit"退出): ");
        fgets(message, sizeof(message), stdin);
        message[strcspn(message, "\n")] = 0; // 移除换行符

        // 判断是否退出
        if (strcmp(message, "exit") == 0) {
            break;
        }

        // 发送数据
        write(client_fd, message, strlen(message));

        // 接收数据
        memset(buffer, 0, sizeof(buffer));
        read(client_fd, buffer, sizeof(buffer));
        printf("收到数据: %s\n", buffer);
    }

    // 关闭连接
    close(client_fd);

    return 0;
}

```

你可能注意到了，上面的 socket 服务器是一个单进程版本，无法同时处理多个客户端连接，只能一个个地处理，效率不高。这种方式虽然简单，但一旦有很多客户端同时连接，服务器就忙不过来了。

### 七、结语：

Linux socket 编程其实不复杂，只要掌握了基本概念和流程，写出一个能运行的 socket 程序其实很容易。不过，单进程处理多个客户端就像排队买票，一个人一个人来，特别慢。这时候，多进程模型就派上用场了，让服务器可以同时处理更多用户，不再让大家等得着急。下一篇文章我们就聊聊怎么用多进程模型来提升服务器能力，让它变得更快更强！

## 最后：

如果你觉得这篇文章对你有帮助，记得给我点个在看和赞 👍，并分享给有需要的小伙伴吧！也欢迎大家来关注我公众号 「跟着小康学编程」，微信搜索 **跟着小康学编程** 即可关注！


## 关注我能学到什么？

- 这里分享 Linux C、C++、Go 开发、计算机基础知识 和 编程面试干货等，内容深入浅出，让技术学习变得轻松有趣。

- 无论您是备战面试，还是想提升编程技能，这里都致力于提供实用、有趣、有深度的技术分享。快来关注，让我们一起成长！

## 怎么关注我的公众号？

**非常简单！扫描下方二维码即可一键关注。**

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

另外我还建了个**技术交流群**，里面都是真正在写代码的同行，不聊虚的，只聊技术。有问题大家一起讨论，比一个人闷头学效率高多了。
  
![](https://files.mdnice.com/user/48364/4ebc72e9-e4bb-447a-9a92-8367a178df6d.png)

记住，技术这条路，一个人走容易迷路，一群人走才能走得更远。

