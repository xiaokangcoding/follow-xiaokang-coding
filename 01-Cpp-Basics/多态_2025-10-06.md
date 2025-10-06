大家好，我是小康。

今天我们要一起走进 C++ 的世界，探索一个非常强大的概念——**多态**。


> 💡 学习建议：
>
> 动手写写看：自己实现几个不同类型的机器人，体会多态带来的灵活性。
>
> 用指针或引用操作基类：观察虚函数如何在运行时切换不同的行为。
>
> 多思考场景：什么时候用继承+多态更方便，什么时候组合更合适。多态不是魔法，是工具；多练习，你就能熟练运用它解决实际问题。

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


在一个被代码环绕的程序员村庄里，三位年轻的程序员：小李、小张和小王，正忙于解决一个全新的问题——"如何设计一个可以执行各种任务的机器人?" 无论是清洁、烹饪，还是修理，他们都希望这个机器人能够根据不同的需求，自如地切换任务。正当他们感到困惑时，多态的概念为他们带来了答案。

## 小李的初步设计：让机器人执行任务

小李首先提出了一个简单的想法：设计一个 `Robot` 基类，所有机器人继承自这个基类，并通过 `performTask()` 方法来执行各自的任务。他的初步代码如下：

```c++
class Robot {
public:
    void performTask() {
        cout << "Robot is performing a general task!" << endl;
    }
};
```

这个设计看起来一切顺利，每个机器人只需要继承 `Robot` 基类，并实现自己的 `performTask()` 方法。于是，小李创建了两个子类：`CleaningRobot`（扫地机器人）和 `CookingRobot`（做饭机器人）：

```c++
class CleaningRobot : public Robot {
public:
    void performTask() {
        cout << "Cleaning robot is sweeping the floor!" << endl;
    }
};

class CookingRobot : public Robot {
public:
    void performTask() {
        cout << "Cooking robot is making dinner!" << endl;
    }
};
```

一开始，小李觉得这样的设计非常完美。每个子类都能根据自己的职责实现 `performTask()` 方法。但很快，他就发现了一个问题……


**问题：修改与扩展的困难**

随着小李设计的机器人种类越来越多，他开始意识到，若要增加新的机器人类型（比如 `WashingRobot` 或 `RepairRobot`），每次都需要手动修改主程序，增加新机器人的实例，并调用它们的 `performTask()` 方法。

举个例子，如果新增了 `RepairRobot`，主程序就可能需要改成这样：

```c++
int main() {
    CleaningRobot cleaningRobot;
    CookingRobot cookingRobot;
    RepairRobot repairRobot;  // 新增机器人

    cleaningRobot.performTask();
    cookingRobot.performTask();
    repairRobot.performTask();  // 新增的任务

    return 0;
}
```

随着机器人种类的增加，主程序变得越来越庞大，代码的扩展性和维护性也受到影响。更糟的是，如果程序中的多个地方都调用 `performTask()`，每次新增机器人类型时，都需要在多个地方进行修改。

小李感叹道：“如果能有一种方法，避免每次修改主程序，而是让系统根据需要自动适应新增的机器人类型，那该有多好！”

## 小张的点拨：为什么引入多态

小李苦思冥想后，终于提出了自己的担忧：“每次我们新增一种机器人类型，主程序都需要修改，这样不太灵活。而且随着机器人种类的增加，代码也会变得越来越复杂。”

小张点了点头，笑着说：“你提到的问题正是传统继承设计的局限性。在你目前的设计中，主程序必须知道每个具体的机器人类型，这样就增加了代码之间的耦合度。每当增加新的机器人时，不得不修改主程序。其实，多态可以帮助我们解决这个问题。”

小李有些疑惑：“那多态和继承的区别到底是什么？继承不是让子类拥有父类的功能吗？为什么说单靠继承不够呢？”

小张耐心地解释道：“继承确实让子类继承了父类的功能，但它并没有解决代码依赖具体类型的问题。具体来说，通过继承，程序仍然需要显式地知道每个子类的类型。而多态的本质是：通过基类指针或引用，你可以在运行时根据对象的实际类型，自动调用正确的子类方法。也就是说，主程序不需要关心对象的具体类型，而只关心一个统一的接口。”

### 对比：普通继承与多态

#### 扩展性与维护性

小张举了一个例子来帮助小李理解。首先，他展示了没有多态时的设计：

```c++
int main() {
    CleaningRobot cleaningRobot;
    CookingRobot cookingRobot;
    RepairRobot repairRobot;  // 新增机器人

    cleaningRobot.performTask();
    cookingRobot.performTask();
    repairRobot.performTask(); // 每次新增任务时，都需要修改主程序
}
```

小张解释道：“看，没多态时，每次我们新增一个机器人类型（例如 `RepairRobot`），就需要修改主程序，手动添加新机器人的实例，并调用其 `performTask()` 方法。这使得代码耦合度变高，也让维护和扩展变得困难。”

接着，小张展示了使用多态后的设计：

```c++
class RepairRobot : public Robot {
public:
    void performTask() {
        cout << "Repair robot is fixing things!" << endl;
    }
};

int main() {
    Robot* robot1 = new CleaningRobot();
    Robot* robot2 = new CookingRobot();
    Robot* robot3 = new RepairRobot();  // 新增机器人

    robot1->performTask();
    robot2->performTask();
    robot3->performTask();

    delete robot1;
    delete robot2;
    delete robot3;
    return 0;
}
```

