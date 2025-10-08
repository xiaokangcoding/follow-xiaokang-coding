大家好，我是小康。

## 前言：你可能一直在用"笨办法"写代码
哥们，说实话，你是不是还在用那种写了10行代码才能搞定的事情？比如找个最大值要写个for循环，判断个条件要if套if？

我之前也是这样，直到我发现了STL里藏着的这些"黑科技"。用了之后，代码不仅变短了，运行速度还提升了几倍！同事们都问我是不是偷偷上了什么培训班。

今天就把这些压箱底的宝贝分享给你，保证你看完之后会拍大腿说："卧槽，还有这种操作！"


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

---

## 1. `std::iota` - 数列生成器，告别手写循环
**以前的你：**

```cpp
vector<int> nums(10);
for(int i = 0; i < 10; i++) {
    nums[i] = i + 1;
}
```

**现在的你：**

```cpp
#include <numeric>
vector<int> nums(10);
std::iota(nums.begin(), nums.end(), 1);
// 输出：1 2 3 4 5 6 7 8 9 10
```

看到没？一行代码搞定！`iota`这个名字来自希腊字母，专门用来生成连续数列。想生成1到100？想生成-5到5？随你折腾！

```cpp
vector<int> countdown(5);
std::iota(countdown.begin(), countdown.end(), -2);
// 输出：-2 -1 0 1 2
```

---

## 2. `std::adjacent_find` - 找重复邻居的神器
你有没有遇到过这种情况：要找数组里第一个重复的相邻元素？比如找字符串里的连续重复字符？

**笨办法：**

```cpp
string s = "programming";
int pos = -1;
for(int i = 0; i < s.length() - 1; i++) {
    if(s[i] == s[i+1]) {
        pos = i;
        break;
    }
}
```

**聪明做法：**

```cpp
#include <algorithm>  // 这个是关键！adjacent_find在这里
#include <string>     

string s = "programming";
auto it = std::adjacent_find(s.begin(), s.end());
if(it != s.end()) {
    cout << "找到重复字符: " << *it << " 位置: " << it - s.begin() << endl;
}
// 输出：找到重复字符: m 位置: 6
```

这个函数还能自定义比较条件！比如找第一对差值大于10的相邻数字：

```cpp
vector<int> nums = {1, 3, 5, 18, 20};
auto it = std::adjacent_find(nums.begin(), nums.end(), 
    [](int a, int b) { return abs(a - b) > 10; });
// 找到5和18这对冤家
```

---

## 3. `std::inner_product` - 向量运算的瑞士军刀
这个函数名字听起来很数学，但其实超级实用！不仅能计算点积，还能做各种神奇的运算。

**计算两个数组的点积：**

```cpp
#include <numeric>  

vector<int> a = {1, 2, 3};
vector<int> b = {4, 5, 6};
int result = std::inner_product(a.begin(), a.end(), b.begin(), 0);
cout << result << endl;  // 输出：32 (1*4 + 2*5 + 3*6)
```

**更骚的操作 - 自定义运算：**

```cpp
// 计算两个数组对应位置的差值之和
vector<int> prices_yesterday = {100, 200, 150};
vector<int> prices_today = {105, 195, 160};
int total_change = std::inner_product(
    prices_today.begin(), prices_today.end(), 
    prices_yesterday.begin(), 0,
    std::plus<>(),  // 累加
    std::minus<>()  // 相减
);
cout << "总体价格变化: " << total_change << endl;  // 输出：10
```

---

## 4. `std::nth_element` - 找第N大元素的速度怪兽
要找第K大的元素，你是不是还在排序然后取下标？兄弟，这样太慢了！

**传统做法（慢）：**

```cpp
vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
sort(nums.begin(), nums.end());
cout << nums[2] << endl;  // 第3小的元素
```

**高效做法：**

```cpp
#include <algorithm>

vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
std::nth_element(nums.begin(), nums.begin() + 2, nums.end());
cout << nums[2] << endl;  // 输出：2
// 而且nums[2]左边的都比它小，右边的都比它大！
```

