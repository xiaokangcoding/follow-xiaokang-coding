哈喽，大家好，我是小康！

今天要和大家聊一个特别有意思的话题——**基数树**。

说实话，我第一次听到这个名词的时候，内心是懵逼的。基数？树？这玩意儿到底是啥？

直到有一天，我在研究TCMalloc内存池源码的时候，发现了一个神奇的现象：为什么Google的工程师不用`std::unordered_map`来做页号映射，而要自己实现一个看起来很复杂的数据结构？

带着这个疑问，我深入研究了一下，结果发现了一个宝藏——**基数树**！

## 一个让我震惊的性能对比

先来看个数据，直接震撼你的小心脏：

```plain
测试场景：100万次查找操作
std::unordered_map：    40878微秒
基数树：               714微秒
性能提升：             57.2521倍！
```

这还只是中等规模的测试，在大规模场景下，差距会更加夸张。

## 基数树到底是个啥玩意儿？
好，现在我用最通俗的话给大家解释一下基数树。

**简单粗暴地说，基数树就是一个超级大数组！**

没错，就这么简单。

比如你要存储键值对，传统的做法是：

+ `std::map`：用红黑树，需要比较、旋转，O(log n)时间
+ `std::unordered_map`：用哈希表，需要计算哈希、处理冲突，平均O(1)但有常数开销

而基数树呢？直接把键当作数组的下标！

```cpp
// 传统方式
unordered_map[123] = ptr;  // 需要计算hash(123)，可能还要处理冲突

// 基数树方式
array[123] = ptr;          // 直接访问！O(1)且常数极小
```

是不是超级简单？

## 动手实现一个基数树（一级）
talk is cheap，show me the code！

```cpp
const uint32_t MAX_KEY = (1 << 20) - 1;

// ============ 基数树实现 ============
template<int BITS = 20>  // 20位，支持100万条目 800 0000
class SimpleRadixTree {
private:
    static const size_t SIZE = 1 << BITS;
    void** array_;
    
public:
    SimpleRadixTree() {
        array_ = new void*[SIZE];
        memset(array_, 0, SIZE * sizeof(void*));
        
        cout << "创建了支持 " << SIZE << " 条目的基数树" << endl;
        cout << "理论内存: " << (SIZE * 8) / (1024) << " KB" << endl;
    }
    
    ~SimpleRadixTree() {
        delete[] array_;
    }
    
    // 设置键值对 - O(1)时间！
    void set(uint32_t key, void* value) {
        if (key >= SIZE) {
            cerr << "错误: 键 " << key << " 超出范围 [0, " << (SIZE-1) << "]" << endl;
            return;
        }
        array_[key] = value;
    }
    
    // 获取值 - O(1)时间！
    void* get(uint32_t key) {
        if (key >= SIZE) return nullptr;
        return array_[key];
    }
    
    // 删除键
    void remove(uint32_t key) {
        if (key < SIZE) {
            array_[key] = nullptr;
        }
    }
    
    // 统计已使用的条目数
    size_t count_used() const {
        size_t count = 0;
        for (size_t i = 0; i < SIZE; ++i) {
            if (array_[i] != nullptr) {
                count++;
            }
        }
        return count;
    }
    
    // 获取支持的最大键值
    uint32_t max_key() const {
        return SIZE - 1;
    }
};

```

就这么几行代码，我们就实现了一个高性能的一级基数树！

> 基数树的核心：**用空间换时间**。
>

## 性能大PK：基数树 VS unordered_map
我专门写了个测试程序，在我的Ubuntu机器上跑了一下：

