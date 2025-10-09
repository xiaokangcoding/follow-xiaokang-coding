大家好，我是小康。

上次我们聊了 CMake 的构建类型、跨平台构建，还有测试和调试的整合，相信大家已经对 CMake 的应用有了更深的理解。

不过，项目越大，需求越复杂，构建流程和模块管理也会变得更棘手。比如，如何让编译更快一点？怎么优雅地管理多个模块，让项目不乱套？别急，这篇文章我们继续补充 CMake 进阶篇的剩余两个，聊聊 CMake 的性能优化和多模块构建的高级技巧！

### 10. CMake 的性能优化

在开发大型项目时，编译时间往往会变得很长，这时候优化 CMake 的构建过程是非常有必要的。幸运的是，CMake 提供了很多技巧和工具来帮助我们加速构建过程，减少无意义的编译时间。我们可以从多个方面来优化性能，今天我们重点讲解几个常见的优化方法。

#### 10.1 使用并行构建

构建项目时，CMake 默认会按照顺序一个任务一个任务地执行，尤其是当项目越来越大时，这样的构建方式效率低下。好消息是，CMake 允许我们启用并行构建，这样多个构建任务就可以同时进行，极大地缩短了构建时间。

**如何启用并行构建？** 

其实，这个不需要在 CMake 配置文件中做任何改动。只需要在运行 `make` 或 `cmake --build` 时，加上 `-j` 参数即可指定并行构建的任务数量：

```bash
# 指定 4 个任务并行编译
make -j4
```

如果你不确定你有多少核心可以用，直接使用 `-j`，CMake 会自动选择合适的并行数：

```bash
make -j
```

这样做会显著提高构建速度，尤其是在多核 CPU 上。

> **注意**：一般推荐设置为 CPU 核心数或略大于核心数。例如： 如果有 4 核心，可以使用 `make -j4` 或   `make -j5`

#### 10.2 降低不必要的重新构建

**1、使用缓存变量**

CMake 可能会因为一些设置或配置的变化触发全量构建。比如，每次你设置了构建类型（Debug、Release 等），CMake 会重新计算一遍所有的配置，造成浪费。为了避免这种情况，我们可以使用 **缓存变量**。

举个例子： 假设你设置了构建类型 `CMAKE_BUILD_TYPE`，你可以通过缓存机制来保存它，避免每次都重新设置。

```makefile
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the build type")
```

这段代码将 `CMAKE_BUILD_TYPE` 设置为 `Release`，并且把它存到缓存中。下次你运行 CMake 时，它会记住这个设置，而不会重新计算一遍，从而加快构建过程。

**2、增量构建**

CMake 的构建很聪明，不会每次都重新编译所有文件。只有当代码发生改动时，才会重新编译相关的部分，比如修改了某个源文件，CMake 只会编译这个文件，不会动其他没改的文件。

这种智能的增量编译是基于目标和文件的依赖关系实现的。比如：

```makefile
set(SRC_FILES lib.c)

add_library(mylib STATIC ${SRC_FILES})
target_link_libraries(mylib otherlib)
```

这里的 `mylib` 依赖 `SRC_FILES`，CMake 会跟踪这些文件的变化，只有文件被修改时，`mylib` 才会重新编译。

**3、缩小编译范围**

有时候，整个项目非常庞大，你只想重新编译其中的一个部分。这时，你可以使用 `--target` 参数来只编译你需要的部分。

```makefile
cmake --build . --target my_target
```

这个命令会只编译 `my_target`，而不是整个项目，这样就节省了很多时间，尤其在调试阶段，非常有用。

#### 10.3 使用预构建的库

在项目中，我们可能会依赖一些外部库，像 Google Test、Boost 等。每次构建时，如果都把这些外部库重新编译一遍，不仅浪费时间，还增加了构建过程中的复杂性。

那么，怎么解决这个问题呢？一个很简单的方法就是 **使用预编译的库**。预编译的库，顾名思义，就是那些已经编译好的二进制文件，像 `.a` 或 `.lib` 文件。我们可以直接在项目中使用它们，而不必每次都重新编译。

**举个例子**：使用 Google Test 预编译库

如果你在项目中使用 Google Test（用于单元测试），你可以选择直接 链接到已编译的 Google Test 库，而不是每次都让 CMake 去下载、编译它。

假设你已经在系统中安装好了 Google Test 库，那你可以直接用 `find_package` 命令来找到这个库，然后链接它。

```makefile
# 查找已安装的 Google Test 库
find_package(GTest REQUIRED)

# 将 Google Test 库链接到你的测试程序
target_link_libraries(test_my_project GTest::GTest)
```

