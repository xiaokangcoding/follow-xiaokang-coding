
大家好！今天咱们聊一个看似简单却经常被忽视的话题：**C++中的map和unordered_map到底有啥区别**？

选错了容器，你的程序可能就慢了 10 倍不止！这可不是危言耸听，而是实打实的性能差距。

> 💡 学习建议：
>
>动手写代码对比：分别用 map 和 unordered_map 处理同样的数据量，打印执行时间，感受差异。
>
>阅读源码或调试 STL 容器实现：尤其是 std::_Rb_tree 和哈希桶实现，对理解底层机制非常有帮助。
>
>不要迷信 unordered_map 更快——哈希冲突严重时，它甚至可能比 map 还慢！
>
>关注内存占用：哈希表为追求速度会牺牲内存，某些嵌入式或高性能场景要谨慎。
>
>理解场景优先级：查找多、插入少、顺序重要 → map；随机访问多、不关心顺序 → unordered_map。
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

## 一、一个真实的"血泪"故事

前几天我同事小王一脸沮丧地走过来："我的程序怎么这么慢啊，数据量一大就卡得不行..."

我瞄了一眼他的代码，发现他在处理几十万条数据时用的是`map`，而不是`unordered_map`。简单改了一下容器类型后，程序速度立马提升了 8 倍多！

小王震惊了："啥？就改个容器名字，速度差这么多？"

是的，就是这么神奇！今天我就带大家彻底搞清楚这两个容器的区别，以后再也不踩这个坑。

## 二、它们到底是啥？

### map：有序的绅士

`map`就像一个有序的字典，它会**自动把你放进去的键值对按键排序**。

```cpp
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> scoreMap;
    
    scoreMap["Zhang"] = 85;
    scoreMap["Li"] = 92;
    scoreMap["Wang"] = 78;
    
    // 遍历时会按键的字母顺序输出
    for (const auto& student : scoreMap) {
        std::cout << student.first << ": " << student.second << std::endl;
    }
    
    return 0;
}

// 输出结果：
// Li: 92
// Wang: 78
// Zhang: 85
// （按照字母顺序）
```

### unordered_map：随性的混世魔王

`unordered_map`则像个不讲究顺序的字典，它只关心能不能快速找到东西，至于排序？不存在的！

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> scoreMap;
    
    scoreMap["Zhang"] = 85;
    scoreMap["Li"] = 92;
    scoreMap["Wang"] = 78;
    
    // 遍历时输出顺序不确定
    for (const auto& student : scoreMap) {
        std::cout << student.first << ": " << student.second << std::endl;
    }
    
    return 0;
}

// 可能的输出结果：
// Wang: 78
// Zhang: 85
// Li: 92
// （顺序可能每次运行都不同）
```

## 三、它们内部是咋实现的？
### map：红黑树（有规有矩的大家族）
`map`内部是用 **红黑树** 实现的。红黑树是一种自平衡的二叉查找树。

想象一下，如果把`map`比作一个图书馆：

+ 每本书（键值对）都有固定的位置
+ 所有书按书名（键）字母顺序排列
+ 要找一本书，图书管理员会从中间的书架开始，然后告诉你"往左边找"或"往右边找"
+ 找书的过程就像二分查找

```plain
// map的简化结构示意图（红黑树）
          D
        /   \
       B     F
      / \   / \
     A   C E   G
```

在上面的图中，每个字母代表一个键。查找键"E"的过程：

1. 从根节点"D"开始
2. "E"比"D"大，向右走到"F"
3. "E"比"F"小，向左走到"E"
4. 找到了！

### unordered_map：哈希表（杂乱但高效的仓库）
`unordered_map`内部是用**哈希表**实现的。

继续用图书馆打比方：

+ 这是一个特殊的图书馆，没有明显的排序
+ 但图书管理员有一个神奇的公式，输入书名就能直接告诉你书在哪个架子上
+ 你直接去那个架子就能找到书，不需要一步步查找
+ 这个"神奇公式"就是哈希函数

```plain
// unordered_map的简化结构示意图（哈希表）
桶0: [C] -> [K]
桶1: [A]
桶2: 
桶3: [D] -> [H]
桶4: [B]
桶5: [G]
桶6: [F] -> [J]
桶7: [E] -> [I]
```

在上图中，每个字母代表一个键。查找键"H"的过程：

1. 计算"H"的哈希值，假设结果为 3
2. 直接检查桶 3
3. 桶 3 有一个链表，检查链表中的每个元素
4. 找到"H"！

## 四、性能对比：差距到底有多大？
让我们做个全面的性能对比，分别测试插入、查找、删除和修改这四种操作：

```cpp
#include <iostream>
#include <map>
#include <unordered_map>
#include <chrono>
#include <string>
#include <vector>
#include <random>

