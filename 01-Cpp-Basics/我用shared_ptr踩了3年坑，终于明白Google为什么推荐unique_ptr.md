大家好，我是小康。

最近在技术群里看到一个有趣的讨论，一个小伙伴问："为什么Google的代码规范总是推荐用unique_ptr，对shared_ptr却很谨慎？是不是shared_ptr有什么坑？"

这个问题瞬间炸出了一群技术大佬，有人说shared_ptr性能差，有人说容易内存泄漏，还有人说设计思路就不对...

说实话，我刚开始学C++的时候也很困惑。明明shared_ptr看起来更"智能"啊，可以自动管理引用计数，多个对象可以共享，听起来就很高级。而unique_ptr好像就是个"独占狂"，一个对象只能有一个主人，显得很"小气"。

但是工作几年后，我终于明白了其中的门道。今天就来给大家深度剖析一下这两个"智能指针兄弟"的恩怨情仇。

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

## 先说说这两兄弟的"出身"

### unique_ptr：独占型的"专一男友"
unique_ptr就像那种专一的男朋友，一旦认定了一个对象，就独占所有权，绝不与其他人分享。

```cpp
std::unique_ptr<int> ptr1(new int(42));
// std::unique_ptr<int> ptr2 = ptr1;  // 编译错误！不能复制
std::unique_ptr<int> ptr2 = std::move(ptr1);  // 只能转移所有权
// 现在ptr1为空，ptr2拥有对象
```

### shared_ptr：共享型的"中央空调"
shared_ptr则像中央空调，可以同时服务多个"用户"，内部维护一个引用计数器。

```cpp
std::shared_ptr<int> ptr1(new int(42));
std::shared_ptr<int> ptr2 = ptr1;  // 可以复制，引用计数变为2
std::shared_ptr<int> ptr3 = ptr1;  // 引用计数变为3
// 当所有shared_ptr都销毁后，对象才会被释放
```

## 为什么Google偏爱unique_ptr？
### 1. 性能差距：不是一个量级的
我们先来看看性能对比，数据会说话：

```cpp
// 性能测试代码
#include <chrono>
#include <memory>

// unique_ptr创建和销毁
auto start = std::chrono::high_resolution_clock::now();
for (int i = 0; i < 1000000; ++i) {
    auto ptr = std::make_unique<int>(i);
}  // 自动销毁
auto end = std::chrono::high_resolution_clock::now();
std::cout << "unique_ptr用时: " << 
    std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() 
    << "微秒" << std::endl;

// shared_ptr创建和销毁
start = std::chrono::high_resolution_clock::now();
for (int i = 0; i < 1000000; ++i) {
    auto ptr = std::make_shared<int>(i);
}  // 自动销毁
end = std::chrono::high_resolution_clock::now();
std::cout << "shared_ptr用时: " << 
    std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() 
    << "微秒" << std::endl;
```

在我的机器上测试结果：

+ unique_ptr：约20072微秒
+ shared_ptr：约60081微秒

**shared_ptr竟然比unique_ptr慢了4倍多！**

为什么会这样？因为shared_ptr需要：

+ 维护引用计数（原子操作，线程安全但开销大）
+ 额外的内存分配（控制块）
+ 复制时需要原子递增
+ 销毁时需要原子递减并检查是否为0

### 2. 内存开销：shared_ptr是个"大胃王"
```cpp
std::cout << "unique_ptr大小: " << sizeof(std::unique_ptr<int>) << "字节" << std::endl;
std::cout << "shared_ptr大小: " << sizeof(std::shared_ptr<int>) << "字节" << std::endl;
```

结果：

+ unique_ptr：8字节（就是一个普通指针）
+ shared_ptr：16字节（指针+控制块指针）

而且shared_ptr还会额外分配一个控制块，存储引用计数等信息，至少需要额外的16个字节。

如果你的程序中有大量的智能指针，这个差距就很明显了。

### 3. 设计哲学：ownership要清晰
Google的代码规范有一个重要原则：**所有权要清晰**。

看这个例子：

