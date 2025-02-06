# 继承与面对对象设计

## Item32：确定你的 public 继承塑模出 is-a 关系

### 请记住

- “public 继承”意味 is-a。适用于base classes身上的每一件事情一定也适用于 derived classes 身上，因为每一个 derived classes 对象也都是一个 base class 对象。

继承分为 private 继承和 public 继承。

假设 D 对象 public 继承了 B 对象，就相当于对编译器宣称“D is a B",即每一个 D 对象都是一个 B 对象，适用于 B 的任何规则一定能适用于 D 身上，反之则不成立。

但是 is-a 并非唯一存在于 classes 之间的关系，另外两个常见的关系是 has-a（有一个）,和 is-implemented-in-terms-of（根据某物实现出），其将在条款 38 和条款 39 讨论。

## Item33：避免遮掩继承而来的名称

### 请记住

- derived classes 内的名称会遮掩 base classes 内的名称。在 public 继承下从来没有人希望如此；
- 为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数（forwarding functions）。

先从一个和继承无关，但是和作用域有关的例子开始：

```C++
int x;
void someFunc()
{
double x;
std::cin>>x;
}
```

编译器在 someFunc 的作用域内遭遇 x 名称时，它会在 local 作用域内查找是否存在 x，找到的话就不会再找其他作用域。尽管 someFunc 的  x是 double 类型而 global 的 x 是 int 类型，但是这不重要，C++ 的名称遮掩规则（name-hiding rules）所做的唯一事情就是：遮掩名称。至于名称是否应该是相同或者不同的类型，并不重要。本例中 double 的 x 就遮掩了一个 int 的 x。

现在再看一个继承的例子，对于继承的 class 来说， derived class 的作用域包含在 base class 内：

```C++
class Base {
private:
    int x;
public:
    virtual void mf1()=0;
    virtual void mf2();
    void mf3();
    ...
};
class Derived:public Base{
    // public 继承
    public:
    virtual void mf1();
    void mf4();
}
```

现在我们假设mf4的内部实现是这样子的：

```C++
void Derived::mf4(){
    ...
    mf2();
    ...
}
```

mf4 内部调用了 mf2，那么：

编译器会首先查找 local 作用域（本例中就是mf4覆盖的作用域）有没有 mf2，没有找到;

接着查找 class Derived 覆盖的作用域，结果还是没有找到;

于是再查找 class Base 覆盖的作用域，发现了一个叫做 mf2 的函数，那么编译器就会使用这个函数并停止下一步查找；

我们假设 Base 没有定义 mf2 这个函数，那么编译器就会在包含了 Base 的那个 namespace（如果有这个命名空间的话）中寻找mf2，还是没有的话就在global作用域中寻找。

如果上一个例子没有什么疑点，那么下面这个例子就特别一点了：

```C++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(dobule);
    ...
};
class Derived:public Base {
    public:
    virtual void mf1();
    void mf3();
    void mf4();
    ...
}
```

这段代码的运行会让第一次面对它的 C++ 程序员大吃一惊，因为 Derived 覆盖了 Base 的 mf1（重写） 和 mf3（覆盖），于是 Base 的所有名为 mf1 和 mf3 的函数在命名遮掩规则下都“消失”了，无论它是不是 virtual，有没有带参数。

```C++
Derived d;
int x;
...
d.mf1();    // 有效，调用的是Derived::mf1
d.mf1(x);   // 错误，Derived::mf1遮掩了Base::mf1后，带参数的那个也被遮掩住了
d.mf2();    // 有效 调用Base::mf2
d.mf3();    // 有效 调用Derived::mf3
d.mf3(x);   // 错误，Derived::mf3遮掩了Base::mf3后，带有参数的那个也被遮掩住了
```

实际上，如果使用 public 继承又不打算继承那些重载函数，是违背了 is-a 的原则的。因此需要一个方法来推翻 C++ 对“继承而来的名称”的缺省遮掩行为。可以使用 using 来达成目标：

```C++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(dobule);
    ...
};
class Derived:public Base {
    public:
    using Base::mf1;
    using Base::mf3;
    virtual void mf1();
    void mf3();
    void mf4();
    ...
}
```

using 让 Base 的 mf1 和 mf3 在 Derived 作用域中都可见了，现在 d.mf1(x) 和 d.mf3(x) 都可以正常使用了。所以，当试图重新覆写（推翻）base class 的重载函数的时候，你必须为那些原本将被遮掩的名称引入 using 声明式，否则他们会被遮掩。

