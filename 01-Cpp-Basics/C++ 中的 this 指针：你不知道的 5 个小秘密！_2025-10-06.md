# 前言：`this` 指针——看似普通，却藏着 C++ 的灵魂
大家好，我是小康。

在写 C++ 的过程中，你一定见过那个神秘的家伙——`this` 指针。  
它总是“默默出现”，但又没人细讲它到底是干嘛的。  
其实，`this` 不仅仅是一个“指向当前对象的指针”，更是 C++ 面向对象机制中至关重要的一环。

今天这篇文章，我们就来用最简单的方式，彻底搞清楚：

- `this` 到底是什么？  
- 它出现在哪些场景？  
- 为什么有些地方必须用它，而有些地方最好别用？

看完这篇文章，你会对 `this` 有一个全新的理解——不只是“语法糖”，而是理解类与对象底层机制的关键。

> 💡 学习建议：
> 强烈建议自己写几个类，打印出 `this` 的地址观察它的变化。
>
> 理解“对象就是数据 + 函数”这一思想，`this` 是它们之间的纽带。
>
> 读完后试着回答自己一个问题：“没有 `this`，C++ 的成员函数还能存在吗？”
>
> 💬 欢迎关注我的公众号「**跟着小康编程**」，我会持续更新 C 、 C++、Linux、后端开发相关的系列文章，  一起进阶系统编程与高性能开发！
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

## 一、this 是谁？

在讲 `this` 指针之前，我们先来理清一个非常基本的概念：C++ 中的对象。你可以把对象想象成一个实体，它是类的实例。当你创建了一个类的对象时，C++ 会为你分配一块内存区域，这块内存区域就是这个对象的“家”。

那么，`this` 指针其实就是指向当前对象的指针，它告诉你——你当前操作的是哪个对象。它的值就是当前对象的地址。

## 二、this 怎么用？

### 1. 访问当前对象的成员

每当你在类的成员函数里使用 `this` 指针时，实际上你是在告诉编译器：“嘿，我现在要操作的这个对象是自己。”

```c++
class MyClass {
public:
    int value;
    MyClass(int v) : value(v) {}

    void print() {
        std::cout << "当前对象的 value 值是: " << this->value << std::endl;
    }
};
```

在上面的代码里，`this->value` 等价于 `value`，但加上 `this` 的话，显得你是在“明确”告诉编译器：我现在操作的是当前对象的成员。

### 2. 用来避免成员名冲突

有时候，我们在类的构造函数或者成员函数里，可能会遇到形参和成员变量同名的情况。此时，`this` 指针就能帮你“明确”区分是成员变量还是形参。

```c++
class MyClass {
public:
    int value;
    MyClass(int value) {
        this->value = value;  // 使用 this 指针区分成员变量和形参
    }
};
```

在这个例子中，构造函数的参数和类的成员变量同名了。为了避免混淆，我们用 `this->value` 来表示成员变量，确保赋值的是对象的成员。

## 三、this 有哪些隐藏的秘密？

### 1. this 只能在成员函数中使用

你不能在类的外部随便用 `this`。它是“专属于”类成员函数的——只有在成员函数内部，编译器才知道你说的 `this` 是哪个对象。

```c++
class MyClass {
public:
    int value;

    void setValue(int v) {
        this->value = v;  // 合法，this 是当前对象的指针
    }

    void print() {
        std::cout << "当前对象的值是: " << this->value << std::endl;
    }
};

void test() {
    MyClass obj;
    obj.setValue(10);  // 在成员函数中可以访问 this 指针
    // 在这里就无法使用 this 指针了
}
```

### 2. this 指针是一个常量指针

`this` 不是普通的指针，它是一个常量指针。这意味着你不能改变 `this` 的指向——你不能让它指向其他对象。它永远指向当前对象。

```c++
void MyClass::changeThis() {
    this = nullptr;  // 错误！this 是常量指针，不能修改。this指针类型是 const MyClass* this
}
```

### 3. this 指针与链式调用

`this` 指针可以用来实现 **链式调用**，即在同一个语句中连续调用多个成员函数。通过返回 `*this`，可以让一个函数返回当前对象的引用，进而可以继续调用其他成员函数。

```c++
class MyClass {
public:
    int value;
    MyClass(int v) : value(v) {}

    MyClass& setValue(int v) {
        value = v;
        return *this;  // 返回当前对象的引用，支持链式调用
    }

    MyClass& print() {
        std::cout << "当前值是: " << value << std::endl;
        return *this;
    }
};

int main() {
    MyClass obj(10);
    obj.setValue(20).print();  // 链式调用
}
```

## 四、对象调用成员函数时，`this` 指针如何传递？

每当你通过对象调用成员函数时，C++ 编译器会自动把该对象的地址作为 `this` 指针传递给成员函数。这一过程对我们来说是透明的，但是它实际上是如何工作的呢？