```cpp
void performance_battle() {
    cout << "\n========== 性能大PK开始 ==========" << endl;
    
    const int TEST_COUNT = 1000000;
    
    // 准备测试数据
    vector<uint32_t> keys;
    cout << "准备 " << TEST_COUNT << " 个测试键..." << endl;
    for(int i = 0; i < TEST_COUNT; i++) {
        //keys.push_back(i * 7);  // 一些分散的键，避免连续访问的缓存优势
        keys.push_back(i);  
    }
    
    cout << "数据准备完成，开始测试..." << endl;
    
    // ========== 测试 unordered_map ==========
    cout << "\n测试 unordered_map..." << endl;
    unordered_map<uint32_t, void*> hashmap;
    auto start = chrono::high_resolution_clock::now();
    
    // 插入测试
    for(int i = 0; i < TEST_COUNT; i++) {
        hashmap[keys[i]] = reinterpret_cast<void*>(i + 1);
    }
    
    // 查找测试
    for(int i = 0; i < TEST_COUNT; i++) {
        volatile void* result = hashmap[keys[i]];
        (void)result; // 防止编译器优化掉
    }
    
    auto end = chrono::high_resolution_clock::now();
    auto hashmap_time = chrono::duration_cast<chrono::microseconds>(end - start);
    
    // ========== 测试基数树 ==========
    cout << "测试基数树..." << endl;
    SimpleRadixTree<20> radix_tree;
    start = chrono::high_resolution_clock::now();
    
    // 插入测试
    for(int i = 0; i < TEST_COUNT; i++) {
        radix_tree.set(keys[i], reinterpret_cast<void*>(i + 1));
    }
    
    // 查找测试
    for(int i = 0; i < TEST_COUNT; i++) {
        volatile void* result = radix_tree.get(keys[i]);
        (void)result; // 防止编译器优化掉
    }
    
    end = chrono::high_resolution_clock::now();
    auto radix_time = chrono::duration_cast<chrono::microseconds>(end - start);
    
    // ========== 输出结果 ==========
    cout << "\n=== 性能大PK结果 ===" << endl;
    cout << "测试规模:       " << TEST_COUNT << " 次操作" << endl;
    cout << "unordered_map:  " << hashmap_time.count() << " 微秒" << endl;
    cout << "基数树:         " << radix_time.count() << " 微秒" << endl;
    
    if (radix_time.count() > 0) {
        double speedup = (double)hashmap_time.count() / radix_time.count();
        cout << "性能提升:       " << speedup << " 倍" << endl;
        
        if (speedup > 5) {
            cout << "基数树性能碾压！" << endl;
        } else if (speedup > 2) {
            cout << " 基数树明显更快！" << endl;
        } else {
            cout << "性能相当" << endl;
        }
    }
    
    // 验证结果正确性
    cout << "\n验证结果正确性..." << endl;
    bool correct = true;
    for(int i = 0; i < 1000; i++) {  // 验证前1000个结果
        void* hash_result = hashmap[keys[i]];
        void* radix_result = radix_tree.get(keys[i]);
        if (hash_result != radix_result) {
            cout << "结果不一致！键: " << keys[i] << endl;
            correct = false;
            break;
        }
    }
    
    if (correct) {
        cout << "结果验证通过，两种方法结果一致！" << endl;
    }
}
```

在我的机器上跑出来的结果：

```plain
=== 性能大PK结果 ===
测试规模:       1000000 次操作
unordered_map:  40878 微秒
基数树:         714 微秒
性能提升:       57.2521 倍
```

**50+倍的性能提升！** 这还只是中等规模的测试。

为什么会有这么大的差距？

1. **基数树**：直接数组访问，没有任何额外计算
2. **unordered_map**：需要计算哈希值、处理冲突、内存分散访问

## 基数树在内存池中的神奇应用
现在来看看基数树的实际应用场景——内存池！

在内存池系统中，我们需要根据内存地址快速找到对应的内存块信息（Span）。

传统方式可能是这样：

```cpp
// 传统内存池的页号映射
unordered_map<PageID, Span*> page_to_span;

// 哈希查找，较慢
Span* find_span(void* ptr) {
    PageID page_id = (uintptr_t)ptr >> 12;  // 假设4KB页面
    std::lock_guard<std::mutex> lock(map_mutex_);// 还要加锁
    auto it = page_to_span.find(id);
    return (it != page_to_span.end()) ? it->second : nullptr;   
}
```

用基数树优化后：

```cpp
// 基数树版本的页号映射
SimpleRadixTree<28> page_map;  // 支持1TB地址空间

// 不需要加锁
Span* find_span(void* ptr) {
    PageID page_id = (uintptr_t)ptr >> 12;
    return (Span*)page_map.get(page_id);    // 直接数组访问，超快！
}
```

