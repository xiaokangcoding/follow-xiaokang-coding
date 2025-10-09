## 前言：编译命令敲累了？是时候学点自动化了！

哈喽大家好，我是小康！  

上次我们聊了 GCC、G++ 和 GDB，搞清楚了这些工具在 C/C++ 开发中的用法。有了它们，你已经可以愉快地编译、调试代码了。

但我问你，每次编译项目是不是手动敲 `gcc` 命令？ 当项目文件一多，命令就像绕口令一样——又长又复杂，还特别容易出错。    
别怕，今天我就带你认识一个“懒人神器”——**Makefile**。

用 Makefile 的好处很简单：

+ 代码编译自动化，轻松又高效；
+ 不用手动敲命令，少掉坑；
+ 项目多大都不怕，它全能搞定。

只要学会写 Makefile，编译这种枯燥的事情，再也不用你操心！让我们从零开始，一步步带你搞清楚它是啥、怎么写，看完就能用！

## 1、什么是 Makefile？

Makefile 就是一个**编译指挥官**，你把编译规则写在里面，之后用一条简单的命令 `make`，它就会按照规则自动完成所有的编译任务。

打个比方，你是项目经理，Makefile 就是你的笔记本，记录着项目的“施工计划”：

+ 每个目标（比如可执行文件 `main`）的来源（哪些源文件）；
+ 这些目标要用什么命令生成；
+ 有哪些需要重复利用的部分（比如中间文件 `*.o`）。

一句话：Makefile 帮你自动化处理那些又多又烦的编译流程！

## 2、为什么用 Makefile？

假设你有两个源文件：`main.c` 和 `utils.c`，手动编译步骤大概是这样：

1. 先把 `main.c` 和 `utils.c` 分别编译成目标文件：

```bash
gcc -c main.c -o main.o
gcc -c utils.c -o utils.o
```

2. 再把目标文件链接成可执行文件：

```bash
gcc main.o utils.o -o main
```

看着简单，但代码一多，命令就会变成这样：

```bash
gcc -c file1.c -o file1.o
gcc -c file2.c -o file2.o
gcc -c file3.c -o file3.o
...
gcc file1.o file2.o file3.o -o my_program
```

多打一条命令，多一个机会掉坑；一改代码，又得全编译一遍，时间都浪费了。  
用 Makefile，只需要：

```bash
make
```

一条命令，全搞定！而且它还会**只编译改动的文件**，效率直接起飞。

简单理解：

+ 没有 Makefile：自己手敲命令，累。
+ 有了 Makefile：只用一句 `make`，剩下的事全自动完成，爽。

## 3、Makefile 的基本结构 (一分钟搞懂）  

Makefile 是由一组 **规则**（rule）组成的，每个规则都包含三部分：

1. **目标**（target）：你想要生成的文件，比如 `main`。
2. **依赖**（dependencies）：目标文件需要哪些源文件或头文件。
3. **命令**（commands）：生成目标需要运行的命令。

举个例子：

```makefile
main: main.o utils.o
	gcc main.o utils.o -o main
  
main.o: main.c
    gcc -c main.c

utils.o: utils.c
    gcc -c utils.c
```

什么意思呢？

上面总共 3 条规则，来说说第一条规则：

1. 目标是 `main`，表示我们要生成一个叫 `main` 的可执行文件。
2. 依赖是 `main.o` 和 `utils.o`，也就是说生成 `main` 需要这两个依赖文件先生成，而这两个依赖是利用规则 2 和 规则 3 生成的。
3. 命令是 `gcc main.o utils.o -o main`，它负责把 `.o` 文件编译成最终的可执行文件。

后两条规则类似，告诉 `make` 怎么生成 `main.o` 和 `utils.o`。  

简单吗？这就相当于告诉 Makefile：“你要先准备好 `main.o` 和 `utils.o`，然后用 gcc 链接它们。”

## 4、Makefile 基础功能：让编译自动化从这里开始  

### 4.1 自动生成目标文件

如果你每次都向上面一样手写 `main.o`、`utils.o` 的生成规则，那 Makefile 就会变得非常繁琐和重复。好消息是，Makefile 支持通配符，可以自动生成规则！

```makefile
%.o: %.c
	gcc -c $< -o $@
```

这段代码怎么用？假设你有 `main.c` 和 `utils.c`，Makefile 会自动生成对应的规则：

+ `main.o`：由 `main.c` 生成，命令是 `gcc -c main.c -o main.o`；
+ `utils.o`：由 `utils.c` 生成，命令是 `gcc -c utils.c -o utils.o`。

**解释一下符号：**

