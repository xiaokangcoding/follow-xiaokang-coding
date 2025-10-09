大家好，我是小康。

## 写在前面

你知道吗？在高并发场景下，频繁的`malloc`和`free`操作就像是程序的"阿喀琉斯之踵"，轻则拖慢系统响应，重则直接把服务器拖垮。

最近我从0到1实现了一个高性能内存池，经过严格的压测验证，在8B到2048B的分配释放场景下，性能相比传统的`malloc/free`平均快了**4.5倍**！今天就来给大家分享这个实现过程，相信看完后你也能写出自己的高性能内存池。

**数据最有说服力，来看看实测结果**:


![](https://files.mdnice.com/user/48364/7b3448c7-7126-45cc-ba6f-93d5058e3896.jpg)


看到了吗？**相比标准malloc/free，平均性能提升4.62倍，最高达到7.37倍！**

## 为什么需要内存池？
在开始撸代码之前，我们先来聊聊为什么要造这个轮子。

### 传统内存分配的痛点
你有没有遇到过这些情况：

1. **频繁分配小对象**：比如游戏服务器中每秒创建成千上万个临时对象
2. **内存碎片化**：明明还有很多空闲内存，但就是分配不出连续的大块
3. **性能瓶颈**：高并发场景下`malloc`成为系统的性能瓶颈
4. **内存泄漏**：忘记`free`导致的内存泄漏，让人头疼不已

这些问题的根源在于：**系统级的内存分配器设计得太通用了**。它要处理各种大小的内存请求，要考虑各种边界情况，这就导致了性能上的妥协。

### 内存池的优势
内存池就像是给程序开了个"专属食堂"：

+ **速度快**：预先分配好内存，拿来就用，不用每次都找系统要
+ **减少碎片**：统一管理，按需切分，内存利用率更高
+ **避免泄漏**：集中管理，程序结束时统一释放
+ **可控性强**：自己的地盘自己做主，可以根据业务特点优化

## 设计思路：三层架构设计
经过大量调研和思考，我采用了类似TCMalloc的三层架构：

```plain
┌─────────────────────────────────────────────────────────┐
│                   应用程序                                │
└─────────────────────┬───────────────────────────────────┘
                      │ ConcurrentAlloc() / ConcurrentFree() 
┌─────────────────────▼───────────────────────────────────┐
│              ThreadCache (线程缓存)                      │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                       │
│  │ 8B  │ │16B  │ │32B  │ │...  │  每个线程独享           │
│  │List │ │List │ │List │ │List │                       │
│  └─────┘ └─────┘ └─────┘ └─────┘                       │
└─────────────────────┬───────────────────────────────────┘
                      │ 批量获取/归还
┌─────────────────────▼───────────────────────────────────┐
│             CentralCache (中央缓存)                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                   │
│  │ 8B Span │ │16B Span │ │32B Span │  全局共享，桶锁    │
│  │ List    │ │ List    │ │ List    │                   │
│  └─────────┘ └─────────┘ └─────────┘                   │
└─────────────────────┬───────────────────────────────────┘
                      │ 申请新Span
┌─────────────────────▼───────────────────────────────────┐
│               PageHeap (页堆)                           │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                  │
│  │ 1页  │ │ 2页  │ │ 4页  │ │ 8页  │  管理大块内存       │
│  │ Span │ │ Span │ │ Span │ │ Span │                  │
│  └──────┘ └──────┘ └──────┘ └──────┘                  │
└─────────────────────┬───────────────────────────────────┘
                      │ 系统调用
┌─────────────────────▼───────────────────────────────────┐
│                  操作系统                               │
│              (mmap/VirtualAlloc)                       │
└─────────────────────────────────────────────────────────┘
```

### 为什么是三层？
这个设计的精妙之处在于**分层解耦**：

+ **ThreadCache**：每个线程都有自己的缓存，分配时无需加锁，速度飞快
+ **CentralCache**：当ThreadCache没有合适的内存块时，向CentralCache申请
+ **PageHeap**：管理大块内存，当CentralCache也没有时，向系统申请内存

这样设计的好处是：大部分情况下分配操作都在ThreadCache完成，无锁且极快；只有在必要时才会涉及锁操作。

### 第一层：ThreadCache（线程本地缓存）
**设计理念**：每个线程拥有独立的内存缓存，消除锁竞争。

```cpp
class ThreadCache {
private:
    FreeList free_lists_[NFREELISTS];  // 208个不同大小的自由链表
    static thread_local ThreadCache* tls_thread_cache_;
    
public:
    void* Allocate(size_t size);
    void Deallocate(void* ptr, size_t size);
};
```

**核心优化点**：

+ **无锁设计**：线程本地存储，天然线程安全
+ **多级缓存**：208个不同大小的自由链表
+ **批量操作**：与中心缓存批量交换，减少交互次数

### 第二层：CentralCache（中心分配器）
**设计理念**：所有线程共享的中心分配器，负责向ThreadCache提供内存。

```cpp
class CentralCache {
private:
    SpanList span_lists_[NFREELISTS];        // Span链表数组
    std::mutex mutexes_[NFREELISTS];         // 桶锁数组
    
public:
    size_t FetchRangeObj(void*& start, void*& end, size_t n, size_t size);
    void ReleaseListToSpans(void* start, size_t size);
};
```

**核心优化点**：

+ **桶锁设计**：每个大小类别独立锁，减少锁竞争
+ **Span管理**：每个Span管理一组连续页面
+ **批量分配**：一次分配多个对象给ThreadCache

### 第三层：PageHeap（页堆管理器）
**设计理念**：管理大块内存页面，是系统内存和应用的桥梁。

```cpp
class PageHeap {
private:
    SpanList span_lists_[POWER_SLOTS];  // 只管理2的幂次页数
    PageMap2<PAGE_MAP_BITS> page_map_;   // 页面到Span映射，采用基数树来管理
    
public:
    Span* AllocateSpan(size_t n);
    void ReleaseSpanToPageHeap(Span* span);
};
```

**核心优化点**：

+ **2的幂次优化**：只分配1,2,4,8,16,32,64,128,256页的Span
+ **智能分裂**：大Span智能分裂成小Span
+ **零开销释放**：释放直接缓存，无需合并操作

## 核心数据结构设计
### 1. 自由链表（FreeList）
这是内存池的基础数据结构，将空闲内存块串成链表：

```cpp
class FreeList {
private:
    void* head_;      // 链表头指针
    size_t size_;     // 当前大小
    size_t max_size_; // 慢启动最大批量大小
    
public:
    void Push(void* obj);
    void* Pop();
    void PushRange(void* start, void* end, size_t n);
    size_t PopRange(void*& start, void*& end, size_t n);
};
```

**巧妙设计**：利用空闲内存块本身存储链表指针，零额外开销！

```cpp
static inline void*& NextObj(void* obj) {
    return *(void**)obj;  // 前8字节存储下一个块的地址
}
```

### 2. Span结构
Span是管理连续页面的核心结构：

```cpp
struct Span {
    PageID page_id_;        // 起始页号
    size_t n_;              // 页数
    Span* next_;            // 双向链表指针
    Span* prev_;
    size_t object_size_;    // 切分的对象大小
    size_t use_count_;      // 已分配对象数
    void* free_list_;       // 切分后的自由链表
    bool is_used_;          // 是否使用中
};
```

## 关键算法实现
### 1. 大小类别映射算法
将任意大小映射到固定的大小类别，这是性能的关键：

```cpp
static inline size_t RoundUp(size_t size) {
    if (size <= 128) {
        return Align(size, 8);        // 8字节对齐
    } else if (size <= 1024) {
        return Align(size, 16);       // 16字节对齐
    } else if (size <= 8 * 1024) {
        return Align(size, 128);      // 128字节对齐
    } else if (size <= 64 * 1024) {
        return Align(size, 1024);     // 1KB对齐
    } else if (size <= 256 * 1024) {
        return Align(size, 8 * 1024); // 8KB对齐
    }
}
```

**设计考量**：小对象精细对齐，大对象粗粒度对齐，平衡内存利用率和性能。

### 2. 慢启动批量分配
动态调整批量大小，平衡内存使用和性能：

```cpp
static size_t NumMoveSize(size_t size) {
    size_t base_batch;
    if (size <= 32) {
        base_batch = 128;    // 小对象大批量
    } else if (size <= 128) {
        base_batch = 64;
    } else if (size <= 512) {
        base_batch = 32;
    } else {
        base_batch = 16;     // 大对象小批量
    }
    return base_batch * batch_multiplier;
}
```

### 3. 页面映射优化
采用二层基数树，快速查找对象所属的Span：

```cpp
template<int BITS>
class PageMap2 {
private:
    static const int LEAF_BITS = BITS / 2;
    static const int ROOT_BITS = BITS - LEAF_BITS;
    
    struct OptimizedLeaf {
        SubLeaf* sub_leafs[SUB_LEAFS_PER_LEAF];
        // 延迟初始化，减少内存开销
    };
    
public:
    inline void* get(size_t k) const;
    inline void set(size_t k, void* v);
};
```

上面展示的是部分核心设计思路的简化代码，实际实现中还包含了更多的边界处理和优化细节。

> **PS**：说实话，能参考TCMalloc架构手搓高性能内存池的人应该不多。我在研究阶段看了网上的几个版本，发现大部分还是基于32位系统设计的，在如今的64位环境下就显得有些局限了。可能是早期教学项目的代码被反复借鉴，缺少针对现代系统的深度优化。
>
>
>**注意**: 我这个版本从头开始针对64位系统设计，不仅支持完整的虚拟地址空间，还考虑了现代CPU架构的特性， 至少在设计思路上更贴近实际应用场景。  
>

## 性能优化技巧
### 1. 分支预测优化
```cpp
// 利用__builtin_expect优化分支预测
if (__builtin_expect(!list.Empty(), 1)) {
    return list.Pop();  // 大概率走这个分支
}
```

### 2. 内联函数优化
```cpp
// 热点函数全部内联
static inline size_t GetPageID(void* addr) {
    return reinterpret_cast<PageID>(addr) >> PAGE_SHIFT;
}
```

### 3. 缓存友好的数据结构
```cpp
// 64字节对齐，匹配CPU缓存行
struct SimpleBatch {
    void* ptrs[32];    
    uint8_t count = 0;   
} __attribute__((aligned(64)));
```

### 4. 锁优化策略
```cpp
// 桶锁：每个大小类别独立锁
std::mutex mutexes_[NFREELISTS];

// 减少持锁时间
{
    std::lock_guard<std::mutex> lock(mutexes_[index]);
    // 最少的临界区代码
}
```

### 5. 基于perf的性能分析优化
在内存池开发过程中，perf是我最重要的性能分析工具。下面分享三个实际优化案例：

**案例1：发现热点函数并优化**

问题发现：使用perf分析发现SizeClass::Index()函数占用了15%的CPU时间

```bash
# 性能分析命令
sudo perf record -g ./test_memory_pool
sudo perf report 

# 发现热点
15.23%  test_memory_pool  [.] SizeClass::Index(unsigned long)
 8.94%  test_memory_pool  [.] ThreadCache::Allocate(unsigned long)
```

优化方案：针对最常用的小对象做特殊优化

```cpp
// 优化前：每次都走复杂的Index计算
size_t index = SizeClass::Index(align_size);

// 优化后：小对象直接计算，避免函数调用
size_t index;
if (__builtin_expect(align_size <= 128, 1)) {
    index = (align_size >> 3) - 1;  // 直接位运算
} else {
    index = SizeClass::Index(align_size);  // 复杂情况才调用
}
```

效果验证：再次perf分析，该函数CPU占用降到3.2%，整体性能提升12%

**案例2：优化Deallocate的批量处理**

问题发现：Deallocate函数中频繁的Push操作CPU耗时较高

```bash
12.45%  test_memory_pool  [.] FreeList::Push(void*)
 7.33%  test_memory_pool  [.] ThreadCache::Deallocate(void*, unsigned long)
```

优化方案：针对小对象使用批量释放策略

```cpp
// 优化前：每次都要操作链表
void ThreadCache::Deallocate(void* ptr, size_t size) {
    size_t index = GetIndex(size);
    free_lists_[index].Push(ptr);  // 每次都要修改链表头
}

// 优化后：使用批量缓冲区
SimpleBatch batches_[32];  // 只为热点大小创建批量

void ThreadCache::Deallocate(void* ptr, size_t size) {
    if (index < 32) {
        SimpleBatch& batch = batches_[index];
        batch.ptrs[batch.count++] = ptr;
        if (batch.count >= 32) {
            FlushSimpleBatch(index, size);  // 批量刷新到链表
        }
    }
}
```

**案例3：解决大量缺页中断问题**

问题发现：程序出现大量缺页处理，perf显示__memset_avx2_erms耗时严重

```bash
33.67%  test_memory_pool  [.] __memset_avx2_erms
11.22%  test_memory_pool  [.] PageMap2::set_range
```

优化方案：优化PageMap二层基数树，减少memset调用

```cpp
// 优化前：每次都要初始化大块内存
class PageMap2 {
    void* values[HUGE_SIZE];  // 直接分配巨大数组，导致大量memset
};

// 优化后：延迟初始化，按需分配
class PageMap2 {
    struct SubLeaf {
        void* values[1024];  // 只有8KB，memset很快
        bool initialized = false;
    };
    
    void ensure_initialized() {
        if (!initialized) {
            memset(values, 0, sizeof(values));  // 只清零8KB
            initialized = true;
        }
    }
};
```

效果：memset调用减少95%，在高并发场景下性能提升显著。由此可见在高并发场景下不能够大量调用memset。

 实际在优化过程中还遇到了很多类似的性能瓶颈，这里只是举了几个例子。perf工具帮助我们精确定位问题，避免了盲目优化，每一次改进都有数据支撑。  

## 实战测试与性能分析
### 测试环境
+ 系统：ubuntu20.04
+ 编译器：GCC -O3 优化
+ 线程数：16
+ 每线程操作：10000次分配释放

### 性能提升分析
1. **小对象优势明显**：8B-128B对象提升2-5倍
2. **中等对象表现优异**：256B-1KB对象提升5-6倍
3. **大对象依然领先**：2KB以上对象提升7倍以上

### 为什么这么快？
1. **减少系统调用**：批量分配减少90%+的系统调用
2. **消除锁竞争**：线程本地缓存 + 桶锁设计
3. **内存局部性**：连续内存分配，缓存友好
4. **算法优化**：O(1)分配释放，无遍历查找

## 使用方法
接口设计简洁，可以直接替换malloc/free：

```cpp
// 基础接口
void* ptr = ConcurrentAlloc(1024);
ConcurrentFree(ptr);
```

## 项目亮点总结
1. **三层架构设计**：清晰的架构层次，职责分离
2. **多种优化技术**：从算法到实现的全方位优化
3. **生产级质量**：完整的错误处理和边界检查
4. **高可扩展性**：支持自定义配置和扩展
5. **实测性能卓越**：平均4.62倍性能提升

## 一些思考和收获
这个内存池项目从构思到完成，前后花了我一个月的业余时间。

最初只是想解决项目中的性能瓶颈，没想到越深入越发现内存管理的复杂性。从最简单的链表管理，到三层架构设计，再到各种微观优化，每一步都让我对系统底层有了新的认识。

**印象最深的是那次perf分析**，发现15%的时间竟然消耗在一个看似简单的Index计算上。这让我意识到，真正的性能优化往往隐藏在最不起眼的地方。

 还有那次为了解决PageMap初始化性能问题，我不得不重新设计了整个二级页表结构。原本简单粗暴的大数组分配导致了严重的缺页中断，perf显示__memset_avx2_erms占用了23%的CPU时间。虽然推翻了之前的设计，改用延迟初始化的SubLeaf结构，但看到最终memset调用减少95%的数据时，一切都值得了。  

**这个项目最大的价值不是代码本身，而是整个优化思维的建立。**

现在我看任何性能问题，都会先想：这是架构问题还是实现问题？是算法复杂度的问题还是常数优化的空间？数据访问模式是否缓存友好？锁的粒度是否合理？

## 如果你也想掌握这项核心技能...
说实话，内存池设计是C++性能优化的核心技能之一。很多大厂面试都会问相关问题，而且在高性能系统开发中经常用到。

这次实现过程中踩过的坑、总结的经验，我觉得对想深入学习C++底层优化的朋友会很有价值。所以我把整个项目的开发过程、设计思路、优化技巧都整理成了一套完整的实战课程。

**课程特色**：

+ **项目驱动**：从0到1完整实现，每天都有可运行的版本
+ **渐进式学习**：10天学习计划，循序渐进不会卡住
+ **实战导向**：重点讲解perf分析、性能调优等实战技能
+ **源码+文档**：4000+行完整代码，详细的设计文档

课程内容涵盖：

+ 三层架构的系统性设计思维
+ 无锁编程和多线程优化技巧
+ perf工具进行性能分析的实战方法
+ CPU缓存友好的数据结构设计
+ 从问题发现到解决的完整优化流程

最重要的是，这不只是理论讲解，而是一个完整的工程实践。学完后你将拥有一个可以写在简历上、在面试中展示的高质量项目。

如果你对这个内存池项目感兴趣，可以看这篇文章：  

想报名的话，赶紧加我微信：**jkfwdkf**，备注「**内存池**」！

**名额有限！手快有，手慢无！**

**或者扫下方二维码加我**：

![](https://files.mdnice.com/user/48364/c0dd2bed-8e46-4e05-b082-e8b021e30a16.png)

在这个AI时代，能深入底层、具备系统级优化能力的工程师反而更加稀缺。这个项目的技术深度和工程实践价值，相信能为你的技术成长和职业发展带来实质帮助。

如果你在性能优化方面有什么心得，或者对内存池实现有什么想法，也欢迎在评论区交流。

