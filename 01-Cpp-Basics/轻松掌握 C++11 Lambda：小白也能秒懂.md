大家好啊，我是小康。

今天咱们聊一个听起来挺唬人，但用起来超级方便的东西——C++11的 Lambda 表达式！啥？听名字就头大？别急，跟着我这篇文章走，保证你看完直呼："原来这么简单啊！"

> 💡 学习建议：
>
>多写小例子，尝试各种捕获方式（值捕获、引用捕获、混合捕获），亲自体验作用域和生命周期差异。
>
>结合 STL 算法练手，像 std::sort、std::for_each、std::transform，用 Lambda 改写一遍，会豁然开朗。
>
>尝试多线程场景，用 Lambda 作为线程函数，观察捕获变量在线程中的表现。
>
>别死记语法，理解 Lambda 的本质——“一个可以直接定义在当前位置的可调用对象”，比记符号更重要。
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


## 什么是 Lambda 表达式？先别慌！

要是有人问你 Lambda 是啥，你可以拍拍胸脯说："不就是个匿名函数嘛，小意思！"

通俗点说，Lambda就是一个没有名字的小函数，写完就能用，不用大费周章地去定义一个正式函数。

举个生活中的例子吧：

+ 正式函数就像你叫了个专业厨师来家里做饭，走正规流程，备菜、炒菜、上菜，还得提前预约。
+ Lambda就像你肚子饿了，随手煮个泡面加个鸡蛋，3分钟搞定，吃完就走人。

是不是瞬间觉得没那么可怕了？

## Lambda表达式长啥样？
先来看看 Lambda 的基本样子：

```plain
[捕获列表](参数列表) -> 返回类型 { 函数体 }
```

看起来挺复杂？别怕，我们把它拆开来看：

1. `[捕获列表]`：决定外部变量怎么进入Lambda的小房间
2. `(参数列表)`：和普通函数一样，接收参数
3. `-> 返回类型`：告诉编译器返回啥东西（通常可以省略，编译器自己能猜出来）
4. `{ 函数体 }`：就是要执行的代码呗

来个最简单的Lambda例子：

```cpp
#include <iostream>

int main() {
    // 一个简单的Lambda，不接收参数，返回整数5
    auto sayHi = []() { return 5; };

    std::cout << "Lambda返回值: " << sayHi() << std::endl;
    return 0;
}
```

输出结果：

```plain
Lambda返回值: 5
```

是不是没想到竟然这么简单？接下来我们再深入了解一下！

## 捕获列表：Lambda的"购物袋"
捕获列表可能是 Lambda 里最让人困惑的部分。简单说，它决定了Lambda能不能使用它外部的变量。

想象 Lambda 是个小超市，捕获列表就是你进门时拿的购物袋，决定了你能把外面的什么东西带进超市。

几种常见的"购物方式"：

1. `[]` - 空购物袋，啥也不带（不捕获任何外部变量）
2. `[=]` - 大购物袋，把所有能看到的外部变量都复制一份带进去
3. `[&]` - 神奇透明袋，不复制外部变量，直接在原地操作它们
4. `[a, &b]` - 混合袋，复制变量a，直接操作变量b
5. `[this]` - 带着自己的身份证（捕获当前对象的this指针）

来看个例子：

```cpp
#include <iostream>

int main() {
    int number = 10;
    int factor = 5;

    // 不捕获任何变量
    auto lambda1 = []() { 
        // std::cout << number; // 错误：看不到外部变量
        std::cout << "我啥也没捕获到" << std::endl;
    };

    // 以值方式捕获所有变量
    auto lambda2 = [=]() {
        std::cout << "我复制了number的值: " << number << std::endl;
        // number = 20; // 错误：复制的值不能修改
    };

    // 以引用方式捕获所有变量
    auto lambda3 = [&]() {
        number = 20; // 可以修改原始变量
        std::cout << "我修改了number: " << number << std::endl;
    };

    // 混合捕获：值捕获number，引用捕获factor
    auto lambda4 = [number, &factor]() {
        // number = 30; // 错误：值捕获的变量不能修改
        factor = 15; // 正确：引用捕获的变量可以修改
        std::cout << "number: " << number << ", factor: " << factor << std::endl;
    };

    lambda1();
    lambda2();
    lambda3();
    lambda4();

    std::cout << "最终number: " << number << ", factor: " << factor << std::endl;

    return 0;
}
```

输出结果：

```plain
我啥也没捕获到
我复制了number的值: 10
我修改了number: 20
number: 20, factor: 15
最终number: 20, factor: 15
```

看出区别了吗？`[=]`只是复制了一份，`[&]`直接改了原始值！

## Lambda参数：和普通函数一样嘛
这部分最简单，和普通函数的参数完全一样：

```cpp
#include <iostream>

int main() {
    // 接收两个参数的Lambda
    auto add = [](int a, int b) {
        return a + b;
    };

    std::cout << "3 + 5 = " << add(3, 5) << std::endl;
    return 0;
}
```

输出结果：

```plain
3 + 5 = 8
```

## Lambda的实战应用：排序、过滤样样行
说了这么多，Lambda到底在哪里特别好用呢？最常见的就是和STL算法的结合使用。

