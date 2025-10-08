大家好啊，我是小康。

今天咱们来聊一个看起来高深，实际上理解起来其实挺简单的话题—— ELF 文件。

不知道你有没有想过：我们敲下`./program`命令的那一刻，计算机是怎么把这个文件变成一个活蹦乱跳的进程的？这背后的"黑魔法"到底是什么？

没错，答案就是今天的主角：**ELF（Executable and Linkable Format）可执行与可链接格式**。你可以把它理解为 Linux 世界里程序的"灵魂容器"！

## 一、什么是 ELF 文件？给个痛快话！

简单来说，ELF 是 Linux 下的可执行文件格式，就像 Windows 下的 .exe 一样。但别被这个简单的解释骗了，ELF 可比 .exe 复杂得多，也强大得多！

ELF 文件可以是：

+ 可执行文件（比如你的`./program`）
+ 目标文件（编译后但还没链接的 .o 文件）
+ 共享库文件（就是 .so 文件，类似 Windows 下的 .dll）
+ 核心转储文件（程序崩溃时的那个core dump）

本质上，ELF 就是一个**容器**，里面装着代码、数据以及程序运行所需的各种信息，按照特定的格式组织起来。

## 二、初见 ELF：第一印象很重要

想知道一个文件是不是 ELF 格式的？超简单：

```bash
$ file /bin/ls
/bin/ls: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, ...
```

看到没？只要文件输出信息的开头是"ELF"，那它就是 ELF 格式的！

再来点儿硬核的，我们直接看一下 ELF 文件的前几个字节：

```bash
$ hexdump -C -n 16 /bin/ls
00000000  7f 45 4c 46 02 01 01 00  00 00 00 00 00 00 00 00  |.ELF............|
```

这里最开始的`7f 45 4c 46`就是 ELF 文件的"魔数"（Magic Number）。其中 45 4c 46 是 ASCII 码中的 "ELF" 三个字母，前面的 7f 是一个特殊字符。这四个字节就是 ELF 文件的"身份证"，操作系统首先会检查这四个字节，确认它是不是一个 ELF 文件。

## 三、ELF 文件的内部结构：化繁为简

很多教程一上来就给你画个复杂的结构图，看得人头晕眼花。咱们先别急，我用一个简单的类比来帮你理解：

把 ELF 文件想象成一本"程序说明书"，这本书有三部分组成：

1. **文件头**（ELF Header）：相当于书的封面和目录，告诉你这本书有什么内容，怎么看
2. **程序头表**（Program Header Table）：相当于给"阅读器"（操作系统）看的指南，告诉它怎么把这本书变成一个活的程序
3. **节区头表**（Section Header Table）：相当于给"编辑器"（链接器、调试器）看的指南，告诉它这本书的内部结构

然后，书的主体内容就是各种**节区**（Sections）或**段**（Segments），里面装着代码、数据等实际内容。

**直观一点，用图来表示就是**：

```plain
+------------------+
|    ELF Header    | <-- 文件开始处的标识信息和总体布局
+------------------+
|     程序头表      | <-- 告诉操作系统如何加载
| Program Header 1 |
| Program Header 2 |
|       ...        |
+------------------+
|    Section 1     | <-- 实际内容，如代码、数据等
|    Section 2     |
|       ...        |
+------------------+
|    节区头表       |  <-- 描述每个Section的信息
| Section Header 1 | 
| Section Header 2 |
|       ...        |
+------------------+
```

哎，你可能会问：什么是节区（Section）？什么又是段（Segment）？它们有什么区别？

简单来说：

+ **节区（Section）**：是 ELF 文件存储的基本单位，针对链接器
+ **段（Segment）**：是运行时内存的基本单位，针对加载器

一个段通常包含多个功能相似的节区。比如，包含代码的所有节区会被归入到一个叫做"TEXT"的段中。

## 四、深入解剖 ELF文件：逐层剥开

### 1. ELF头（ELF Header）

ELF 头是整个文件的"门面"，包含了文件的基本信息和指向其他部分的指针。用`readelf -h`命令可以查看：

```bash
$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x5850
  Start of program headers:          64 (bytes into file)
  Start of section headers:          136912 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
```

这里面最重要的信息是：

+ **Entry point address**：程序执行的起点地址
+ **Start of program headers**：程序头表的位置
+ **Start of section headers**：节区头表的位置

### 2. 程序头表（Program Header Table）

程序头表告诉操作系统如何创建进程映像，用`readelf -l`命令查看：