有时候，我们并不想继承 base classes 的所有函数，这在 public 继承下，这是绝对不能发生的，因为这违反了 public 继承所坚持的 is-a 关系，但是在 private 继承下它却可能是有意义的。

如果在上述的例子中，我们只需要 Derived 继承来自 Base 的mf1()，但是不需要带参数的 mf1，也就是只需要重载函数的一部分被继承，这个时候 using 就不合适了，而转交函数会比较奏效：

```C++
class Derived:public Base {
    public:
    ...
    virtual void mf1()  // 方便起见写成inline
    {
    Base::mf1();
    }
   ...
}
```

## Item34：区分接口继承和实现继承

### 请记住

- 接口继承和实现继承不同。在 public 继承之下，derived classes 总是继承 base class 的接口；
- pure virtual 函数只具体指定接口继承；
- 简朴的（非纯） impure virtual 函数具体指定接口继承及缺省实现继承。
- non-virtual 函数具体指定接口继承以及强制性实现继承。

1. 函数的继承

    作为 class 的设计者，有时候你会希望 derived classes 只继承成员函数的接口（也就是声明，纯虚函数 + 额外的定义）；有时候你又会希望 derived classes 同时继承函数的接口和实现，但是又可以覆写这些实现（普通虚函数）；有时候你又会希望 derived classes 同时继承函数的接口和实现的同时不允许覆写任何东西（非虚函数）。

    ```C++
    class Shape {
        public：
            virtual void draw() const = 0;
            virtual void error(const std::string& msg);
            int objectID const;
        ...
    };
    class Rectangle:public Shape{...};
    class Ellipse:public Shape{...};
    ```

    因为 draw 函数是一个 pure virtual 函数，所以 Shape 是一个抽象的 class，客户不能创建 Shape 的实体，而是只能创建它的 derived class 的实体。根据例子可以看出 Base class的如下特点：

    1. 成员函数的接口总是会被继承

        由于 public 继承坚持的 is-a 特性，任何对 base class 为真的特性对 derived class 也一定为真.

2. 声明 pure virtual 函数的目的是让 derived class 只继承接口

     例子中的 draw 是一个 pure virtua l函数，pure virtual 函数有如下特点：

      1. derived class 必须重新声明该函数

      2. 通常它们在抽象 class 中是没有定义的

    所以 pure virtual 函数的目的是让 derived class 只继承接口，换句话说，pure virtual 函数并不提供缺省的实现，而是由它的 derived class 自己实现它们。虽然 pure virtual 函数本身也是可以提供一个定义的，C++ 并不排斥给 pure virtual 函数提供实现代码，但是由于在子类中必须重新声明它，所以一般的调用是不会使用到 pure virtual 在 base class 的实现的，唯一的使用方式是明确指定其 class 的名称：

    ```C++
    Shape* ps = new Shape;      // 错误 Shape是抽象的
    Shape* ps1 = new Rectangle;
    ps1->draw();                // 调用的是Rectangle::draw
    Shape* ps2 = new Ellipse;
    ps2->draw();                // 调用的是Ellipse::draw
    ps1->Shape::draw();         // 调用的是Shape::draw
    ps1->Shape::draw();         // 调用的是Shape::draw
    ```

    一般而言这项性质没什么用，但它确实在特殊情形下可以实现一些比impure virtual函数更安全的缺省实现

3. 声明 impure virtual 函数的目的是让 derived class 同时继承接口和实现

    如果 derived class 没有覆写 impure virtual 函数，那么就会默认继承 base class 的实现。比如例子中的 error 声明为 impure virtual 意味着告诉 derived class 的设计者“你必须支持一个error函数，但是如果你不想自己写一个，可以使用Shape class提供的缺省版本”。

4. 声明 non-virtual 函数的目的是为了令 derived classes 继承函数的接口以及一份强制性的实现

    这意味着该函数不打算在 derived classes 中有不同的行为，其不变形凌驾于其特异性。

5. 设计时需要避免的问题

    1. 把所有成员函数设计为non-virtual，这将不给derived class 留下任何特意化的余地；
    2. 把所有成员函数设计为virtual，这是设计者立场不坚定的体现，就应该有一些特质是在该类和继承它的类中保持不变的。

## Item35：考虑 virtual 函数以外的其他选择

### 请记住

- virtual 函数的替代方案包括 NVI（non-virtual interface）手法及 strategy 设计模式的多种形式。NVI 手法自身是一个特殊形式的 template method 设计模式；
- 将机能从成员函数转移到 class 外部函数，带来的一个缺点是，非成员函数无法访问 class 的 non-public 成员
- tr1::function 对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式（target signature）兼容”的所有可调用物（callable entities）。