// 计时辅助函数
template<typename Func>
long long timeOperation(Func func) {
    auto start = std::chrono::high_resolution_clock::now();
    func();
    auto end = std::chrono::high_resolution_clock::now();
    return std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
}

int main() {
    const int COUNT = 100000;
    
    // 准备随机数据
    std::vector<int> keys;
    for (int i = 0; i < COUNT; i++) {
        keys.push_back(i);
    }
    
    // 打乱顺序用于随机访问
    std::random_device rd;
    std::mt19937 g(rd());
    std::shuffle(keys.begin(), keys.end(), g);
    
    std::map<int, int> orderedMap;
    std::unordered_map<int, int> unorderedMap;
    
    // 1. 插入性能
    auto mapInsertTime = timeOperation([&]() {
        for (int i = 0; i < COUNT; i++) {
            orderedMap[i] = i * 2;
        }
    });
    
    auto unorderedMapInsertTime = timeOperation([&]() {
        for (int i = 0; i < COUNT; i++) {
            unorderedMap[i] = i * 2;
        }
    });
    
    // 2. 查找性能（顺序访问）
    auto mapLookupTime = timeOperation([&]() {
        int result = 0;
        for (int i = 0; i < COUNT; i++) {
            result += orderedMap[i];
        }
        // 防止编译器优化掉
        volatile int dummy = result;
    });
    
    auto unorderedMapLookupTime = timeOperation([&]() {
        int result = 0;
        for (int i = 0; i < COUNT; i++) {
            result += unorderedMap[i];
        }
        // 防止编译器优化掉
        volatile int dummy = result;
    });
    
    // 3. 查找性能（随机访问）
    auto mapRandomLookupTime = timeOperation([&]() {
        int result = 0;
        for (int key : keys) {
            result += orderedMap[key];
        }
        // 防止编译器优化掉
        volatile int dummy = result;
    });
    
    auto unorderedMapRandomLookupTime = timeOperation([&]() {
        int result = 0;
        for (int key : keys) {
            result += unorderedMap[key];
        }
        // 防止编译器优化掉
        volatile int dummy = result;
    });
    
    // 4. 修改性能
    auto mapUpdateTime = timeOperation([&]() {
        for (int i = 0; i < COUNT; i++) {
            orderedMap[i] = i * 3;
        }
    });
    
    auto unorderedMapUpdateTime = timeOperation([&]() {
        for (int i = 0; i < COUNT; i++) {
            unorderedMap[i] = i * 3;
        }
    });
    
    // 5. 删除性能
    auto mapEraseTime = timeOperation([&]() {
        for (int key : keys) {
            if (key % 2 == 0) {  // 删除一半的元素
                orderedMap.erase(key);
            }
        }
    });
    
    auto unorderedMapEraseTime = timeOperation([&]() {
        for (int key : keys) {
            if (key % 2 == 0) {  // 删除一半的元素
                unorderedMap.erase(key);
            }
        }
    });
    
    // 打印结果
    std::cout << "操作\t\tmap(微秒)\tunordered_map(微秒)\t性能比" << std::endl;
    std::cout << "插入\t\t" << mapInsertTime << "\t\t" << unorderedMapInsertTime 
              << "\t\t\t" << (float)mapInsertTime / unorderedMapInsertTime << std::endl;
    
    std::cout << "顺序查找\t" << mapLookupTime << "\t\t" << unorderedMapLookupTime 
              << "\t\t\t" << (float)mapLookupTime / unorderedMapLookupTime << std::endl;
    
    std::cout << "随机查找\t" << mapRandomLookupTime << "\t\t" << unorderedMapRandomLookupTime 
              << "\t\t\t" << (float)mapRandomLookupTime / unorderedMapRandomLookupTime << std::endl;
    
    std::cout << "修改\t\t" << mapUpdateTime << "\t\t" << unorderedMapUpdateTime 
              << "\t\t\t" << (float)mapUpdateTime / unorderedMapUpdateTime << std::endl;
    
    std::cout << "删除\t\t" << mapEraseTime << "\t\t" << unorderedMapEraseTime 
              << "\t\t\t" << (float)mapEraseTime / unorderedMapEraseTime << std::endl;
    
    return 0;
}

