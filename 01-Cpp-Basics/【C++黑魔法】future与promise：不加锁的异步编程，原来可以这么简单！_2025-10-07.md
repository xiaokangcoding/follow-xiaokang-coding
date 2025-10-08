大家好，我是小康。

你是否曾经为了让程序同时做多件事而绞尽脑汁？是否被多线程编程的各种锁、条件变量搞得头昏脑胀？今天，我要告诉你一个秘密武器，让你轻松驾驭异步编程的海洋！

## 前言：为什么要学future和promise？

朋友，想象一下这个场景：你在餐厅点了一份需要20分钟才能做好的复杂菜品。你有两个选择：

1. 坐在那里盯着厨房门口，等待20分钟（同步等待）
2. 服务员给了你个取餐码，菜品好了会通知你，同时你可以刷刷手机或聊聊天（异步等待）

显然，第二种方式更高效，对吧？

在C++编程中，`future`和`promise`就像是这个"取餐码+通知"系统，让你的程序能够优雅地处理异步任务。它们是C++11引入的现代并发编程工具，比传统的线程、互斥锁和条件变量更加简单易用。

> 💡 学习建议：
>
>先理解异步思想，再写代码：别一上来就开线程，先搞清楚同步 vs 异步的区别。
>
>用小例子实验：比如用 promise 模拟点餐，用 future 获取结果，感受“非阻塞”的快感。
>
>配合调试工具，观察线程执行顺序与 future 的状态变化，加深理解。
>
>逐步过渡到更复杂场景：比如线程池、任务分发，future 是这些高级玩法的基石。
>
>理解机制比背 API 更重要：真正掌握了它的模型，你写异步代码就像喝水一样自然。
>
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

## 一、异步任务是个啥？通俗地说就是"后台运行"

在解释`future`和`promise`之前，我们先聊聊什么是异步任务。

**异步任务**就是指那些可以在"后台"执行，不需要主线程等待的任务。比如：

+ 下载一个大文件
+ 复杂计算（如图像处理）
+ 访问远程服务器

想象一下你的电脑在下载游戏的同时，你还能继续刷视频、聊天，这就是异步的魅力！

## 二、future：未来会得到的结果
`future`可以理解为"未来的结果"，它就像一张电影票根：

+ 你现在拿着票根（future）
+ 电影（异步任务）正在后台准备中
+ 当电影准备好了，你可以用票根进场（获取结果）

用代码说话：

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute_answer() {
    // 假装这是个复杂计算
    std::cout << "开始计算终极问题的答案..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟耗时操作
    std::cout << "计算完成！" << std::endl;
    return 42; // 返回结果
}

int main() {
    // 启动异步任务，立即返回一个future
    std::cout << "主线程：启动一个耗时任务" << std::endl;
    std::future<int> answer_future = std::async(compute_answer);

    std::cout << "主线程：哇，不用等待，我可以继续做其他事情！" << std::endl;

    // 做一些其他工作...
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "主线程：我在异步任务计算的同时做了些其他事" << std::endl;

    // 当需要结果时，我们可以获取它
    // 如果结果还没准备好，这会阻塞直到结果可用
    std::cout << "主线程：好了，现在我需要知道答案了，等待结果..." << std::endl;
    int answer = answer_future.get();

    std::cout << "终极答案是：" << answer << std::endl;

    return 0;
}
```

**输出结果**：

```plain
主线程：启动一个耗时任务
开始计算终极问题的答案...
主线程：哇，不用等待，我可以继续做其他事情！
主线程：我在异步任务计算的同时做了些其他事
计算完成！
主线程：好了，现在我需要知道答案了，等待结果...
终极答案是：42
```

看到了吗？主线程启动了计算，但并不立即等待结果，而是继续执行其他代码。只有当真正需要结果时（调用`get()`），才会等待异步任务完成。

## 三、promise：我保证会给你结果
如果说`future`是领取结果的凭证，那么`promise`就是一个承诺："我保证会在某个时刻设置一个值"。它们是一对好搭档：

+ `promise`负责在某个时刻设置结果
+ `future`负责在需要时获取结果

这就像你和朋友的约定：

+ 你：我承诺会告诉你考试成绩（promise）
+ 朋友：我会等你告诉我（future）

来看个例子：

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

void producer(std::promise<int> my_promise) {
    std::cout << "生产者：我要开始生产一个重要的值了..." << std::endl;

    // 假装我们在做一些复杂的计算
    std::this_thread::sleep_for(std::chrono::seconds(2));

    int result = 42;
    std::cout << "生产者：计算完成，设置结果到promise" << std::endl;

    // 设置promise的值，这会通知相关的future
    my_promise.set_value(result);
}

int main() {
    // 创建一个promise
    std::promise<int> answer_promise;

    // 从promise获取一个future
    std::future<int> answer_future = answer_promise.get_future();

    // 启动一个线程，传入promise
    std::cout << "主线程：启动生产者线程" << std::endl;
    std::thread producer_thread(producer, std::move(answer_promise));

    // 主线程继续做其他事情
    std::cout << "主线程：我可以做自己的事，不用等待..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));

    // 当需要结果时
    std::cout << "主线程：现在我需要结果了，等待future..." << std::endl;
    int answer = answer_future.get();
    std::cout << "主线程：收到结果：" << answer << std::endl;

    // 别忘了等待线程结束
    producer_thread.join();

    return 0;
}
```

