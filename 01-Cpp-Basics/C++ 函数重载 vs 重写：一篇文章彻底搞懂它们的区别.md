大家好，我是小康。

今天咱们聊个老生常谈但又经常被搞混的话题——函数重载和函数重写。

说真的，每次面试的时候，面试官总爱问这个问题。我敢打赌，十个程序员有九个都在这儿栽过跟头。要么就是概念搞混了，要么就是说得云里雾里的，让面试官一脸懵逼。

今天我就用最简单粗暴的方式，把这俩货给你讲明白。保证你看完之后，再也不会傻傻分不清楚了！

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


## 先来个形象的比喻

想象一下，你家楼下有个包子铺，老板姓张。

**函数重载**就像是：张老板会做包子，但他能做肉包子、菜包子、豆沙包子。虽然都叫"做包子"，但根据你给的材料不同，他做出来的包子也不一样。**同一个人，同一个技能名字，但是根据输入的不同，输出也不同。**

**函数重写**就像是：张老板退休了，他儿子小张接手了包子铺。小张也会"做包子"，但他的做法跟他爸完全不一样，可能更好吃，也可能更难吃。**不同的人，同一个技能名字，但实现方式完全不同。**

怎么样，是不是一下子就明白了？

## 函数重载：同名不同参
### 啥是函数重载？
简单说，就是**同一个类里面，方法名字一样，但参数不一样**。编译器会根据你传的参数来决定调用哪个方法。

就像你去餐厅点菜：

+ "来份炒饭！"（无参数）
+ "来份炒饭，要辣的！"（一个参数）
+ "来份炒饭，要辣的，多放肉！"（两个参数）

服务员会根据你的要求做出不同的炒饭。

### 代码实战
```cpp
#include <iostream>
#include <string>
using namespace std;

class Cook {
public:
    // 基础版炒饭
    void makeFriedRice() {
        cout << "做了一份普通炒饭" << endl;
    }
    
    // 带辣度的炒饭
    void makeFriedRice(string spicy) {
        cout << "做了一份" << spicy << "的炒饭" << endl;
    }
    
    // 带辣度和配菜的炒饭
    void makeFriedRice(string spicy, string ingredient) {
        cout << "做了一份" << spicy << "的炒饭，加了" << ingredient << endl;
    }
    
    // 连数量都能指定
    void makeFriedRice(int count, string spicy) {
        cout << "做了" << count << "份" << spicy << "的炒饭" << endl;
    }
    
    // 还可以重载构造函数
    Cook() {
        cout << "厨师准备就绪！" << endl;
    }
    
    Cook(string name) {
        cout << "厨师" << name << "准备就绪！" << endl;
    }
};

// 测试一下
int main() {
    Cook chef("老王");
    
    chef.makeFriedRice();                      // 输出：做了一份普通炒饭
    chef.makeFriedRice("微辣");                // 输出：做了一份微辣的炒饭
    chef.makeFriedRice("中辣", "牛肉");        // 输出：做了一份中辣的炒饭，加了牛肉
    chef.makeFriedRice(3, "变态辣");           // 输出：做了3份变态辣的炒饭
    
    return 0;
}
```

看到了吗？同样是`makeFriedRice`这个方法名，但根据你传的参数不同，执行的逻辑也不同。编译器很聪明，它会自动帮你选择合适的方法。

### 重载的规则（划重点！）
1. **方法名必须相同** - 这是基本要求
2. **参数列表必须不同** - 要么数量不同，要么类型不同，要么顺序不同
3. **返回值类型可以相同也可以不同** - 但不能仅仅通过返回值类型来区分重载
4.  **在同一个作用域内**(同一个类)  

 **记住**：编译器是通过参数来区分调用哪个函数的，跟返回值没关系！  

## 函数重写：子承父业，青出于蓝
### 啥是函数重写？
函数重写发生在继承关系中。**子类重新实现父类的方法**，方法名、参数都一样，但实现逻辑不同。

就像爸爸教你骑自行车的方法是"勇敢地骑上去"，但你教你儿子的方法可能是"先学会平衡，再慢慢来"。同样是"学骑车"这个方法，但实现方式完全不同。