来看下面的例子：

```c++
#include <iostream>
using namespace std;

class MyClass {
public:
    int value;

    MyClass(int v) : value(v) {}

    void printValue() {
        std::cout << "当前对象的地址是: " << this << std::endl;  // 输出当前对象的地址
        std::cout << "当前对象的 value 值是: " << this->value << std::endl;  // 输出对象的值
    }
};

int main() {
    MyClass obj(10);
    obj.printValue();  // 通过对象调用成员函数

    return 0;
}
```

### this 指针如何传递？

当我们调用 `obj.printValue()` 时，C++ 编译器背后实际上会做这样的事情：

```c++
(&obj)->printValue();  // 通过 &obj 获取对象地址，将其传递给 printValue 函数
```

也就是说，`&obj` 传递给了 `this` 指针，指向了当前调用该函数的对象 `obj`。你可以通过 `this` 指针访问到 `obj` 对象的成员。这个传递过程是隐式的，编译器会自动帮你做。

### 为了更加直观的理解：

虽然在实际编码中，我们不需要手动传递 `this` 指针，但为了帮助大家更清楚地理解它的作用，我们可以通过显式传递 `this` 指针来做个对比。想象一下，如果我们手动传递 `this` 指针，代码可能会是这样的：

```c++
void MyClass::printValue(const MyClass* this) {
    std::cout << "当前对象的地址是: " << this << std::endl;  // 输出当前对象的地址
    std::cout << "当前对象的 value 值是: " << this->value << std::endl;  // 输出对象的值
}
```

在这种情况下，你可以像这样显式地传递对象的地址：

```c++
obj.printValue(&obj);  // 显式传递对象的地址
```

这里的 &obj 其实就是 obj 对象的地址，它被传递给了 `printValue()` 函数。此时，函数内部的 this 指针指向了 obj，并且你依然可以通过 this 来访问对象的成员。

实际上，这就是我们平时调用成员函数时，编译器自动做的事情：**将对象的地址隐式地传递给 this 指针**。所以，`(&obj)->printValue();` 和 `obj.printValue(&obj)` 在本质上是相同的，只不过前者是自动传递，后者是我们手动传递 this 指针。

## 五、this 不是万能的！

虽然 `this` 指针在很多情况下非常有用，但它也有局限性：

+ 在静态成员函数中，没有 `this` 指针。因为静态成员函数是属于类的，而不是某个具体的对象，所以它没有“当前对象”的概念。  

```c++
class MyClass {
public:
    static void staticMethod() {
        // this->value = 10;  // 错误！静态函数没有 this 指针
    }
};
```

+ `this` 指针不适用于全局函数。它只和类的成员函数相关联。

## 六、总结：this 指针的妙用

- **指向当前对象**：this 指针总是指向当前调用成员函数的对象，让你在代码中明确知道正在操作的是哪个对象。
- **解决成员变量和参数同名问题**：当成员变量和函数参数同名时，this 指针帮你轻松区分它们，避免混淆。
- **链式调用的秘密武器**：通过返回 *this，你可以让多个成员函数在同一行代码中依次执行，让代码更简洁、流畅。
- **常量指针，保持不变**：this 是常量指针，它始终指向当前对象，不能指向其他对象，保证了代码的稳定性和一致性。

理解 `this` 指针，就像是在编写 C++ 代码时拥有了一把“精确定位”的工具。它帮助你更加清晰地理解对象的行为，让你的代码更加清晰、可控。

今天的 `this` 指针小课堂就到这里，希望通过这篇文章，你对 `this` 指针有了更加直观的理解。并在实际编码中得心应手。如果觉得这篇文章有帮助，记得点赞、在看和分享哦~，让更多人一起掌握 this 指针的精髓！

也欢迎大家来关注我公众号 「**跟着小康学编程**」，这里会持续分享计算机编程硬核技术文章！

## 关注我能学到什么？

- 这里分享 Linux C、C++、Go 开发、计算机基础知识 和 编程面试干货等，内容深入浅出，让技术学习变得轻松有趣。

- 无论您是备战面试，还是想提升编程技能，这里都致力于提供实用、有趣、有深度的技术分享。快来关注，让我们一起成长！

## 怎么关注我的公众号？

**非常简单！扫描下方二维码即可一键关注。**

![](https://files.mdnice.com/user/71186/0dde803d-d52f-4ed8-b74b-b7f3da5817b9.png)

另外我还建了个**技术交流群**，里面都是真正在写代码的同行，不聊虚的，只聊技术。有问题大家一起讨论，比一个人闷头学效率高多了。
  
![](https://files.mdnice.com/user/48364/4ebc72e9-e4bb-447a-9a92-8367a178df6d.png)

记住，技术这条路，一个人走容易迷路，一群人走才能走得更远。
