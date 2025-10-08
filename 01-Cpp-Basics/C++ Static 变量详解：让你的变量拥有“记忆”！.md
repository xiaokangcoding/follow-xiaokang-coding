大家好，我是小康，今天我们来聊聊 C++ 中一个非常有用的关键字 —— static 。

#### 前言：你知道变量也能有“记忆”吗？

在 C++ 编程中，我们经常使用变量来存储信息。一般来说，变量只在它的作用域内有效，一旦超出了作用域，变量就“消失”了。

但是，C++ 提供了一个非常有意思的关键字——`static`，它能让变量在函数外的作用域之外“活”得更久，甚至能记住它上次的值。这就像是给一个普通的变量装上了“记忆卡”，它能在函数调用结束后仍然保持状态。听起来是不是有点神奇？

今天，我们就来聊聊 `static` 变量，看看它如何在 C++ 中改变了变量的生命周期和行为。


> 💡 学习建议：
>
>多实验：在函数里加 static，多次调用看看变量值变化；
>
>理解生命周期：体会普通变量和静态变量在作用域结束后的区别；
>
>结合类学习：static 成员变量和普通成员的区别是面试高频考点。
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

### 什么是 static  变量？

首先，我们得弄清楚，什么是 `static` 变量。简单来说，`static` 变量有一个特别的属性：它们的生命周期不再是函数调用的周期，而是程序整个运行期间。

#### 举个例子：

假设你有一个函数，每次调用时都希望记录函数调用的次数。正常情况下，函数的局部变量只在函数执行时存在，执行完之后就销毁了。但是如果你希望记录上一次函数调用时的值，该怎么办呢？

这时候，`static` 变量就派上用场了。它会记住自己上次的值，即使函数执行完毕，它也会保持状态。

```c++
#include <iostream>
using namespace std;

void countCalls() {
    static int count = 0;  // static 变量
    count++;
    cout << "This function has been called " << count << " times." << endl;
}

int main() {
    countCalls();  // 输出：This function has been called 1 times.
    countCalls();  // 输出：This function has been called 2 times.
    countCalls();  // 输出：This function has been called 3 times.
    return 0;
}
```

在这个例子中，`count` 变量被声明为 `static`，意味着它的值会在多次调用函数时持续存在。每次调用 `countCalls()`，`count` 都会增加，而不会被重新初始化为 0。

### static 变量的分类：局部变量与全局变量

现在，我们了解了 `static` 变量的作用：它能让函数内的变量保持状态、记住上次的值。接下来，我们就来看看 `static` 变量的两种常见类型：**静态局部变量**和**静态全局变量**。它们看起来很像，但其实有一些细微的区别。

#### 1、静态局部变量（static 局部变量）

静态局部变量就像是“**有记忆的局部变量**”。它在函数内声明，但其生命周期从程序开始到结束都存在。即使函数执行完毕，变量也不会被销毁，下次再调用该函数时，它依然保持着上次的值。

我们在上面的例子中已经看到过 `static` 局部变量 `count`的效果。每次调用 `countCalls()` 函数时，`count` 都会记住上次调用时的值，不会重新初始化。这样就实现了记录函数调用次数的功能。

#### 2、静态全局变量（static 全局变量）

那么，静态全局变量是什么呢？简单来说，它与静态局部变量的区别在于 **它是在全局作用域中声明的**。虽然它是全局变量，但它的作用范围限制在定义它的文件内，不能在其他文件中访问。

换句话说，`static` 全局变量就像是“私有”的全局变量，只有当前文件能够访问它，其他文件无法访问或修改它，这样就避免了命名冲突的风险。

**例如**：

```c++
#include <iostream>
using namespace std;

static int count = 0;  // static 全局变量

void increase() {
    count++;  // 修改全局变量
}

int main() {
    increase();
    cout << "Global count: " << count << endl;  // 输出：Global count: 1
    return 0;
}
```

在这个例子中，`count` 是一个 static 全局变量，它虽然是全局的，但只能在当前文件内使用。如果你尝试在其他文件中引用它，编译器就会报错，无法访问。