这样，CMake 就会直接使用已经安装好的 Google Test 库，而不会重新编译它。每次构建时，只有你修改的代码会被重新编译，大大节省了时间。

#### 10.4 启用 CMake 的编译缓存（CMake's Build Cache）

每次修改代码后，构建项目可能会花费很多时间，特别是大型项目。CMake 提供了构建缓存机制，帮助我们加速构建。

**1、构建缓存怎么工作的？**

每次运行 CMake 配置项目时，它会生成一个 `CMakeCache.txt` 文件，记录下所有配置信息（比如编译器、构建类型等）。下次构建时，CMake 会读取缓存，跳过重复配置，从而加快构建速度。

**2、使用 ccache 加速编译**

除了 CMake 自带的缓存，还有一个叫 **ccache** 的工具，它会缓存已经编译过的文件。下次相同的文件不需要重新编译，直接用缓存结果，节省时间。

**3、如何启用 ccache？**

+ 安装 ccache：

```bash
sudo apt-get install ccache
```

+ 在`CMakeLists.txt` 中启用：

```makefile
set(CMAKE_CXX_COMPILER_LAUNCHER ccache)
```

#### 小结 :

通过这些优化方法，你可以显著提高 CMake 构建过程的效率。无论是通过并行构建、避免不必要的重新构建，还是使用预编译库和编译缓存，都能帮助你更快速、更高效地构建大型项目。通过优化 CMake 配置，开发者可以节省大量的构建时间，把更多的精力投入到实际的开发和测试工作中。

### 11. CMake 高效构建：自定义模块与多项目管理

在上文中，我们讲了 CMake 性能优化的技巧，比如如何加速构建、如何避免不必要的重复构建等。接下来，我们要谈的是如何让你的 CMake 构建系统更加灵活和高效，特别是在处理 **多个项目** 或 **模块** 时，如何通过 **自定义模块** 来简化你的构建过程。

让我们一步一步来，掌握如何通过自定义和多项目管理，提升构建效率。

#### 11.1 自定义 CMake 模块：让构建更智能

自定义模块是 CMake 中非常有用的功能，它可以帮助我们把重复的构建逻辑封装起来，让构建过程变得更清晰、简洁。

想象一下，你每次都需要设置编译选项、库路径，或者定义一些通用的构建规则。每次都在 `CMakeLists.txt` 中重复写一遍，既麻烦又容易出错。这个时候，自定义 CMake 模块就派上用场了。

**怎么做呢？**

+ 创建一个新的 CMake 文件，比如 `MyModule.cmake`，然后把一些常用的构建逻辑放到里面。
+ 之后，在项目的 `CMakeLists.txt` 中通过 `include()` 把这个模块引入进来，就可以直接使用这些构建规则。

通常你可以在项目根目录下创建一个 `cmake/` 文件夹，用来存放所有的自定义模块。比如，你可以在 cmake 文件夹下创建一个 `MyModule.cmake` 文件来设置常见的编译选项：

```makefile
# MyModule.cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
set(CMAKE_BUILD_TYPE Debug)
```

 然后，在你的主 `CMakeLists.txt` 文件中，通过 `include()` 命令引入这个模块：  

```makefile
# CMakeLists.txt
include(MyModule.cmake)
```

这样，每次你都可以直接用这个模块提供的构建选项，避免每个项目都重复配置。

你也可以根据需要创建更多的模块，比如设置库路径、配置编译器、查找依赖等，都是非常简单的。

#### 11.2 自定义函数：更简洁的构建任务

除了模块，CMake 还允许我们定义 **自定义函数**，帮助简化重复性的构建任务。例如，你经常需要为多个源文件设置一些额外的标志，或者将不同的源文件组合成一个库，你可以通过函数来封装这些操作。

**怎么做呢？**

你可以定义一个函数来统一设置目标：

```makefile
# CMakeLists.txt

# 定义一个函数来创建一个可执行文件
function(create_executable target_name)
    add_executable(${target_name} ${ARGN}) # ${ARGN} 是 CMake 内置变量，表示传递给函数的其余参数（这里是源文件列表）
    target_compile_options(${target_name} PRIVATE -Wall) # -Wall：启用编译器的所有常见警告，有助于捕获潜在问题。
endfunction()

# 调用函数来创建项目的可执行文件
create_executable(my_app main.cpp foo.cpp bar.cpp)
```

这里我们定义了一个 `create_executable()` 函数，它接受一个目标名称和多个源文件作为参数，自动设置编译选项并生成可执行文件。通过这种方式，我们可以避免在每次添加可执行文件时都手动配置编译选项。

#### 11.3 多项目管理：如何处理大型项目

随着项目的复杂性增加，单一的 `CMakeLists.txt` 文件往往会变得庞大且难以管理。这时，你就需要用到 **多项目管理**。