**输出结果**：

```plain
主线程：启动生产者线程
生产者：我要开始生产一个重要的值了...
主线程：我可以做自己的事，不用等待...
主线程：现在我需要结果了，等待future...
生产者：计算完成，设置结果到promise
主线程：收到结果：42
```

这个例子展示了如何使用`promise`和`future`在线程间传递结果。生产者线程通过`promise`设置值，主线程通过`future`获取值。

## 四、future的几种获取方式
除了通过`promise`获取`future`，C++11还提供了其他便捷方式：

### 1. 通过async获取future
`std::async`是最简单的方式，它自动创建线程并返回`future`：

```cpp
std::future<int> result = std::async([]() {
    return 42;
});
```

### 2. 通过packaged_task获取future
`std::packaged_task`包装了一个可调用对象，并允许你获取其`future`：

```cpp
#include <iostream>
#include <future>
#include <thread>

int main() {
    // 创建一个packaged_task，包装一个lambda函数
    std::packaged_task<int(int, int)> task([](int a, int b) {
        std::cout << "计算 " << a << " + " << b << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟耗时计算
        return a + b;
    });

    // 获取future
    std::future<int> result = task.get_future();

    // 在新线程中执行任务
    std::thread task_thread(std::move(task), 10, 32);

    // 主线程做其他事情...
    std::cout << "主线程：等待计算结果..." << std::endl;

    // 获取结果
    int sum = result.get();
    std::cout << "结果是：" << sum << std::endl;

    task_thread.join();
    return 0;
}
```

**输出结果**：

```plain
主线程：等待计算结果...
计算 10 + 32
结果是：42
```

## 五、实用功能：future的超时等待
有时候，我们不想无限期地等待异步任务。`future`提供了带超时的等待功能：

```cpp
#include <iostream>
#include <future>
#include <chrono>

int long_calculation() {
    std::this_thread::sleep_for(std::chrono::seconds(3));
    return 42;
}

int main() {
    auto future = std::async(std::launch::async, long_calculation);

    // 设置1秒超时
    auto status = future.wait_for(std::chrono::seconds(1));

    if (status == std::future_status::ready) {
        std::cout << "任务已完成，结果是：" << future.get() << std::endl;
    } else if (status == std::future_status::timeout) {
        std::cout << "等待超时！任务还没完成" << std::endl;

        // 我们仍然可以继续等待完成
        std::cout << "继续等待..." << std::endl;
        std::cout << "最终结果：" << future.get() << std::endl;
    }

    return 0;
}
```

**输出结果**：

```plain
等待超时！任务还没完成
继续等待...
最终结果：42
```

## 六、异常处理：当异步任务出错时
异步任务中的异常会被捕获并存储在`future`中，当你调用`get()`时会重新抛出：

```cpp
#include <iostream>
#include <future>
#include <stdexcept>

int may_throw() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    throw std::runtime_error("哎呀，出错了！");
    return 42;
}

int main() {
    auto future = std::async(may_throw);

    try {
        int result = future.get();
        std::cout << "结果：" << result << std::endl;
    } catch (const std::exception& e) {
        std::cout << "捕获到异常：" << e.what() << std::endl;
    }

    return 0;
}
```

**输出结果**：

```plain
捕获到异常：哎呀，出错了！
```

这种设计非常优雅——无论异步任务是成功返回值还是抛出异常，都能通过同一个`future`接口处理。

## 七、实际应用案例：并行计算求和
让我们用一个更实用的例子来巩固理解：并行计算大数组的和。

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <future>
#include <chrono>