这个函数的时间复杂度是O(n)，比排序的O(nlogn)快多了！找中位数、找前K大元素都是它的拿手好戏。

---

## 5. `std::partial_sum` - 前缀和计算机
算前缀和还在手写双重循环？这个函数一步到位！

```cpp
#include <numeric>  // partial_sum在这里

vector<int> nums = {1, 2, 3, 4, 5};
vector<int> prefix_sum(nums.size());
std::partial_sum(nums.begin(), nums.end(), prefix_sum.begin());
// prefix_sum: [1, 3, 6, 10, 15]

for(int x : prefix_sum) {
    cout << x << " ";
}
```

还能玩出花来：

```cpp
vector<int> nums = {2, 3, 4, 5};
vector<int> products(nums.size());
std::partial_sum(nums.begin(), nums.end(), products.begin(), 
                 std::multiplies<int>());
// products: [2, 6, 24, 120] (累乘)
```

---

## 6. `std::rotate` - 数组旋转大师
左旋转、右旋转，一个函数全搞定！

```cpp
#include <algorithm>  // std::rotate在这里

vector<int> nums = {1, 2, 3, 4, 5, 6, 7};
// 向左旋转3位
std::rotate(nums.begin(), nums.begin() + 3, nums.end());
// 结果：[4, 5, 6, 7, 1, 2, 3]

for(int x : nums) {
    cout << x << " ";
}
```

字符串操作也超级方便：

```cpp
string s = "abcdefg";
std::rotate(s.begin(), s.begin() + 2, s.end());
cout << s << endl;  // 输出：cdefgab
```

---

## 7. `std::set_intersection` - 求交集的专家
两个有序数组求交集，不用自己写双指针了！

```cpp
#include <algorithm>  // std::set_intersection在这里

vector<int> a = {1, 2, 3, 4, 5};
vector<int> b = {3, 4, 5, 6, 7};
vector<int> result;

std::set_intersection(a.begin(), a.end(), b.begin(), b.end(),
                     std::back_inserter(result));
// result: [3, 4, 5]

cout << "交集: ";
for(int x : result) {
    cout << x << " ";
}
```

还有`set_union`（并集）、`set_difference`（差集）等兄弟函数，处理集合运算简直不要太爽！

---

## 8. `std::lexicographical_compare` - 字典序比较王者
真正的字典序比较，就像查字典一样！比较两个序列谁在"字典"里排在前面：

```cpp
#include <algorithm>  // std::lexicographical_compare在这里

vector<string> words1 = {"apple", "banana"};
vector<string> words2 = {"apple", "cherry"};

bool result = std::lexicographical_compare(
words1.begin(), words1.end(),
words2.begin(), words2.end()
);

cout << (result ? "words1 在字典中排在前面" : "words2 在字典中排在前面") << endl;
// 输出：words1 在字典中排在前面 (因为 banana < cherry)
```

字符串比较也很直观：

```cpp

string s1 = "abc";
string s2 = "abd";

bool result = std::lexicographical_compare(s1.begin(), s1.end(), 
s2.begin(), s2.end());
cout << (result ? "s1 < s2" : "s1 >= s2") << endl;  // 输出：s1 < s2
```

自定义比较规则，比如忽略大小写：

```cpp

string s1 = "Apple";
string s2 = "banana";

bool result = std::lexicographical_compare(
s1.begin(), s1.end(), s2.begin(), s2.end(),
[](char a, char b) { 
    return tolower(a) < tolower(b); 
}
);
// 按忽略大小写的字典序比较
```

---

## 9. `std::inplace_merge` - 原地归并排序
有两个已排序的数组段，想合并成一个？这个函数就是为你准备的！

