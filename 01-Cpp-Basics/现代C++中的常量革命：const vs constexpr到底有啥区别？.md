大家好，我是小康。

今天咱们来聊一个看起来很小但用起来很有门道的话题——C++里的常量定义。如果你也曾经被`const`和`constexpr`这两兄弟搞得一头雾水，那这篇文章就是为你准备的！

## 从一个烧水的故事说起

想象一下，你家里有两种烧水壶：

+ 老式水壶：得放到火上烧，等水开了才能用
+ 新式热水瓶：里面的水随时都是热的，想用就能用

这两种水壶在 C++ 里分别对应着啥呢？没错，就是今天的主角：

+ `const`就像那个老式水壶，需要程序运行时才能确定它的值
+ `constexpr`则像那个随时备好热水的热水瓶，在编译时就把值算好了

听起来好像差不多？别急，接着往下看，你会发现它们的差别可大了！


> 💡 学习建议：
>
>多做小实验！ 写几个简单的例子，用 const 和 constexpr 定义变量，看看编译器报不报错。
>
>注意使用场景！ 当你需要编译期常量（例如数组长度、模板参数）时，一定要用 constexpr。
>
>理解底层原理！ 想清楚“编译期常量”和“运行期常量”的区别，这才是掌握 C++ 的关键。
>
>多读标准库源码！ 比如 <array>、<chrono> 里大量使用 constexpr，是学习最佳范例。
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

## const：我保证不变，但不保证提前知道

先说说大家比较熟悉的`const`。它的意思很简单："**这个值一旦被初始化，就不能再改变了**"。

```cpp
const int MAX_PLAYERS = 10;  // 这个值不能再改变了
MAX_PLAYERS = 11;  // 编译错误！常量不能被修改
```

`const`有一个重要特性：它可能在编译时确定值，但也可能在运行时才确定。这取决于初始化的方式：

```cpp
// 编译时就能确定的const
const int MAX_PLAYERS = 10;  // 编译器在编译时就知道这是10

// 运行时才能确定的const
int GetMaxPlayers() {
    return 10;  // 假设这个值是从配置文件或用户输入得到的
}
const int RUNTIME_MAX_PLAYERS = GetMaxPlayers();  // 运行时才知道具体值
```

关键点是：`const`**本身并不保证编译时求值，它只保证变量不可修改。** 编译器可能会在编译时计算 const 的值，但这不是 const 关键字的承诺，而是编译器的优化行为。

这就像那个老式水壶，有时候你可能提前准备好了热水，有时候却需要现烧——但无论如何，水烧开后就不会再变。

## constexpr：我不但不变，我还保证提前知道！

再来看看`constexpr` ，它是 C++11 才引入的，它比`const`更进一步，明确地告诉编译器："**嘿，我的值必须在编译时就能确定，不能等到运行时**"。

```cpp
constexpr int MAX_PLAYERS = 10;  // 编译时就确定了值

// 甚至可以做一些简单的计算
constexpr int TOTAL_PLAYERS = MAX_PLAYERS * 2;  // 编译时计算为20
```

这就像那个随时有热水的热水瓶，编译器提前就把答案算好了，程序运行时直接用现成的。`constexpr`**同时保证了编译时求值和不可修改**，它是一个"双保险"。

但如果你尝试这样写：

```cpp
int GetMaxPlayers() {
    return 10;
}

constexpr int MAX_PLAYERS = GetMaxPlayers();  // 错误！无法在编译时计算
```

编译器会生气地报错："你让我在编译时算出`GetMaxPlayers()`的结果？我做不到啊！"

## 什么时候用哪个？实用指南

那么问题来了，我们该怎么选择用`const`还是`constexpr`呢？

### 用const的情况
1、 **当值在运行时才能确定**

```cpp
const int user_age = getUserInputAge();  // 用户输入的值
const std::string message = loadMessageFromFile();  // 从文件读取
```

2、 **定义指针变量时**

```cpp
const int* p = &value;  // 指针指向的内容不能通过p修改
int* const p = &value;  // 指针本身不能改变指向
```

### 用constexpr的情况
1、 **编译时就能确定的常量**

```cpp
constexpr double PI = 3.14159265358979;
constexpr int DAYS_IN_WEEK = 7;
```

2、 **需要在编译时计算的表达式**

```cpp
constexpr int SECONDS_PER_DAY = 24 * 60 * 60;  // 86400
```

3、 **函数返回值在编译时需要用到**

```cpp
constexpr int square(int n) {
    return n * n;
}

constexpr int result = square(5);  // 编译时计算为25
```

## 真实案例：constexpr的威力
来看一个实际的例子，体会一下`constexpr`的强大之处。

假设我们要计算斐波那契数列的第n个数：

```cpp
// 使用constexpr的斐波那契函数
constexpr int fibonacci(int n) {
    return (n <= 1) ? n : fibonacci(n-1) + fibonacci(n-2);
}

// 编译时计算斐波那契数列的第10个数
constexpr int fib10 = fibonacci(10);  // 编译时计算为55

int main() {
    // 程序运行时，fib10已经是55了，不需要再计算
    std::cout << "斐波那契数列的第10个数是：" << fib10 << std::endl;
    return 0;
}
```

如果用`const`而不是`constexpr`，这个计算就会在运行时进行，效率就会低很多。

## 最佳实践总结
根据我的经验，这里有几条实用建议：

1、 **能用constexpr就用constexpr**  
它不仅保证了变量不可变，还能提高程序的性能。

2、 **复杂运算尽量在编译时完成**  
把能在编译时做的事情就在编译时做完，让运行时更轻松。

3、 **函数参数常量用const**

```cpp
void processData(const std::vector<int>& data);  // 防止函数内部修改data
```

4、 **类成员常量用constexpr**

```cpp
class GameSettings {
public:
    static constexpr int MAX_PLAYERS = 10;
};
```

## 一句话区分

如果你只想记住一句话：

`const`只承诺"我不变"，但不保证编译时求值；`constexpr`则双重承诺"我不变"，并且"我一定在编译时就知道值是多少"。

## 最后的思考题
看看下面这段代码，思考一下哪里用了`const`，哪里用了`constexpr`，为什么这样用？

```cpp
constexpr int BUFFER_SIZE = 1024;

void processBuffer(const char* buffer, const int size) {
    constexpr int MAX_SIZE = BUFFER_SIZE * 2;
    const int actualSize = (size > MAX_SIZE) ? MAX_SIZE : size;
    // 处理逻辑...
}
```

答案留给各位读者思考啦！希望这篇文章能让你对 C++ 中的常量定义有更清晰的认识。学会了这两兄弟的正确用法，代码不仅更安全，还能跑得更快！

如果你觉得这篇文章对你有帮助，别忘了**点赞、在看、分享**哦~ 你的支持是我持续创作的动力！

如果有疑问或想了解更多 C++ 的小技巧，欢迎在评论区留言交流~

---

更多精彩文章，请关注本公众号！

下期见！👨‍💻

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