// 输出结果：
// 操作            map(微秒)       unordered_map(微秒)     性能比
// 插入            225419          116690                  1.93178
// 顺序查找        103715          20122                   5.15431
// 随机查找        127432          25890                   4.92205
// 修改            104918          20597                   5.09385
// 删除            130040          29996                   4.33524
```

### 性能分析：
从测试结果可以清晰地看出：

1. **插入操作**：`unordered_map`比`map`快约1.9倍。这是因为`map`每次插入都需要维护红黑树的平衡，而`unordered_map`只需计算哈希值并放入对应的桶。
2. **查找操作**：
    - 顺序查找：`unordered_map`比`map`快约5.2倍
    - 随机查找：`unordered_map`比`map`快约4.9倍

查找是`unordered_map`最显著的优势，无论是顺序还是随机访问模式下都有明显提升。

3. **修改操作**：`unordered_map`比`map`快约5.1倍。修改操作本质上是先查找再赋值，所以性能差距与查找操作接近。
4. **删除操作**：`unordered_map`比`map`快约4.3倍。`map`删除元素后可能需要重新平衡树，而`unordered_map`只需从哈希表中删除节点。

### 小结：
综合来看，`unordered_map`在所有操作上都显著优于`map`，特别是在查找和修改操作上，性能提升达到了5倍左右。这意味着在大多数不需要有序遍历的场景下，`unordered_map`是更优的选择。

记住这些差异，在实际开发中选择合适的容器，可以为你的程序带来显著的性能提升。

## 五、什么时候用哪个？
### 用 map 的情况
1、 **需要有序遍历**：如果你需要按键的顺序遍历元素

```cpp
// 想按学生姓名字母顺序打印成绩单
std::map<std::string, int> scoreCard;
// ... 添加数据 ...
for (const auto& item : scoreCard) {
    std::cout << item.first << ": " << item.second << std::endl;
}
```

2、 **需要范围查询**：找出所有键在某个范围内的元素

```cpp
// 查找所有名字在A到C之间的学生
auto start = scoreCard.lower_bound("A");
auto end = scoreCard.upper_bound("C");
for (auto it = start; it != end; ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```

3、 **需要稳定的性能**：最坏情况下查找复杂度是确定的O(log n)

### 用 unordered_map 的情况

1、 **只关心查找速度**：大多数情况下只是用来查找，不关心顺序

```cpp
// 快速查找某个学生的成绩
std::unordered_map<std::string, int> scoreDB;
// ... 添加数据 ...
std::cout << "Zhang's score: " << scoreDB["Zhang"] << std::endl;
```

2、 **数据量大**：对于大数据量，unordered_map 的常数时间复杂度 O(1) 明显优于 map 的 O(log n)

3、 **不需要排序**：如果不需要按键排序，就没必要付出排序的成本

## 六、常见坑点

### 坑1：自定义类型做键

对于`unordered_map`，如果用自定义类型作为键，必须提供哈希函数和相等比较函数：

```cpp
struct Student {
    std::string name;
    int id;
    
    bool operator==(const Student& other) const {
        return id == other.id;
    }
};

// 为Student提供哈希函数
namespace std {
    template<>
    struct hash<Student> {
        size_t operator()(const Student& s) const {
            return hash<int>()(s.id);
        }
    };
}

// 现在可以使用了
std::unordered_map<Student, int> studentScores;
```

### 坑2：性能波动
`unordered_map`在某些情况下可能会遇到哈希冲突，导致性能下降。如果你的应用对性能稳定性要求高，可能需要考虑使用`map`。

### 坑3：内存占用
`unordered_map`通常比`map`消耗更多内存，因为哈希表为了降低冲突概率，会预留一定的空间。

## 七、实际使用建议
1. **默认选择unordered_map**：除非有特殊需求，一般情况下优先使用`unordered_map`
2. **测试决定**：对于性能关键的代码，最好实际测试两种容器的性能差异
3. **根据需求选择**：如果需要有序遍历或范围查询，选择`map`；如果只需要快速查找，选择`unordered_map`
4. **考虑数据规模**：数据量越大，两者的性能差距可能越明显

## 总结
选对容器，事半功倍；选错容器，徒增烦恼。

+ **map**：有序、稳定、支持范围查询，但速度较慢（O(log n)）
+ **unordered_map**：无序、速度快（O(1)），但内存占用较大，且不支持范围查询

记住这些区别，下次写代码时，就能轻松选择正确的容器，让你的程序飞起来！

你遇到过因为选错容器导致的性能问题吗？欢迎在评论区分享你的"血泪"经历~

---

## 🔍 拨开性能迷雾，探索 C++ 之美

如果你也对性能优化着迷，或想让你的代码更加高效，不妨关注我的公众号「**跟着小康学编程**」。

在这里，我用简单的比喻解释复杂概念，用有趣的案例展示技术原理。无论是 STL 容器选择，还是内存优化策略，都能找到通俗易懂的解答。

**每周更新，不见不散！** 👋

如果你觉得这篇文章对你有帮助，别忘了**点赞、在看、分享**哦~ 你的支持是我持续创作的动力！

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