+ `%.o` 和 `%.c`：`%` 是通配符，表示文件名匹配，比如 `main.o` 和 `main.c`。
+ `$<`：依赖文件，比如 `main.c`。
+ `$@`：目标文件，比如 `main.o`。

用这个规则，Makefile 直接帮你生成所有目标文件，舒服吧？

而当项目文件越多，使用 Makefile 的优势就越大。

### 4.2 增量编译：只编译改动的文件
Makefile 有个超棒的功能：**只编译需要更新的文件**。  
它会检查每个目标的依赖文件，如果依赖文件没有变化，就跳过编译。

比如你改了 `main.c`，Makefile 只会重新生成 `main.o`，而 `utils.o` 完全不动。  
这个功能在项目文件很多的时候，能节省一大堆时间。

### 4.3 清理临时文件
编译后，会留下很多 `.o` 文件和中间文件。Makefile 可以加一个 `clean` 规则，帮你一键清理：

```makefile
clean:
	rm -f *.o main
```

直接运行 `make clean`，干净清爽！

## 5、Makefile 的进阶玩法

了解了基本用法后，咱们来看一些能提升开发效率的进阶功能。 

### 5.1 基础玩法 - 提高可读性和可维护性  

#### 1. 使用变量：让 Makefile 更加简洁 
 
变量怎么使用？比如 `CC = gcc`。变量可以让 Makefile 更加灵活和易维护。

**变量的基本用法：**

```makefile
# 定义变量
CC = gcc
CFLAGS = -Wall -g
TARGET = main
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o) 

# $(SRCS:.c=.o) 是 Makefile 中的一种变量替换，它的作用是把变量 SRCS 中的每个 .c 文件名换成对应的 .o 文件名。
# 替换之后 OBJS = main.o utils.o
```

在命令中使用变量时，需要用 `$()` 的形式引用：

```makefile
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $(OBJS) -o $(TARGET)
    
# 使用$()替换变量之后的规则如下:    
main: main.o utils.o
	gcc -Wall -g main.o utils.o -o main

```

这样，如果你要修改编译器或优化选项，只需要改动变量部分，而不需要手动修改每条规则。

#### 2、伪目标：让 Makefile 更灵活

在 Makefile 中，有些目标（比如 `clean`）不会生成文件，而是用来执行特定的命令，比如清理临时文件。这种目标我们称为 伪目标。

问题来了：如果目录中刚好有个文件名就叫 `clean`，运行 `make clean` 时，Makefile 会误以为这个文件已经存在，导致规则不执行。

**怎么解决？**  

用 `.PHONY` 声明伪目标，告诉 `make` 这个目标不是文件，应该直接执行命令。

示例：声明伪目标

```makefile
.PHONY: clean 
clean:
	rm -f $(OBJS) $(TARGET)
```

这样，即使目录中有个名为 `clean` 的文件，`make clean` 仍然会按规则执行，删除目标文件和中间文件。


**记住：凡是不生成文件的目标，都建议用 `.PHONY` 声明！**

### 5.2  进阶玩法 - 构建更强大的 Makefile  

#### 模式规则：适配更多文件类型	

有时候我们的项目里，不只有 `.c` 文件，还有 `.cpp` 文件。如果要分别写规则，那就太麻烦了！这时候，**模式规则** 就能帮上大忙。

什么是模式规则？

模式规则就是一种通用规则，用来告诉 Makefile：  
**“遇到这种类型的文件，该怎么处理。”**  
比如，告诉 Makefile：

+ `.c` 文件用 `gcc` 编译；
+ `.cpp` 文件用 `g++` 编译。

这样，Makefile 会根据文件后缀自动选择正确的规则，不用你手动一个一个写。

怎么用？支持 C++ 文件

假设项目里有 `.cpp` 文件，我们可以加一个模式规则：

```makefile
%.o: %.cpp
	g++ -c $< -o $@
```
这样，Makefile 会自动把所有 `.cpp` 文件编译成 `.o` 文件，完全不用你操心。

如何同时支持 C 和 C++ 文件？

如果项目里既有 `.c` 文件，也有 `.cpp` 文件，那我们可以写两条规则：

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

%.o: %.cpp
	g++ -c $< -o $@
