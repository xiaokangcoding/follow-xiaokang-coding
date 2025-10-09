#### 前言：

程序员的世界，总绕不开两样东西：**一个是写代码，另一个是配环境。** 
尤其是在 Linux 下开发 C 和 C++，很多新人被各种工具链、依赖库和配置搞得头昏脑涨，仿佛一不小心就能把系统玩崩溃。但其实，搭建一个好用的开发环境并不难，只要方法对头，分分钟让你觉得 “编程，原来这么丝滑！”

今天，就跟着小康一步步把 Linux 的 C、C++ 环境搭建得妥妥的，学会之后，能让你写代码写到飞起！！

### Step 1: 装上基础设施，起步不迷路

#### 1.1 装系统：选择 Linux 发行版

搭建开发环境的第一步，得有块地啊！Linux 系统是主场：

1. **常见的 Linux 发行版**：Ubuntu（新手友好、 资料丰富），Debian（ 稳定可靠，适合进阶用户  ）。
2. **安装方式**：
- 如果是新手，直接在 Windows 上用 VMware 虚拟机 或 WSL（Windows Subsystem for Linux）搞起，简单又省事。
- 资深开发者：直接装双系统，甚至用云服务器（阿里云/腾讯云随便挑）。

**Tips：**

+ 初学者优先考虑 Ubuntu 或 WSL，简单不折腾，Ubuntu 建议安装 Desktop 版(桌面版)。
+ 硬盘空间建议分配 50GB 以上，避免后续安装工具时告急。
+ 内存建议分配至少 4GB，运行开发工具更流畅；8GB 以上更佳。
+ CPU 核心数至少设置为 4 核，保证编译和多任务处理不卡顿。
+ 网络适配器选择桥接模式，虚拟机可以直接访问宿主机网络，联网更方便。

> **PS**：没必要为环境花大钱，初学者一台普通电脑 + 免费虚拟机环境足够用。

因为小康的主力开发环境是 Ubuntu，所以接下来的步骤会基于 Ubuntu 系统，如果你选的也是 ubuntu，那跟着我操作准没错！

#### 1.2 安装必备的编译工具链  (GCC、G++ 和 Make 等基础工具包)
装上 C/C++ 编译器，Linux 下最常见的是 GCC：

```bash
sudo apt update  # 更新软件包列表，确保你的系统知道有哪些最新的软件包和更新。
sudo apt install build-essential  # 安装构建工具包，包含编译 C/C++ 程序所需的基础工具，如 gcc、g++、make 等。

```

`sudo apt install build-essential` 这条命令干了几件事：

+ 安装 GCC 和 G++ 编译器。
+ 配上 Make 工具，让你能用 `Makefile` 自动化编译。

安装完后，试试它们是否装好了：

```bash
gcc --version
g++ --version
make --version
```

如果版本号跳出来了，恭喜你，第一步搞定！

### Step 2: 配个顺手的代码编辑器(Vim、VS Code、CLion，三剑客)

#### 2.1 终端党：Vim（必会）

+ **Vim** 是 Linux 系统的标配编辑器，但刚上手可能有点懵圈。

装一下：

```bash
sudo apt install vim
```

初次使用可以先简单学几个快捷键，比如：

+ `i` 进入插入模式，可以正常写代码了。
+ `Esc` 退出插入模式。
+ `:wq` 保存并退出。

#### 2.2 GUI 党：VS Code (强烈建议)

如果你更习惯图形化界面，想要轻便又功能强大的编辑器，直接选 **VS Code**，上手简单，插件生态丰富，能满足绝大多数 C/C++ 开发需求。  

在 Ubuntu 上通过命令行安装也很简单：

- 下载 VS Code：

  直接去官网 https://code.visualstudio.com/，把 .deb 文件下载下来：

- 安装 VS Code：
  用系统自带的安装工具装上去：
```bash
sudo dpkg -i vscode.deb
```

- 修复依赖（如果有问题）：
  安装过程中如果提示有依赖没装，运行这行命令搞定：

```bash
sudo apt-get -f install
```

-  安装插件：打开 VS Code 左侧插件市场（小积木图标），搜索以下插件并安装：

    - C/C++：代码补全、语法检查必备。
    - Code Runner(可选)：一键运行代码(运行小型代码片段、测试代码逻辑时非常方便)
    - CMake Tools：如果项目用到 CMake，这个插件很方便。

装完后，直接在项目根目录输入 `code .` 就能启动 VS Code，界面清爽、功能强大，写 C/C++ 程序不要太舒服！

#### 2.3 高级党：CLion(可选) 

如果你追求更强大的功能和更智能的体验，可以试试 JetBrains 出品的 CLion。这是专为 C/C++ 开发打造的 IDE，优点一大堆：

- 自动管理 CMake 配置：再也不用手动折腾 CMakeLists.txt。
- 智能代码补全：效率高得离谱，写代码特别顺手。
- 内置调试器：支持 GDB 和 LLDB，调试功能强大，界面直观好用。
- 跨平台一致性：不管是 Windows、Linux，还是 macOS，都能保证一致流畅的开发体验。

