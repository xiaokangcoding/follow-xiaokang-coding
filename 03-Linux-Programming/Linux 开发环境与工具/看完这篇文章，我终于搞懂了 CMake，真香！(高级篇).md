大家好，我是小康。

上次介绍的 CMake 性能优化和多项目管理，你搞定了吗？ 没搞定，赶紧去看：

上篇我们聊了如何优化构建速度，解决了模块太多不好管理的问题。是不是感觉“我终于懂点 CMake 了”？

但今天的内容可不止是“懂点”。项目做好了，配置太复杂怎么办？代码生成要写手动脚本？分享项目时依赖全丢了？这些问题如果还没碰到，迟早会找到你。

没关系，这次我们来聊聊 **CMake 的高级用法**：

1、动态配置（生成器表达式），轻松应对复杂需求；

2、自动生成代码，让重复工作交给 CMake；

3、包管理与安装，分享项目变简单；

4、避开那些踩过的“坑”（最佳实践与常见问题）。

学完这些，你会发现，原来 CMake 不只是工具，更是一个全能助手！

**一张思维导图帮你快速了解 CMake 的全貌**：

![](https://files.mdnice.com/user/71186/820508b9-8c2b-433a-9dda-b37c525877af.png)

> **友情提醒**：原创不易，如果觉得内容对你有帮助，别忘了点赞、转发，或者点个「在看」支持一下！非常感谢！🌟


## 高级篇：高级技巧与最佳实践  

### 12.CMake 的生成器表达式：高级构建技巧

生成器表达式，听起来是不是有点“高大上”？其实没那么复杂，简单来说，它就是 CMake 提供的一种动态判断工具。通过它，你可以根据一些条件（比如当前是 Debug 模式还是 Release 模式、目标是静态库还是动态库）来自动调整配置。

换句话说，生成器表达式就是 CMake 的“条件公式”，让你能在构建时灵活地选择编译选项、链接库、文件路径等等。

**它为什么重要？**

在实际项目中，构建需求往往很复杂，比如：

+ Debug 模式需要加调试信息，Release 模式需要开启优化；
+ 不同平台（Windows、Linux）需要链接不同的系统库；
+ 静态库和动态库需要不同的处理逻辑……

如果不用生成器表达式，你可能需要写很多 `if` 判断，代码又长又乱。而有了它，只用一行表达式就能搞定这些逻辑，是不是很香？

**它怎么用？**  

生成器表达式的基本格式是这样的：`$<expression>`。CMake 会在构建时动态解析它，比如：

+ `$<CONFIG:Debug>`：如果当前是 Debug 模式，这个表达式就会被替换成对应的值。
+ `$<TARGET_FILE:my_target>`：动态获取目标 `my_target` 生成的文件路径。

说了这么多，别担心，接下来我们会通过几个简单的例子，一步步教你怎么用它，把复杂的构建需求化繁为简！

#### 例子 1：根据构建类型选择编译选项

假设你有一个项目，在 **Debug 模式** 下需要开启调试信息（比如 `-g`），而在 **Release 模式** 下则需要开启优化（比如 `-O3`）。用生成器表达式，可以很优雅地解决这个问题，而不用手动写一堆 `if` 判断。

**来看代码**：

```makefile
# 给目标 my_target 设置编译选项
target_compile_options(my_target
  PRIVATE
  $<$<CONFIG:Debug>:-g>       # 如果是 Debug 模式，启用调试符号
  $<$<CONFIG:Release>:-O3>    # 如果是 Release 模式，启用优化
)
```

**这里发生了什么？**

1.`my_target` 是什么？
- 这是你的构建目标，比如一个可执行文件（`add_executable(my_target main.cpp)`）或者一个库（`add_library(my_target STATIC mylib.cpp)`）。

2.`target_compile_options()` 是干嘛的？

- 它是用来给某个目标设置编译选项的，比如这里我们根据不同的构建模式，动态添加 `-g` 或 `-O3`。

3.生成器表达式 `$<$<CONFIG:Debug>:-g>` 怎么工作？

+ `$<CONFIG:Debug>`：如果当前构建模式是 Debug，这个表达式就生效，添加 `-g` 选项（调试符号）。
+ `$<CONFIG:Release>`：如果当前构建模式是 Release，该表达式就生效，添加 `-O3` 选项（优化级别 3）。

**为什么用生成器表达式更好？**  

如果不用生成器表达式，你可能需要写成这样：

```makefile
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  target_compile_options(my_target PRIVATE -g)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
  target_compile_options(my_target PRIVATE -O3)
endif()
```

这种写法虽然也能用，但代码会变得又长又啰嗦，逻辑分散。而用生成器表达式，你只需要一行代码，CMake 会根据当前的构建模式（Debug 或 Release）自动选择合适的选项，清晰又高效！

#### 例子 2：在不同平台使用不同的库
假设你在 Windows 上要链接 `ws2_32` 库，而在 Linux 上要链接 `pthread` 库。你可以使用生成器表达式来做平台判断：

```makefile
target_link_libraries(my_target
  PRIVATE
  $<$<TARGET_OS:Windows>:ws2_32>
  $<$<TARGET_OS:Linux>:pthread>
)
```

在这里，`$<$<TARGET_OS:Windows>:ws2_32>` 表示当目标平台是 Windows 时链接 `ws2_32` 库，而 `$<$<TARGET_OS:Linux>:pthread>` 表示当目标平台是 Linux 时链接 `pthread` 库。这种方式可以大大简化跨平台的配置。

#### 例子 3：根据目标类型动态设置链接库
如果你的目标需要链接不同的库（比如静态库和动态库）。你可以使用生成器表达式来根据库的类型选择不同的库。

```makefile
target_link_libraries(my_target
  PRIVATE
  $<$<TARGET_TYPE:STATIC_LIBRARY>:libstatic.a>
  $<$<TARGET_TYPE:SHARED_LIBRARY>:libshared.so>
)
```

- 这里的 `$<$<TARGET_TYPE:STATIC_LIBRARY>:libstatic.a>` 意思是，如果 `my_target` 是静态库，就链接 `libstatic.a`；

- 而 `$<$<TARGET_TYPE:SHARED_LIBRARY>:libshared.so>` 则表示如果是动态库，就链接 `libshared.so`。

- 通过生成器表达式，CMake 会自动根据目标的类型选择正确的库。

#### 例子 4：获取目标文件的路径

假设你需要在构建后复制一个目标的可执行文件或者库文件到某个目录，这时候你可以用 `$<TARGET_FILE>` 表达式来获取目标的文件路径。

```makefile
add_executable(my_app main.cpp)

# 在构建完成后复制可执行文件到 /path/to/output
add_custom_command(TARGET my_app POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:my_app> /path/to/output/
)
```

**解释：**

+ `$<TARGET_FILE:my_app>` 会动态获取目标 `my_app` 的输出文件路径（比如可执行文件 `my_app` 的具体路径）。
+ `add_custom_command` 定义了一个构建后的操作：复制文件到目标目录。

这个用法非常适合需要在构建后对文件进行处理的场景，比如将二进制文件复制到指定目录，或者作为输入给后续的工具使用。

#### 例子 5：根据编译器设置选项

有时候，你的项目需要支持多种编译器，比如 GCC 和 Clang。不同的编译器可能有自己独特的选项，所以我们需要根据使用的编译器，动态设置合适的编译选项。这时候，`$<CXX_COMPILER_ID>` 生成器表达式就能派上用场了！

**看个例子**：

```makefile
target_compile_options(my_target
    PRIVATE
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra>
    $<$<CXX_COMPILER_ID:Clang>:-Weverything>
)
```

**解释一下**：

+ `my_target`：这是你的目标，比如你通过 `add_executable(my_target main.cpp)` 创建的目标。
+ `$<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra>`：如果你用的是 GNU 编译器（比如 GCC），这段表达式就会添加 `-Wall` 和 `-Wextra` 选项，用来启用常见的警告。
+ `$<$<CXX_COMPILER_ID:Clang>:-Weverything>`：如果你用的是 Clang 编译器，这段表达式就会添加 `-Weverything`，它会启用 Clang 的所有警告。

不同编译器的选项并不完全兼容，比如 Clang 支持 `-Weverything`，但 GCC 就不认这个选项。直接写这些选项会导致构建失败。所以我们用生成器表达式来动态判断当前的编译器，选择正确的选项，避免出错。

#### 例子 6：根据操作系统设置路径

如果你的项目需要在不同的操作系统（比如 Windows 和 Linux）上运行，可能会用到不同的路径来查找库或头文件。这时候，你可以用 `$<PLATFORM_ID>` 表达式来自动判断当前的操作系统，并设置对应的路径。

**看个例子**：

```makefile
target_include_directories(my_target
    PRIVATE
    $<$<PLATFORM_ID:Windows>:C:/libs/include>
    $<$<PLATFORM_ID:Linux>:/usr/local/include>
)
```

**解释一下**：

+ `$<$<PLATFORM_ID:Windows>:C:/libs/include>`：如果当前的平台是 Windows，就添加 `C:/libs/include` 作为头文件路径。
+ `$<$<PLATFORM_ID:Linux>:/usr/local/include>`：如果当前的平台是 Linux，就添加 `/usr/local/include` 作为头文件路径。

CMake 会根据你当前的系统环境（Windows 还是 Linux）自动选择合适的路径。这样，你就不用手动写一堆 `if` 判断，配置会更加简洁。

#### 例子 7：设置不同的运行时库

有些项目在 Debug 和 Release 模式下需要使用不同的运行时库，比如在 Debug 模式用带调试符号的库，而在 Release 模式用优化过的库。这种需求很常见，用 `$<CONFIG>` 表达式就能轻松搞定。

**来看个简单的例子**：

```makefile
target_link_libraries(my_target
    PRIVATE
    $<$<CONFIG:Debug>:debug_runtime>
    $<$<CONFIG:Release>:release_runtime>
)
```

**解释一下**：

+ `my_target`：这是你的目标，比如一个可执行文件或者一个库，你通过 `add_executable` 或 `add_library` 创建的目标。
+ `$<$<CONFIG:Debug>:debug_runtime>`：如果当前构建模式是 `Debug`，CMake 会自动链接 `debug_runtime` 库。
+ `$<$<CONFIG:Release>:release_runtime>`：如果当前构建模式是 `Release`，CMake 会链接 `release_runtime` 库。

**有什么用？**

+ 在调试模式下（`Debug`），你可能需要一个包含调试符号的库（比如更容易追踪问题）。
+ 在发布模式下（`Release`），你通常需要一个优化后的库来提高运行性能。

CMake 会根据当前的构建类型自动选择合适的库，无需手动判断，干净又高效。

#### 例子 8：添加条件性的编译定义

有时候，我们的项目需要根据配置来启用或禁用某些功能。比如：

+ 开启一个实验性功能。
+ 启用调试日志。
+ 控制某些代码片段是否参与编译。
+ ...

这类需求可以通过生成器表达式 `$<BOOL>` 来实现，非常方便。

**来看代码**：

```makefile
target_compile_definitions(my_target
    PRIVATE
    $<$<BOOL:${MY_FEATURE_ENABLED}>:FEATURE_ENABLED>
)
```

**代码解读：**

1.`my_target`
+ 这是你的构建目标，比如一个库或可执行文件。

2.`${MY_FEATURE_ENABLED}`
+ 这是一个变量，用来决定某个功能是否启用。
+ 比如，你可以在 `CMakeLists.txt` 里设置它：

```makefile
set(MY_FEATURE_ENABLED ON)  # 启用特性
```

如果是 `ON` 或非空值，表示启用；如果是空或 `OFF`，表示关闭。

3.`$<$<BOOL:${MY_FEATURE_ENABLED}>:FEATURE_ENABLED>`
+ **意思很简单**：
    - 如果 `MY_FEATURE_ENABLED` 是 `ON`，那么 CMake 会添加 `FEATURE_ENABLED` 这个宏到编译器里。
    - 如果 `MY_FEATURE_ENABLED` 是 `OFF` 或空值，CMake 不会添加这个宏。
+ **效果**：动态决定是否定义 `FEATURE_ENABLED` 宏。

**举个例子**：在你的 C++ 代码里，可以这样用：

```c++
#ifdef FEATURE_ENABLED
void enableFeature() {
    // 新功能代码
}
#endif
```

+ **这样做的好处是**：你可以通过配置开关来控制功能是否启用，而不用每次都去代码里手动加或删 `#define`，让你的代码更灵活，管理起来也更方便。


#### 小结

生成器表达式是 CMake 的一大利器，可以帮你根据条件（比如构建类型、平台、目标类型等）动态配置项目。它让配置更灵活，也能避免写很多繁琐的 `if` 判断。

虽然一开始可能有点难懂，但用熟了之后，你会发现它非常方便，能让你的 CMake 文件又简洁又好维护。

### 13. CMake 的代码生成与自定义命令

接着我们刚才讲的生成器表达式，CMake 还提供了一个非常强大的功能——**代码生成与自定义命令**。这部分其实是让你在构建过程中添加一些自定义的操作，比如自动生成代码、运行脚本，或编译之后执行某些操作。CMake 提供了两种非常有用的命令，`add_custom_command` 和 `add_custom_target`，让你能够在构建过程中做一些个性化的操作。

这两个命令看起来有点像，但其实有些不同。我们接着了解一下它们到底有什么区别，适用在哪些场景。

#### 1. `add_custom_command`：自定义构建命令

`add_custom_command` 主要是用来在构建过程中执行某些命令的，通常是生成一些文件或者运行外部程序。比如，你可能需要在编译之前生成一些源代码文件，或者调用外部工具来做点事情。

**怎么用？**

```makefile
add_custom_command(
    OUTPUT output_file          # 生成的文件
    COMMAND some_command        # 执行的命令
    DEPENDS input_file          # 依赖的文件，只有这些文件改动了才会重新执行命令
    COMMENT "Generating files"  # 运行时会显示的提示信息
)
```

+ `OUTPUT` 表示你最终要生成的文件。
+ `COMMAND` 是你要运行的命令，比如执行脚本、调用程序等。
+ `DEPENDS` 是你这个命令依赖的文件，只有这些文件变化了，命令才会被执行。
+ `COMMENT` 是你执行命令时显示的提示，类似日志信息，帮助你知道命令在做什么。

**例子1：**

假设你有一个 Python 脚本 `generate_code.py`，它会根据一些配置文件(比如一个 JSON 文件 `config.json`）生成 `.cpp` 文件。那么你可以用 `add_custom_command` 来运行这个 Python 脚本。

```makefile
add_custom_command(
    OUTPUT generated_code.cpp
    COMMAND python3 generate_code.py config.json generated_code.cpp
    DEPENDS generate_code.py config.json
    COMMENT "Generating C++ code from JSON configuration"
)
```

每次 `generate_code.py` 或 `config.json` 改动时，CMake 就会重新运行这个命令，生成 `generated_code.cpp` 文件。而这个 .cpp 文件在编译时会作为源代码编译进你的项目。

**例子 2：**

你可能会遇到需要在构建完成后做一些额外工作的情况，比如复制某些生成的文件到某个目录。这时候就可以用到 `add_custom_command` 的 `POST_BUILD` 选项。

```makefile
add_executable(my_app main.cpp)

# 在构建完成后，复制生成的可执行文件
add_custom_command(TARGET my_app POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:my_app> /path/to/output/
  COMMENT "Copying executable to output directory"
)
```

这里：

+ 我们通过 `add_custom_command` 指定了一个后置构建命令（`POST_BUILD`）。
+ 它会在 `my_app` 构建完成后，自动将生成的可执行文件复制到 `/path/to/output/`。
+ `$<TARGET_FILE:my_app>` 会自动获取 `my_app` 目标生成的文件路径。

通过这种方式，你可以自动化构建后的文件处理任务，避免手动操作。

#### 2. `add_custom_target`：自定义构建目标

有时，你可能需要在构建过程中做一些额外的工作，比如生成一个文档、清理临时文件、或者执行一些特殊的操作。但是，这些工作并不是用来生成最终的可执行文件或库的。这个时候，你可以使用 `add_custom_target`。

`add_custom_target` 就是用来创建一个“自定义任务”的。它和 add_executable 或 add_library 类似，但不是用来生成可执行文件或库，而是执行一些你需要的额外命令。

**怎么用？**

```makefile
add_custom_target(
    target_name             # 自定义目标的名字
    COMMAND command [args...]  # 你要执行的命令
    DEPENDS dependency_files   # 依赖的文件，确保这些文件变动时目标会执行
    COMMENT "Custom target"    # 显示提示信息
)
```

+ `target_name` 是你定义的目标名字，类似 `my_app` 或 `my_lib`，但是这个目标不一定会生成实际文件。
+ `COMMAND` 是你要执行的命令或脚本。
+ `DEPENDS` 是目标的依赖文件，文件有变化时，CMake 会重新执行目标。
+ `COMMENT` 是显示的提示信息。

**例子1**：

假设你要生成文档、打包文件等任务，可以通过 `add_custom_target` 来创建一个“生成文档”的目标：

```makefile
add_custom_target(
    generate_docs
    COMMAND doxygen Doxyfile
    COMMENT "Generating project documentation"
)
```
然后，你可以设置让文档生成与其他构建任务关联，比如在编译时自动生成文档：

```makefile
add_dependencies(my_project generate_docs)
```

这样每次构建 `my_project` 时，CMake 就会先生成文档，确保文档是最新的。

**例子2**：

例如你想在每次构建时清理掉一些临时文件，但这些文件并不是你的项目的主要输出，这时候你可以用 `add_custom_target` 来做一个清理任务。

```makefile
# 创建一个名为 clean_temp 的自定义目标
add_custom_target(clean_temp
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/temp_folder
)
```

这里，`clean_temp` 这个目标并不会生成什么文件，而是执行一个清理文件夹的任务。你可以在构建项目时或者其他需要的时候调用这个任务。

那么，`add_custom_command` 和 `add_custom_target` 到底有什么不同呢？

+ `add_custom_command` 主要是用来定义 **某个具体的构建步骤**，比如执行一个命令生成文件。它可以跟具体的文件生成和依赖有关，是和生成的文件绑定的。一般来说，你不会直接调用它，而是通过它生成的文件来触发其他操作。
+ `add_custom_target` 更像是定义了一个 **独立的任务**，它是一个目标，可以把多个命令绑定在一起，成为一个执行步骤。你可以把它当作一个特殊的构建目标来执行，像生成文档、清理文件等任务都可以使用它。

**总结起来**：

+ 如果你只是想在构建时运行一个命令生成文件，就用 `add_custom_command`。
+ 如果你需要把多个命令组织成一个大任务（比如自动生成文档、压缩文件、清理缓存等），就用 `add_custom_target`。

#### 小结：

通过 `add_custom_command` 和 `add_custom_target`，你可以在 CMake 构建过程中加入一些额外的操作。这些操作不仅可以用来生成文件，还能执行脚本、复制文件、自动化测试等。简而言之，你可以让 CMake 自动化完成一些重复性的任务，让项目的构建更加智能和高效。  

### 总结：

今天我们聊了两个 CMake 的高级玩法，实战性十足：

+ **生成器表达式**：通过动态条件配置，比如针对不同平台、目标类型或构建模式，轻松选编译选项、链接库。这东西有点像“万能钥匙”，写配置再也不怕复杂场景了！
+ **代码生成与自定义命令**：用 `add_custom_command` 和 `add_custom_target`，把那些枯燥重复的任务丢给 CMake，比如自动生成代码、运行脚本、后置构建操作。构建流程智能化，就是这么爽！

这两个技巧不只是“高级”，更是“好用”。学会它们，你的 CMake 技能直接跃升一个档次，复杂项目配置起来也能稳得住。

### 下篇预告：

感觉 CMake 越来越有趣了吧？别急，下篇内容更实用：

1. **包管理与安装配置**：库写好了，怎么优雅地分发给别人用？教你一步步搞定 `find_package()` 配置，保证你的库用起来又方便又高效。
2. **最佳实践与常见问题**：CMake 的坑，说多不多，说少不少。下篇我们直接总结最常见的那些“雷”，让你轻松避开！

学完下一篇，你的 CMake 技能将覆盖从配置到分发的全流程，还能掌握一套高效的最佳实践，避开那些常见的开发陷阱。别忘了点个 「赞」 或 「在看」 支持一下，也欢迎分享给你身边的小伙伴，一起把 CMake 玩明白！我们下篇见！

#### 怎么关注我的公众号？

点击下方公众号名片即可关注。

![](https://files.mdnice.com/user/48364/65158d3c-cd38-4604-861a-8f0379066dc0.png)

此外，小康最近创建了一个技术交流群，专门用来讨论技术问题和解答读者的疑问。在阅读文章时，如果有不理解的知识点，欢迎大家加入交流群提问。我会尽力为大家解答。期待与大家共同进步！

![](https://files.mdnice.com/user/48364/971ccaa3-8f57-4e33-8bc9-d0863eeade81.png)