这就是为什么TCMalloc、jemalloc这些高性能内存池都在用基数树的原因！

## 一个完整的内存池应用示例
```cpp
// ============ 内存池应用演示 ============
struct Span {
    uint32_t page_id;
    uint32_t num_pages;
    bool in_use;
    void* free_list;
    
    Span(uint32_t pid, uint32_t pages, bool used = true) 
        : page_id(pid), num_pages(pages), in_use(used), free_list(nullptr) {}
};

class MemoryPool {
private:
    SimpleRadixTree<24> page_map_;  // 24位，支持16M页，相当于64GB地址空间
    vector<Span*> spans_;  // 保存所有Span，用于清理
    
public:
    ~MemoryPool() {
        for(auto* span : spans_) {
            delete span;
        }
    }
    
    // 注册内存块
    void register_span(Span* span) {
        spans_.push_back(span);
        
        // 为这个Span的每一页都建立映射
        for(uint32_t i = 0; i < span->num_pages; i++) {
            uint32_t page_id = span->page_id + i;
            page_map_.set(page_id, span);
        }
        
        cout << "注册Span: 起始页号=" << span->page_id 
             << ", 页数=" << span->num_pages << endl;
    }
    
    // 根据地址快速找到所属的内存块
    Span* addr_to_span(void* ptr) {
        // 假设页面大小为4KB，即PAGE_SHIFT=12
        uint32_t page_id = (reinterpret_cast<uintptr_t>(ptr) >> 12) & 0xFFFFFF;  // 取低24位
        return reinterpret_cast<Span*>(page_map_.get(page_id));
    }
    
    // 根据页号查找Span
    Span* page_to_span(uint32_t page_id) {
        return reinterpret_cast<Span*>(page_map_.get(page_id));
    }
    
    // 释放内存时，快速定位到对应的Span
    bool free_memory(void* ptr) {
        Span* span = addr_to_span(ptr);  // O(1)时间找到Span！
        if(span && span->in_use) {
            cout << "找到地址 " << ptr << " 对应的Span: 页号" << span->page_id << endl;
            return true;
        }
        return false;
    }

};
```

看到没？有了基数树，内存池的地址查找变成了O(1)操作，这对高并发场景下的内存分配性能提升是巨大的！

 当然，这里只是简单展示一下基数树的应用思路。实际的内存池项目要复杂得多，想学习完整实现的同学不妨看看：  

## 什么时候该用基数树？
基数树虽然牛逼，但也不是万能的。什么时候用呢？

**适合用基数树的场景：** 

+ 键的范围相对固定且已知
+ 键分布比较密集
+  对查找性能要求极高
+  内存充足的环境

**不适合用基数树的场景：** 

+ 键的范围非常大且分布稀疏
+  内存严重受限的环境
+  键是字符串等复杂类型

## 进阶：多层基数树
当键的范围太大时（比如64位地址空间），1层基数树就不够用了。这时候我们可以用多层基数树。

简单来说，就是把一个超级大数组拆分成多个小数组的组合：