```cpp
#include <algorithm>  // std::inplace_merge在这里

vector<int> nums = {1, 3, 5, 2, 4, 6};
// 前半部分[1,3,5]已排序，后半部分[2,4,6]已排序
std::inplace_merge(nums.begin(), nums.begin() + 3, nums.end());
// 结果：[1, 2, 3, 4, 5, 6]

for(int x : nums) {
    cout << x << " ";
}
```

内存效率超高，时间复杂度O(n)，归并排序的核心就是它！

---

## 10. `std::sample` - 随机采样专家
从大数据集里随机选几个样本？这个函数帮你搞定！

```cpp
#include <random>
#include <algorithm>  // std::sample在这里

vector<int> data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
vector<int> samples;

std::random_device rd;
std::mt19937 gen(rd());

std::sample(data.begin(), data.end(), std::back_inserter(samples), 
           3, gen);  // 随机选3个

cout << "随机样本: ";
for(int x : samples) {
    cout << x << " ";
}
// 可能输出：随机样本: 2 7 9
```

做数据分析、机器学习预处理的时候超级有用！

---

## 11. `std::partition` - 数据分类专家
想把数组按条件分成两部分？比如把奇数放前面，偶数放后面？

```cpp
#include <algorithm>  // std::partition在这里

vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9};
auto it = std::partition(nums.begin(), nums.end(), 
                        [](int x) { return x % 2 == 1; });

cout << "奇数部分: ";
for(auto i = nums.begin(); i != it; ++i) {
    cout << *i << " ";  // 输出：1 9 3 7 5
}
cout << "\n偶数部分: ";
for(auto i = it; i != nums.end(); ++i) {
    cout << *i << " ";  // 输出：6 4 8 2
}
```

还有个更稳定的版本`std::stable_partition`，能保持原有的相对顺序！

---

## 12. `std::mismatch` - 找不同点的侦探
比较两个序列，找出第一个不同的位置：

```cpp
#include <algorithm>  // std::mismatch在这里

string s1 = "programming";
string s2 = "programing";  // 少了一个m

auto result = std::mismatch(s1.begin(), s1.end(), s2.begin());
if(result.first != s1.end()) {
    cout << "第一个不同的位置: " << (result.first - s1.begin()) << endl;
    cout << "字符分别是: '" << *result.first << "' 和 '" << *result.second << "'" << endl;
}
// 输出：第一个不同的位置: 7
//      字符分别是: 'm' 和 'i'
```

做字符串diff、数据校验的时候特别有用！

---

## 13. `std::equal_range` - 二分查找三件套
在有序数组里查找某个值的所有出现位置，一次性给你上下边界：

```cpp
#include <algorithm>  // std::equal_range在这里

vector<int> nums = {1, 2, 2, 2, 3, 4, 5};
auto range = std::equal_range(nums.begin(), nums.end(), 2);

cout << "数字2的范围: [" << (range.first - nums.begin()) 
     << ", " << (range.second - nums.begin()) << ")" << endl;
// 输出：数字2的范围: [1, 4)

cout << "数字2出现了 " << (range.second - range.first) << " 次" << endl;
// 输出：数字2出现了 3 次
```

比`lower_bound`和`upper_bound`分别调用要高效！

---

## 14. `std::clamp` - 数值限制器
把数值限制在某个范围内，超出就截断：

```cpp
#include <algorithm>

int score = 150;
int clamped = std::clamp(score, 0, 100);  // 限制在0-100之间
cout << "原始分数: " << score << ", 限制后: " << clamped << endl;
// 输出：原始分数: 150, 限制后: 100

double temperature = -10.5;
double safe_temp = std::clamp(temperature, -5.0, 35.0);
cout << "温度调节: " << temperature << " -> " << safe_temp << endl;
// 输出：温度调节: -10.5 -> -5
```

游戏开发、数据处理时经常用到！

---

## 15. `std::generate_n` - 批量生成器
想生成N个随机数？想批量初始化数据？这个函数帮你搞定：

```cpp
#include <random>
#include <algorithm>

vector<int> random_nums(10);
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<> dis(1, 100);

std::generate_n(random_nums.begin(), 10, [&]() { return dis(gen); });

cout << "10个随机数: ";
for(int x : random_nums) {
    cout << x << " ";
}
```