```bash
$ readelf -l /bin/ls

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000004428 0x0000000000004428  R      0x1000
  LOAD           0x0000000000005000 0x0000000000005000 0x0000000000005000
                 0x0000000000012d1c 0x0000000000012d1c  R E    0x1000
  LOAD           0x0000000000018000 0x0000000000018000 0x0000000000018000
                 0x0000000000004d40 0x0000000000004d40  R      0x1000
  LOAD           0x000000000001d520 0x000000000001e520 0x000000000001e520
                 0x0000000000001640 0x0000000000002270  RW     0x1000
...
```

最重要的是那些类型为`LOAD`的段，它们会被加载到内存中。

注意看`Flags`：

+ `R`表示可读（Read）
+ `W`表示可写（Write）
+ `E`表示可执行（Execute）

这就是为什么有的内存区域可执行，有的只能读不能写，这些权限在 ELF 文件里就定义好了！

### 3. 节区头表（Section Header Table）

节区头表描述了文件中各个节区的信息，用`readelf -S`查看：

```bash
$ readelf -S /bin/ls
There are 30 section headers, starting at offset 0x21730:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.build-i NOTE             0000000000000338  00000338
       0000000000000024  0000000000000000   A       0     0     4
...
  [11] .text             PROGBITS         00000000000052a0  000052a0
       0000000000012a7c  0000000000000000  AX       0     0     16
...
  [23] .data             PROGBITS         000000000001e520  0001d520
       0000000000000e60  0000000000000000  WA       0     0     32
  [24] .bss              NOBITS           000000000001f380  0001e380
       0000000000001410  0000000000000000  WA       0     0     32
...
```

常见的重要节区包括：

+ **.text**：存放程序的机器代码
+ **.data**：已初始化的全局变量和静态变量
+ **.bss**：未初始化的全局变量和静态变量（不占用文件空间）
+ **.rodata**：只读数据（如字符串常量）
+ **.symtab**：符号表，存储程序中定义和引用的函数、变量
+ **.strtab**：字符串表，通常存储符号名
+ **.dynamic**：动态链接信息

## 五、ELF 文件的生命周期：从编译到执行

为了彻底搞懂 ELF 文件，我们需要了解它的整个生命周期：

```plain
源代码(.c) --编译--> 目标文件(.o) --链接--> 可执行文件 --加载--> 进程
```

### 1. 编译阶段：生成目标文件(.o)

当你写完 C 代码，运行`gcc -c hello.c`时，会得到一个`hello.o`的目标文件。这个文件已经是 ELF 格式的了，但它还不能直接执行，因为里面有很多"坑"等着被填上。

这些"坑"在 ELF 文件中表现为"重定位表"，用`readelf -r`可以看到：

```bash
$ readelf -r hello.o

Relocation section '.rela.text' at offset 0x2d0 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000013  000a00000004 R_X86_64_PLT32    0000000000000000 printf - 4
000000000023  000b00000004 R_X86_64_PLT32    0000000000000000 exit - 4
```

这表示代码中调用了`printf`和`exit`函数，但编译器不知道它们在哪儿，所以留了个"坑"等着链接器来填。

### 2. 符号表：程序的"通讯录"

说到这些函数(printf 、exit)，咱们不得不提 ELF 文件中的"符号表"。简单来说，符号表就像是程序的"通讯录"，记录了程序中所有函数和变量的名字和位置。

来看看符号表长啥样：

```bash
$ readelf -s hello.o

Symbol table '.symtab' contains 12 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
    ...
     9: 0000000000000000    41 FUNC    GLOBAL DEFAULT    1 main
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf
    11: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND exit
```

瞧，这里面有`main`函数（我们自己定义的），还有`printf`和`exit`（外部函数）。注意它们的`Ndx`（索引）列：`main`是1，表示在第1个节区；而`printf`和`exit`是`UND`，表示"未定义"，这就是前面说的"坑"。

这个目标文件的符号表就像一张"半成品通讯录"，只记录了自己有什么函数，以及自己需要哪些外部函数，但还不知道那些外部函数在哪里。所以它还不能独立工作，需要链接器来帮忙找到这些外部函数。

### 3. 动态链接：程序的"即插即用"

说到外部函数，就不得不提 ELF 的一个超强功能：动态链接。

还记得 Windows 上安装软件时经常冒出的"DLL缺失"错误吗？Linux 上也有类似概念，不过实现得更优雅，这就是动态链接库（.so文件）。

动态链接的好处简直不要太多：

+ 节省内存：多个程序共享同一个库
+ 节省磁盘：不用把所有代码都打包进可执行文件
+ 方便升级：库更新后，程序自动用上新版本，不用重新编译

那么问题来了：程序怎么知道自己需要哪些库？又是如何找到这些库的呢？