CMake 提供了 `add_subdirectory()` 命令，可以帮助你把多个子项目（或子模块）拆分开，每个子项目有自己独立的 `CMakeLists.txt` 文件，这样更方便管理和维护。

假设你有一个主项目，并且这个项目依赖于两个子模块：`moduleA` 和 `moduleB`。

+ 项目结构可能是这样：

```objectivec
project_root/
├── CMakeLists.txt        # 主项目的 CMake 配置
├── moduleA/
│   ├── CMakeLists.txt    # moduleA 的 CMake 配置
│   ├── moduleA.cpp       # moduleA 的实现文件
│   └── moduleA.h         # moduleA 的头文件（可选）
├── moduleB/
│   ├── CMakeLists.txt    # moduleB 的 CMake 配置
│   ├── moduleB.cpp       # moduleB 的实现文件
│   └── moduleB.h         # moduleB 的头文件（可选）
└── main.cpp              # 主项目入口文件
```

+ 然后你可以在主项目的 `CMakeLists.txt` 中这样写：

```makefile
# 主项目的 CMakeLists.txt
add_subdirectory(moduleA)
add_subdirectory(moduleB)

add_executable(main_app main.cpp)
target_link_libraries(main_app moduleA moduleB)
```

+ 子模块`moduleA` 的 `CMakeLists.txt` 可能是这样：

```makefile
# moduleA 的 CMakeLists.txt

# 创建一个静态库 moduleA，包含 moduleA.cpp 源文件
add_library(moduleA STATIC moduleA.cpp)

# 为 moduleA 添加编译选项：启用常见警告提示 (-Wall)
target_compile_options(moduleA PRIVATE -Wall)

# 设置头文件的包含路径，PUBLIC 表示其他目标也能使用该路径
# ${CMAKE_CURRENT_SOURCE_DIR} 指向当前目录，即 moduleA 的目录
target_include_directories(moduleA PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

```

在这种方式下，每个子模块都可以有自己的独立的构建逻辑，主项目通过 `add_subdirectory()` 引入它们，然后进行统一管理和链接。

#### 11.4 使用 FetchContent 或 ExternalProject 管理外部依赖

在开发 C++ 项目的过程中，你可能会依赖一些第三方库或者其他项目。为了避免手动下载和配置这些依赖项，CMake 提供了两个很有用的工具：`FetchContent` 和 `ExternalProject`。

通过这两个模块，CMake 可以自动下载、构建并集成外部项目，免去你手动设置复杂依赖的麻烦。接下来，我们简单介绍一下这两种方法。

**1、使用 FetchContent 下载外部库**

`FetchContent` 模块是一个非常方便的工具，它让你能够直接从 GitHub（或者其他地方）下载第三方库，并且让 CMake 自动处理这些库的构建。这个过程是透明的，也就是说，你只需要配置好它，CMake 会自动帮你下载并把它集成到项目中。

举个例子，假设我们要使用 `jsoncpp` 这个库，它是一个 JSON 解析库。我们可以使用 `FetchContent` 来下载并集成它。

**代码示例：**

```makefile
# 引入 FetchContent 模块
include(FetchContent)

# 通过 FetchContent_Declare 告诉 CMake 要下载 jsoncpp 库
FetchContent_Declare(
    jsoncpp                          # 你要下载的库的名字
    GIT_REPOSITORY https://github.com/open-source-parsers/jsoncpp.git   # Git 仓库的地址
    GIT_TAG master                    # 使用最新的版本（master 分支）
)

# FetchContent_MakeAvailable 会自动下载并集成 jsoncpp 到你的项目中
FetchContent_MakeAvailable(jsoncpp)

# 创建一个可执行文件
add_executable(my_app main.cpp)

# 将 jsoncpp 库链接到你的应用程序中
target_link_libraries(my_app JsonCpp::JsonCpp)
```

**解释：**

+ `FetchContent_Declare()`：告诉 CMake 从 GitHub 下载 `jsoncpp` 库，并指定库的版本（`master` 分支）。
+ `FetchContent_MakeAvailable()`：CMake 会自动下载并构建 `jsoncpp`，然后将它集成到项目中。你可以像使用本地库一样使用它。
+ `target_link_libraries()`：将 `jsoncpp` 库链接到你的可执行文件 `my_app` 上。

通过这种方式，`jsoncpp` 库会自动从 GitHub 下载，构建并集成到你的项目中，完全不需要你手动处理库路径或安装。

**2、使用 ExternalProject 管理外部依赖**

`ExternalProject` 模块是 CMake 提供的另一种方式来处理外部依赖。与 `FetchContent` 不同的是，`ExternalProject` 更适合处理那些需要独立构建的外部库。它会启动一个外部进程来构建这些库，而不是直接将它们集成到主项目的构建中。