假设我们正在设计一款游戏，游戏中的人物拥有一个函数计算他们的生命值。考虑到不同人物可能计算的方式不一样，可能会有这样的设计：

```C++
class GameCharacter {
public:
virtual int healthValue() const;
...
}
```

这种设计是简洁明了的，但是其实还可以有其他的选择方案。

1. 藉由 Non-Virutal Interface 手法实现 Template Method 模式
这个流派主张 virutal 函数应该总是为 private 的。这个流派的拥护者建议，保留 healthValue 为 public 成员函数，但是将它从 virtual 函数改变成 non-virtual，在它的内部调用 private virtual 函数进行实际工作：

    ```C++
    class GameCharacter{
    public:
        int healthValue() const
        {
            ...
                int retVal = doHealthValue();
            ...
                return retVal;
        }
        ...
    private:
        virtual int doHealthValue() const
        {
            ...
        }
    };
    ```

    这一设计让客户通过 public non-virtual 成员函数间接调用 private virtual 函数，称为 non-virtual interface（NVI）手法。也就是 Template Method（模板方法）设计模式的体现。（这个模板和 C++ 模板无关）调用 virtual 函数的这个 non-virtual 接口又可以叫做外覆器（wrapper）。

    NVI 的优势在于确保 virtual 函数在进行真正工作之前可以设定好场景，调用结束后可以清理场景，比如说调用之前可以锁定互斥器，验证先决条件等等，调用之后可以接触互斥器，记录日志等等。NVI 重写的 virtual 函数是 private 的，这意味着外部不能直接调用 virtual 函数，必须通过该类提供的 Non-virtual 接口间接调用它，这就让 derived class 享有“virtual 函数如何实现”的权利的同时，让 base class 保留了“virtual 函数如何被调用”的权利，用户只需要重写 virtual 函数自定义中间的部分即可，而不是重写所有的步骤。

2. 藉由 Funtion Pointers 实现 Strategy 模式

    NVI 手法虽然有它的优点，不过最终“计算生命值”依然是在 virtual 函数内进行了，下面介绍的这种方法更加特殊。它主张“人物健康指数的计算和人物本身无关”，这种计算完全不需要“人物”这个成分。

    ```C++
    class GameCharacter;    // 前置声明
    // 默认的计算函数
    int defaultHealthCalc(const GameCharacter& gc);
    class GameCharacter{
        public:
        // 定义HealthCalcFunc是一个函数指针，它的参数是 reference to GameCharacter，返回类型是 int
        typedef int (*HealthCalcFunc)(const GameCharacter&);
        explicit GameCharacter (HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf){}
        int healthValue() const
        {return healthFunc(*this);}
        ...
        private:
        HealthCalcFunc healthFunc;
    };
    ```

    和 virtual 方法或者 NVI 风格相比，Strategy 模式即策略模式提供了更多的弹性:

    1. 同属于 GameCharacter 的不同实体可以使用不同的生命值计算函数;
    2. 生命值计算函数可以在运行时替换.

    不过让生命值计算函数不再和人物本身有关联也意味着这个函数不能访问人物实体的 non-public 部分

3. 藉由 tr1::function 来完成 Strategy 模式

    更进一步，完成生命值计算的不一定非要是一个函数，也可以是一种“类似函数的事物”（比如函数对象），甚至可以是其他对象的成员函数，返回类型也未必一定是 int，而是任何可以转换成 int 的类型。

    tr1::function 就是能解决这些问题的一种对象，该对象可以持有任何的“可调用物”（callable entity，也就是函数指针，函数对象，或者成员函数指针）；

    ```C++
    class GameCharacter;     // 前置声明
    // 默认的计算函数
    int defaultHealthCalc(const GameCharacter& gc);
    class GameCharacter{
        public:
        // 定义 HealthCalcFunc 是一个函数指针，它的参数是 reference to GameCharacter，返回类型是 int
        typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
        explicit GameCharacter (HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf){}
        int healthValue() const
        {return healthFunc(*this);}
        ...
        private:
        HealthCalcFunc healthFunc;
    };
    ```

    std::tr1::function 和原本的函数指针的区别在于它是一个指向函数的泛化指针，这个函数的参数可以是任何能够转化成 GameCharacter 类型的引用，而返回类型也可以是任何可以转化成 int 的类型。

    ```C++
    short calcHealth(const GameCharacter&); // 健康计算函数，它的返回值为 non-int
    struct HealthCalculator{
        int operator()(const GameCharacter&) const{...}
    };  // 一个函数对象
    class GameLevel{
    public:
        float health(const GameCharacter&) const;   // 成员函数，用来计算健康，返回值为non-int
    };
    class EvilBadGuy:public GameCharacter{
        ...
    };
    class EyeCandyCharacter:public GameCharacter{
        ...
    };
    EvilBadGuy ebg1(calcHealth);    // 人物一，使用某个函数计算健康指数
    EyeCandyCharacter eccl(HealthCalculator()); // 人物二，使用某个函数对象计算就健康指数

    GameLevel currentLevel;
    ...
    EvilBadGuy ebg2(
    std::tr1::bind(&GameLevel::health, currentLevel, _1)
    );  // 人物三，使用某个成员函数计算健康指数
    ```

    可以看到 GameCharacter 可以采用返回其他类型的函数，一个函数对象，甚至别的类的成员函数来作为自己的生命值计算函数。这赋予了计算生命值时的极大弹性。

