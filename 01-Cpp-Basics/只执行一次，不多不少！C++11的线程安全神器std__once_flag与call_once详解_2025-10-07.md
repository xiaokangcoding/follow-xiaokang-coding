大家好啊，我是小康。

今天咱们聊一个看起来很小但实际超级实用的话题 —— C++11中的`std::once_flag`和`call_once`。听起来很高大上？别担心，我保证用最接地气的方式给你讲明白，绝不玩文绉绉的！

> 💡 学习建议：
>
>别死记 API，先理解它“解决了什么问题”：这比背语法有用一百倍。
>
>多写几个多线程小 Demo，比如单例模式、资源加载，体会 call_once 的威力。
>
>对比手写加锁和 call_once，你会发现它不仅更简洁，还更高效。
>
>打断点调试 看看它的执行顺序，你会对它的内部机制印象更深。
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

## 一个生活中的小故事

想象一下这个场景：你和三个朋友一起住，早上大家都要用洗衣机。如果每个人都去按一次洗衣机的"开始"按钮会发生什么？对，洗衣机会被重启好几次，衣服永远洗不完！

理想情况是：**无论谁先到洗衣机前，只需要有一个人按一次开始按钮就够了**。其他人看到洗衣机已经在运转，就不需要再按了。

这就是我们今天要讲的`std::once_flag`和`call_once`要解决的问题！

## 为什么需要"只执行一次"的机制？
在多线程程序中，有些初始化工作我们希望：

+ 必须执行（不能少）
+ 只能执行一次（不能多）
+ 所有线程都要等这个初始化完成后才能继续

最典型的例子就是单例模式（Singleton Pattern）：无论多少个线程同时请求，系统中某个对象只能有一个实例。

没有`call_once`之前，大家都这么写：

```cpp
// 传统写法，容易出问题
class Singleton {
private:
    static Singleton* instance;
    static std::mutex mutex;
    
    Singleton() { /* 构造函数 */ }
    
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {  // 第一次检查
            std::lock_guard<std::mutex> lock(mutex);
            if (instance == nullptr) {  // 第二次检查
                instance = new Singleton();
            }
        }
        return instance;
    }
};
```

看看这个"双重检查锁定"模式，又臭又长，还容易写错。更糟的是，在某些内存模型下，这段代码仍然可能有问题！

## 救星来了：std::once_flag 和 call_once
##

C++11引入了两个救星：

+ `std::once_flag`：一面"只执行一次"的旗帜
+ `std::call_once`：确保某个函数在多线程环境下只执行一次的工具

让我们看看它们是怎么用的：

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::once_flag flag;  // 创建一个"只执行一次"的旗帜

void do_once() {
    std::cout << "这段代码只会执行一次！" << std::endl;
}

void thread_func() {
    // 无论多少线程调用这个函数，do_once()只会执行一次
    std::call_once(flag, do_once);
    std::cout << "线程 " << std::this_thread::get_id() << " 执行完毕" << std::endl;
}

int main() {
    std::thread t1(thread_func);
    std::thread t2(thread_func);
    std::thread t3(thread_func);
    
    t1.join();
    t2.join();
    t3.join();
    
    return 0;
}
```

运行这段代码，你会发现"这段代码只会执行一次！"这句话确实只打印了一次，而"线程xx执行完毕"会打印三次。

## 工作原理：它是怎么做到的？
`std::once_flag`和`call_once`背后的工作原理其实很简单：

1. `once_flag`是一个标记，记录关联的函数是否已经执行过
2. 当第一个线程调用`call_once`时，它会执行指定的函数
3. 其他线程调用同一个`call_once`（使用同一个flag）时，会等待第一个线程执行完毕
4. 一旦函数执行完成，所有线程都会继续运行，但函数不会被重复执行

## 单例模式的最佳实践
现在，让我们用`std::call_once`来改写单例模式：

```cpp
class Singleton {
private:
    static std::once_flag init_flag;
    static Singleton* instance;
    
    Singleton() { 
        std::cout << "创建单例对象！" << std::endl;
    }
    
public:
    static Singleton* getInstance() {
        std::call_once(init_flag, []() {
            instance = new Singleton();
        });
        return instance;
    }
    
    void doSomething() {
        std::cout << "单例正在工作..." << std::endl;
    }
};