`ExternalProject` 更适合那些构建过程比较复杂或者需要独立安装的库。

**代码示例：**

```makefile
# 引入 ExternalProject 模块
include(ExternalProject)

# 定义一个外部项目 glog（Google 的日志库）
ExternalProject_Add(
    glog
    GIT_REPOSITORY https://github.com/google/glog.git  # Git 仓库地址
    GIT_TAG master                                   # 使用 master 分支
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/glog  # 安装路径
)

# 确保在构建 main 应用之前，glog 库已经构建完成
add_dependencies(my_app glog)

# 包含 glog 库的头文件和链接库
include_directories(${CMAKE_BINARY_DIR}/glog/include)
link_directories(${CMAKE_BINARY_DIR}/glog/lib)

# 创建应用程序，并将 glog 链接到该应用程序
add_executable(my_app main.cpp)
target_link_libraries(my_app glog)
```

**解释：**

+ `ExternalProject_Add`：定义了一个外部项目 `glog`，CMake 会从 GitHub 下载并构建它。`CMAKE_ARGS` 中的 `-DCMAKE_INSTALL_PREFIX` 参数告诉 CMake 把 `glog` 安装到指定的目录（在这里是构建目录下的 `glog` 文件夹）。
+ `add_dependencies`：这行确保在 `my_app` 构建之前，`glog` 库已经被成功构建。
+ `include_directories` 和 `link_directories`：将 `glog` 库的头文件和库路径添加到你的项目中，以便能够使用 `glog`。
+ `target_link_libraries`：将 `glog` 库链接到 `my_app`。

通过这种方式，`glog` 库会独立构建，并且在构建完成后，你就可以将它链接到你的项目中。

**3、如何选择：**

+ `FetchContent`：如果你希望轻松地集成外部库，并让 CMake 自动处理下载和构建，`FetchContent` 是非常适合的选择。它适用于简单的、可以直接集成的外部依赖库。
+ `ExternalProject`：如果你的外部依赖有独立的构建流程或者需要单独安装，`ExternalProject` 更适合。它可以让你完全控制外部项目的构建过程，但它的使用比 `FetchContent` 更复杂一些。

对于大多数初学者来说，`FetchContent` 是更简单易用的选择，但如果你遇到需要复杂构建流程的库，`ExternalProject` 也不失为一个强大的工具。希望通过这些示例，你能更轻松地管理你的外部依赖！

#### 小结一下：灵活、高效的构建

通过自定义模块、函数以及合理的多项目管理，我们可以大大提升 CMake 构建系统的灵活性和效率。无论是小型项目还是大型复杂项目，CMake 都能够帮你构建一个高效、可扩展的构建系统。

+ 自定义模块和函数可以封装常用的构建逻辑，简化重复配置。
+ `add_subdirectory()` 可以帮助管理多模块的项目，层次清晰、便于维护。
+ `ExternalProject` 和 `FetchContent` 能够处理外部依赖，使构建过程自动化、透明化。

通过掌握这些技巧，你将能够轻松管理多模块、多依赖的项目，构建出高效、可维护的构建系统。

### 总结

这篇文章，我们聚焦了 CMake 的高效构建，主要解决了两个问题：

+ **性能优化**：通过并行构建、增量构建以及缓存机制，学会了如何大幅缩短构建时间，让编译速度飞快。
+ **多模块构建**：掌握了自定义模块和多项目管理的技巧，让复杂的大型项目结构更加清晰、灵活可控。

这些内容让你在处理大型项目时，更得心应手，也让构建效率和项目管理能力更上一层楼。

### 下篇预告

接下来，我们将进入 高级篇：高级技巧与最佳实践，聊聊 CMake 的更多进阶玩法：

+ **生成器表达式**：如何用 CMake 的高级语法实现更灵活的构建配置？
+ **代码生成与自定义命令**：自动化生成代码和文件，让构建流程更智能。
+ **包管理与安装配置**：优雅地处理项目的安装和依赖管理，提升分发效率。
+ **最佳实践与常见问题**：总结实用技巧，帮你避开开发中的各种坑。

这些内容将为你的项目构建流程提供更多灵活性和可扩展性，敬请期待，我们下篇见！

看完是不是感觉 CMake 没那么难了？点个「赞」或者「在看」支持下，也可以 分享 给身边的小伙伴，一起把 CMake 玩明白！

也欢迎大家关注我的公众号「跟着小康学编程」，下一篇将继续带你深入 CMake 的高级篇，别忘了关注！我们下篇见！

#### 怎么关注我的公众号？

点击下方公众号名片即可关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)