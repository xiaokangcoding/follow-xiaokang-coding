大家好，我是小康。

在之前的基础篇中，我们已经学会了用 CMake 管理项目目录和源代码，像搭积木一样将项目轻松组织起来。到这里，你已经迈出了用 CMake 管理项目的第一步。

但实际开发中，需求往往更加复杂：

+ 怎么让配置文件更智能，比如根据不同的条件切换编译选项？
+ 面对第三方库，比如 Boost 或 OpenCV，怎么高效地管理依赖？
+ 如何优化构建流程，轻松切换调试版和发布版？

这些都是更深入使用 CMake 时不可回避的问题。在这一篇，我们将带你从“会用”到“用好”，一起探索 CMake 的更多功能，让你的项目管理更加灵活高效！

## 提升篇：常用功能  

### 5、CMake 高效配置：条件判断、循环与字符串操作  

当项目变复杂时，CMake 需要更聪明地处理各种情况。通过学会条件判断、循环和字符串操作，你可以让 CMake 变得更灵活、高效！  

#### 5.1 条件判断

CMake 就像一个聪明的决策者，能够根据不同的条件做出选择。比如，假设你正在编译一个跨平台的项目，Windows 下需要链接不同的库，macOS 下还需要额外配置一些特性，Linux 下则不需要。CMake 通过条件判断可以轻松解决这些问题。

举个例子，假设我们需要在不同平台上使用不同的库：

```makefile
if(WIN32)
    # Windows 上使用特定的库
    set(MY_LIBRARY "C:/libs/windows_lib.lib")
    # 打印调试信息
    message("Using library for Windows")
elseif(APPLE)
    # macOS 上使用其他库
    set(MY_LIBRARY "/usr/local/lib/mac_lib.dylib")
    message("Using library for macOS")
else()
    # Linux 上用另一个库
    set(MY_LIBRARY "/usr/lib/linux_lib.so")
    message("Using library for Linux")
endif()

# 最终链接库
target_link_libraries(my_project ${MY_LIBRARY})
```

**解释**：

+ **Windows**：如果是在 Windows 上，CMake 会设置 `MY_LIBRARY` 为 Windows 特有的库路径。
+ **macOS**：如果是在 macOS 上，CMake 会设置为 macOS 版本的库。
+ **Linux**：如果是 Linux，CMake 会选择 Linux 上的库。

通过这种方式，无论你在哪个操作系统上，CMake 都能自动选择正确的库，而不需要你每次都去手动更改代码。 

#### 5.2 循环：`foreach` 和 `while`

1、`foreach` 循环：

`foreach` 循环特别适合用来遍历一个列表。比如，假设你有一堆源文件，想要对每个文件做一些操作，比如打印文件名，或者设置编译选项。`foreach` 就是帮你自动遍历这些文件，逐个处理。

**举个例子**：

```makefile
set(SOURCES main.cpp utils.cpp helper.cpp)

foreach(SRC ${SOURCES})
    message("Compiling source file: ${SRC}")
    # 这里可以做更多操作，比如添加每个源文件到目标中
    target_sources(hello PRIVATE ${SRC})
endforeach()
```

**怎么理解**：

+ 这里，`foreach` 就是从 `SOURCES` 里一项一项地取出文件名，每取出一个文件，就执行一次 `message()`，把文件名打印出来。
+ 你还可以在循环中做其他操作，比如给每个文件设置编译选项，或者把这些文件都加到目标中去。

**什么时候用**：

+ 当你有一堆文件、选项或其他内容需要一 一处理时，`foreach` 就非常有用。

2、`while` 循环：

`while` 循环则是根据一个条件反复执行，直到条件不再满足为止。简单来说，就是你设置一个“门槛”，每次检查这个条件，直到它满足了为止。就像一个不停跑步的机器，直到到达设定的终点。

比如，假设你想从数字 1 开始，逐步增加，直到数字达到 5 为止：

```makefile
set(COUNT 1)

while(COUNT LESS 5)
    message("Current count: ${COUNT}")
    math(EXPR COUNT "${COUNT} + 1")
endwhile()
```

**怎么理解？**