4. 古典的Strategy模式

    在传统的 Strategy 模式实现中，我们可以把 virtual 函数替换为另外一个继承体系内的 virtual 函数：

    ```C++
    class GameCharacter;
    class HealthCalcFunc{
    publci:
        ...
        virtual int calc(const GameCharacter& gc) const
        {...}
        ...
    };
    HealthCalcFunc defaultHealthCalc;
    class GameCharacter{
    pbulic:
        explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc):pHealthCalc(phcf)
        {}
        int healthValue() const
        {
            return pHealthCalc->calc(*this);
        }
        ...
    private:
        HealthCalcFunc* pHealthCalc;
    }
    ```

## Item36：绝不重新定义继承而来的 non-virtual 函数

### 请记住

- 绝对不要重新定义继承而来的 non-virtual 函数。

假设有如下代码：

```C++
class B{
    public:
    void mf();
    ...
};
class D:public B{
    public:
    void mf();
    ...
};
```

当我们指定两个不同类型的指针指向同一个对象并调用 mf 时，结果可能会让大吃一惊：

```C++
D x;
B* pB=&x;
D* pD=&x;
pB->mf();
pD->mf();
// 两者调用的竟然不是同一个函数，pB调用B::mf，pD调用D::mf
```

这是因为 mf 是 non-virtual 函数，在编译器看来，non-virtual 函数是静态绑定的，也就是说因为 pB 是一个 B 类型的指针，无论它运行时指向的对象实际上是什么类型，在调用 mf 函数时始终都调用的是 B 定义的版本。

而 virtual 函数则不同，执行的是动态绑定，只有在调用时才会确定具体执行的是哪个版本的方法。条款7 就是本条款的一条特例，尽管它的重要性使它能单独被列出来。

## Item37：绝不重新定义继承而来的缺省参数值

### 请记住

- 请绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定的，而 virtual 函数——你唯一应该覆写的东西——确实动态绑定的。

从上一条款（36.绝不重新定义继承而来的 non-virtual 函数）可知，不应该重新定义 non-virtual 函数，所以本条款只讨论 virtual 函数。

不能重新定义缺省参数值的原因是，virtual 函数是动态绑定的，但是缺省参数值是静态绑定的。

```C++
class Shape{
public :
    enum ShapeColor{Red, Green, Blue};
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
}
class Rectangle:public Shape{
public:
    //错误的
    virtual void draw(ShapColor color = Green) const;
    ...
}
class Circle:public Shape
{
public:
    virtual void draw(ShapeColor color) const;
}
```

现在考虑这些指针：

```C++
Shape* ps;
Shape* pc = new Circle;
Shape* pr = new Rectangle;
```

无论这些指针实际指向谁，它们的静态类型都是 Shape。而动态类型则是运行时实际指向的对象，ps 没有指向任何对象，pc 的动态类型是 Circle，pr 的动态类型是 Rectangle。如名称所示，动态类型可以在运行时中途改变，当指针指向其他对象时，它的动态类型就有可能变化。

但是如下的代码却和期望不符：

```C++
pr->draw(); // 调用Rectangle::draw，这符合期望，但是参数是ShapeColor::Red 
```

原本我们重新定义了 color 的默认值，期望 Rectangle 执行 draw 时它的参数缺省为 Green，但是最后执行的时候却依然是 Shape 所定义的 Red。这说明， virtual 函数虽然是动态绑定，但是它的参数缺省值却是静态绑定的。

C++为什么要这么做呢？主要还是为了运行时的效率考虑，如果缺省值是动态绑定，编译器就必须设法在运行期给 virtual 函数决定适当的缺省值，这比静态绑定机制更慢且更加复杂，所以 C++ 做出了这样的取舍。