“通过多态，你看，”小张继续说道，“我们几乎不需要修改主程序，只需新增一个 `RepairRobot` 类并实现它自己的 `performTask()` 方法，程序会自动根据对象的实际类型来选择调用对应的 `performTask()` 方法。这样，主程序和机器人类型之间的依赖就大大减少了，扩展性和维护性也得到了提升。”

在这里，可能会有人问，不管是采用继承还是多态，新增一个机器人，主程序都需增加2行代码

#### 统一接口，解耦主程序

为了进一步强调多态的优势，小张又讲解了如何通过统一接口减少代码冗余。没有多态时，我们可能需要为每种机器人类型编写不同的函数：

```C++
void doSomethingWithRobot(CleaningRobot& robot) {
    robot.performTask();
}

void doSomethingWithRobot(CookingRobot& robot) {
    robot.performTask();
}
```

“每新增一种机器人类型，我们就要写一个新的函数，”小张说，“这样不仅让代码变得冗长，也导致维护起来更加麻烦。”

而使用多态后，情况就变得简单了。只需要写一个函数：

```c++
void doSomethingWithRobot(Robot* robot) {
    robot->performTask();
}
```

“看，使用多态后，”小张继续说道，“无论新增多少种机器人类型，主程序都不需要修改。主程序和具体实现解耦，代码更简洁，也更易于维护。”

> **注意**：无论是通过继承关系还是多态机制，新增机器人时，都需要定义新的类并实现相应的接口。

### 实现多态：关键点

最后，小张总结道：“要实现多态，关键在于将基类的 `performTask()` 方法声明为 `virtual`，这样程序就可以在运行时正确调用子类的重写方法。”

```c++
class Robot {
public:
    virtual void performTask() {
        cout << "Robot is performing a general task!" << endl;
    }
    virtual ~Robot() = default;  // 确保基类有虚析构函数
};

class CleaningRobot : public Robot {
public:
    void performTask()  {
        cout << "Cleaning robot is sweeping the floor!" << endl;
    }
};

class CookingRobot : public Robot {
public:
    void performTask() {
        cout << "Cooking robot is making dinner!" << endl;
    }
};
```

通过 `virtual` 关键字，基类的 `performTask()` 方法就成了虚函数，子类实现的 `performTask()` 方法将在运行时被正确调用。这就是多态的实现，它让程序的扩展和维护变得更加灵活和高效。  

## 小王的补充： 如何使用多态 ？

通过前面的讲解，你已经了解了多态的基本概念：通过基类指针，我们可以在运行时动态选择调用哪个子类的方法。小王补充道：“多态的关键就在于，通过基类指针或引用，我们可以调用统一的接口，而不需要关心具体的子类实现。”

**示例代码：**

```c++
int main() {
    Robot* robot1 = new CleaningRobot();
    Robot* robot2 = new CookingRobot();

    robot1->performTask();  // 输出：Cleaning robot is sweeping the floor!
    robot2->performTask();  // 输出：Cooking robot is making dinner!

    delete robot1;
    delete robot2;
    return 0;
}
```

在这个示例中，`robot1` 和 `robot2` 分别指向 `CleaningRobot` 和 `CookingRobot` 类型的对象。通过基类指针 `Robot*`，我们调用 `performTask()` 方法时，程序会自动根据实际的对象类型选择正确的方法实现。即使我们不明确指定子类类型，程序依然能正确地执行不同的任务。

这就是多态的优势：主程序不需要关心每个机器人对象的具体类型，它只需要通过基类接口来进行调用。通过这种方式，程序与具体的子类解耦，极大地提高了代码的灵活性和可维护性。

## 小李的收获：多态的优势

小李总结了多态带来的几个关键优势：

1. **统一接口**：通过基类接口调用方法，主程序无需关心具体的子类类型。无论是 `CleaningRobot` 还是 `CookingRobot`，都能通过相同的接口来执行任务，简化了代码。
2. **扩展性强**：当我们需要新增一个机器人类型时，只需创建一个新的子类，并实现必要的方法，主程序的代码无需修改。程序会自动适应新增的子类，极大提高了代码的扩展性和灵活性。

## 总结：多态的魔力

通过小李的探索和小张、小王的补充，我们已经掌握了多态的基本概念：通过基类指针或引用，程序能够在运行时自动选择调用不同子类的方法。这不仅让程序的结构更加简洁，还极大地提升了代码的灵活性和可扩展性。

正如我们所看到的，无论机器人种类如何增加，程序的主体结构几乎不需要修改，新增的机器人只需要实现基类定义的接口，程序便能自动适配。这种特性使得多态成为了实现代码复用和解耦的强大工具，帮助我们更轻松地应对不断变化的需求。

这正是多态的魅力所在：它让我们的代码变得更加模块化、易于扩展和维护。在下篇文章中，我们将进一步探讨多态的实现原理，揭秘它是如何在编译和运行时发挥作用的。敬请期待！


"觉得有收获吗？别忘了点赞、在看、分享，让更多的小伙伴一起进步！如果你有疑问或者想深入讨论，评论区见，我会一一解答。

想了解更多编程内容？关注我的公众号「跟着小康学编程」，更多实用的技术分享等你来学！"


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