### 代码实战
```cpp
#include <iostream>
#include <string>
using namespace std;

// 父类 - 老爸的教学方式
class OldTeacher {
public:
    virtual void teachBikeRiding() {  // virtual关键字是重点！
        cout << "老爸的方法：别怕，直接骑上去，摔几次就会了！" << endl;
    }
    
    virtual void teachSwimming() {
        cout << "老爸的方法：扔到水里，不会游多喝水，自然就学会了！" << endl;
    }
    
    // 虚析构函数，养成好习惯
    virtual ~OldTeacher() {
        cout << "老爸累了，休息去了" << endl;
    }
};

// 子类 - 新一代的教学方式
class ModernTeacher : public OldTeacher {
public:
    void teachBikeRiding() override {  // override关键字确保我们真的在重写
        cout << "现代方法：先练平衡，戴好护具，循序渐进，安全第一！" << endl;
    }
    
    void teachSwimming() override {
        cout << "现代方法：从浅水区开始，学会漂浮，再学动作，科学训练！" << endl;
    }
    
    // 子类还可以有自己独有的方法
    void teachOnline() {
        cout << "现代独有：在线视频教学辅助指导" << endl;
    }
    
    ~ModernTeacher() {
        cout << "现代老师下班了" << endl;
    }
};

// 测试一下
int main() {
    OldTeacher dad;
    ModernTeacher son;
    
    cout << "爸爸的教学方法：" << endl;
    dad.teachBikeRiding();      // 输出：老爸的方法：别怕，直接骑上去，摔几次就会了！
    dad.teachSwimming();        // 输出：老爸的方法：扔到水里，不会游多喝水，自然就学会了！
    
    cout << "\n儿子的教学方法：" << endl;
    son.teachBikeRiding();      // 输出：现代方法：先练平衡，戴好护具，循序渐进，安全第一！
    son.teachSwimming();        // 输出：现代方法：从浅水区开始，学会漂浮，再学动作，科学训练！
    son.teachOnline();          // 输出：现代独有：在线视频教学辅助指导
    
    // 多态的魅力 - C++的精髓所在！
    cout << "\n多态演示：" << endl;
    OldTeacher* teacher = new ModernTeacher();  // 父类指针指向子类对象
    teacher->teachBikeRiding(); // 输出：现代方法：先练平衡，戴好护具，循序渐进，安全第一！
    
    delete teacher;  // 记得释放内存
    
    return 0;
}
```

最后那个多态的例子特别有意思！虽然`teacher`的类型是`OldTeacher`，但它实际指向的是`ModernTeacher`对象，所以调用的是子类重写后的方法。这就是面向对象编程的魅力所在！

### 重写的规则（又是重点！）
1. **必须有继承关系** - 没有父子关系就不叫重写
2. **父类方法必须是virtual** - 这是C++特有的，没有virtual就不是真正的重写
3. **方法名、参数列表、返回值类型都必须相同** - 一模一样
4. **子类建议使用 override 关键字** -  （C++11推荐，不是必须但建议用）  
5. **基类析构函数最好声明为virtual** - 避免内存泄漏问题

## 来个终极对比
| 特征 | 函数重载(Overload) | 函数重写(Override) |
| --- | --- | --- |
| **发生位置** | 同一个类内 | 父子类之间 |
| **方法名** | 必须相同 | 必须相同 |
| **参数列表** | 必须不同 | 必须相同 |
| **返回值类型** | 可以不同 | 必须相同 |
| **决定时机** | 编译时决定 | 运行时决定 |
| **关键词** | 无特殊关键词 | virtual + override |
| **目的** | 提供多种调用方式 | 改变父类行为 |
| **C++特色** | 支持操作符重载 | 需要virtual才能多态 |


## C++独有的骚操作
### 操作符重载
C++最牛逼的地方就是可以重载操作符！比如你可以让两个对象直接用`+`号相加：

```cpp
#include <iostream>
using namespace std;

class Point {
private:
    int x, y;
    
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}
    
    // 重载+操作符
    Point operator+(const Point& other) {
        return Point(x + other.x, y + other.y);
    }
    
    // 重载<<操作符，方便输出
    friend ostream& operator<<(ostream& os, const Point& p) {
        os << "(" << p.x << ", " << p.y << ")";
        return os;
    }
};

int main() {
    Point p1(1, 2);
    Point p2(3, 4);
    Point p3 = p1 + p2;  // 直接用+号！
    
    cout << p1 << " + " << p2 << " = " << p3 << endl;
    // 输出：(1, 2) + (3, 4) = (4, 6)
    
    return 0;
}
```

### 函数模板重载
C++还支持模板函数的重载：

```cpp
#include <iostream>
using namespace std;

// 普通函数
int add(int a, int b) {
    cout << "调用了int版本的add" << endl;
    return a + b;
}

// 模板函数
template<typename T>
T add(T a, T b) {
    cout << "调用了模板版本的add" << endl;
    return a + b;
}

int main() {
    cout << add(1, 2) << endl;          // 调用普通函数
    cout << add(1.5, 2.5) << endl;     // 调用模板函数
    cout << add<int>(1, 2) << endl;    // 强制调用模板函数
    
    return 0;
}
```

### 重载的应用
想想你平时用的`cout`：

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "字符串" << endl;     // 输出字符串
    cout << 123 << endl;          // 输出整数
    cout << 3.14 << endl;         // 输出浮点数
    cout << true << endl;         // 输出布尔值
    
    return 0;
}
```

这就是重载！同一个`<<`操作符，但可以接受不同类型的参数。

还有构造函数重载：

```cpp
class Student {
private:
    string name;
    int age;
    
public:
    // 默认构造函数
    Student() : name("未知"), age(0) {
        cout << "创建了一个未知学生" << endl;
    }
    
    // 只有姓名的构造函数
    Student(string n) : name(n), age(0) {
        cout << "创建了学生：" << name << endl;
    }
    
    // 完整信息的构造函数
    Student(string n, int a) : name(n), age(a) {
        cout << "创建了学生：" << name << "，年龄：" << age << endl;
    }
};