还能生成其他类型的数据：

```cpp
vector<string> passwords(5);
std::generate_n(passwords.begin(), 5, []() {
    return "pwd_" + to_string(rand() % 1000);
});
// 生成5个随机密码
```

---

## 16. `std::search_n` - 连续元素搜索专家
找连续出现N次的元素？比如找股票连续3天涨停，或者找字符串里连续的空格？

```cpp
#include <algorithm>  // std::search_n在这里

vector<int> stock_changes = {1, -1, 1, 1, 1, -1, 1, 1};  // 1表示涨，-1表示跌
auto it = std::search_n(stock_changes.begin(), stock_changes.end(), 3, 1);

if(it != stock_changes.end()) {
    cout << "找到连续3天上涨，开始位置: " << (it - stock_changes.begin()) << endl;
    // 输出：找到连续3天上涨，开始位置: 2
}

// 字符串中找连续空格
string text = "hello    world";  // 4个连续空格
auto pos = std::search_n(text.begin(), text.end(), 3, ' ');
if(pos != text.end()) {
    cout << "找到3个连续空格，位置: " << (pos - text.begin()) << endl;
}
```

还能自定义条件：

```cpp
vector<int> temps = {25, 26, 27, 28, 29, 20, 15};  // 温度数据
// 找连续3天温度都超过25度
auto it = std::search_n(temps.begin(), temps.end(), 3, 25, 
                       [](int actual, int threshold) { return actual > threshold; });

if(it != temps.end()) {
    cout << "找到连续3天超过25度，开始位置: " << (it - temps.begin()) << endl;
    // 输出：找到连续3天超过25度，开始位置: 0 (25,26,27都>25)
}
```

---

## 17. `std::partial_sort` - 部分排序高手
只想要前K个最大/最小值？不用全排序！这个函数比`sort`快多了：

```cpp
#include <algorithm>

vector<int> scores = {85, 92, 78, 96, 88, 73, 91, 89, 94, 77};
// 只要前3名，不用全部排序
std::partial_sort(scores.begin(), scores.begin() + 3, scores.end(), std::greater<int>());

cout << "前3名分数: ";
for(int i = 0; i < 3; i++) {
    cout << scores[i] << " ";  
}
// 输出：前3名分数: 96 94 92

// 后面的元素顺序是未定义的，但前3个一定是最大的3个！
```

找中位数也超级高效：

```cpp
vector<double> data = {3.2, 1.5, 4.8, 2.1, 5.7, 1.2, 6.3, 2.9};
int mid = data.size() / 2;
std::partial_sort(data.begin(), data.begin() + mid + 1, data.end());
cout << "中位数: " << data[mid] << endl;
```

**性能对比**：

+ `sort`全排序：O(n log n)
+ `partial_sort`前K个：O(n log k) - 当k很小时，快很多！

---

## 18. `std::unique_copy` - 去重复制专家
想去重但不想修改原数组？这个函数帮你复制一份去重后的数据：

```cpp
#include <algorithm>

vector<int> nums = {1, 1, 2, 2, 2, 3, 3, 4, 5, 5};
vector<int> unique_nums;

// 前提：原数组要先排序！
std::unique_copy(nums.begin(), nums.end(), std::back_inserter(unique_nums));

cout << "原数组: ";
for(int x : nums) cout << x << " ";
cout << "\n去重后: ";
for(int x : unique_nums) cout << x << " ";
// 输出：去重后: 1 2 3 4 5
```

处理字符串也很方便：

```cpp
string text = "aabbccddee";
string result;
std::unique_copy(text.begin(), text.end(), std::back_inserter(result));
cout << "去重字符串: " << result << endl;  // 输出：abcde
```

**高级用法** - 自定义去重条件：