+ 这个例子里，`while` 会先检查 `COUNT` 是否小于 5，如果小于，就打印出来，然后让 `COUNT` 增加 1。然后继续检查，直到 `COUNT` 达到 5 时停止。
+ 就是不断循环，直到你满足了某个条件（这里是 `COUNT` 小于 5）。

**什么时候用？**

+ 当你需要根据某些条件反复做事情，直到条件不再满足时，`while` 就很有用了。比如动态配置选项、批量生成文件、自动调整设置等。

**小结一下**：

+ `foreach`：用来处理一组东西（文件、库、选项等），逐个操作，就像你在一个清单上逐项勾选。
+ `while`：根据条件反复执行，直到达到某个标准，就像你反复跑步，直到终点到达。

这两个循环能帮助你批量处理任务，避免重复的手动操作，让 CMake 配置变得更灵活，也更高效。

#### 5.3 字符串操作：拼接、分割、查找

**1、字符串拼接（STRING(CONCAT ...)）**

拼接就是把两个或多个字符串合成一个大字符串。就像你把两段话拼起来，或者把文件名和路径组合成一个完整的文件路径。

实战例子：

假设你有文件夹路径 `/home/user/project/` 和文件名 `main.cpp`，你想拼接成一个完整的文件路径。

```makefile
set(DIR "/home/user/project/")
set(FILE_NAME "main.cpp")

# 拼接字符串
set(FULL_PATH "${DIR}${FILE_NAME}")

message("Full path: ${FULL_PATH}")
```

**解释**：

+ 我们把路径和文件名拼接起来，最终形成 `/home/user/project/main.cpp`。
+ 这里用到的拼接方法就是将两个字符串用 `${}` 插入到一起，CMake 会自动拼接它们。

**2、字符串分割（STRING(SUBSTRING ...)）**

有时候，我们需要从一个长字符串中提取某一部分。这就像你从长长的句子中挑出关键词。

实战例子：

假设你有一个文件路径 `/home/user/project/main.cpp`，你想提取出文件名 `main.cpp`。

```makefile
set(PATH "/home/user/project/main.cpp")

# 获取文件名
string(REGEX REPLACE "^.*/" "" FILE_NAME ${PATH})
# ^ 是一个锚点，表示匹配字符串的开始位置。 
# .* 会匹配任意数量的字符，甚至是零个字符。它会一直匹配，直到遇到我们指定的下一个字符(/)。

message("File name: ${FILE_NAME}")
```

**解释**：

+ 这里用了 `string(REGEX REPLACE ...)` 来删除路径中的所有内容，留下文件名部分。
+ 正则表达式 `^.*\/` 的意思是“匹配从头到最后一个 `/` 之前的所有字符”。
+ REPLACE "^.*/" "" 表示将匹配到的字符串（路径部分）替换成空字符串（""）。换句话说，它会删除路径中的所有内容，只保留文件名部分。
+ 这样就能轻松获取路径中的文件名 `main.cpp`。

**3、字符串查找（STRING(FIND ...)）**

有时候，你需要查找字符串中的某个子串。比如，检查路径中是否包含某个特定的文件夹名，或者确认某个构建选项是否已经设置。

实战例子：

假设你有路径 `/home/user/project/`，你想查找这个路径中是否包含 `user`。

```makefile
set(PATH "/home/user/project/")

# 查找字符串中的子串
string(FIND ${PATH} "user" POSITION)

# 如果找到了，POSITION 会返回位置，找不到则返回 -1
if(POSITION GREATER -1)
    message("Found 'user' in the path at position: ${POSITION}")
else()
    message("'user' not found in the path.")
endif()
```

**解释**：

+ `string(FIND ...)` 会返回子串在字符串中的位置，如果找不到会返回 `-1`。
+ 在这个例子中，CMake 会查找路径中 `user` 出现的位置，返回位置 `5`，表示从第 6 个字符开始（因为字符串从 0 开始计数）。
+ 你可以根据这个返回值来判断是否找到了某个子字符串，进而执行相应的操作。

小结一下：

+ **拼接**：你可以用 `set()` 和 `${}` 把多个字符串组合起来，形成一个新的字符串。
+ **分割**：通过正则表达式，可以从一个长字符串中提取你需要的部分，像从文件路径中提取文件名。
+ **查找**：用 `string(FIND ...)` 查找子字符串的位置，帮助你确定某个内容是否存在。