```

**这两条规则的作用：**

1. **第一条**：告诉 Makefile，`.c` 文件用 `gcc` 编译。
2. **第二条**：告诉 Makefile，`.cpp` 文件用 `g++` 编译。

这样，不管你的文件是 `.c` 还是 `.cpp`，Makefile 都会自动搞定。

**总结一下：**

+ 它会根据文件类型，自动选择合适的编译方式；
+ 你只需要写一条规则，Makefile 就能帮你搞定一大堆文件；
+ 再也不用重复写规则了，省事又高效！

**记住：文件后缀不同？用模式规则全搞定！**

#### 条件语句：让 Makefile 更聪明

条件语句可以让 Makefile 根据实际情况调整规则，比如不同的操作系统、不同的编译模式，用起来既灵活又省心。

**2.1 适配不同平台**

不同操作系统的命令可能不一样，比如删除文件，Linux 用 `rm`，Windows 用 `del`。通过条件语句，Makefile 可以自动选择正确的命令：

```makefile
OS = $(shell uname)

ifeq ($(OS), Linux)
	CLEAN_CMD = rm -f
else
	CLEAN_CMD = del
endif

clean:
	$(CLEAN_CMD) *.o $(TARGET)
```

+ 在 Linux 上运行 `make clean`：执行 `rm -f`；
+ 在 Windows 上运行 `make clean`：执行 `del`。

这样，无论在哪个平台都不用手动改命令了，省事！

**2.2 切换编译模式**

开发过程中经常需要在调试模式（debug）和发布模式（release）之间切换：

+ **调试模式**：包含调试信息（方便排查问题）。
+ **发布模式**：优化性能（适合生产环境）。

用条件语句很容易实现：

```makefile
ifeq ($(MODE), debug)
	CFLAGS = -g -O0
else
	CFLAGS = -O2
endif

all:
	$(CC) $(CFLAGS) -o $(TARGET) $(SRCS)
```

+ **运行调试模式：**

```bash
make MODE=debug
```

用 `-g` 和 `-O0` 编译，生成带调试信息的程序。

+ **运行发布模式：**

```bash
make
```
或：
```bash
make MODE=release
```
用 `-O2` 编译，生成优化后的高性能程序。

**小结一下**：

+ **适配不同平台：** 条件语句让 Makefile 在 Linux 和 Windows 上都能用。
+ **切换编译模式：** 方便开发阶段调试和生产环境优化。

#### 3. 自动化依赖管理： 让 Makefile 更聪明！  

在写代码时，`.c` 文件往往会用到头文件 `.h`。比如，你的 `main.c` 里可能有一句：

```c
#include "utils.h"
```

如果有一天你修改了 `utils.h`，Makefile 怎么知道它需要重新编译 `main.c` 呢？  

靠你手动写依赖规则？别开玩笑了，项目文件一多，光靠手动写依赖会把人累趴。

这时候，**自动化依赖管理**就派上用场了。

什么是自动化依赖管理？

- 自动化依赖管理的核心是用 `gcc -M` 命令，它能帮你自动生成 `.c` 文件和 `.h` 文件的依赖关系。每次你修改头文件时，Makefile 会自动触发相关的 `.c` 文件重新编译。

代码怎么写？

看下面这个 Makefile 示例：

```makefile
# 定义依赖文件列表
DEPS = $(SRCS:.c=.d)

# 生成 .d 文件，写入依赖规则
%.d: %.c
	$(CC) -M $< > $@

# 包含依赖文件
include $(DEPS)
```

它到底做了什么？

1. 定义依赖文件

```makefile
DEPS = $(SRCS:.c=.d)
```

把源文件列表 `SRCS` 中的每个 `.c` 文件，替换成对应的 `.d` 文件，比如：

+ `main.c` → `main.d`
+ `utils.c` → `utils.d`

这些 `.d` 文件就是用来记录 `.c` 和 `.h` 之间的关系。

2. 自动生成依赖规则

```makefile
%.d: %.c
	$(CC) -M $< > $@
```

这条规则会用 `gcc -M` 为每个 `.c` 文件生成一个 `.d` 文件，里面记录了它依赖哪些头文件。

比如，如果你的 `main.c` 包含了 `utils.h`，生成的 `main.d` 文件可能是这样的：

```makefile
main.o: main.c utils.h
```

3. 包含依赖规则

```makefile
include $(DEPS)
```

这句话告诉 Makefile，把所有 `.d` 文件里的内容加载进来。每次运行 Makefile 时，它都会检查 `.d` 文件里的规则，看哪些文件需要重新编译。

效果如何？

假设你有以下文件：

+ `main.c` 依赖 `utils.h`；
+ `utils.c` 不依赖任何头文件。

如果你修改了 `utils.h`，Makefile 会自动发现这个改动，然后只重新编译 `main.c`，而不会动 `utils.c`。

**自动化依赖管理的好处：**

1. 再也不用手动写依赖规则，让 Makefile 更智能；
2. 每次头文件更新时，Makefile 自动判断哪些文件需要重新编译；
3. 即使项目文件多到爆，也能轻松应对。

**简单记住：“有 `.h` 文件，就用 `gcc -M` 自动生成依赖！”**

#### 4. 多目标支持：用 Makefile 管理多个模块

当你的项目文件越来越多，甚至分成了多个模块，比如 `lib` 是核心功能模块，`app` 是主程序模块，光靠一个 Makefile 已经很难搞定了。  

这时候，聪明的做法是：

+ 每个模块有自己的 Makefile，单独管理自己的规则；
+ 用一个主 Makefile 调度所有模块，让项目更清晰、更高效！

分模块的做法：

**1. 给每个模块单独写一个 Makefile**

比如，在 `lib` 模块的目录下，我们写一个 `lib/Makefile`：

```makefile
# lib/Makefile
lib.a: lib.o               # 定义目标 lib.a
	ar rcs lib.a lib.o       # 把 lib.o 打包成静态库 lib.a