// 在类外初始化静态成员
std::once_flag Singleton::init_flag;
Singleton* Singleton::instance = nullptr;
```

是不是简洁了很多？没有复杂的双重检查，没有手动管理互斥锁，代码量减少了一半！

## 更多实际应用场景
除了单例模式，`std::call_once`还适用于很多场景：

### 1. 延迟初始化昂贵资源
```cpp
class DatabaseConnection {
private:
    std::once_flag conn_init;
    Connection* conn;
    
public:
    Connection* getConnection() {
        std::call_once(conn_init, [this]() {
            std::cout << "建立数据库连接（耗时操作）..." << std::endl;
            conn = new Connection("db://example");
        });
        return conn;
    }
};
```

### 2. 线程池的一次性初始化
```cpp
class ThreadPool {
private:
    std::once_flag init_flag;
    std::vector<std::thread> threads;
    
public:
    void initialize(size_t thread_count) {
        std::call_once(init_flag, [this, thread_count]() {
            for (size_t i = 0; i < thread_count; ++i) {
                threads.emplace_back(&ThreadPool::worker_thread, this);
            }
        });
    }
    
    void worker_thread() {
        // 线程工作内容...
    }
};
```

### 3. 配置文件的一次性加载
```cpp
class Configuration {
private:
    std::once_flag load_flag;
    std::map<std::string, std::string> settings;
    
public:
    std::string getSetting(const std::string& key) {
        std::call_once(load_flag, [this]() {
            std::cout << "加载配置文件..." << std::endl;
            // 加载配置到settings映射表...
        });
        return settings[key];
    }
};
```

## 比较：call_once vs 其他线程安全方式
那么，`std::call_once`比其他方法好在哪里呢？

1. **对比静态局部变量**：C++11保证静态局部变量的初始化是线程安全的，但`call_once`更灵活，可以在任何地方调用，不限于函数作用域
2. **对比互斥锁**：比手动管理互斥锁更简洁，不容易出错
3. **对比原子操作**：比使用`std::atomic`编写复杂的初始化逻辑更清晰
4. **性能优势**：在多数实现中，`call_once`采用了类似读写锁的优化，多线程并发时性能更好

## 容易踩的坑
使用`std::call_once`虽然简单，但还是有一些坑需要注意：

1. **一个flag只能用于一次初始化**：如果你需要多个不同的一次性初始化，就需要多个`once_flag`
2. **异常处理**：如果被调用的函数抛出异常，`call_once`会认为初始化失败，下次还会尝试执行
3. **避免死锁**：不要在`call_once`调用的函数中再次使用同一个`once_flag`，否则会死锁
4. **线程退出**：`once_flag`不会在线程退出时自动重置，它是全局状态

来看一个异常处理的例子：

```cpp
std::once_flag flag;

void may_throw() {
    std::call_once(flag, []() {
        std::cout << "尝试初始化..." << std::endl;
        throw std::runtime_error("初始化失败！");
    });
}

int main() {
    try {
        may_throw();  // 第一次尝试初始化，会抛出异常
    } catch (const std::exception& e) {
        std::cout << "捕获异常: " << e.what() << std::endl;
    }
    
    try {
        may_throw();  // 由于上次失败，会再次尝试初始化
    } catch (const std::exception& e) {
        std::cout << "再次捕获异常: " << e.what() << std::endl;
    }
    
    return 0;
}
```

输出结果会显示初始化被尝试了两次！

## 总结：什么时候用call_once？
`std::once_flag`和`call_once`最适合以下场景：

1. 需要线程安全的一次性初始化
2. 单例模式的实现
3. 共享资源的延迟初始化
4. 需要确保某个操作在多线程环境下只执行一次

记住，它是C++11标准库给我们提供的"只执行一次"的保证，远比我们自己实现双重检查锁定更可靠、更简单。

你们有没有在项目中用过这个功能？欢迎在评论区分享你的经验！


## 写在最后
感觉对线程安全有点"开窍"了吗？如果这篇文章帮你解决了困惑，不妨动动手指支持一下！👇

❤️ **一键三连**：点赞、在看、转发 ~

🔒 **关注我的公众号「跟着小康学编程」**，这里没有枯燥说教，只有接地气的技术分享！

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