这些字符串操作非常有用，尤其在处理路径、文件名或动态生成构建选项时，能帮你省下不少麻烦。通过这些简单的操作，你可以让 CMake 的配置变得更灵活和智能！

### 6、CMake的依赖管理

随着项目规模的增长，难免会用到一些外部依赖库，比如 Boost、SQLite 或 OpenCV。这些库的引入不仅能节省开发时间，还能让功能更强大。但是，管理这些依赖可不简单：路径怎么设置？库文件怎么链接？尤其是在跨平台开发时，这些问题更是让人头大。

幸运的是，CMake 为我们提供了非常强大的工具来搞定这些依赖管理。无论是查找现有的第三方库，还是管理自定义路径中的特殊库，CMake 都能帮我们自动化这些繁琐的工作。

#### 6.1 查找和使用外部库

假设你正在开发一个项目，突然需要用到一个外部库，比如 Boost，SQLite 或者 OpenCV。手动设置这些库的路径、头文件、库文件的链接，工作量很大，尤其是在跨平台开发时。幸好，CMake 提供了 `find_package()` 命令，可以自动帮你找到这些库，并且配置好相关路径。

**比如，我们要使用 Boost 库，你可以这样做**：

```makefile
# 查找 Boost 库，要求版本至少是 1.70(版本>=1.70)
find_package(Boost 1.70 REQUIRED)
```

这行代码的意思是告诉 CMake，“帮我找一下 Boost 库，至少是版本 1.70，如果找不到就报错”。如果 CMake 成功找到了 Boost，接下来，它会自动设置一些变量，例如：

+ `Boost_INCLUDE_DIRS`：Boost 的头文件路径
+ `Boost_LIBRARIES`：Boost 的库文件路径

然后，你就可以将这些信息传给 CMake，让它处理编译过程。举个例子，如果你有一个目标 `my_target`（比如你要编译的可执行文件或库），你可以这样告诉 CMake，`my_target` 需要用到 Boost：

```makefile
# 告诉 CMake，my_target 需要使用 Boost 的头文件
target_include_directories(my_target PRIVATE ${Boost_INCLUDE_DIRS})

# 告诉 CMake，my_target 需要链接 Boost 库
target_link_libraries(my_target PRIVATE ${Boost_LIBRARIES})
```

**这段代码的意思是**：

+ `target_include_directories()`：把 Boost 的头文件路径加到编译器的搜索路径中，确保编译时能够找到 Boost 的头文件。
+ `target_link_libraries()`：把 Boost 的库文件链接到 `my_target`，确保生成的程序能正确链接到 Boost 库。

简单来说，你只需要告诉 CMake 哪些库要用，CMake 会自动帮你处理好所有的路径和链接问题。

#### 6.2 自定义路径依赖库的查找
在实际开发中，你可能会遇到这样的情况：某些库并没有安装在系统默认的路径（比如 `/usr/lib` 或 `/usr/local/lib`），导致 `find_package()` 无法找到它们。这时候，就需要告诉 CMake 应该去哪里找这些库。

**find_package() 的默认查找顺序：**

在运行 `find_package()` 时，CMake 会按照以下顺序依次查找库：

1、用户指定的提示路径

+ 如果在 `find_package()` 中使用了 `HINTS` 或 `PATHS` 参数，这些路径会被优先考虑。

2、全局配置的路径

CMake 会优先查找全局指定的路径：

+ `CMAKE_PREFIX_PATH`：这是一个全局变量，可以通过命令行或 CMakeLists.txt 配置。
+ 库支持的特定环境变量。例如：
    - Boost 支持 `BOOST_ROOT` 环境变量。
    - OpenCV 支持 `OpenCV_DIR` 环境变量。

这些全局配置的路径是解决自定义路径问题的常见方法。

3、库自带的配置文件路径

如果库是使用 CMake 构建的，通常会自带 `<PackageName>Config.cmake` 或 `<lowercasename>-config.cmake` 文件。这些配置文件的路径可能位于：

+ 库安装路径下的 `lib/cmake/<PackageName>/`。
+ 用户定义的安装目录。

CMake 会根据这些配置文件提供的信息定位头文件和库文件。

