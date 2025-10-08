大家好，我是小康。

你有没有过这样的经历？写了一大堆代码，明明逻辑没问题，程序跑得却像蜗牛一样慢。特别是当你在处理大量数据，往容器里疯狂塞东西的时候。

如果你经常和 C++ 的 vector、list 这些容器打交道，那么今天这篇文章绝对值得你花几分钟时间——因为我要告诉你一个小技巧，它能让你的代码不仅写起来更爽，还能跑得更快！

它就是容器的"隐藏技能"：`emplace_back()`。

> 💡 学习建议：
>
>自己动手写个对比实验：用 push_back() 和 emplace_back() 各插入一万条对象，观察构造次数和运行时间的差别。
>
>理解完美转发（perfect forwarding）：emplace_back() 背后依赖了它，理解这点能让你深入掌握 C++11 的精髓。
>
>看 STL 源码实现：看看 vector::emplace_back() 是怎么用模板和 std::forward 实现“原地构造”的。
>
>不要盲目替换：某些情况下（尤其是简单类型），emplace_back() 和 push_back() 区别并不大，记得根据场景选用。
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

## "又是一个新函数？我的push_back不香了吗？"

别急，咱们先来看个例子，感受一下这两者的区别：

```cpp
#include <vector>
#include <string>

class Person {
public:
    Person(std::string name, int age) : m_name(name), m_age(age) {
        printf("构造了一个人：%s, %d岁\n", name.c_str(), age);
    }
    
    // 拷贝构造函数
    Person(const Person& other) : m_name(other.m_name), m_age(other.m_age) {
        printf("拷贝构造了一个人：%s\n", m_name.c_str());
    }
    
private:
    std::string m_name;
    int m_age;
};

int main() {
    std::vector<Person> people;
    
    printf("=== 使用push_back ===\n");
    people.push_back(Person("张三", 25));
    
    people.clear();  // 清空容器
    
    printf("\n=== 使用emplace_back ===\n");
    people.emplace_back("李四", 30);
}
```

运行这段代码，你会看到这样的输出：

```plain
=== 使用push_back ===
构造了一个人：张三, 25岁
拷贝构造了一个人：张三

=== 使用emplace_back ===
构造了一个人：李四, 30岁
```

看出区别了吗？

+ 使用`push_back`时，我们先创建了一个临时的 Person 对象，然后 vector 把它拷贝到容器里
+ 而使用`emplace_back`时，我们直接传入构造 Person 所需的参数，vector 直接在容器内部构造对象

结果就是：`push_back`额外调用了一次拷贝构造函数，而`emplace_back`没有！

## "所以emplace_back就是直接传构造函数参数？"

没错！这就是它最大的特点。

+ `push_back(x)` 需要你先构造好一个对象x，然后把它放进容器
+ `emplace_back(args...)` 则是直接把构造函数的参数 args 传进去，在容器内部构造对象

这个差别看似小，实际上在性能上却能带来很大的提升，尤其是当：

1. 对象构造成本高（比如有很多成员变量）
2. 拷贝成本高（比如内部有动态分配的内存）
3. 你需要插入大量对象时

## "来点实际的例子！"

好的，我们来看一个真正能展示差异的例子。我们创建一个拷贝成本真正很高的类，这样才能看出 emplace_back 的威力：

```cpp
#include <vector>
#include <string>
#include <chrono>
#include <iostream>
#include <memory>

// 设计一个拷贝成本很高的类
class ExpensiveToCopy {
public:
    // 构造函数 - 创建一个大数组
    ExpensiveToCopy(const std::string& name, int dataSize) 
        : m_name(name), m_dataSize(dataSize) {
        // 分配大量内存，模拟昂贵的资源
        m_data = new int[dataSize];
        for (int i = 0; i < dataSize; i++) {
            m_data[i] = i;  // 初始化数据
        }
    }
    
    // 拷贝构造函数 - 非常昂贵，需要复制整个大数组
    ExpensiveToCopy(const ExpensiveToCopy& other)
        : m_name(other.m_name), m_dataSize(other.m_dataSize) {
        // 深拷贝，非常耗时
        m_data = new int[m_dataSize];
        for (int i = 0; i < m_dataSize; i++) {
            m_data[i] = other.m_data[i];
        }
        
        // 输出提示以便观察拷贝构造函数的调用情况
        std::cout << "拷贝构造: " << m_name << std::endl;
    }
    
    // 析构函数
    ~ExpensiveToCopy() {
        delete[] m_data;
    }
    
    // 禁用赋值运算符以简化例子
    ExpensiveToCopy& operator=(const ExpensiveToCopy&) = delete;
    
private:
    std::string m_name;
    int* m_data;
    int m_dataSize;
};

// 计时辅助函数
template<typename Func>
long long timeIt(Func func) {
    auto start = std::chrono::high_resolution_clock::now();
    func();
    auto end = std::chrono::high_resolution_clock::now();
    return std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
}

int main() {
    const int COUNT = 100;  // 对象数量，减少一点以便看到输出
    const int DATA_SIZE = 100000;  // 每个对象中数组的大小
    
    std::cout << "=== 测试push_back ===\n";
    long long pushTime = timeIt([&]() {
        std::vector<ExpensiveToCopy> objects;
        objects.reserve(COUNT);  // 预分配空间避免重新分配的影响
        
        for (int i = 0; i < COUNT; i++) {
            // 创建临时对象然后放入vector
            // 这个过程会调用拷贝构造函数
            objects.push_back(ExpensiveToCopy("对象" + std::to_string(i), DATA_SIZE));
        }
    });
    
    std::cout << "\n=== 测试emplace_back ===\n";
    long long emplaceTime = timeIt([&]() {
        std::vector<ExpensiveToCopy> objects;
        objects.reserve(COUNT);  // 预分配空间避免重新分配的影响
        
        for (int i = 0; i < COUNT; i++) {
            // 直接传递构造函数参数
            // 直接在vector内部构造对象，避免了拷贝
            objects.emplace_back("对象" + std::to_string(i), DATA_SIZE);
        }
    });
    
    std::cout << "\npush_back耗时: " << pushTime << " 微秒" << std::endl;
    std::cout << "emplace_back耗时: " << emplaceTime << " 微秒" << std::endl;
    double percentDiff = (static_cast<double>(pushTime) / emplaceTime - 1.0) * 100.0;
    std::cout << "性能差异: push_back比emplace_back慢了 " << percentDiff << "%" << std::endl;
}
```

