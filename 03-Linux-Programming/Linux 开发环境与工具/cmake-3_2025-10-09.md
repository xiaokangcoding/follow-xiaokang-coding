
大家好，我是小康。

在上篇 CMake提升篇 中，我们已经掌握了 CMake 的基础操作：配置灵活的构建流程、管理外部依赖。现在，用 CMake 搭建项目已经不是什么难事了。

但是，开发中的挑战远不止这些：

+ 当项目需要支持不同平台时，构建流程该如何适配？
+ 如何集成测试框架和调试工具，提升代码质量？
+ 面对构建速度慢的问题，如何优化流程，让构建快起来？
+ 大型项目里，多个子模块怎么高效管理？

这些都是 CMake 在更复杂场景中的核心能力。接下来，我们将带你从“用好”到“精通”，解锁 CMake 的更多高级玩法，让你的项目管理如虎添翼！

**一张思维导图帮你快速了解 CMake 的全貌**：

![](https://files.mdnice.com/user/71186/820508b9-8c2b-433a-9dda-b37c525877af.png)

> **友情提醒**：原创不易，如果觉得内容对你有帮助，别忘了点赞、转发，或者点个「在看」支持一下！非常感谢！🌟

在聊进阶篇之前，先来补充一下上篇提升篇没来得及详细讲的内容：**构建类型、调试和优化配置**，让基础打得更牢一点，再继续深入！  

### 7. 构建类型与调试、与优化配置

在开发过程中，常常需要根据不同的需求来配置不同的构建类型。比如，你可能需要一个调试版本（Debug）来追踪问题，也可能需要一个优化版本（Release）来提升性能。而 CMake 提供了简单的方法来管理这些构建类型，确保你可以根据不同的需求灵活切换。  

#### 7.1 常见的构建类型

CMake 支持几种常见的构建类型，你可以根据项目需求选择合适的类型。这里有四种常见的类型：

**1、Debug**：这个类型用于调试时使用。它会保留调试信息，并且没有做任何优化，程序运行速度比较慢，但能让你更容易找到 bug。

**2、Release**：用于发布版本。它会去除调试信息，并且进行各种优化，最终生成的程序执行速度最快。

**3、RelWithDebInfo**：这个类型介于 Debug 和 Release 之间，既会进行优化，又会保留一些调试信息，适合调试性能问题时使用。

**4、MinSizeRel**：这个类型的目标是让程序的体积最小化，同时也会做一些优化，适合嵌入式或者资源有限的场景。

#### 7.2 设置构建类型
我们可以通过 `CMAKE_BUILD_TYPE` 变量来选择构建类型。可以在命令行配置时通过 `-DCMAKE_BUILD_TYPE` 来设置，也可以在 CMakeLists.txt 中指定默认值。

**示例1**：命令行设置构建类型

```bash
cmake -DCMAKE_BUILD_TYPE=Release ..
```

**示例2**：CMakeLists.txt 中设置默认构建类型

```makefile
# 如果没有在命令行中指定构建类型，默认使用 Release 类型
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()
```

#### 7.3 调试信息配置

在 `Debug` 模式下，CMake 会为源代码生成调试信息，这样你可以在调试器（如 `gdb` 或 `Visual Studio`）中查看程序的变量、调用栈等。

你可以通过设置 CMake 变量 `CMAKE_CXX_FLAGS_DEBUG` 来调整调试信息的详细程度。比如，禁用优化并生成调试信息：

```makefile
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")  # 生成调试信息，禁用优化
```

+ `-g`：生成调试信息
+ `-O0`：禁用优化，确保调试时代码行为不被改变

#### 7.4 优化配置

CMake 也让你可以根据不同的构建类型调整代码的优化级别：

+ **Debug**：一般不启用优化，因为开发时你希望看到最精确的代码行为。
+ **Release**：启用高级优化（比如 `-O2` 或 `-O3`），提升代码执行效率，但会去除调试信息。
+ **RelWithDebInfo**：启用适度的优化（如 `-O2`），同时保留调试信息，适合发布后调试。

举个例子，在 `Release` 模式下你可以设置如下优化：

```makefile
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
```

+ `-O3`：开启最高级别的优化。
+ `-DNDEBUG`：禁用断言代码（通常用于发布版本）。

#### 7.5 针对不同平台的配置

不同的操作系统和编译器可能需要不同的调试和优化配置。你可以通过条件判断来设置平台特定的标志。例如，Windows 上使用 `MSVC` 编译器时，可能需要不同的配置：

```makefile
if(MSVC)
  # 对于 MSVC 编译器
  set(CMAKE_CXX_FLAGS_DEBUG "/Od /Zi")
  set(CMAKE_CXX_FLAGS_RELEASE "/O2 /DNDEBUG")
else()
  # 对于 GCC 或 Clang 编译器
  set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
  set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
endif()
```

#### 7.6 调试与优化结合

有时你希望在调试时既能保留调试信息，又能进行一定的优化。此时可以选择 `RelWithDebInfo` 构建类型，它在优化性能的同时保留了调试信息：

```makefile
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
```

这将启用适度的优化（`-O2`），并保留调试信息（`-g`），既能提高性能，也能方便调试。

#### 小结一下：

在 CMake 中，不同的构建类型让我们能够更好地控制程序在开发、调试和发布时的表现。选择合适的构建类型，可以帮助我们更高效地开发和优化程序。

接下来，我们来进入 CMake 进阶篇：专业构建的内容讲解。

## 进阶篇：专业构建  

### 8. CMake 的跨平台构建与工具链配置

#### 8.1 跨平台构建：为什么要用？

你可能有这样的需求：你在 Windows 上开发，但你的程序最终需要在 Linux、macOS 或嵌入式设备上运行。

传统的做法是直接在目标平台上编译，但这种方式有时不太现实。比如，开发者通常没有目标平台的硬件资源，或者目标平台的操作系统与开发机器差异很大，这时就需要使用 **交叉编译**（Cross-compilation）。

交叉编译就是指在一个平台（比如 Windows）上开发，编译出在另一个平台（比如 Linux）上运行的程序。CMake 就能轻松帮你实现这一点，让你在开发机器上生成目标平台可用的可执行文件。

#### 8.2 工具链文件（Toolchain）是什么？

CMake 支持通过 **工具链文件**（Toolchain file）来配置交叉编译。工具链文件是一个简单的文本文件，它告诉 CMake 使用哪个编译器和工具来构建程序，尤其是当你想从一个平台为另一个平台构建程序时。

比如，你可能想在 Windows 上构建一个程序，但目标平台是 Linux。为了让 CMake 知道你想使用的 Linux 编译器和工具链，你需要提供一个工具链文件来配置这一切。

这个工具链文件通常包括以下几项内容：

+ **编译器设置**：指定用于编译代码的编译器（比如 GCC 或 Clang）。
+ **系统信息**：告诉 CMake 目标平台是什么，比如是 Linux、ARM 或其他平台。
+ **库和头文件路径**：指定工具链中包含的库和头文件，以便交叉编译器能够找到它们。

#### 8.3 如何使用工具链文件？

假设你在 Windows 上开发，目标是构建一个在 Linux 上运行的程序，你可以通过以下步骤来配置工具链文件：

**1. 编写工具链文件（toolchain.cmake）**

工具链文件主要用来定义编译器、系统信息以及相关的路径设置。比如，为了交叉编译一个 Linux 程序，你可以这样写：

```makefile 
# 设置交叉编译的系统信息
set(CMAKE_SYSTEM_NAME Linux)         # 目标系统是 Linux
set(CMAKE_SYSTEM_PROCESSOR x86_64)   # 目标处理器是 x86_64 架构

# 设置交叉编译器路径
set(CMAKE_C_COMPILER /usr/bin/gcc)   # 指定交叉编译的 C 编译器
set(CMAKE_CXX_COMPILER /usr/bin/g++) # 指定交叉编译的 C++ 编译器

# 设置交叉编译所需的库路径
set(CMAKE_FIND_ROOT_PATH /path/to/linux/libs)
```

+ `CMAKE_SYSTEM_NAME`和`CMAKE_SYSTEM_PROCESSOR`： 是用来告诉 CMake 你要构建程序的目标系统(Linux)和处理器架构(x86_64)。
+ `CMAKE_C_COMPILER` 和 `CMAKE_CXX_COMPILER`：指定交叉编译器路径，比如 GCC/G++。
+ `CMAKE_FIND_ROOT_PATH`：告诉 CMake 去哪里找目标平台的库和依赖。

这个文件的意思就是告诉 CMake：“我要为 Linux 系统编译程序，并且使用 GCC 编译器。”

> **注意**：每个目标平台需要对应一个工具链文件，但只需要写一次就行。之后每次构建这个平台的程序，都可以重复使用这个文件，不用反复修改。

**2. 指定工具链文件进行构建**

在命令行中运行 CMake 配置时，通过 `CMAKE_TOOLCHAIN_FILE` 指定工具链文件路径，CMake 就会自动按照里面的设置来构建项目：

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake ..
```

+ 这条命令的意思是告诉 CMake：“用这个工具链文件的配置，为目标平台（比如 Linux）生成构建文件。”

通过这种方式，你可以在 Windows 上为 Linux 平台构建程序，而无需手动配置编译器路径和库路径。CMake 会根据工具链文件的内容自动完成这些工作。

**统一的构建配置:**

虽然每个平台需要一个对应的工具链文件，但项目的主配置文件（CMakeLists.txt）只需要写一次，不用为每个平台单独调整。换句话说，工具链文件解决的是平台的差异，而主配置文件则保持一致。这种“分工合作”的方式，让项目既灵活又高效。

比如，你的主配置文件 CMakeLists.txt 文件可以这样写：

```makefile
project(MyProject)
...

add_executable(my_app main.cpp)
```

无论是为 Windows、Linux 还是其他平台构建，只需要切换工具链文件即可，主配置文件完全不用改！

#### 小结一下：

工具链文件让跨平台开发变得简单：

+ 每个平台只需要配置一次工具链文件。
+ 主配置文件（CMakeLists.txt）可以保持统一，无需针对不同平台进行调整。

#### 8.4 工具链配置的常见用途

+ **为嵌入式系统交叉编译**：比如，你在 Windows 上开发程序，但目标是树莓派或其他嵌入式 Linux 设备。你可以在工具链文件中指定交叉编译器（如 ARM 编译器），然后 CMake 就会自动为目标设备生成可执行文件。
+ **为不同平台构建**：比如开发一个多平台支持的应用（Windows、Linux、macOS），你可以根据不同平台配置不同的工具链文件，CMake 会自动选择合适的工具链来构建目标平台的程序。
+ **自动化构建过程**：如果你在 CI/CD 系统中工作，可以通过工具链文件自动化交叉编译过程，确保在开发环境与目标平台之间的构建一致性。

#### 总结:

通过 CMake 工具链文件，CMake 可以让跨平台构建变得轻而易举。比如，你可以在 Windows 上配置一个交叉编译环境，为 Linux 或其他平台构建应用，而不用每次切换平台时手动配置编译器和依赖。

这大大简化了跨平台开发的流程，无论是单一平台的应用还是多平台支持的项目，CMake 都能帮你省去繁琐的配置工作，让你把更多精力放在代码开发和优化上。

### 9. CMake 的测试与调试

当你的项目越来越复杂时，单纯的构建和编译已经不够了，你需要确保代码没有 bug，并且在不同的环境下都能正常工作。CMake 提供了强大的工具来帮助你进行 **单元测试** 和 **调试**，让你在开发过程中能够更轻松地找到问题。

#### 9.1 如何在 CMake 项目中集成单元测试
在开发大型项目时，单元测试是确保代码正确性的一种非常有效的方式。

CMake 支持与多个测试框架（如 Google Test、Catch2 等）集成，帮助你自动化测试流程。CMake 自带的 `CTest` 工具也非常好用，它能让你方便地运行所有测试，并生成测试报告。

**集成 Google Test 示例：**

```makefile
# CMake 版本要求：3.11 及以上
cmake_minimum_required(VERSION 3.11)

project(MyProject)

# 添加 Google Test 库，CMake 3.11 以上支持 FetchContent
include(FetchContent)

# 下载 Google Test
FetchContent_Declare(
    gtest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG master  # 你可以指定特定的版本，例如 v1.11.0
)

# 下载并添加到项目中
FetchContent_MakeAvailable(gtest)

# 将测试代码编译为可执行文件
add_executable(test_my_project test_my_project.cpp)

# 将 Google Test 库链接到测试可执行文件
target_link_libraries(test_my_project gtest gtest_main)

# 在构建过程中启用测试
enable_testing()

# 添加测试目标
add_test(NAME MyTest COMMAND test_my_project)
```

通过上述配置，你就能在 CMake 中运行 `ctest`命令来执行你的单元测试了。`enable_testing()` 告诉 CMake 启用测试功能，而 `add_test()` 用来注册测试。

#### 9.2 调试 CMake 构建问题

在构建大型项目时，可能会遇到一些构建问题，比如编译器找不到某个文件，或者库的版本不匹配等。CMake 提供了一些调试工具，帮助你快速找到问题。

使用 `cmake --debug` 来调试：

你可以通过 `cmake --debug` 命令来查看 CMake 配置过程中的详细信息。如果遇到问题，打开调试日志可以帮助你快速定位错误。

```bash
cmake --debug-output ..
```

这个命令会显示 CMake 执行的每一步，帮助你理解 CMake 在做什么，以及在哪个步骤出现了问题。比如，如果你发现某些文件没有被正确找到，CMake 会告诉你具体是哪一步出了问题。

#### 9.3 使用 CMake 构建类型进行调试

除了基本的构建和调试，前文提到 CMake 还支持不同的 **构建类型**（如 Debug 和 Release），这对于调试和优化都非常有帮助。

+ **Debug 模式**：启用调试信息（如 `-g`），并禁用优化（如 `-O0`）。这意味着编译器不会优化你的代码，保证你能看到最原始的代码行为。Debug 模式下，你可以使用 GDB 或 Visual Studio 等工具进行逐步调试。

```makefile
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
```

+ **Release 模式**：启用优化（如 `-O2` 或 `-O3`），同时去除调试信息，生成高效的二进制文件。通常发布版本会使用这种模式。

```makefile
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
```

#### 9.4 常见的 CMake 调试技巧

+ **打印变量值**：在 CMake 中，你可以使用 `message()` 函数打印变量值来帮助调试。例如：

```makefile
message(STATUS "CMAKE_BUILD_TYPE is ${CMAKE_BUILD_TYPE}")
```

这会在 CMake 配置过程中打印出 `CMAKE_BUILD_TYPE` 的值，帮助你检查是否设置正确。

+ **调试 CMakeLists.txt 配置**：如果遇到复杂的配置问题，尝试将 `CMakeLists.txt` 中的部分代码注释掉，逐步恢复，这样可以帮助你定位具体是哪一行配置出了问题。

#### 小结

+ **单元测试**：你可以通过 CMake 配置和管理项目的测试，确保代码质量。
+ **调试 CMake 构建问题**：通过 `cmake --debug` 和 `message()` 函数，可以帮助你快速定位配置问题。
+ **构建类型调试**：Debug 模式适合调试代码，Release 模式适合发布，CMake 让你能轻松切换这些构建类型。

CMake 不仅仅是一个构建工具，还成为了你开发调试过程中的得力助手，帮助你提高开发效率，确保项目顺利完成。

### 总结

这篇文章，我们先补充了之前没有详细讲到的 CMake 构建类型与优化配置，学会了：

+ **构建类型与优化**：调试版、发布版的切换，以及针对不同平台的优化配置，让项目跑得又快又稳。

然后进入了 进阶篇：专业构建 的部分内容，详细讲解了：

+ **跨平台构建与工具链配置**：如何通过统一的项目配置文件（CMakeLists.txt），结合不同平台的工具链文件，实现无缝跨平台构建。
+ **测试与调试**：用 CMake 集成测试框架和调试工具，提升代码质量，开发效率更上一层楼。

这些内容进一步增强了 CMake 在实际项目中的应用能力，让你对复杂场景的构建管理有了更深的理解。

### 下篇预告

为了不让篇幅过长，进阶篇的剩余两个主题，我们放到下篇文章中再详细讲解：

+ **CMake 的性能优化**：如何缩短构建时间、优化依赖管理，让构建流程更加高效？
+ **高效构建：自定义模块与多项目管理**：面对大型项目和多个模块，如何用 CMake 做到模块化管理、灵活配置？

如果觉得这篇文章还不错，别忘了点个「赞」、点个「在看」，或分享给更多对 cmake 感兴趣的小伙伴！

也欢迎大家关注我的公众号「跟着小康学编程」，下一篇将继续带你深入 CMake 的高阶玩法，别忘了关注！我们下篇见！

#### 怎么关注我的公众号？

非常简单！扫描下方二维码即可一键关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)