ELF 文件中有一个特殊的`.dynamic`节区，专门记录这些信息：

```bash
$ readelf -d /bin/ls | grep NEEDED
 0x0000000000000001 (NEEDED)             Shared library: [libselinux.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```

这告诉我们，`ls`命令依赖于这两个共享库。如果你想更直观地看到所有依赖及它们的实际位置，可以用`ldd`命令：

```bash
$ ldd /bin/ls
	linux-vdso.so.1 (0x00007ffc961cd000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f27f989e000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f27f96b3000)
	...
```

看到没？`ldd`不仅告诉你需要哪些库，还告诉你它们的实际位置和加载地址。

那程序又是怎么找到这些库的呢？它会按照以下顺序查找：

1. 环境变量`LD_LIBRARY_PATH`指定的目录
2. 可执行文件的`RPATH`属性指定的目录
3. `/etc/ld.so.cache`缓存中记录的位置
4. 默认目录如`/lib`、`/usr/lib`等

动态链接器（ld.so）会在程序启动时自动处理这些依赖关系，把所有需要的库都加载进来，就像乐高积木一样把程序拼装完整，非常巧妙！

### 4. 链接阶段：生成可执行文件

链接器会把多个目标文件和库文件链接在一起，解决那些"坑"（重定位），最终生成可执行文件。

那么链接器具体是怎么解决这些"坑"的呢？简单来说就是做个"牵线搭桥"的活：

1. 收集所有目标文件中的符号表，建立一个全局符号表
2. 找到所有标记为"未定义"（UND）的符号
3. 在全局符号表或者库文件中寻找这些符号的定义
4. 把找到的地址填回原来的"坑"中

比如当链接器找到`printf`函数在 libc.so 中的实际地址后，就会修改原来调用 printf 的指令，让它指向正确的地址。

链接完成后，再看同一个程序的符号表，会发现那些 UND 的符号要么有了实际地址（静态链接），要么指向了动态链接的跳转表（动态链接）。

在动态链接的情况下，还会在 ELF 文件中记录运行时需要哪些共享库，前面已经说过了。

### 5. 加载阶段：从文件到进程

当你执行`./program`时，操作系统（确切地说是加载器 ld.so ）会做这些事：

1. 检查 ELF 头的合法性
2. 根据程序头表，将需要的段加载到内存
3. 如果是动态链接的，还会找到并加载所需的共享库
4. 跳转到 `Entry Point` 开始执行

这个过程可以用`strace`命令观察：

```bash
$ strace ./hello
execve("./hello", ["./hello"], 0x7ffcef8db490 /* 52 vars */) = 0
brk(NULL)                               = 0x55c84f34c000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
...
```

`execve`就是创建新进程的系统调用，后面一系列操作就是在加载和准备程序运行环境。

## 六、ELF实用工具箱：玩转ELF文件

好了，了解了 ELF 的原理后，来看看有哪些工具可以帮我们操作 ELF 文件：

1、**file**：判断文件类型

```bash
$ file /bin/ls
```

2、**readelf**：查看ELF文件的所有信息

```bash
$ readelf -a /bin/ls  # 显示全部信息
```

3、**objdump**：反汇编 ELF 文件

```bash
$ objdump -d /bin/ls  # 反汇编代码段
```

4、**nm**：列出符号表

```bash
$ nm /bin/ls  # 显示符号（函数、变量）
```

5、**ldd**：查看动态依赖

```bash
$ ldd /bin/ls  # 显示依赖的共享库
```

6、**strings**：提取文件中的字符串

```bash
$ strings /bin/ls | grep "GNU"  # 查找包含"GNU"的字符串
```

7、**strip**：移除ELF文件中的符号表和调试信息

```bash
$ strip -s program  # 减小文件体积
```

8、**patchelf**：修改 ELF 文件的属性

```bash
$ patchelf --set-interpreter /lib64/ld-custom.so program  # 修改解释器
```

## 七、实际应用：ELF文件的那些神奇玩法

ELF文件的知识不仅仅是理论，来看看一些实际的例子：

### 1. 程序加固与混淆

想象你开发了一个软件不想被轻易破解：

```bash
# 删除符号表，让逆向分析更困难
$ strip --strip-all myprogram

# 对比前后大小
$ ls -lh myprogram*
-rwxr-xr-x 1 user user 236K myprogram
-rwxr-xr-x 1 user user 176K myprogram.stripped
```

看，文件体积一下减少了几十k，因为符号信息都被删掉了！

### 2. 程序补丁与热修复

假设你想修改程序使用的解释器路径：