```cpp
// 不好的设计：所有权不明确
class DataProcessor {
    std::shared_ptr<Data> data_;
public:
    void setData(std::shared_ptr<Data> data) { data_ = data; }
};

class DataCache {
    std::shared_ptr<Data> cached_data_;
public:
    void cache(std::shared_ptr<Data> data) { cached_data_ = data; }
};

// 使用时：谁拥有data？谁负责生命周期？不清楚！
auto data = std::make_shared<Data>();
processor.setData(data);
cache.cache(data);
```

```cpp
// 好的设计：所有权清晰
class DataManager {
    std::unique_ptr<Data> data_;  // 明确的拥有者
public:
    Data* getData() { return data_.get(); }  // 只提供访问权
    void setData(std::unique_ptr<Data> data) { 
        data_ = std::move(data); 
    }
};

// 使用时：DataManager明确拥有data
auto data = std::make_unique<Data>();
manager.setData(std::move(data));
```

## shared_ptr的"隐形炸弹"
### 1. 循环引用：程序员的噩梦
这是shared_ptr最著名的坑：

```cpp
class Parent;
class Child;

class Parent {
public:
    std::shared_ptr<Child> child_;
    ~Parent() { std::cout << "Parent析构" << std::endl; }
};

class Child {
public:
    std::shared_ptr<Parent> parent_;  // 危险！循环引用
    ~Child() { std::cout << "Child析构" << std::endl; }
};

{
    auto parent = std::make_shared<Parent>();
    auto child = std::make_shared<Child>();
    
    parent->child_ = child;
    child->parent_ = parent;  // 形成循环引用
}
// 程序结束，但你会发现析构函数都没被调用！
// 内存泄漏了！
```

这种bug特别隐蔽，很难调试。而unique_ptr从设计上就避免了这个问题。

### 2. 线程安全的假象
很多人以为shared_ptr是线程安全的，其实只是**引用计数的操作是线程安全的**，对象本身的访问并不安全：

```cpp
std::shared_ptr<std::vector<int>> ptr = std::make_shared<std::vector<int>>();

// 这些操作是线程安全的：
std::shared_ptr<std::vector<int>> ptr2 = ptr;  // 拷贝shared_ptr
ptr.reset();  // 重置shared_ptr
auto count = ptr.use_count();  // 查看引用计数

// 这些操作不是线程安全的：
// 线程1
ptr->push_back(1);  // 不安全！ 修改vector内容

// 线程2  
ptr->push_back(2);  // 不安全！

// 引用计数的增减是安全的，但对vector的操作不安全
```

### 3. 性能陷阱：不经意的拷贝
```cpp
// 看似无害的代码
void processData(std::shared_ptr<Data> data) {  // 注意：按值传递
    // 处理数据...
}  // 函数结束时，局部的shared_ptr被销毁，引用计数-1

// 每次调用都会发生什么？
auto data = std::make_shared<Data>();  // 引用计数=1
for (int i = 0; i < 1000000; ++i) {
    processData(data);  // 1. 拷贝shared_ptr，引用计数+1（原子操作）
                        // 2. 函数执行
                        // 3. 函数结束，引用计数-1（原子操作）
}
// 100万次原子操作的开销！
```

应该改为：

```cpp
// 方案1：按引用传递shared_ptr（推荐）
void processData(const std::shared_ptr<Data>& data) {
    // 处理数据，无拷贝开销
}

// 方案2：传递原始指针（如果不需要延长生命周期）
void processData(Data* data) {
    // 处理数据...
}

// 方案3：传递引用（如果确定对象存在）
void processData(const Data& data) {
    // 处理数据...
}
```

## 什么时候才用shared_ptr？
虽然我们一直在"黑"shared_ptr，但它确实有适用场景，关键是要**真正需要共享所有权**：

### 1. 缓存系统：多个持有者，不确定谁先释放
```cpp
class ResourceCache {
    std::unordered_map<std::string, std::shared_ptr<ExpensiveResource>> cache_;
public:
    std::shared_ptr<ExpensiveResource> get(const std::string& key) {
        if (cache_.find(key) == cache_.end()) {
            cache_[key] = std::make_shared<ExpensiveResource>(key);
        }
        return cache_[key];  // 调用者和缓存都持有，谁都可能先释放
    }
    
    void cleanup() {
        // 清理缓存，但如果外部还在使用，对象不会被销毁
        cache_.clear();
    }
};
```