lib.o: lib.c               # 编译规则：生成 lib.o
	gcc -c lib.c -o lib.o
```

+ lib.a 是静态库，`ar rcs` 是打包命令。
+ 这个 Makefile 只关心 `lib` 模块自己的文件，不影响其他模块。

**2. 用主 Makefile 调度所有模块**

主 Makefile 位于项目根目录，负责把所有模块串起来。它并不关心每个模块的具体规则，而是递归调用每个模块自己的 Makefile：

```makefile
# 主 Makefile
SUBDIRS = lib app          # 定义模块目录

all: $(SUBDIRS)            # 主目标：编译所有模块

$(SUBDIRS):                # 递归调用每个模块的 Makefile
	$(MAKE) -C $@

clean:                     # 清理所有模块
	for dir in $(SUBDIRS); do $(MAKE) -C $$dir clean; done
```

代码解释：

1. `SUBDIRS` 是模块列表： 

这里定义了项目中的模块，比如 `lib` 和 `app`。每个模块目录都有自己的 Makefile。

2. `$(MAKE) -C $@`  是关键： 

这条命令的意思是切换到指定目录（`-C`），然后运行这个目录里的 Makefile。  
比如，`$(MAKE) -C lib` 就是到 `lib` 目录运行它的 Makefile。  

3. 递归清理: 

`clean` 目标会循环进入每个模块目录，调用它们的 `clean` 规则。注意 `$$dir` 中的双 `$`，是为了让 Makefile 能正确解析。  

**整体效果**:

+ 你可以在主目录运行 `make`，它会自动编译所有模块；
+ 运行 `make clean` 时，它会递归清理所有模块的临时文件；
+ 每个模块的规则独立，清晰又方便维护。

用主 Makefile 调度多个模块的好处：

1. **结构清晰**：每个模块的规则独立管理，主 Makefile 只负责调度。
2. **易于维护**：修改或新增模块时，只需在 `SUBDIRS` 添加对应模块目录即可。
3. **高效递归**：通过 `$(MAKE) -C` 调用子目录的 Makefile，模块间互不干扰。

**简单来说：分模块管理，用主 Makefile 调度，一切井井有条！**

### 5.3 高阶玩法 - 优化效率与灵活性  

#### 1. 并行编译：提高效率

Makefile 的 `make` 命令支持并行执行多个规则，用 `-j` 参数指定并行任务数。

**示例：并行编译**

```bash
make -j4
```

这会同时运行最多 4 个任务，充分利用多核 CPU，显著提升大项目的编译速度。

#### 2. 自定义函数：复用逻辑

在写 Makefile 时，如果规则中有重复的编译逻辑，比如把 `.c` 文件编译成 `.o` 文件，一直重复写 `$(CC) $(CFLAGS)` 就很麻烦。这时候，我们可以用自定义函数来统一管理这些重复操作，既方便又省事！

定义函数：

用 `define` 和 `endef` 定义一个编译函数：

```makefile
define compile
	$(CC) $(CFLAGS) -c $< -o $@
endef
```

+ `compile` 是函数名，表示编译的逻辑；
+ `$<` 是依赖文件（比如 `main.c`），`$@` 是目标文件（比如 `main.o`）。

使用函数:

调用自定义函数时，用 `$(call 函数名)`：

```makefile
%.o: %.c
	$(call compile)
```

这条规则会自动把 `.c` 文件编译成对应的 `.o` 文件。

小结一下：

1. 自定义函数减少了重复代码；
2. 修改逻辑时，只需改函数定义，其他地方不用动；
3. 让 Makefile 简洁易读，清晰高效。

**一句话：把重复的逻辑封装成函数，Makefile 也能优雅起来！**

#### 3. 静态模式规则：批量生成目标文件

当多个文件需要用相似的规则编译时，一个个写太麻烦，用 **静态模式规则** 就能一次性搞定！

先来看个简单示例：
假设我们要把多个 `.c` 文件编译成 `.o` 文件：

```makefile
OBJS = main.o utils.o io.o