在我的电脑上，大约是这样的结果：

```plain
=== 测试emplace_back ===

push_back耗时: 66979 微秒
emplace_back耗时: 35858 微秒
性能差异: push_back比emplace_back慢了 86.7896%
```

这意味着`push_back`比`emplace_back`慢了约86%！这可不是小数目，尤其是在处理大对象时。

## "看起来emplace_back完胜啊！为什么还有人用push_back？"

好问题！`emplace_back`虽然在大多数情况下更快，但并不是所有场景都适合用它：

1. **当你已经有一个现成的对象时**，`push_back`可能更直观
2. **对于基本类型**（int, double等），两者性能差异可以忽略不计
3. **对于某些编译器优化情况**，比如移动语义，差距可能不明显

来看一个例子，说明什么时候两者其实差不多：

```cpp
std::vector<int> numbers;

// 对于基本类型，这两个是等价的
numbers.push_back(42);
numbers.emplace_back(42);

// 如果已经有一个现成的对象
std::string name = "张三";
std::vector<std::string> names;

// 这种情况下，如果 string 支持移动构造，两者性能接近
names.push_back(name);               // 拷贝name
names.push_back(std::move(name));    // 移动name（推荐）
names.emplace_back(name);            // 拷贝name
names.emplace_back(std::move(name)); // 移动name（推荐）
```

## "完美转发是什么鬼？听说emplace_back跟这个有关？"

没错！`emplace_back`的强大之处，部分来自于它使用了"完美转发"(Perfect Forwarding)技术。

简单来说，完美转发就是把函数参数"原汁原味"地传递给另一个函数，保持它的所有特性（比如是左值还是右值，是const还是non-const）。

在C++中，这通常通过模板和`std::forward`实现：

```cpp
template <typename... Args>
void emplace_back(Args&&... args) {
    // 在容器内部直接构造对象
    // 完美转发所有参数
    new (memory_location) T(std::forward<Args>(args)...);
}
```

这样的设计让`emplace_back`能够接受任意数量、任意类型的参数，并且完美地转发给对象的构造函数。这就是为什么你可以直接这样写：

```cpp
people.emplace_back("张三", 25);  // 直接传构造函数参数
```

而不需要先构造一个对象。

## "还有其他 emplace 系列函数吗？"

是的！STL容器中有一系列的emplace函数：

+ `vector`、`deque`、`list`: `emplace_back()`
+ `list`, `forward_list`: `emplace_front()`
+ 所有容器: `emplace()`（在指定位置构造元素）
+ 关联容器（`map`, `set`等）: `emplace_hint()`（带提示的插入）

它们的共同点是：直接在容器内部构造元素，而不是先构造再拷贝/移动。

## 实战建议：什么时候用 emplace_back？

1、 **复杂对象插入**：当你要插入的对象构造成本高、拷贝代价大时

2、 **大量数据操作**：需要插入大量元素时，性能差异会更明显

3、 **直接传参更方便时**：比如插入 pair 到 map

```cpp
// 不那么优雅
std::map<int, std::string> m;
m.insert(std::make_pair(1, "one"));

// 更优雅，也更高效
m.emplace(1, "one");
```

4、 **临时对象场景**：当你需要创建临时对象并插入容器时

## 总结

`emplace_back`本质上是通过减少不必要的对象创建和拷贝来提升性能。它利用了 C++ 的完美转发功能，让你可以直接传递构造函数参数，而不需要先创建临时对象。

在处理复杂对象或大量数据时，这种优化尤为明显。当然，对于简单类型或已有对象，两者差异不大。

所以下次当你在写：

```cpp
myVector.push_back(MyClass(arg1, arg2));
```

的时候，不妨试试：

```cpp
myVector.emplace_back(arg1, arg2);
```

代码更简洁，运行更高效，何乐而不为呢？

记住，在编程世界里，这种看似微小的优化，累积起来就是质的飞跃！

你的容器操作用的是 push_back 还是 emplace_back ？欢迎在评论区分享你的经验！

---

如果你喜欢这种通俗易懂的 C++ 技术讲解，欢迎关注我的公众号「**跟着小康学编程**」，这里不仅有性能优化技巧，还有更多 C++11/14/17/20 的现代特性解析，让你的代码既现代又高效！


## 每天进步一点点，C++技能升级不停歇
看完这篇文章，你是不是对 C++ 容器操作有了新的认识？这只是现代C++高效编程的冰山一角。如果你想掌握更多这样的实用技巧，欢迎关注我的公众号「**跟着小康学编程**」。

在那里，你会发现：

+ 性能调优的秘密武器
+ STL容器和算法的高级应用
+ C++面试中的那些坑和解法
+ Linux C/C++后端技术分享
+ 计算机基础原理与网络编程实战

我不喜欢枯燥的理论堆砌，而是用生动例子和实际问题来解释复杂概念。正如你在这篇文章中看到的那样。

如果这篇文章对你有帮助，别忘了点个**赞**，点个**在看**，这是支持我继续创作的动力！我们下期再见~

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