**安装方式：**

1. 下载：直接去官网 https://www.jetbrains.com/clion/ 下载对应环境的版本即可(支持 windows 和 Linux )
2. 安装：根据提示一步步安装即可。
3. 费用: 学生或新手可以申请免费使用（JetBrains 提供学生版和试用期），不想付费可以网上自行百度破解版。

### Step 3: 让代码跑起来：编译与运行的正确姿势

#### 3.1 编译 & 运行

写了第一段 C 代码，保存成 `hello.c`，可以这么搞：

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

编译运行：

```bash
gcc hello.c -o hello
./hello
```

C++ 同理：

```c++
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, C++!" << endl;
    return 0;
}
```

编译运行：

```bash
g++ hello.cpp -o hello
./hello
```

#### 3.2 常见问题排查
+ **报错: `gcc: command not found`？**
说明 GCC 没装对，回到 Step 1.2 再试试。
+ **缺少 `stdio.h` 或 `iostream`？**  
可能是编译器路径出问题了，用 `sudo apt reinstall build-essential` 重新装一遍。


### Step 4: 配好调试工具，开发效率翻倍

#### 4.1 安装 GDB

调试工具 GDB 是 C/C++ 开发的必备利器：

```bash
sudo apt install gdb
```

简单调试流程：

```bash
g++ -g hello.cpp -o hello  # 编译的时候必须要加上 -g 参数，不然调试不了。
gdb ./hello
```

进入 gdb 后：

+ `run`：运行程序。
+ `break main`：在 `main()` 函数处打断点。
+ `next`：单步执行。
+ `print <变量名>`：打印变量值。

#### 4.2 配合 VS Code 调试

如果你用的是 VS Code，装个 C/C++ 调试插件后，直接配置 `launch.json` 文件，整个调试流程会更直观。

#### 4.3 排查内存问题：Valgrind

内存泄漏是 C/C++ 开发中常见的坑，**Valgrind** 是个强大的工具，可以帮你检查程序的内存问题。

**1、安装 Valgrind**：

```bash
sudo apt install valgrind -y
```

**2、使用方法：**假设你有一个编译好的程序 `your_program`，直接运行：

```bash
valgrind ./your_program
```

它会分析你的程序运行过程中分配和释放的内存。

**3、看结果：**

+ **正常情况**：  如果看到 `All heap blocks were freed`，恭喜，没有内存泄漏！
+ **有问题**：  如果显示 `definitely lost` 或 `indirectly lost`，说明有内存没有正确释放，需要检查代码，比如 `malloc` 和 `free` 是否匹配。

**一句话总结：** 装个 Valgrind，跑一遍程序，就能快速找到内存泄漏问题。简单高效，C/C++ 开发必备！

### Step 5: 开发效率提升：从配置到自动化

#### 5.1 编写 Makefile，解放双手

手动敲编译命令容易出错，Makefile 让一切变得自动化：

```Makefile
# 定义编译器，这里选择 GCC
CC = gcc  

# 定义编译选项：
# -Wall：打开所有警告
# -g：生成调试信息
CFLAGS = -Wall -g  

# 定义目标规则 'all'，默认目标是 main
all: main  

# 定义如何生成 main 可执行文件
# 使用目标 main.o 和 func.o 链接生成 main
main: main.o func.o  
	$(CC) $(CFLAGS) -o main main.o func.o  

# 定义生成 .o 文件的通用规则
# 这里 $< 表示匹配的 .c 文件
%.o: %.c  
	$(CC) $(CFLAGS) -c $<  

# 定义清理规则，删除所有中间文件和最终生成的目标文件
clean:  
	rm -rf *.o main
```

执行命令：

```bash
make         # 自动编译，生成目标文件 main
make clean   # 清理编译生成的中间文件和可执行文件
```

#### 5.2 使用 CMake：大型项目福音
写 Makefile 太麻烦？那就用 CMake！步骤如下：

+ 安装：

```bash
sudo apt install cmake -y
```

+ 项目根目录创建 `CMakeLists.txt` 文件：

```bash
# 指定最低版本要求，这里是 3.10
cmake_minimum_required(VERSION 3.10)  

# 定义项目名称，这里是 MyProject
project(MyProject)  

# 添加可执行文件 main，并指定需要的源文件
# main.cpp 和 func.cpp 是需要编译的 C++ 源文件
add_executable(main main.cpp func.cpp)
```

+ 执行命令：

```bash
# 创建一个名为 build 的目录，存放生成的构建文件
mkdir build && cd build  

# 使用 CMake 生成 Makefile，.. 表示查找上级目录中的 CMakeLists.txt
cmake ..  

# 使用 make 根据生成的 Makefile 编译项目
make
```

#### 5.3  Makefile 和 CMake 的对比
+ **Makefile**：适合小型项目，手动维护规则，简单直接。
+ **CMake**：适合跨平台和大型项目，自动生成 Makefile 等构建文件，更灵活强大。

### Step 6: 环境美化：打造专属开发体验

#### 6.1 别再盯着黑乎乎的终端了！

换个好看的终端工具，比如 **zsh + oh-my-zsh**。