$(OBJS): %.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

**这是什么意思？**

+ `$(OBJS)` 是目标文件列表，比如 `main.o`、`utils.o`；
+ `%.o: %.c` 说明每个 `.o` 文件由对应的 `.c` 文件生成；
+ `$<` 是源文件（如 `main.c`），`$@` 是目标文件（如 `main.o`）。

优点：

1. **减少重复**：一条规则批量处理，省时省力；
2. **自动匹配**：文件名自动对应，无需手动写每条规则。

这里顺便提下通配模式规则，这两种模式用法很相似。

对于更简单的项目，你可以用通配模式规则来实现类似效果：

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

**解释一下：**

+ 每个 `.o` 文件由对应的 `.c` 文件生成；
+ 通配符 `%` 会匹配任意文件名，比如 `main.c` 自动对应 `main.o`。

**静态模式规则 vs. 通配模式规则**

| 特性 | 静态模式规则 | 通配模式规则 |
| --- | --- | --- |
| 匹配范围 | 针对特定目标列表（如 `$(OBJS)`<br/>） | 自动匹配所有符合`%`<br/> 的文件 |
| 灵活性 | 控制更精确，只处理指定的目标文件| 简单统一，适合全局规则 |
| 适用场景 | 文件多、规则复杂，特定文件需要特殊处理| 文件少、规则统一，简单项目 |


**总结：**

+ 通配模式规则适合简单项目，一条规则处理所有文件；
+ 静态模式规则适合复杂项目，可以精确控制哪些文件应用规则。

**记住：简单全局用通配，精准处理选静态！**

#### 4. 跨平台构建：用 CMake 生成 Makefile

如果项目需要在多个平台（如 Windows、Linux、macOS）上编译，直接写 Makefile 会很麻烦。这时，可以用 CMake 自动生成适配不同平台的 Makefile。

使用方法：

**1. 创建 CMake 配置文件**  

在项目目录下新建 `CMakeLists.txt`，内容如下：

```makefile
# 声明最低版本要求
cmake_minimum_required(VERSION 3.10)

# 定义项目名称
project(MyProject)

# 指定可执行文件
add_executable(main main.c utils.c)
```

**2. 生成 Makefile**  
在终端运行：
```bash
cmake .
```

3.**编译项目**  
使用生成的 Makefile：

```bash
make
```

优点:

+ **跨平台**：适配 Windows、Linux、macOS 等操作系统；
+ **简化管理**：无需手写复杂的 Makefile。

**一句话总结：用 CMake 自动生成 Makefile，跨平台编译就是这么简单！**

## 五、完整示例

结合前面学到的内容，来看看一个完整的 Makefile：

```makefile
# 定义变量
CC = gcc                     # 编译器
CFLAGS = -Wall -g            # 编译参数：开启所有警告和调试信息
SRCS =  $(wildcard *.c)      # 获取当前目录下所有的 .c 文件，并赋值给 SRCS 变量，例如：SRCS = main.c utils.c 
OBJS = $(SRCS:.c=.o)         # 把 .c 文件替换成 .o 文件，替换之后，OBJS = main.o utils.o 
TARGET = main                # 最终生成的可执行文件

# 编译规则
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) $(OBJS) -o $(TARGET)

# 生成 .o 文件规则
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# 加上 .PHONY 声明伪目标
.PHONY: clean

clean:
	rm -f $(OBJS) $(TARGET)

```

使用：

+ `make`：生成 可执行文件`main`；
+ `make clean`：会删除所有 `.o` 文件和可执行文件 `main`，保持项目目录干净。

## 六、写在最后：从 Makefile 开始，走向编译自动化！

Makefile 就是编译中的“懒人神器”，一旦用上，你会发现：

+ 不再手动敲命令，编译变得更简单；
+ 即使项目越来越大，管理起来也毫不费力；
+ 提高效率，节省时间，轻松搞定复杂编译！

如果你还在手动敲命令，赶紧试试写个 Makefile，体验一下自动化的快乐吧～  

下篇文章我们将带你进入 CMake 的世界，了解如何跨平台管理项目，敬请期待！

如果文章对你有帮助，记得点个在看和赞 👍 再走～  有什么疑问，评论区聊！也欢迎大家来关注我公众号 「**跟着小康学编程**」，这里会持续分享计算机编程硬核技术文章！

#### 怎么关注我的公众号？

非常简单！扫描下方二维码即可一键关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)