// 计算数组部分和的函数
long long partial_sum(const std::vector<int>& data, size_t start, size_t end) {
    return std::accumulate(data.begin() + start, data.begin() + end, 0LL);
}

int main() {
    // 创建一个大数组
    const size_t size = 100000000; // 1亿个元素
    std::vector<int> data(size, 1); // 全是1的数组

    auto start_time = std::chrono::high_resolution_clock::now();

    // 单线程计算
    long long single_result = std::accumulate(data.begin(), data.end(), 0LL);

    auto single_end = std::chrono::high_resolution_clock::now();
    auto single_duration = std::chrono::duration_cast<std::chrono::milliseconds>(single_end - start_time);

    std::cout << "单线程结果: " << single_result << " (耗时: " 
        << single_duration.count() << "ms)" << std::endl;

    // 使用4个线程并行计算
    auto multi_start = std::chrono::high_resolution_clock::now();

    const size_t num_threads = 4;
    const size_t block_size = size / num_threads;

    std::vector<std::future<long long>> futures;

    for (size_t i = 0; i < num_threads; ++i) {
        size_t start = i * block_size;
        size_t end = (i == num_threads - 1) ? size : (i + 1) * block_size;

        // 启动异步任务
        futures.push_back(std::async(std::launch::async, 
            partial_sum, std::ref(data), start, end));
    }

    // 收集结果
    long long multi_result = 0;
    for (auto& f : futures) {
        multi_result += f.get();
    }

    auto multi_end = std::chrono::high_resolution_clock::now();
    auto multi_duration = std::chrono::duration_cast<std::chrono::milliseconds>(multi_end - multi_start);

    std::cout << "多线程结果: " << multi_result << " (耗时: " 
        << multi_duration.count() << "ms)" << std::endl;

    std::cout << "加速比: " << static_cast<double>(single_duration.count()) / multi_duration.count() << "x" << std::endl;

    return 0;
}
```

**可能的输出结果**（取决于你的硬件）：

```plain
单线程结果: 100000000 (耗时: 570ms)
多线程结果: 100000000 (耗时: 171ms)
加速比: 3.33333x
```

看到没？多线程版本明显更快！这正是`future`的价值所在——让并行编程变得简单而高效。

## 八、总结：为什么future和promise这么香？
现在，你已经了解了C++11中`future`和`promise`的基本用法。它们的优势在于：

1. **简化异步编程**：比直接管理线程、互斥锁和条件变量简单得多
2. **清晰的所有权模型**：`promise`负责生产值，`future`负责消费值
3. **异常传递**：异步任务中的异常会自动传递给等待的`future`
4. **超时控制**：可以设置等待超时，避免无限阻塞
5. **与现代C++完美融合**：配合lambda、智能指针等现代特性使用更加优雅

记住这个类比就行：`promise`就像一个"承诺给你结果的人"，`future`就像"等待结果的凭证"。

下次当你需要在程序中执行耗时操作又不想阻塞主线程时，就想到`future`和`promise`吧！它们会让你的代码更加现代、高效，还能充分利用多核处理器的威力。

最后一个小提示：虽然C++11的`future`和`promise`已经很强大，但如果你追求更高级的异步编程，可以考虑看看C++20的协程(coroutine)特性，那又是另一个让人兴奋的话题了～

---

## 写在最后：这只是异步编程旅程的开始...
嘿，朋友！看到这里，你已经掌握了C++异步编程的一把利器了！但编程世界就像宇宙一样浩瀚无边，我们的探索才刚刚开始～

**学到了新知识？有疑问？有心得？**  
👉 快到评论区和大家分享吧！我会亲自解答你的每一个问题。

**如果这篇文章让你有所收获...**  
请点个「**赞**」和「**在看**」，这对我来说，就像异步任务完成时收到的一个`promise.set_value()`一样珍贵！转发给你的程序猿/媛朋友，他们一定也会感谢你分享这个编程"黑魔法"！

**想要更多编程"神器"吗？**  

关注我的公众号「跟着小康学编程」，这里有：

🚀 **多线程编程实战秘籍** —— 让你的程序飞起来，CPU核心全利用！  
🔍 **C++面试题深度解析** —— 面试官：这水平，必须给offer！  
🛠️ **现代C++高级技巧** —— C++11/14/17/20新特性全解析，代码立减50%！  
🧩 **STL源码探秘** —— 看大神是怎么写代码的，学就要学最好的！  
💻 **实战项目分享** —— 纸上得来终觉浅，实战出真知！


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