```cpp
vector<string> words = {"hello", "HELLO", "world", "WORLD", "test"};
vector<string> unique_words;

// 按忽略大小写去重
std::sort(words.begin(), words.end(), [](const string& a, const string& b) {
    string lower_a = a, lower_b = b;
    transform(lower_a.begin(), lower_a.end(), lower_a.begin(), ::tolower);
    transform(lower_b.begin(), lower_b.end(), lower_b.begin(), ::tolower);
    return lower_a < lower_b;
});
for(string x : words) cout << x << " ";

std::unique_copy(words.begin(), words.end(), std::back_inserter(unique_words),
                [](const string& a, const string& b) {
                    string lower_a = a, lower_b = b;
                    transform(lower_a.begin(), lower_a.end(), lower_a.begin(), ::tolower);
                    transform(lower_b.begin(), lower_b.end(), lower_b.begin(), ::tolower);
                    return lower_a == lower_b;
                });
for(string x : unique_words) cout << x << " ";
// 结果：保留一个hello和一个world
```

**使用技巧**：

+ `unique_copy`要求输入已排序
+ 用`back_inserter`可以自动扩容目标容器
+ 可以配合自定义比较函数做复杂去重

---

## 奖励关：几个超级实用的组合技
### 技巧1：快速去重并保持顺序
```cpp
vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6, 5};
// 方法：先排序，再unique
sort(nums.begin(), nums.end());
nums.erase(std::unique(nums.begin(), nums.end()), nums.end());
// 结果：[1, 2, 3, 4, 5, 6, 9]
```

### 技巧2：统计满足条件的元素个数
```cpp
vector<int> scores = {85, 92, 78, 96, 88, 73, 91};
int high_scores = std::count_if(scores.begin(), scores.end(), 
                               [](int x) { return x >= 90; });
cout << "高分人数: " << high_scores << endl;  // 输出：3
```

### 技巧3：检查是否全部/任意元素满足条件
```cpp
vector<int> ages = {18, 19, 20, 21, 22};

bool all_adult = std::all_of(ages.begin(), ages.end(), 
                            [](int x) { return x >= 18; });
cout << "都是成年人: " << (all_adult ? "是" : "否") << endl;

bool has_senior = std::any_of(ages.begin(), ages.end(), 
                             [](int x) { return x >= 65; });
cout << "有老年人: " << (has_senior ? "是" : "否") << endl;
```

---

## 总结：从此告别"笨代码"
这18个STL算法就像武功秘籍一样，每一个都能让你的代码变得更短、更快、更优雅。关键是它们都经过了高度优化，性能比你手写的循环要好得多。

记住几个使用技巧：

1. **头文件要记牢** - `<algorithm>`、`<numeric>`、`<iterator>`是三大宝库
2. **善用lambda表达式** - 自定义比较条件超级灵活，比写函数对象简单多了
3. **迭代器是万金油** - 不只是vector，string、deque、list、set都能用
4. **性能优先选择** - `nth_element`比排序快，`partial_sort`比全排序快，根据需求选最合适的
5. **组合使用威力大** - 比如`sort + unique`去重，`partition + sort`分类排序
6. **前置条件要注意** - 很多算法要求输入已排序（如`unique`、`binary_search`）
7. **善用插入迭代器** - `back_inserter`、`front_inserter`让容器自动扩容
8. **自定义比较函数** - 大部分算法都支持传入比较函数，让功能更强大

下次写代码之前，先问问自己："有没有STL算法能帮我？"说不定就能找到现成的轮子，何必自己造呢？

最后送你一句话：**好的程序员不是写代码最多的，而是写代码最少的**。用好STL，让你的代码简洁如诗！

---

好了，今天的STL算法分享就到这里！如果你觉得有用的话，别忘了点个赞和在看，分享让更多小伙伴看到这些实用技巧。

我是小康，专注Linux C/C++后台开发，定期分享一些实用的编程干货和踩坑经验。如果你也在做后台开发，或者想学习相关技术，欢迎关注我的公众号「**跟着小康学编程**」，咱们一起交流进步！

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