```bash
# 查看当前解释器
$ readelf -l myprogram | grep interpreter
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]

# 修改为自定义解释器
$ patchelf --set-interpreter /opt/mylibs/ld-linux.so myprogram

# 确认修改成功
$ readelf -l myprogram | grep interpreter
      [Requesting program interpreter: /opt/mylibs/ld-linux.so]
```

这样程序就会使用你自定义的动态链接器，而不需要重新编译！

更酷的是，Linux 还提供了一种不用重启程序就能热修复的黑科技——`LD_PRELOAD`环境变量！它可以让你悄悄地"替换"程序中的函数实现。

来看一个简单实用的例子 —— 监控程序的内存分配：

**创建一个简单的内存跟踪库**: memtrace.c

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>

// 原始malloc函数指针
static void* (*real_malloc)(size_t) = NULL;

// 拦截 malloc 函数
void* malloc(size_t size) {
    // 延迟初始化原始函数
    if (real_malloc == NULL) {
        real_malloc = dlsym(RTLD_NEXT, "malloc");
    }
    
    // 调用原始malloc
    void* ptr = real_malloc(size);
    
    // 打印跟踪信息
    fprintf(stderr, "malloc(%zu) = %p\n", size, ptr);
    
    return ptr;
}

```
编译成共享库:
```bash
$ gcc -shared -fPIC memtrace.c -o libmemtrace.so -ldl
```
接着使用我们的库监控任何程序的内存分配：
```bash
LD_PRELOAD=./libmemtrace.so ./my_program
```
输出：
```bash
malloc(100) = 0x55e930e2f6b0
malloc(200) = 0x55e930e2f720
malloc(300) = 0x55e930e2f7f0
```

看到了吗？我们只用了十几行代码，就实现了一个能够监控任何程序内存分配的工具！这个例子的工作原理很简单：

1. 定义一个与系统函数同名的`malloc`
2. 用`dlsym(RTLD_NEXT, "malloc")`找到真正的malloc函数
3. 在调用真正的 malloc 前后添加我们的代码（这里是打印日志）
4. 通过`LD_PRELOAD`让系统优先加载我们的库

这种技术经常用于：

+ 调试内存问题
+ 给程序添加日志
+ 修改程序行为而不用改源码
+ 临时修复运行中的服务

当然，这项技术也常被黑客利用来劫持程序函数，所以理解它不仅能提升编程能力，也对安全防护很重要！

## 八、总结：ELF 文件的精髓

好了，咱们来总结一下 ELF 文件的核心要点：

1. **ELF是容器**：装载了代码、数据和各种元数据
2. **分层结构**：ELF 头、程序头表、节区、节区头表
3. **两种视角**： 
    - 执行视角：段（Segments）- 加载器关心
    - 链接视角：节（Sections）- 链接器关心
4. **生命周期**：从源代码到目标文件，再到可执行文件，最后变成进程

当你理解了 ELF 文件的本质，Linux 下的很多问题就迎刃而解了：为什么有些程序不能在不同版本的 Linux 上运行？为什么动态库版本不匹配会导致程序崩溃？为什么有些恶意软件难以检测？——这些问题的答案都藏在 ELF 文件的结构中！

记住，ELF 文件不仅仅是一个格式，它是 Linux 世界中程序的"灵魂容器"，承载着程序从编译到执行的整个生命周期。

---

"哇，原来这就是 ELF 文件的秘密！"如果这是你读完文章的感受，那我的目的就达到了。

我是小康，一个热衷于用故事解读技术的 Linux 老玩家。如果你喜欢这种深入浅出的技术解读，欢迎关注我的公众号「**跟着小康学编程**」，这里没有生硬的术语堆砌，只有接地气的技术剖析。

你学会了吗？欢迎在评论区分享你的心得！有什么 Linux 或底层技术问题？留言告诉我，你的困惑可能就是下一篇文章的主题。

动动手指点个赞、点个在看，让更多技术小伙伴一起告别晦涩难懂的技术文档！

## 探索不止，学习不停！

学完 ELF，你的 Linux 底层之旅才刚刚开始！

如果你想继续深入探索系统原理，提升技术内功，欢迎扫描下方二维码，关注「**跟着小康学编程**」公众号：

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

🔥 **为什么要关注我们？**

+ 每周推送易懂硬核技术文章，告别晦涩难懂
+ 独家整理的 Linux 底层知识地图，循序渐进提升
+ 定期分享面试高频考点，助你职场破局

👥 **想和更多技术爱好者交流？** 

扫描下方二维码加入技术交流群，遇到问题随时提问，大神在线解答！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)

学技术，我们是认真的。