+ 安装 zsh：

```bash
sudo apt install zsh
chsh -s $(which zsh)
```

+ 安装 oh-my-zsh：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

```

**如果上面的命令卡住：**

+ 打开浏览器访问： [https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh](https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)
+ 将内容复制到你的 Linux 主机用户主目录` ~  `下的 `myinstall.sh` 的文件中：

```bash
vim ~/myinstall.sh
```

+ 保存后运行本地脚本：

```bash
sh ~/myinstall.sh
```

#### 6.2 打造个性化界面

+ 配置终端主题，推荐 Powerlevel10k：

Powerlevel10k 是一款功能强大且高颜值的 Zsh 主题，可以让你的终端界面更直观、更炫酷。安装方法如下：

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/custom/themes/powerlevel10k
echo 'ZSH_THEME="powerlevel10k/powerlevel10k"' >> ~/.zshrc
source ~/.zshrc
```

+ 配置常用插件：

安装常用插件可以显著提升终端的效率和体验。以下是几个常用且实用的插件：

- 自动补全插件：zsh-autosuggestions : 这个插件可以根据历史记录自动补全命令，非常省事：
- 语法高亮插件：zsh-syntax-highlighting ，输入命令时，正确的命令会显示为绿色，错误的命令则高亮红色：

配置过程:

```bash

# 1、下载相关库 
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

# 2、vim 打开 ~/.zshrc 文件，找到 plugins=(...)，手动添加插件

plugins=(git zsh-autosuggestions zsh-syntax-highlighting)

# 3、加载配置，使其生效
source ~/.zshrc

```


zsh 安装与配置参考链接 ：https://www.hackerneo.com/blog/dev-tools/better-use-terminal-with-zsh

### Step 7: 使用 Git 管理代码

工作写代码时，版本管理是少不了的，Git 是一个强大的工具，可以帮助你记录代码的每次修改，还能和团队协作。

#### 7.1 安装 Git

在 Ubuntu 中直接执行以下命令安装：

```bash
sudo apt install git -y
```

检查安装是否成功：

```bash
git --version
```

#### 7.2 初始化 Git
进入项目文件夹 myproject（你自己新建的），初始化一个 Git 仓库：

```bash
git init
```

这会在 myproject 目录下创建一个 `.git` 文件夹，用来管理版本信息。

#### 7.3 配置用户名和邮箱
第一次使用 Git 时需要配置用户名和邮箱：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

#### 7.4 基本使用

**1. 添加文件到暂存区**  
告诉 Git 要追踪哪些文件的修改：

```bash
git add 文件名    # 添加单个文件
git add .         # 添加所有修改的文件
```

**2. 提交修改到版本库**  
将暂存区的修改提交到 Git 仓库：

```bash
git commit -m "你的提交说明"
```

**3. 查看状态**  
随时检查哪些文件有修改或者未被提交：

```bash
git status
```

**4. 查看提交历史**  
检查项目的历史提交记录：

```bash
git log
```

#### 7.5 推送到远程仓库

如果你想保存代码到 GitHub 或 GitLab 等远程仓库，先关联远程地址：

```bash
git remote add origin 远程仓库地址  

# 远程仓库地址: git@github.com:xiaokangcoding/test.git 
```

然后把代码推送到远程仓库：

```bash
git push -u origin master  
# -u 是 --set-upstream 的简写，用于设置本地分支与远程分支的 跟踪关系，方便后续的 git push 和 git pull 操作。

git push    # 推送当前分支到远程
# 后续推送直接使用 git push，不需要分支名。
```

#### **7.6 克隆远程仓库**

如果你需要下载远程仓库的代码到本地：

```bash
git clone 远程仓库地址

# 远程仓库地址: git@github.com:xiaokangcoding/test.git 
```

#### 7.7 小结

Git 是代码管理的神器。记住几个核心命令：

+ `git add`：把修改添加到暂存区。
+ `git commit`：把暂存区提交到版本库。
+ `git push`：把本地代码上传到远程仓库。
+ `git pull`：从远程仓库同步代码。

简单又高效，用好 Git，你工作开发效率会飞起！

### 总结：开发环境好了，效率也飞起了

从选择 Linux 系统，到安装开发工具、配置调试环境，再到美化终端和提升效率，你已经掌握了一套完整的 Linux C/C++ 开发环境搭建攻略。下一步？就是撸起袖子写代码啦！


## 最后：

如果这篇文章有帮到你，记得点个在看和赞👍，也可以分享给身边的小伙伴，让更多人一起进步！另外，欢迎关注我的公众号「**跟着小康学编程**」，后续还有更多有趣的技术干货等着你！ 👋


#### 关注我能学到什么？

- 这里分享 Linux C、C++、Go 开发、计算机基础知识 和 编程面试干货等，内容深入浅出，让技术学习变得轻松有趣。

- 无论您是备战面试，还是想提升编程技能，这里都致力于提供实用、有趣、有深度的技术分享。快来关注，让我们一起成长！

#### 怎么关注我的公众号？

**非常简单！扫描下方二维码即可一键关注。**

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)

