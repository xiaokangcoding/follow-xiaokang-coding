### 前言：Makefile 能干的，CMake 干得更优雅

在上篇文章中，我们聊了聊 Makefile。虽然它是 C/C++ 项目编译的“老司机”，但写起来真的是让人头大。尤其是当项目文件一多，手写依赖就像在搬砖，费时又费力。

那么问题来了，难道我们就没有更优雅的工具了吗？答案是：有！

这时候，CMake 就像一个专业的项目管家，它会帮你处理所有琐碎的编译细节。你只需要告诉它“我要干什么”，剩下的事情交给它就行了。关键是，它还能跨平台支持，能帮你生成各种构建系统，比如 Makefile、Visual Studio 工程文件、Xcode 工程，堪称“编译界的瑞士军刀”！

今天我们就从头到尾，用最简单的大白话，把 CMake 的用法讲清楚！看完这篇文章，你一定会觉得：“CMake 原来这么简单！”

在正式开始之前，先通过一张思维导图，帮你快速了解 CMake 的全貌，这样你心里会更有底：

![](https://files.mdnice.com/user/71186/820508b9-8c2b-433a-9dda-b37c525877af.png)

> **友情提醒**：记得点赞、在看、转发支持我哦！多谢！🌟

我计划分四篇文章来介绍 cmake，今天只讲基础篇部分。

## 基础篇

### 1. 什么是 CMake？为什么用它？

CMake 是一个跨平台的项目构建工具，通俗点说，它会帮你生成 Makefile 或其他编译系统需要的构建文件。

#### 用 CMake 的好处：

+ **不用手写复杂的 Makefile**：你只需要写一个简单的 `CMakeLists.txt`，CMake 会自动帮你生成 Makefile。
+ **支持跨平台开发**：不管是 Linux、Windows 还是 MacOS，CMake 都能轻松搞定。
+ **自动化依赖管理**：只要源文件有改动，它会自动处理文件依赖，不用你手动维护。
+ **支持大项目**：几百个源文件的项目，CMake 比你还细心，编译起来稳得一批。

简单来说，**CMake = 省心 + 高效**。

### 2. CMake 的基本使用流程

#### 2.1 基本流程

学习 CMake，就像学做饭，只有几个简单步骤：

1. **写菜单**：创建一个 `CMakeLists.txt` 文件，告诉 CMake 要编译哪些源文件。
2. **配置厨房**：运行 CMake，生成构建文件（比如 Makefile）。
3. **开火做饭**：用生成的构建文件开始编译，得到程序。

以下是 `CMake` 与 `Makefile` 的构建流程，帮助你更直观地理解这些步骤：

```csharp
项目源码
   │
   └──► [CMakeLists.txt]  # 编写 CMake 构建文件
           │
           ▼
         cmake (命令)  # 运行 CMake 命令生成 Makefile
           │
           ▼
      [Makefile]      # 自动生成的 Makefile
           │
           ▼
         make (命令)  # 使用 Makefile 构建目标
           │
           ▼
       目标文件        # 编译生成的目标文件（如可执行程序或库）
```

#### 2.2 关键命令

假设你的代码放在项目目录 `my_project/` 下，使用 CMake 的流程如下：

```bash
mkdir build           # 创建编译目录（推荐在项目外部）
cd build              # 进入编译目录
cmake ..              # 生成 Makefile
make                  # 开始编译
./hello               # 运行可执行文件
```

### 3. 第一个简单的 CMake 项目

#### 3.1 文件结构

我们写一个最简单的项目，只有一个 `main.cpp` 文件：

```bash
my_project/
├── main.cpp
├── CMakeLists.txt
```

`main.cpp`：

```c++
#include <iostream>
int main() {
    std::cout << "Hello, CMake!" << std::endl;
    return 0;
}
```

#### 3.2 编写 CMakeLists.txt

在项目根目录下，新建一个名为 `CMakeLists.txt` 的文件，内容如下：

```makefile
# 指定 CMake 的最低版本
cmake_minimum_required(VERSION 3.10)

# 设置项目名称
project(HelloCMake)

# 添加可执行文件
add_executable(hello main.cpp)
```

**解释：**

+ `cmake_minimum_required()`：告诉 CMake 使用的最低版本，避免版本兼容问题。
+ `project()`：设置项目名称。
+ `add_executable()`：告诉 CMake，我们要生成一个名字叫 `hello` 的可执行文件，它的源文件是 `main.cpp`。

#### 3.3 编译运行

1、在项目目录下新建 `build` 文件夹，并进入：

```bash
mkdir build
cd build
```
2.运行 CMake 配置命令：
```bash
cmake ..
```
此时，CMake 会生成 `Makefile`。

3.执行 `make` 开始编译：

```bash
make
```
4.运行生成的程序：

```bash
./hello
```

你会看到输出：

```plain
Hello, CMake!
```

恭喜你，第一个简单的 `CMake` 项目完成了！

### 4. 支持多源文件和目录

当你的项目越来越大，源代码文件越来越多时，你一定会觉得直接在 `CMakeLists.txt` 中写一堆文件名特别麻烦。别担心，CMake 给你提供了更优雅的解决方案，让代码管理更简单！

#### 4.1 添加多个源文件

如果你的项目中有多个源文件，像 `main.cpp` 和 `utils.cpp`，你可以直接把它们放到 `add_executable()` 中：

```makefile
add_executable(hello main.cpp utils.cpp)
```

这样，CMake 就会把这些文件一起编译成一个可执行文件 `hello`。简单明了，对吧？

#### 4.2 使用变量管理源文件

但当源文件越来越多时，手动列出每个文件就显得很麻烦。这时候，我们可以用变量来管理这些文件，CMake 会帮你搞定。

```makefile
set(SOURCES main.cpp utils.cpp)
add_executable(hello ${SOURCES})
```

通过这种方式，你可以轻松管理文件列表，未来如果需要增加或删除文件，只需要修改这个变量，就能一劳永逸。

是不是很方便？

除了自己定义的变量，CMake 还可以访问系统环境变量。例如，获取当前用户的 `HOME` 目录：  

```makefile
set(MY_PATH $ENV{HOME})
```

CMake 提供了 `$ENV{ }`语法，用于访问系统的环境变量，通过 `$ENV{HOME}`，CMake 会自动读取你系统中的 `HOME` 环境变量，把它的值赋给 `MY_PATH` 变量。  

#### 4.3 多目录管理

随着项目逐渐变大，你的代码可能会分散到多个目录中。比如，你可能会有一个 `src/` 目录存放核心代码，一个 `include/` 目录存放头文件，还有一个 `libs/` 目录存放外部库的文件。

到了这个阶段，你可能不希望将所有文件都写在一个 CMakeLists.txt 文件里了。没错，CMake 完全支持这种方式，允许你轻松管理多个目录的代码。

**使用 cmake 来管理项目，目录结构一般是这样**：

```objectivec
my_project/
├── CMakeLists.txt       # 根目录的 CMake 配置
├── src/
│   ├── CMakeLists.txt   # src 目录下的 CMake 配置
│   ├── main.cpp
│   ├── utils.cpp
│   └── helper.cpp
├── libs/
│   ├── CMakeLists.txt   # libs 目录下的 CMake 配置
│   ├── lib.cpp
│   └── lib.h
├── include/
│   ├── utils.h
│   └── helper.h
└── build/               # 构建目录（自动创建）
```

在这个结构中，根目录下有一个 `CMakeLists.txt` 文件，它负责管理整个项目。而 `src/` 和 `libs/` 目录下分别有各自的 `CMakeLists.txt` 文件，用来管理这些目录中的源文件和库。这样，不仅让项目更有条理，而且也方便管理。

**1、根目录的 CMakeLists.txt**

在根目录的 `CMakeLists.txt` 文件中，我们通常做一些全局的配置，比如设置项目的名字、指定 CMake 最低版本要求，以及引入子目录等。

```makefile
# 根目录的 CMakeLists.txt

# 设置项目名和 CMake 最低版本
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 添加 src 和 libs 子目录
add_subdirectory(src)
add_subdirectory(libs)
```

这里做了两件事：

- 使用 `add_subdirectory()` 告诉 CMake 去处理 `src/` 和 `libs/` 子目录的 `CMakeLists.txt` 文件。
- 设置项目名为 `MyProject`，并指定了最低版本要求（`3.10`）。

**2、src/ 目录的 CMakeLists.txt**

接下来，在 `src/` 目录下的 `CMakeLists.txt` 文件中，负责编译 `src/` 目录下的源文件（比如 `main.cpp` 和 `utils.cpp`）。如果有多个源文件，你可以通过变量来管理这些文件。

```makefile
# src/CMakeLists.txt

# 设置源文件
set(SOURCES
    main.cpp
    utils.cpp
    helper.cpp
)

# 创建可执行文件
add_executable(hello ${SOURCES})

# 设置头文件路径
target_include_directories(hello PRIVATE ${PROJECT_SOURCE_DIR}/include)
# PROJECT_SOURCE_DIR：这是一个 CMake 内置的变量，表示项目的源代码根目录。
# 无论你在哪个子目录中，它都会指向项目的最外层目录：即 my_project/

# 链接库
target_link_libraries(hello PRIVATE mylib)
```

这段代码做了几件事：

- 使用 `set(SOURCES ...)` 将 `src/` 目录下的源文件名赋值给变量 SOURCES，方便下面使用。
- 使用 `add_executable()` 创建可执行文件 `hello`。
- 使用 `target_include_directories()` 设置头文件路径，指向根目录下的 `include/` 目录，这样编译器就能找到头文件。
- 使用 `target_link_libraries()` 连接一个外部库 `mylib`（假设它在 `libs/` 目录下）。

**3、libs/ 目录的 CMakeLists.txt**

`libs/` 目录下的 `CMakeLists.txt` 文件类似地负责编译库文件。

```makefile
# libs/CMakeLists.txt

# 设置库文件源代码
set(LIB_SOURCES
    lib.cpp
)

# 创建静态库
add_library(mylib STATIC ${LIB_SOURCES}) # STATIC 可改为 SHARED 生成动态库

# 设置头文件路径
target_include_directories(mylib PUBLIC ${PROJECT_SOURCE_DIR}/include)
```

在这里：

- 使用 `set(LIB_SOURCES ...)` 设置变量 LIB_SOURCES，将 `libs/` 目录下的源文件名赋值给它。
- 使用 `add_library()` 创建了一个静态库 `mylib`。
- 使用 `target_include_directories()` 设置库的头文件路径，指向根目录下的 `include/` 目录，通过 PUBLIC 关键字，确保其他需要引用 `mylib` 的目标能够找到头文件。

**4、项目根目录的头文件（include/）**

在 `include/` 目录下存放了项目的头文件。需要注意的是，你不需要在 `src/` 和 `libs/` 的 CMakeLists.txt 文件中列出这些头文件。

CMake 会自动知道你从 `include/` 目录中包含头文件，只要你确保`src/` 和 `libs/` 目录下的 CMakeLists.txt 文件中正确配置了 `target_include_directories()`。

```bash
my_project/
├── include/
│   ├── utils.h
│   └── helper.h
```

**5、添加 `build/` 目录**

到这里，你的代码管理已经非常清晰了，但接下来，我们需要谈一谈 `build/` 目录。`build/` 目录是构建过程中的输出目录，它将存放所有编译产生的中间文件、目标文件和最终的可执行文件。

通常，在 CMake 构建项目时，我们不会直接在源码目录下运行构建命令，而是会创建一个单独的 `build/` 目录，这样可以保持源码目录的整洁，同时也可以更方便地管理构建文件。你可以这样做：

+ 创建 `build/` 目录：

在项目根目录下创建一个 `build/` 目录：

```bash
mkdir build
cd build
```

+ 配置项目：

然后在 `build/` 目录下运行 `cmake` 命令，CMake 会自动查找根目录下的 `CMakeLists.txt` 文件并配置整个项目。

```bash
cmake ..
```
这样，CMake 会根据根目录的 `CMakeLists.txt` 配置生成必要的构建文件。

+ 开始构建：

配置完成后，你可以运行 `make` 或其他构建工具来进行编译：

```bash
make
```

+ 构建输出：

编译完成后，所有生成的文件（比如目标文件、可执行文件等）都会被放到 `build/` 目录里。源代码目录就不会被任何临时文件污染了，保持整洁。

#### 小结：

通过这种结构和配置方式，CMake 可以非常有效地管理多个目录下的源代码和库：

+ 根目录的 `CMakeLists.txt` 负责引入子目录和进行全局配置。
+ 每个子目录（如 `src/` 和 `libs/`）都有自己的 `CMakeLists.txt` 来管理该目录下的源文件和库。
+ `target_include_directories()` 确保头文件可以被正确引用。
+ `build/` 目录用来存放所有构建过程中生成的中间文件和最终的可执行文件，保持源代码目录干净整洁。


这种多目录的管理方式，会让你的项目变得更加模块化、易于扩展，同时也方便多人协作开发。如果以后需要添加新的库或源文件，只需在对应子目录的 `CMakeLists.txt` 中做调整，而根目录只需要引入子目录即可。

### 最后:

如果这篇文章对你有帮助，别忘了点个「赞」、点个「在看」，或分享给更多对 cmake 感兴趣的小伙伴！

也欢迎大家关注我的公众号「跟着小康学编程」，下一篇，我们接着为大家介绍 CMake 提升篇。

### 下篇预告：CMake 提升篇

接下来，我们会一起探索 CMake 更强大的功能，让你从“会用”到“用好”：

+ **高效配置**：怎么用条件判断、循环和字符串操作让 CMakeLists.txt 变得更灵活？
+ **依赖管理**：如何优雅地处理第三方库，比如 Boost、OpenCV？
+ **构建优化**：调试版和发布版该如何配置？如何让你的程序跑得更快？

这些内容会让你在实际项目中更加得心应手，敬请期待！

#### 怎么关注我的公众号？

非常简单！扫描下方二维码即可一键关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)