4、标准系统路径

如果以上路径都找不到，CMake 会查找系统默认的库路径：

+ **Linux**：`/usr/lib`, `/usr/local/lib`。
+ **macOS**：`/opt/local/lib`, `/usr/local/lib`。
+ **Windows**：`C:/Program Files/`, `C:/Program Files (x86)/`。

这些路径是编译器和链接器的默认搜索路径，通常用于全局安装的库。



如果上述路径中都找不到符合要求的库，CMake 会报错，停止配置过程。

**那如何解决呢？**

告诉 CMake 自定义路径即可：

当库不在默认路径时，可以通过以下几种方法告诉 CMake 自定义路径，从而让 `find_package()` 正常工作：

**1、使用 HINTS 或 PATHS 提供路径提示**

除了 `CMAKE_PREFIX_PATH`，你还可以通过 `HINTS` 或 `PATHS` 参数告诉 `find_package()` 去哪里找库，适合单个库需要自定义路径的场景。

+ `HINTS`：路径提示

`HINTS` 为 CMake 提供额外的查找路径，优先级高于系统默认路径。例如：

```makefile
find_package(MyLib REQUIRED HINTS /custom/path/to/mylib)
```

CMake 会先查找标准路径，再根据 `HINTS` 提供的路径寻找 `MyLib`。

+ `PATHS`：明确路径

`PATHS` 用于指定明确的路径，CMake 会优先在这些路径中查找。例如：

```makefile
find_package(MyLib REQUIRED PATHS /custom/path/to/mylib)
```

`PATHS` 优先级更高，适合需要严格指定路径的场景。

**示例：**

```makefile
find_package(MyLib REQUIRED HINTS /custom/path/to/mylib)
# 或
find_package(MyLib REQUIRED PATHS /custom/path/to/mylib)

if (MyLib_FOUND) # MyLib_FOUND 为 TRUE：表示成功找到 MyLib
    target_include_directories(my_target PRIVATE ${MyLib_INCLUDE_DIRS})
    target_link_libraries(my_target PRIVATE ${MyLib_LIBRARIES})
else()
    message(FATAL_ERROR "MyLib not found!")
endif()
```

+ `HINTS`：路径提示，不改变默认搜索逻辑。
+ `PATHS`：直接覆盖默认路径查找，优先级更高。

**2、设置 CMAKE_PREFIX_PATH 变量：**

`CMAKE_PREFIX_PATH` 是 CMake 的一个全局变量，它告诉 CMake 在标准路径之外，还要去哪些地方查找依赖库。你只需要把库所在的路径加到这个变量里，`find_package()` 就能顺利找到它。 

可以通过以下两种方式设置：  

+ **通过命令行指定路径**

假设你有一个库 `MyLib`，安装在路径 `/custom/path/to/mylib`，可以在运行 CMake 的时候设置 `CMAKE_PREFIX_PATH`：

```bash
cmake .. -DCMAKE_PREFIX_PATH=/custom/path/to/mylib
```

这样，CMake 会在 `/custom/path/to/mylib` 以及它的子目录中查找 `MyLib`，包括头文件和库文件。

+ **在 CMake 脚本中设置路径**

如果你不想每次运行命令都指定路径，也可以直接在 CMake 脚本里设置：

```makefile
set(CMAKE_PREFIX_PATH "/custom/path/to/mylib")
find_package(MyLib REQUIRED)
```

通过这种方式，`find_package()` 会自动在 `/custom/path/to/mylib` 下查找 `MyLib`。

这里顺便提下 `find_library()` 和 `find_path()`:

有时候，`find_package()` 并不适用，比如：

+ 库没有提供 `Config.cmake` 文件，CMake 不知道该怎么找。
+ 你只想简单地指定库文件或头文件的位置，而不需要完整的依赖管理。

这时候，`find_library()` 和 `find_path()` 就派上用场了。这两个命令可以帮你分别找到库文件和头文件。

用 `find_library()` 查找库文件:

`find_library()` 是用来找库文件的，比如 `.so`（Linux）、`.a`（静态库）或 `.lib`（Windows）。

例子：假设库文件 `mylib` 在 `/custom/path/to/lib`：

```makefile
find_library(MYLIB_PATH NAMES mylib PATHS /custom/path/to/lib)
```