#### 3、静态全局变量 vs 普通全局变量

那么，静态全局变量和普通全局变量到底有什么区别呢？其实，它们最大的区别在于作用域。

+ **静态全局变量**：虽然是全局变量，但它的作用范围仅限于声明它的文件内。其他文件不能访问它。这使得它在大型项目中非常有用，避免了全局命名空间污染，提升了封装性。
+ **普通全局变量**：它的作用范围是全程序的任何地方，只要通过 `extern` 关键字进行声明，其他文件都可以访问。

**举个例子**：

```c++
// file1.cpp
#include <iostream>
using namespace std;

int globalVar = 10;  // 普通全局变量
static int staticVar = 20;  // 静态全局变量

void printVars() {
    cout << "globalVar: " << globalVar << endl;   // 输出：globalVar: 10
    cout << "staticVar: " << staticVar << endl;   // 输出：staticVar: 20
}

// file2.cpp
#include <iostream>
using namespace std;

// 访问全局变量
extern int globalVar;  // 通过 extern 声明全局变量，告诉编译器它在 file1.cpp 中定义

// error: 'staticVar' is private within this context
// extern int staticVar;  // 静态全局变量无法跨文件访问

void printGlobalVar() {
    cout << "globalVar from file2: " << globalVar << endl;   // 输出：globalVar from file2: 10
}

int main() {
    printGlobalVar();
    return 0;
}
```

在这个例子中，`globalVar` 是普通全局变量，它能在多个文件中访问，而 `staticVar` 是静态全局变量，只能在 `file1.cpp` 文件中使用，不能被其他文件引用。

### 为什么要用 static  变量？

好问题！你可能会想：“为什么不用普通的局部变量呢？它不是更简单吗？”的确，局部变量简单，但 `static` 变量在某些场景下有它的独特优势：

+ **保持状态**：`static` 变量可以在函数之间保持状态，它不会在每次函数调用时重新初始化。这对很多程序来说是非常有用的，特别是那些需要跨函数调用保持数据的场景，比如计数器、缓存、状态标志等。
+ **内存管理更加高效**：`static` 变量在程序运行期间只会分配一次内存，并且在整个程序生命周期内都存在。这意味着它不像普通局部变量那样每次函数调用时都要重新分配内存，避免了重复的内存开销。  

### 小结：

到这里，大家应该已经对 static 变量有了更清晰的认识。它不仅能延长变量的生命周期，还能让变量在不同的函数调用之间保持状态，这对于需要累加或追踪数据的场景特别有用。

接下来，我们将继续深入探讨 static 的更多应用，尤其是在类设计和多线程编程中的作用。相信通过后续的内容，你会对它有更深的理解，并能在实际开发中灵活运用。

如果你觉得这篇文章对你有帮助，别忘了点个「赞」或「在看」，也欢迎大家关注我的公众号「跟着小康学编程」。

#### 下一篇预告：C++ static 成员：让类变量不再“孤单”

在下一篇文章中，我们将继续探讨 `static` 在类中的应用，看看它如何让类的对象之间共享数据，如何利用静态成员函数提高类的设计效率。不见不散哦！

## 🚀 跟我学，你能收获啥？
在这里，你可以学到：

+ Linux、C/C++、Go 的实战技巧
+ 计算机基础知识梳理
+ 编程面试干货、算法套路和优化思路

内容**深入浅出、实用有趣**，不再让你看着书就犯困。  
无论是面试冲刺，还是想升级技能，这里都是你的“技术加油站”。


## 👀 想加入？很简单！
**扫一扫下面二维码**，一键关注公众号，开启你的技术学习之旅！

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

另外，我还建了一个**技术交流群**，里面都是认真写代码的小伙伴，不吹牛、不闲聊，只聊技术。  
有问题？大家一块儿讨论，比一个人闷头学效率高多了！

![](https://files.mdnice.com/user/48364/4ebc72e9-e4bb-447a-9a92-8367a178df6d.png)

技术这条路，一个人走容易迷路，一群人走才有方向。  
跟上节奏，我们一起变强 💪