但是如果遵守上述不改动缺省值的规定，代码应该怎么写呢？下述的代码有着一定的缺点：

```C++
class Shape{
public:
    enum ShapeColor{Red, Green, Blue};
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};
class Rectangle:public Shape {
public:
    virtual void draw (ShapeColor color = Red) const;
    ...
};
```

上面代码的缺点是：代码重复且有相依性（with dependencies），如果哪一天 Shape 的 color 缺省值改动了，所有 Derived class 的 color 缺省值全部都要跟着改动。这个时候可以尝试使用条款35所提到的 virtual 函数的代替设计，例如 NVI：

```C++
class Shape{
public:
    enum ShapeColor {Red, Green, Blue};
    void draw(ShapeColor color = Red) const
    {
        doDraw(color);
    }
    ...
private:
    virtual void doDraw(ShapeColor color) const = 0;
};
class Rectangle:public Shape{
public:
    ...
private:
    virtual void doDraw(ShapeColor color) const;
    ...
}
```

## Item38：通过复合塑模出 has-a 或“根据某物实现出”

### 请记住

- 复合（composition）的意义和 public 继承完全不同；
- 在应用域（application domain），复合意味着 has-a（有一个）。在实现域（implementation domain），复合意味着 is-implemented-in-terms-of（根据某物实现出）。

复合（composition），指的是某种类型的对象包含另一种类型的对象。在程序员中复合这个术语有很多同义词，包括layering（分层），containment（内含），aggregation（聚合）和embedding（内嵌）。

复合一般有两种用途，制造出 has-a 关系，或者is-implemented-in-terms-of。has-a 关系顾名思义就是拥有，持有另一事物的意思。比如游戏角色持有武器，车辆持有车灯等等，这一点不难理解。is-implemented-in-terms-of 则是内部使用了某种对象，最终实现某个效果，比如用 list 来实现自定义 set，set 内部持有 list，但是 list 只是一个保存数据的容器，set 自己编写了存取数据的逻辑，使它能够实现 set 的特性，或者用链表的 Node 和 map 来实现自定义 LRU，LFU 等等，了解数据结构相关知识的应该清楚，很多时候容器不只有一种实现方式，很多情况下一种容器都可以根据另外一种容器实现。

## Item39：明智而审慎地使用 private 继承

### 请记住

- Private 继承意味 is-implemented-in-terms-of（根据某物实现出）。它通常比复合（composition）级别更低。但是当 derived class 需要访问 protected base class 的成员，或需要重新定义继承而来的 virtual 函数时，这么设计是合理的；
- 和复合（composition）不同，private 继承可以造成 empty base 最优。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。

private 继承并不意味着两个 class 之间有 is-a 的关系，换句话说，假设 D 以 private 的方式继承了 B，编译器并不认为任何对 B 为真的事情，对 D 也为真。private 继承相当于宣称 D 和 B 是 is-implemented-in-terms-of 的关系。

从条款34来讲，private 继承意味只有实现部分被继承，接口部分应略去。

从 private base class 继承过来的所有成员，无论其原本是 public，protected 还是 private，在 private derived class 中一律将成为private，因为所有成员都是 private derived class 的实现枝节而已。private 继承仅仅意味着 D 对象根据 B 对象实现而来，没有其他意义。

## Item40：明智而审慎地使用多重继承

### 请记住

- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对 virtual 继承的需要；
- virtual 继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果 virtual base classes 不带任何数据，将是最具使用价值的情况；
- 多重继承的确有正当用途。其中一个情节涉及“public 继承某个 Interface class”和“private 继承某个协助实现的 class”的两相组合。

C++ 社群对于多重继承的观点分为两个阵营，其中之一认为如果单一继承是好的，那么多重继承一定是更好的。另一个阵营则主张，单一继承是好的，但是多重继承不值得拥有。

多重继承可能会导致歧义，即 derived class 调用函数或者访问成员时，如果多个 base class 都有这个成员或函数，编译器会不知道具体指的是哪个，这种情况需要把明确指出使用哪个 base class 的成员才行。

钻石型多重继承：即继承的 base class 又继承了其他更上级 base class，而有若干个 base class 的更上级 base class 是相同的。C++ 默认对于这种情况将会复制多份 base class 的重复内容，如果继承时特定指明是 virtual public 继承，那么就只会复制一份。理论上对于所有的 public 继承，为了后续的维护，都应该改成 virtual public 继承，但是理论并不等于实践，因为 virtual 继承所产生的对象往往比 non-virtual 继承所产生的对象要更加大。