### 自定义排序
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {5, 3, 1, 4, 2};

    // 使用Lambda进行降序排序
    std::sort(numbers.begin(), numbers.end(), [](int a, int b) {
        return a > b; // 降序排列
    });

    std::cout << "降序排序后: ";
    for (int n : numbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出结果：

```plain
降序排序后: 5 4 3 2 1
```

如果没有Lambda，你得单独定义一个比较函数，代码会变得又臭又长。有了Lambda，一行搞定！

### 自定义查找
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

struct Person {
std::string name;
int age;
};

int main() {
    std::vector<Person> people = {
    {"张三", 25},
    {"李四", 30},
    {"王五", 22}
};

    // 查找年龄大于25的第一个人
    auto it = std::find_if(people.begin(), people.end(), [](const Person& p) {
        return p.age > 25;
    });

    if (it != people.end()) {
        std::cout << "找到年龄大于25的人: " << it->name << ", " << it->age << "岁" << std::endl;
    } else {
        std::cout << "没找到符合条件的人" << std::endl;
    }

    return 0;
}
```

输出结果：

```plain
找到年龄大于25的人: 李四, 30岁
```

## Lambda欢乐进阶：闭包那些事儿
Lambda还有个高级玩法叫"闭包"，简单说就是Lambda可以"记住"它创建时的环境。听着复杂？看例子就明白了：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// 工厂函数，返回一个计数器Lambda
auto makeCounter(int startFrom) {
    return [startFrom]() mutable {
        return startFrom++;
    };
}

int main() {
    // 创建两个不同的计数器
    auto counter1 = makeCounter(5);
    auto counter2 = makeCounter(100);

    // 使用计数器
    std::cout << "计数器1: ";
    for (int i = 0; i < 3; i++) {
        std::cout << counter1() << " ";
    }
    std::cout << std::endl;

    std::cout << "计数器2: ";
    for (int i = 0; i < 3; i++) {
        std::cout << counter2() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出结果：

```plain
计数器1: 5 6 7 
计数器2: 100 101 102
```

注意这里的`mutable`关键字，它允许Lambda修改捕获的值。如果没有`mutable`，Lambda捕获的值就是只读的。

## 实用技巧：Lambda你能这样玩
### 技巧1：使用泛型Lambda处理不同类型（C++14特性，但可在C++11中模拟）
```cpp
// C++11中使用模板函数对象模拟泛型Lambda
struct GenericMultiplier {
template<typename T, typename U>
auto operator()(T a, U b) -> decltype(a * b) {
    return a * b;
}
};

int main() {
    GenericMultiplier multiplier;

    // 可以处理不同类型
    std::cout << "整数相乘: " << multiplier(5, 10) << std::endl;
    std::cout << "浮点数相乘: " << multiplier(2.5, 4.0) << std::endl;
    std::cout << "混合类型: " << multiplier(5, 3.14) << std::endl;

    return 0;
}
```

输出结果:

```plain
整数相乘: 50
浮点数相乘: 10
混合类型: 15.7
```

虽然 C++11 不直接支持泛型Lambda，但我们可以通过函数对象模拟这一功能，这是 Lambda 底层实现的本质。

### 技巧2：递归Lambda（自己调用自己）
```cpp
#include <iostream>
#include <functional>

int main() {
    // 计算阶乘的递归Lambda
    std::function<int(int)> factorial = [&factorial](int n) {
        return n <= 1 ? 1 : n * factorial(n - 1);
    };

    std::cout << "5的阶乘: " << factorial(5) << std::endl;
    return 0;
}
```

输出结果：

```plain
5的阶乘: 120
```

为啥要用`std::function`？因为递归需要Lambda引用自己，普通auto变量在定义前不能使用。

### 技巧3：条件筛选
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> evenNumbers;

    // 筛选出偶数
    std::copy_if(numbers.begin(), numbers.end(), std::back_inserter(evenNumbers),
        [](int n) { return n % 2 == 0; });

    std::cout << "偶数有: ";
    for (int n : evenNumbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出结果：

```plain
偶数有: 2 4 6 8 10
```

## 总结：Lambda，你值得拥有！
看到这里，相信你已经不再害怕Lambda表达式了！简单总结一下：

1. Lambda是匿名函数，用完就走，干净利落
2. 基本语法：`[捕获列表](参数列表) -> 返回类型 { 函数体 }`
3. 捕获列表控制外部变量如何进入Lambda：`[]`, `[=]`, `[&]`, `[变量名]`
4. Lambda最适合用在需要传递函数作为参数的场景，特别是和STL算法结合使用

掌握了Lambda，你的C++代码会变得更简洁、更灵活、更现代！不再需要为了一个小功能定义一大堆函数了，用Lambda一行搞定！

最后送大家一句话："代码越短，bug越少；Lambda一出，天下安宁！"

你学会了吗？欢迎在评论区分享你的 Lambda 使用心得！

---

## Lambda表达式没学够？这里还有更多！

代码写累了？点个「**在看**」休息一下呗！顺手「转发」给你的小伙伴，告诉他们 Lambda 原来这么简单~

**👉 关注我的公众号「跟着小康学编程」，每周解锁新技能！**

在这里，我们不只谈Lambda：

+ 🚀 **进阶秘籍**：那些面试官最爱问的C++难题，我都给你拆得明明白白
+ 🔍 **源码探秘**：STL 容器背后藏着什么？跟我一起扒开看个究竟
+ ⚡ **性能魔法**：让你的代码飞起来，学会这些技巧，同事都惊呆了
+ 🛠️ **实战案例**：大厂项目是怎么用 C++ 的？学会这些少走3年弯路
+ 🐧 **全栈进化**：从 C++ 到 Linux 后端开发，全栈工程师就是你！

每周持续更新，保证干货满满，不整虚的！

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