+ `NAMES`：要找的库名，CMake 会自动匹配后缀（如 `.so` 或 `.lib`）。
+ `PATHS`：指定查找路径。

如果找到，`MYLIB_PATH` 会保存库的完整路径，比如 `/custom/path/to/lib/libmylib.so`。

用 `find_path()` 查找头文件:

`find_path()` 是用来找头文件的，比如 `.h` 文件。

例子：假设头文件 `mylib.h` 在 `/custom/path/to/include`：

```makefile
find_path(MYLIB_INCLUDE_PATH NAMES mylib.h PATHS /custom/path/to/include)
```

+ `NAMES`：头文件的名字。
+ `PATHS`：指定查找路径。

如果找到，`MYLIB_INCLUDE_PATH` 会保存头文件所在的目录，比如 `/custom/path/to/include`。

配置到目标：

找到库和头文件后，把它们加到目标的编译和链接过程中：

```makefile
target_include_directories(my_target PRIVATE ${MYLIB_INCLUDE_PATH})
target_link_libraries(my_target PRIVATE ${MYLIB_PATH})
```

**完整示例：**

```makefile
# 找库和头文件
find_library(MYLIB_PATH NAMES mylib PATHS /custom/path/to/lib)
find_path(MYLIB_INCLUDE_PATH NAMES mylib.h PATHS /custom/path/to/include)

# 检查是否找到
if (MYLIB_PATH AND MYLIB_INCLUDE_PATH)
    target_include_directories(my_target PRIVATE ${MYLIB_INCLUDE_PATH})
    target_link_libraries(my_target PRIVATE ${MYLIB_PATH})
else()
    message(FATAL_ERROR "Could not find mylib or its headers!")
endif()
```

**小结：**

+ `find_library()`：找库文件。
+ `find_path()`：找头文件。
+ 用这两个命令可以简单指定路径，灵活处理没有配置文件的库。简单直接，很实用！

#### 6.4 使用包管理器：vcpkg 和 Conan

我们刚刚讲了怎么用 `find_package()` 找库，以及怎么设置自定义路径解决那些“找不到库”的问题。这个方法虽然好用，但总归还是有点麻烦——比如你得自己下载库，自己安装，还得知道库的路径在哪。对于小项目来说还行，但如果你的项目依赖越来越多，就会感觉有点头大了。

想象一下：

+ 不同平台下，库的安装路径不一样，每次都要重新配置。
+ 某些库还得去官网下载、解压、编译，耗时又费力。
+ 团队里的每个人可能用的都是不同的环境，调试起来更是让人崩溃。

这时候，我们可以用一个更加省事的方法——**包管理器**！它能帮你自动下载、安装和配置好依赖库，甚至还能帮你处理那些复杂的版本兼容问题。接下来，我们就来聊聊两个特别流行的包管理器——**vcpkg** 和 **Conan**，看看它们是怎么帮我们轻松搞定这些烦人的事儿的。

**1、vcpkg**

首先，vcpkg 是一个非常流行的跨平台包管理器，它可以帮你轻松安装和管理各种 C++ 库。假设你想在项目中使用 OpenCV，你只需要运行一个命令：

```bash
# 安装 OpenCV 库
vcpkg install opencv
```

安装完成后，接下来就只需要告诉 CMake 去哪里找到这些库了。你只需要设置 vcpkg 的工具链文件路径，然后像平常一样用 `find_package()` 来查找库：

```makefile
# 设置 vcpkg 的工具链文件
set(CMAKE_TOOLCHAIN_FILE "path_to_vcpkg/scripts/buildsystems/vcpkg.cmake")

# 查找 OpenCV 库
find_package(OpenCV REQUIRED)

# 链接到你的目标
target_link_libraries(my_target PRIVATE ${OpenCV_LIBS})
```

这样，CMake 就会自动帮你找到 OpenCV，并且配置好相关的路径。你不再需要手动去处理每个库的安装和配置，简直是省了不少事。

**2、Conan**

除了 vcpkg，Conan 也是一个非常流行的 C++ 包管理器。它和 vcpkg 类似，也可以帮助你自动管理外部依赖，不用再手动下载、配置路径。