### 2. 异步编程：延长对象生命周期
```cpp
class DataProcessor {
public:
    void processAsync(std::shared_ptr<Data> data) {
        // 启动异步任务，data的生命周期不确定
        std::thread([data]() {
            std::this_thread::sleep_for(std::chrono::seconds(5));
            // 即使调用方已经结束，data依然有效
            processData(*data);
        }).detach();
    }
    
    // 如果用unique_ptr会怎样？
    void processBad(std::unique_ptr<Data> data) {
        std::thread([&data]() {  // 危险！引用可能悬空
            processData(*data);   // 可能崩溃
        }).detach();
    }
};
```

### 3. 资源池：共享昂贵资源
```cpp
class DatabaseConnectionPool {
    std::vector<std::shared_ptr<Connection>> pool_;
public:
    std::shared_ptr<Connection> getConnection() {
        if (!pool_.empty()) {
            auto conn = pool_.back();
            pool_.pop_back();
            return conn;  // 多个客户端可能同时使用同一连接
        }
        return std::make_shared<Connection>();
    }
    
    void returnConnection(std::shared_ptr<Connection> conn) {
        // 简单地放回池中，shared_ptr会自动管理生命周期
        // 即使有其他地方还在使用这个连接，也没关系
        // 当所有引用都释放后，连接会自动销毁
        pool_.push_back(conn);
    }
};
```

### 4. 插件系统：多模块共享
```cpp
class PluginManager {
    std::unordered_map<std::string, std::shared_ptr<Plugin>> plugins_;
public:
    std::shared_ptr<Plugin> loadPlugin(const std::string& name) {
        if (plugins_.find(name) == plugins_.end()) {
            plugins_[name] = std::make_shared<Plugin>(name);
        }
        return plugins_[name];  // 多个模块可能同时需要同一个插件
    }
};

// 使用场景
auto audioPlugin = pluginManager.loadPlugin("audio");
auto videoPlugin = pluginManager.loadPlugin("audio");  // 返回同一个实例
// audioPlugin和videoPlugin指向同一对象，任何一个都可以安全使用
```

## 实战指南：如何选择智能指针？
遵循这个简单的决策流程：

### 第一步：默认选择unique_ptr
除非有特殊需求，否则总是从unique_ptr开始。它性能最好，语义最清晰。

### 第二步：遇到问题时再考虑升级
+ **需要转移所有权？** → 继续用unique_ptr + std::move
+ **需要多个拥有者？** → 问自己三个问题：
    1. 是否真的有多个"拥有者"需要这个对象？
    2. 这些拥有者的生命周期是否不确定？
    3. 任何一个拥有者都不应该单独决定对象何时销毁？

如果三个答案都是"是"，才用shared_ptr

+ **可能有循环引用？** → 用weak_ptr打破循环

## 总结：智能指针的"智慧"选择

Google推荐优先使用unique_ptr不是没有道理的：

1. **性能更好**：没有引用计数开销
2. **内存更省**：只有一个指针的大小
3. **设计更清晰**：明确的所有权语义
4. **更安全**：避免循环引用等陷阱

记住这个原则：**能用unique_ptr就不用shared_ptr，能用引用就不用指针**。

最后想说，智能指针的"智能"不在于功能有多复杂，而在于设计思路的清晰和使用场景的准确。就像写代码一样，简单往往比复杂更难，但也更有价值。

---

**写在最后**

说实话，写这篇文章的时候，我想起了刚工作时被shared_ptr坑得死去活来的经历😅。那时候觉得shared_ptr这么"智能"，应该到处用，结果踩了不少坑。

如果你也在C++的路上摸爬滚打，或者对Linux后台开发感兴趣，欢迎关注我的公众号「**跟着小康学编程**」。我会定期分享一些实战踩坑经验，都是工作中真实遇到的问题， 绝对不是那种网上随处可见的教科书式废话。  

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