```cpp
// 2层基数树示意
template<int BITS = 36>  // 支持Linux 64位地址空间  
class PageMap2 {
private:
    // 位数分配：尽量均匀分配
    static const int LEAF_BITS = BITS / 2;        // 18位
    static const int ROOT_BITS = BITS - LEAF_BITS; // 18位
    static const size_t ROOT_SIZE = 1ULL << ROOT_BITS; 
    static const size_t LEAF_SIZE = 1ULL << LEAF_BITS;
    
    struct Leaf {
        void* values[LEAF_SIZE];
        
        Leaf() {
            memset(values, 0, sizeof(values));
        }
    };
    
    Leaf* root_[ROOT_SIZE];
    
public:
    PageMap2() {
        memset(root_, 0, sizeof(root_));
        cout << "二级基数树初始化完成！" << endl;
        cout << "支持地址空间: " << (1ULL << BITS) << " 页 ("
             << ((1ULL << BITS) * 4096) / (1024ULL*1024*1024*1024) << " TB)" << endl;
        cout << "固定内存开销: " << sizeof(root_) / (1024*1024) << " MB" << endl;
    }
    
    ~PageMap2() {
        // 清理所有分配的叶子节点
        for(size_t i = 0; i < ROOT_SIZE; i++) {
            delete root_[i];
        }
    }
    
    // 获取值 - 两次数组访问，仍然是O(1)！
    void* get(uint64_t key) {
        if(key >= (1ULL << BITS)) return nullptr;
        
        uint64_t i1 = key >> LEAF_BITS;      // 高位作为一级索引
        uint64_t i2 = key & (LEAF_SIZE - 1); // 低位作为二级索引
        
        if(root_[i1] == nullptr) return nullptr;
        return root_[i1]->values[i2];
    }
    
    // 设置值 - 按需分配叶子节点
    void set(uint64_t key, void* value) {
        if(key >= (1ULL << BITS)) return;
        
        uint64_t i1 = key >> LEAF_BITS;
        uint64_t i2 = key & (LEAF_SIZE - 1);
        
        // 按需分配叶子节点
        if(root_[i1] == nullptr) {
            root_[i1] = new Leaf();
            cout << "分配新的叶子节点: " << i1 << endl;
        }
        
        root_[i1]->values[i2] = value;
    }
};
```

这样可以支持36位键空间（256TB地址），但内存是按需分配的，不会浪费。

在我最近带领学员做的高性能内存池项目中，我们就采用了这种二级基数树的设计思路，不过做了更多的工程优化。实际效果非常不错，相比原来的unordered_map方案，性能提升了10倍以上。  

感兴趣的朋友可以看这篇文章：

## 我的使用心得

用了基数树这么久，我总结几个心得：

1. **不要被"树"这个名字迷惑了**，基数树本质上就是数组
2. **在高性能场景下，基数树是神器**，特别是内存池、路由表这些场景
3. **多层基数树是进阶版本**，1层够用就别搞复杂

## 编译运行试试看

想要自己试试的同学，把文章中的代码整合一下，用这个命令编译：

```bash
g++ -std=c++11 -O2 radix_tree_test.cpp -o test
./test
```

在我的Ubuntu 20.04上跑得飞快！

## 写在最后

基数树真的是一个被低估的数据结构。

在合适的场景下，它能带来数倍甚至数十倍的性能提升。Google、Facebook这些大厂的底层库都在大量使用基数树，绝对不是没有道理的。

下次如果你遇到需要快速键值查找的场景，特别是键范围相对固定的情况，不妨试试基数树。说不定会有意想不到的惊喜！

好了，今天就聊到这里。如果你觉得这篇文章对你有帮助，记得**点赞、在看、转发**三连哦！

有什么问题欢迎在评论区讨论，我看到会第一时间回复的。

## C++ 项目实战课程

不过话说回来，这些底层性能优化技巧，光看理论是远远不够的，必须在真实项目中摸爬滚打才能融会贯通。  

这段时间我一直在带学员做一些硬核的C++项目，从底层数据结构到高并发组件，每个项目都是企业级的真实场景。看到不少同学通过这些实战项目，技术水平有了质的飞跃，面试时也更有底气了。

**目前正在招生的几个实战项目**：



这些项目都会深入涉及多线程编程、并发优化、 高并发处理、系统级性能调优等企业级开发的核心技术，而且是在真实的项目环境中应用。不是纸上谈兵，而是真刀真枪地写代码、调优化、解决实际问题。

**特别提醒**：现在是首期开班，价格是最低的。随着报名人数的增加和课程的不断完善，后续肯定会涨价一波。所以现在报名最划算，既能享受首期的优惠价格，又能获得完整的项目经验。

想报名的同学，赶紧加我微信：**jkfwdkf**，备注「**项目实战**」！

首期名额有限，先到先得。一起在实战中提升技术，在项目中积累经验！


**或者扫下方二维码加我**：(备注你感兴趣的项目名称)

![](https://files.mdnice.com/user/48364/a8c0b6f1-ed19-4151-9511-c8ab272ccbd5.png)