**安装 Conan 也很简单**，只需要确保系统里有 Python 环境（3.6 或更高版本），然后运行以下命令：

```bash
pip install conan
```

安装完成后，用以下命令验证是否成功：

```bash
conan --version
```

配置好后，你就可以用 Conan 来安装依赖库了。比如，你的项目需要使用 Boost，可以通过以下方式快速安装并集成到 CMake 中：

```bash
# 安装 Boost
conan install boost/1.70.0@
```

安装完成后，在 CMake 文件中，你只需要包含 Conan 的工具链文件，并告诉 CMake 使用 Conan 管理依赖：

```makefile
# 设置 Conan 的工具链文件：加载由 Conan 生成的 conanbuildinfo.cmake 文件。
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
# 应用加载的 conanbuildinfo.cmake 文件中的配置信息，设置项目的依赖环境。
conan_basic_setup()

# 使用 Boost 库
target_link_libraries(my_target PRIVATE Boost::Boost)
```

通过这种方式，CMake 会自动找到 Boost 库并链接到你的项目中。你不需要去手动下载、配置和处理路径，Conan 会为你搞定所有这些麻烦事。

#### 6.5 使用 Git 子模块来管理依赖

有时候，你的外部依赖库是通过 Git 仓库管理的，尤其是一些开源库。CMake 能够与 Git 完美配合，直接把 Git 仓库作为子模块添加到项目中。比如，你想要引入一个 Git 仓库中的库，可以按照以下步骤：

```bash
# 添加 Git 子模块
git submodule add https://github.com/boostorg/boost.git external/boost
git submodule update --init
```

然后，在 CMake 中通过 `add_subdirectory()` 引入该子模块：

```makefile
# 引入 Git 子模块
add_subdirectory(external/boost)

# 使用 boost 库
target_link_libraries(my_target PRIVATE boost)
```

这样，每次你构建项目时，CMake 会自动下载并构建这个外部依赖库，完全自动化，免去了手动管理的烦恼。

### 7. 构建类型与调试、优化配置

为了不让篇幅过长，我将构建类型与调试、优化配置这部分的讲解放在下篇文章：cmake 进阶篇—专业构建

#### 小结一下：

CMake 的依赖管理功能真的很强大，能让我们轻松处理外部库和依赖。你可以用 `find_package()` 查找常用的库，或者用像 vcpkg 和 Conan 这样的包管理器自动安装和配置库，甚至还能通过 Git 子模块来管理库。总之，CMake 简化了整个过程。

通过这些工具，我们能省下不少时间，把精力集中在项目开发上，而不必为外部库的安装和配置烦恼。这样，不仅开发更高效，项目也更容易维护和扩展。

### 总结

到这里，我们已经搞定了 CMake 的常用功能，项目管理也变得更加顺手了：

+ **高效配置**：学会了用条件判断、循环和字符串操作，让 CMakeLists.txt 写起来更灵活、更聪明。
+ **依赖管理**：不管是通过 `find_package()`、包管理器，还是 Git 子模块，都能轻松应对外部库的集成问题。

这些技巧够你应付大部分日常开发了！不过，CMake 的玩法可不止于此，下篇文章我们就要进入更高阶的内容了。敬请期待！

### 下篇预告：进阶篇——专业构建

如果说之前的内容是让你用好 CMake，那么接下来的内容就是带你玩转 CMake！我们会聊一聊更专业的构建技巧：

+ **跨平台构建**：怎么写一次配置文件就能在不同平台编译通过？
+ **测试与调试**：用 CMake 集成测试框架和调试工具，帮你提升代码质量。
+ **性能优化**：构建速度慢？我们来教你如何加速！
+ **模块化构建**：面对复杂的大型项目，怎么用 CMake 管理多个子项目和模块？

这些技能，能让你在更复杂的场景下也能得心应手。别着急，下一篇文章带你逐个击破！

### 最后:

如果觉得这篇文章还不错，别忘了点个「赞」、点个「在看」，或分享给更多对 cmake 感兴趣的小伙伴！

也欢迎大家关注我的公众号「跟着小康学编程」，下一篇，我们接着为大家介绍 CMake 进阶篇——专业构建。

#### 怎么关注我的公众号？


非常简单！扫描下方二维码即可一键关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)