int main() {
    Student s1;                    // 调用默认构造函数
    Student s2("小明");            // 调用单参数构造函数  
    Student s3("小红", 18);        // 调用双参数构造函数
    
    return 0;
}
```

### 重写的应用
比如做一个图形绘制程序：

```cpp
#include <iostream>
using namespace std;

class Shape {
public:
    virtual void draw() = 0;  // 纯虚函数，子类必须实现
    virtual double getArea() = 0;
    virtual ~Shape() {}  // 虚析构函数
};

class Circle : public Shape {
private:
    double radius;
    
public:
    Circle(double r) : radius(r) {}
    
    void draw() override {
        cout << "画一个圆形 ⭕，半径：" << radius << endl;
    }
    
    double getArea() override {
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;
    
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    void draw() override {
        cout << "画一个矩形 ⬜，宽：" << width << "，高：" << height << endl;
    }
    
    double getArea() override {
        return width * height;
    }
};

class Triangle : public Shape {
private:
    double base, height;
    
public:
    Triangle(double b, double h) : base(b), height(h) {}
    
    void draw() override {
        cout << "画一个三角形 🔺，底：" << base << "，高：" << height << endl;
    }
    
    double getArea() override {
        return 0.5 * base * height;
    }
};

int main() {
    Shape* shapes[] = {
        new Circle(5),
        new Rectangle(4, 6),
        new Triangle(3, 8)
    };
    
    for (int i = 0; i < 3; i++) {
        shapes[i]->draw();
        cout << "面积：" << shapes[i]->getArea() << endl << endl;
        delete shapes[i];  // 释放内存
    }
    
    return 0;
}
```

每个子类都重写了`draw`和`getArea`方法，实现自己特有的绘制逻辑。

## 面试官最爱问的陷阱题
**陷阱一：函数隐藏（最坑的那种！）**

```cpp
class Parent {
public:
    void show() {
        cout << "Parent的无参show" << endl;
    }
    
    void show(int x) {
        cout << "Parent的带参show: " << x << endl;
    }
};

class Child : public Parent {
public:
    void show() {  // 注意：这里没有virtual！
        cout << "Child的show" << endl;
    }
};

int main() {
    Child c;
    c.show();      // 正常调用Child的show
    c.show(100);   // 编译错误！为什么？
    
    return 0;
}
```

**答案：这既不是重载也不是重写，而是函数隐藏！**

Child类的`show()`把Parent类的所有`show`方法都隐藏了！即使Parent有`show(int)`版本，Child对象也看不见。

要解决这个问题，需要用`using`关键字：

```cpp
class Child : public Parent {
public:
    using Parent::show;  // 把父类的show方法都"拉"过来
    
    void show() {
        cout << "Child的show" << endl;
    }
};
```

**陷阱二：非虚函数的伪重写**

```cpp
class Base {
public:
    void func() {  // 注意：没有virtual
        cout << "Base::func" << endl;
    }
};

class Derived : public Base {
public:
    void func() {  // 看起来像重写，其实不是！
        cout << "Derived::func" << endl;
    }
};

int main() {
    Base* ptr = new Derived();
    ptr->func();  // 输出什么？
    
    delete ptr;
    return 0;
}
```

**答案：输出"Base::func"！**

因为Base的func不是虚函数，所以这不是重写，只是函数隐藏。通过父类指针调用时，永远调用的是父类版本。

**陷阱三：const重载的陷阱**

```cpp
class Test {
public:
    void print() {
        cout << "非const版本" << endl;
    }
    
    void print() const {  // 这是重载！
        cout << "const版本" << endl;
    }
};

int main() {
    Test t1;
    const Test t2;
    
    t1.print();  // 调用哪个？
    t2.print();  // 调用哪个？
    
    return 0;
}
```

**答案：**

+ t1调用非const版本
+ t2调用const版本

这是C++特有的const重载，很多人不知道const也能构成重载条件！

## 记忆口诀
最后给大家一个超好记的口诀：

**重载看参数，参数不同才叫重载，重写看继承，父子同名才叫重写(父类要有virtual)**

## 总结
好了，到这里应该彻底搞明白了吧？

+ **函数重载**：同一个类里，方法名相同，参数不同，给用户提供多种调用方式
+ **函数重写**：父子类间，父类方法必须是virtual，子类用override重新实现，方法签名完全相同 ,子类改变父类的实现。

下次面试官再问这个问题，你就可以自信地回答了。不仅要说出区别，最好还能举个生动的例子，保证让面试官印象深刻！

记住，编程不是死记硬背，而是要理解其中的道理。这两个概念理解了，面向对象编程的大门就向你敞开了一半！

---

**最后啰嗦两句：**

如果这篇文章对你有帮助，别忘了**给个赞和在看**哦！我是小康，专注分享 Linux C/C++ 后台开发的那些事儿。

欢迎关注我的公众号「**跟着小康学编程**」，每周都有干货文章，从基础语法到项目实战，从面试技巧到职场进阶，咱们一起在编程路上越走越远！

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

