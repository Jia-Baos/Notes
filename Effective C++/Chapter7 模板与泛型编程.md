# 模板与泛型编程

## Item41：了解隐式接口和编译期多态

### 请记住

- classes 和 templates 都支持接口（inferences）和多态（polymorphism）；
- 对 classes 而言接口是显式的（explicit），以函数签名式（函数名称、参数类型、返回类型）为中心。多态则是通过 virtual 函数发生于运行期；
- 对 template 参数而言，接口是隐式的（implicit），奠基于有效表达式（valid expression）。多态则是通过 template 具现化和函数重载解析（function overloading resolution）发生于编译期。

## Item42：了解 typename 的双重意义

### 请记住

- 声明 template 参数时，前缀关键词 class 和 typename 可以互换；
- 请使用关键字 typename 表示嵌套从属类型名称；但不得在 base class list（基类列）或 member initialization list（成员初值列）内以它作为 base class 修饰符。

对于 template 声明式而言，class 和 typename 对于 C++ 是没有任何区别的。

```C++
template<class T> class Widget;
template<typename T> class Widget;  // 没有区别
```

但是并不是所有情况 typename 和 class 都是等价的

假设我们有如下一段不能通过编译的代码：

```C++
template<typename C>
void print2nd(const C& container)
{
    if(container.size()>=2){
        C::const_iterator iter(container.begin());
        ++iter;
        int value=*iter;
        std::cout << value;
    }
}
```

问题出在 C::const_iterator 上面,对于一个 template，C::const_iterator 这个名称具体取决于 C 到底是什么名称，这种名称叫做从属名称，如果某个类型定义在另一个类型内，又可以被叫做嵌套从属名称，所以 C::const_iterator 是嵌套从属名称，而 int 并不依赖任何一种模板类型，叫做非从属类型。问题就出在编译器不清楚 C::const_iterator 究竟是一类型还是一个静态变量，这造成编译器的解析困难，比如说如下情况：

```C++
C::const_iterator *x;
```

一般人可能认为定义了一个 C::const_iterator 的指针,名称为x，但是 C::const_iterator 可能是一个静态变量，x 可能是一个全局变量，结果 * 就成了乘法符号，那么这一语句就意味着 C::const_iterator 和 x 相乘。C++ 编译器必须考虑所有的可能情况，这就是为什么上述代码不能通过编译的原因。为了告诉编译器，C::const_iterator 是一种类型，必须在前面加上 typename 作为提示，一般来说，任何当你想要在 template 中指涉一个嵌套从属类型名称，就必须在紧邻它的前一个位置放上一个 typename 关键字。

```C++
template<typename C>
void print2nd(const C& container)
{
    if(container.size()>=2){
        typename C::const_iterator iter(container.begin());
        ++iter;
        int value=*iter;
        std::cout << value;
    }
}
```

typename 只被用来验明嵌套从属类型名称；其它名称不该有它的存在。

```C++
template<typename C>    // 允许使用 typename 或 class
void f(
    const C& container，    // 不允许使用 typename
    typename C::iterator iter   // 一定要使用 typename
)；
```

这一规则的例外是，typename 不用出现在 base classes list 中的嵌套从属类型名称之前，也不可以在 member initialization list（成员初值列）中作为 base class 修饰符，原因也许是因为在这些场所编译器不需要提醒也知道它们代表的意思。例如：

```C++
template<typename T>
class Derived:public Base<T>::Nested    // 不允许 typename
{
public:
    explicit Derived(int x):Base<T>::Nested(x)  // 不允许 typename
    {
        typename Base<T>::Nested temp;  // 一定要 typename
    }
}
```

## Item43：学习处理模板化基类内的名称

### 请记住

- 可在 derived class templates 内通过“this->”指涉 base class templates 内的成员名称，或藉由一个明白写出的“based class 资格修饰符”完成。

1. 编译器会拒绝在模板化基类当中寻找名称

    假设我们要写一个程序，给不同公司发送消息，信息可以不做加工，也可以加密，我们可以采用template的写法：

    ```C++
    class CompanyA{
        public:
        ...
            void sendCleartext(const std::string& msg);
            void sendEncrypted(const std::string& msg);
        ...
    }
    class CompanyB {
        public:
        ...
            void sendCleartext(const std::string& msg);
            void sendEncrypted(const std::string& msg);
        ...
    }
    ...
    class MsgInfo {...};

    template<typename Company>
    class MsgSender{
        public:
        ...
            void sendClear(const MsgInfo& info)
        {
            std::string msg;
            // 根据info产生信息
            Company c;
            c.sendCleartext(msg);
        }
            void sendSecret(const MsgInfo& info)
        {
            std::string msg;
            // 根据info产生信息
            Company c;
            c.sendEncrypted(msg);
        }
    }
    ```

    但是假如我们为 MsgSender 设计一个 Derived class，下面的代码无法通过编译：

    ```C++
    template<typename Company>
    class LoggingMsgSender: public MsgSender<Company> {
        public:
        ...
            void sendClearMsg(const MsgInfo& info)
        {
            //补充日志
            ...
                sendClear(info);//本语句无法通过编译
            //补充日志
            ...
        }
    }
    ```

    原因在于 LoggingMsgSender 继承了 MsgSender\<Company\>，但是 Company 是个 template 参数，所以编译器并不确定 base class 是否拥有一个叫做 sendClear 的方法，这有点奇怪，MsgSender 不是定义了一个叫做 sendClear的 方法吗？这个方法应该跟 template 参数无关才对。但是编译器需要考虑特化的情况，假设 CompanyZ 坚持使用加密发送的方式，那么 CompanyZ 就不应该有 sendClear 的方法：

    ```C++
    class CompanyZ{
    public:
    ... 
    void sendEncrypted(const std::string& msg);
    ...
    }
    ```

    于是 MsgSender 对于 CompanyZ 就不合适了，因为 MsgSender 提供的 sendClear 方法内部调用了 Company 的 sendCleartext，但是其实 CompanyZ 是没有该方法的。这时候我们需要针对 CompanyZ 产生一个特化版本：

    ```C++
    template<>
    class MsgSender<CompanyZ> {
    public:
    ...
    void sendSecret(const MsgInfo& info)
    {
    ...
    }
    }
    ```

    这个全特化版本制定了当 template 参数是 CompanyZ 是 MsgSender 应该具有的特化版本是什么。

    所以对于 LoggingMsgSender 这个 Derived class，编译器担心出现 template 参数是 CompanyZ 的情况，因为这种情况下，Base class 直接就没有 sendClear 这个方法，所以编译器以防万一拒绝了本次调用——也就是说编译器会拒绝在模板化基类当中寻找名称。

2. 解决方案

    为了解决这个问题，有三种方案：

   1. 在调用前加上this->

        ```C++
        template<typename Company>
        class LoggingMsgSender: public MsgSender<Company> {
            public:
            ...
                void sendClearMsg(const MsgInfo& info)
            {
                // 补充日志
                ...
                    this->sendClear(info);  // 加上this->
                // 补充日志
                ...
            }
        }
        ```

        sendClear 是一个非从属名称，但是 this 在模板类中是一个隐式的从属名称，所以 this->sendClear 也是从属名称，编译器会推迟到对象被实例化才去查找 sendClear。

   2. 使用 using

        ```C++
        template<typename Company>
        class LoggingMsgSender: public MsgSender<Company> {
            public:
            using MsgSender<Company>::sendClear;
            ...
                void sendClearMsg(const MsgInfo& info)
            {
                //补充日志
                ...
                sendClear(info);
                //补充日志
                ...
            }
        }
        ```

        使用 using 告知编译器要使用 MsgSender\<Company\> 的 sendClear 后，原本不会再基类查找名称的编译器就会去查找。

   3. 指出该名称就在 base class 中（不推荐）

        ```C++
        template<typename Company>
        class LoggingMsgSender: public MsgSender<Company> {
            public:
            ...
                void sendClearMsg(const MsgInfo& info)
            {
                //补充日志
                ...
                    MsgSender<Company>::sendClear(info);
                //补充日志
                ...
            }
        }
        ```

        这么做会导致调用的如果是virtual函数，上述指定范围的行为会导致不进行动态绑定，所以不推荐。

## Item44：将与参数无关的代码抽离 templates

### 请记住

- Templates 生成多个 classes 和多个函数，所以任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系；
- 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可以消除，做法是以函数参数或 class 成员变量替换 template 参数；
- 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representstions）的具现类型（instantiation types）共享实现码。

当编写某个函数，其中的一部分实现和另一个函数相同时，较好的办法是把相同的部分抽出来放进一个单独的函数，再在两个函数中调用这个单独的函数。当编写某个 class，这个 class 的某些部分和其他 class 相同，较好的方法是把相同的部分放在一个新的类中，然后用复合或者继承的方式把令其他的 class 取用这些共同的特性，而保留互异的部分在原来的 class 中。但是在编写 template 的代码的时候，代码膨胀比编写函数或者一个普通的 class 是更加难以察觉的，尽管编写的代码看起来数量上没有膨胀，但是编译后根据 template 产生的类或者方法的目标码实际上有很多的重复部分。

假设正在写一个关于矩阵的 template，该矩阵有一个方法用来进行逆矩阵计算：

```C++
template<typename T,std::size_t n>
class SquareMatrix{
    public:
    ...
        void invert();
}
```

其中 T 是模板类型参数，n 是模板非类型参数，然后执行下述代码：

```C++
SquareMatrix<double,5> sm1;
sm1.invert();
SquareMatrix<double,10> sm2;
sm2.invert();
```

这份代码会具现化出两个 invert 方法，但这两个方法一个操作 5 \* 5 矩阵，另一个操作 10 \* 10 矩阵，除此之外没有两个函数没有任何区别，这也是一种难以察觉的代码膨胀。它并不体现在编写的代码上，而是编译产生的目标码上面。

解决方案如下：

```C++
template<typename T>
class SquareMatrixBase {
protected:
...
    void invert(std::size_t matrixSize);
...
};
template<typename T,std::size_t n>
class SquareMatrix:private SquareMatrixBase<T> {
private:
    using SquareMatrixBase<T>::invert;  // SquareMatrix 对 invert 进行了重载，要防止编译器看不到 SquareMatrixBase 的 invert
public:
    ...
        void invert() {this->invert(n);}    // 使用模板化基类的方法，加上了 this
}
```

带参数的 invert 处于 base class 中，和 SquareMatrix 一样，SquareMatrixBase 也是个 template，但是它只有一个模板类型参数，也就是只确定模板的元素对象是什么类型，而尺寸不进行参数化。对于固定类型的元素，所有的矩阵都共享一个 SquareMatrixBase，也共享同一个 SquareMatrixBase 的 invert，这样就避免了代码膨胀。两者之间是 private 继承，表明只是用 SquareMatrixBase 来实现 SquareMatrix，两者不是 is-a 关系。使用函数参数或者成员变量来替代非类型 template 参数，可以有效减少代码膨胀。

类型 template 参数也会造成代码膨胀，比如说 template 分别为 int 和 long 实现了两个具现版本，尽管 int 和 long 在某些平台上有相同的二进制表述。更有代表性的情况是，对于指针而言，绝大多数平台上，所有指针类型都有相同的二进制表述，所以如果在某个 template 中操作强型指针（即T\*），可以尝试让它们调用另外一个操作无类型指针（即void *）的函数，如果你确实打算阻止因为类型 template 参数导致的代码膨胀的话。像很多 C++ 标准库的实现版本确实为 vector，list，deque 等 template 做了这些事。

## Item45：运用成员函数模板接受所有兼容类型

### 请记住

- 请使用 member function templates（成员函数模板）生成“可接受所有兼容类型”的函数；
- 如果你声明 member templates 用于“泛化 copy 构造”或“返回 copy assignment 操作”，你还是需要声明正常的 copy 构造函数和 copy assignment 操作符。

智能指针指的是行为像指针的对象，它们还可以提供普通指针所没有的机能。比如 auto_ptr 可以自动释放资源，再比如 STL 容器内基本都使用智能指针作为迭代器，因为一个普通指针不可能通过“++”操作符转移到 linked list 的下一个节点，但是 list::iterators 就做得到。

不仅仅是 STL 提供的智能指针，有时候我们也会想要用代码自定义自己的智能指针，而普通指针是支持隐式转换的，比如从 Derived Class 转换成 Base Class，从 non-const 转换成 const。

```C++
class Top{...};
class Middle:public Top{...};
class Bottom:public Middle{...};
Top* pt1 = new Middle;  // Middle*转换为Top*
Top* pt2 = new Bottom;  // Bottom* 转换为 Top*
const Top* pct2 = pt1;  // Top* 转换为const Top*
```

但是如果想要在用户自定的指针中完成上面的转换，就比较麻烦：

```C++
template<typename T>
class SmartPtr{
public:
    explicit SmartPtr(T* realPtr);  // 智能指针通常以内置的原始指针完成初始化
    ...
};
SmartPtr<Top> pt1 = SmartPtr<Middle>(new Middle);   // SmartPtr<Middle>转换为SmartPtr<Top>
SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom);   // SmartPtr<Bottom>转换为SmartPtr<Top>
SmartPtr<const Top> pct2 = pt1; // SmarPtr<Top>转换为SmartPtr<const Top>
```

我们想要上述代码能够成立，但是虽然 Top，Middle，Bottom 之间是 public 继承代表的 is-a 关系，但是以它们为模板参数完成的 SmartPtr 之间并不具备这种继承的关系。很显然，转型部分应该和构造函数这部分相关，我们需要重写构造函数，来让新的构造函数能够适应这些转型，但是另外一个很显然的事实是，我们不可能写出所有的构造函数，因为这意味着，一旦日后添加了新的 Derived class，我们又需要重新为 SmartPtr 补充新的构造函数，可维护性会大大降低。我们应该做的不是重写构造函数，而是做一个类似的事情，为 SmartPtr 写一个构造模板：

```C++
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other); // member template，生成copy构造函数的模板
}
```

根据一个 SmartPtr\<U\> 生成一个 SmartPtr\<T\>，两者是一个 template 的不同具现体，这种写法又叫做泛化 copy 构造函数。上述泛化 copy 构造函数并没有被声明为 explicit，因为应当允许隐式的转换，就像普通指针所做的一样。

但是我们并不希望 SmartPtr\<Top\> 可以转化为 SmartPtr\<Bottom\>，或者 SmartPtr\<double\> 可以转化为 SmartPtr\<int\>，也就是不是任意的 SmartPtr\<U\> 都可以生成一个 SmartPtr\<T\>，而是只有在 U 可以转换为 T 的情况下才允许 SmartPtr\<U\> 转换 SmartPtr\<T\>，所以必须在泛化 copy 构造函数中创建的成员函数群进行一个筛选。

假设自定义的智能指针和 auto_ptr 或者 tr1:share_ptr 一样，提供一个 get 函数返回自己指向的对象，那么我们可以在构造模板的实现中实现对隐式转换的约束：

```C++
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other)
    :heldPtr(other.get())   // 以other的heldPtr初始化this的heldPtr
    {...}
    T* get() const {return heldPtr;}
    ...
    private:
    T* heldPtr;//持有的内置原始指针
}
```

因为条款 41 提到的对于 template 而言，接口是隐式的，奠基于有效表达式,所以只有在 heldPtr(other.get())是有效表达式，也就是存在某个隐式转换可以把 U \* 转换成 T \* 时，才能够通过编译。

一个泛化的 copy 构造模板函数并不影响编译器为 class 生成缺省的 copy 构造函数。所以如果你想全方位的掌控 copy 相关的方方面面，除了泛化的 copy 构造模板函数，最好再声明一个正常的 copy 构造函数，这个道理对 copy assignment 赋值操作符也是适用的。

## Item46：需要类型转化时请为模板定义非成员函数

### 请记住

- 当我们编写一个 class template，而它所提供之“与此 template 相关的”函数支持“所有参数值隐式类型转换”时，请将那些函数定义为“class template内部的 friend 函数”。

假设现在设计一个有理数类，但是设计为一个模板类，分子分母的类型不是 int 而是模板类型参数：

```C++
template<typename T>
class Rational {
    public:
    Rational(const T& numerator = 0, const T& denominator = 1);
    const T numerator() const;
    const T denominator() const;
    ...
};
template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,
    const Rational<T>& rhs)
{...}
```

以下代码将会无法通过编译：

```C++
Rational<int> oneHalf(1,2);
Rational<int> result = oneHalf * 2;//无法通过编译
```

这是因为 \* 操作符是一个模板函数，对于模板函数一般通过它的实参来推断模板参数类型，对于第一个参数 oneHalf 很容易推断出 T 是 int，但是对于第二个参数2，编译器看不出来 2 对应的 Rational\<T\> 当中的T到底是什么类型。尽管 2 是可以隐式转换成一个 Rational 的，但是在 template 实参类型推导的过程中，编译器不会考虑隐式转换，这是因为此时函数本身都没有被具现化出来，编译器不确定 T 到底是什么。

解决方法是，把 * 操作符设计为一个 friend 函数而不是一个函数模板：

```C++
template<typename T>
class Rational {
public:
...
    friend const Rational operator* (const Rational& lhs,
        const Rational& rhs);
};
template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,
                            const Rational<T>& rhs)
{...}
```

这样做能够行得通的原因是，当 oneHalf 被构造出来的时候，class Rational\<int\> 已经根据模板具现化出来了，友元函数 operator\*（接受Rational\<int\> 作为参数) 也随之被具现化出来，operator\* 并不是一个函数模板而是一个随着 class template 被具现化而被确定的函数，所以能够接受隐式转换。

## Item47：请使用 traits classes 表现类型信息

### 请记住

- Traits classes 使得“类型相关信息”在编译期可用。它们以 template 和 “template 特化”完成实现；
- 整合重载技术（overloading）后，traits classes 有可能在编译期对类型执行 if...else 测试。

1. STL的迭代器分类

    STL的迭代器具有5种分类：

    1. Input迭代器

        只能向前移动，只能用来读取，且只能读取一次

    2. Output迭代器

        只能向前移动，只能用来写入，且只能写入一次

    3. forward迭代器

        只能向前移动，可以用来读取或者写入多次

    4. Bidirectional迭代器

        可以前后移动

    5. random access迭代器

        可以进行迭代器算数，一口气向前向后移动任意距离，类似指针算数一样

    C++ 给 5 个分类提供专属的卷标结构加以确认，而且它们之间有继承关系：

    ```C++
    struct input_iterator_tag{};
    struct output_iterator_tag{};
    struct forward_iterator_tag:public input_iterator_tag{};
    struct bidirectional_iterator_tag:public forward_iterator_tag{};
    struct random_access_iterator_tag:public bidirectional_iterator_tag{};
    ```

2. advance 函数

    STL 中提供了一个模板方法，将迭代器向前或者向后移动若干个单位

    ```C++
    template<typename IterT, typename DistT>
    void advance(IterT& iter, DistT d); // 迭代器向前移动 d 个单位，d < 0 的话那么向后移动
    ```

    对于非 random_access 的迭代器，我们只能在内部一次次调用 iter++ 或者 iter--，而对于 random_access 迭代器，我们可以使用迭代器算数一步到位。换句话说，理想的实现应该是这样的：

    ```C++
    template<typename IterT,typename DistT>
    void advance(IterT& iter,DistT d)
    {
        if(iter is a random access iterator)
        {
            iter+=d;//只针对random access iterator进行迭代器算数运算
        }
        else
        {
            if(d>=0){while (d--) ++iter;}
            else{while (d++) }-- iter;
        }
    }
    ```

    所以我们必须知道 iter 是不是一个 random access，要取得类型的某些信息。这正是 traits 所做的：帮助用户在编译期间知道某些类型信息。

3. traits

    Traits 并不是什么关键字或者一个预先定义好的构件，而是 C++ 程序员共同遵守的定义。该技术的要求之一是，它对内置类型和用户自定义类型的表现必须一样好。举个例子，如果 advance 受到的实参是一个 const char\* 和一个 int，那么 advance 必须仍然可以有效运作，那意味着 traits 技术必须可以施行于内置类型比如指针上面。

    “traits 必须能够施行于内置类型"意味着”类型内的嵌套信息“这种东西出局了，因为我们无法把信息嵌套在原始指针里面。所以类型的 traits 信息必须位于类型自身的外面。标准技术是把它放在一个 template 以及它的一个或者多个特化版本中，这样的 templates 在标准程序库中有若干个，和迭代器相关的被命名为 Iterator_traits：

    ```C++
    template<typename IterT>
    struct iterator_traits;
    ```

    习惯上 traits 总是被实现为 struct，但是它们常常被称呼为 traits classes。

    iterator_traits 的运作方式是，针对每一个类型 IterT，在 struct iterator_traits\<IterT\> 内声明某个 typedef 名为 iterator_category，用来确认 IterT 的迭代器分类。

    首先对于用户自定义的迭代器类型，要求必须嵌套一个 typedef，名为 iterator_category。比如说 deque 的迭代器可以随机访问，所以针对 deque 迭代器设计的 class 看起来是这个样子：

    ```C++
    template<...>   // 参数略写
    class deque{
    public:
        class iterator {
        public:
            typedef random_access_iterator_tag iterator_category;
            ...
        };
        ...
    };
    ```

    对于 list 的迭代器：

    ```C++
    template<...>
    class List
    {
    public:
        class iterator {
        public:
            typedef bidirectional_iterator_tag iterator_category;
            ...
        };
        ...
    }
    ```

    而对于 iterator_trait 本身而言只需要响应 iterator class 的嵌套式 typedef 就好：

    ```C++
    template<typename IterT>
    struct iterator_traits {
        typedef typename IterT::iterator_category iterator_category;
        ...
    };
    ```

    其次，对于原始指针，因为其属于内部类型，它不可能嵌套 typedef，所以 iterator_traits 专门提供一个偏特化版本（partial template specialization）。

    ```C++
    template<typename IterT>
    struct iterator_traits<IterT *> // 对于内置指针的偏特化
    {
    typedef random_access_iterator_tag iterator_category;   // 因为内置指针支持迭代器运算，所以分类为random_access_iterator_tag
    ...
    }
    ```

    这样一来，可以实现之前的伪代码了:

    ```C++
    template<typename IterT, typename DistT>
    void advance(IterT& iter, DistT d)
    {
    if(typeid(typename std::iterator_traits<Iter>::iterator_category)==typeid(std::random_access_iterator_tag))
        {
            iter+=d;//只针对random access iterator进行迭代器算数运算
        }
        else
        {
            if(d>=0){while (d--) ++iter;}
            else{while (d++) }-- iter;
        }
    }
    ```

    但是这么写还是有缺点的，iter 的类型早在编译期就确定了，但是判断却拖到运行期才执行，我们可以利用重载解决这个问题，让 advance 调用一个重载函数，重载函数真正完成对应的任务。

    ```C++
    template<typename IterT, typename DistT>
    void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag)
    {
        iter+=d;
    }

    template<typename IterT, typename DistT>
    void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag)
    {
        if(d>=0){while(d--) ++iter;}
        else {while(d++) --iter;}
    }

    template<typename IterT, typename DistT>
    void doAdvance(IterT& iter, DistT d, std::input_iterator_tag)
    {
    if(d<0)
    {
        throw std::out_of_range("Negative distance");
    }
        while(d--) ++iter;
    }
    ```

    然后再advance中调用重载函数：

    ```C++
    template<typename IterT, typename DistT>
    void advance(IterT& iter, DistT d, std::input_iterator_tag)
    {
    doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category())
    }
    ```

## Item48：认识 template 元编程

### 请记住

- Template metaprogramming（TMP，模板元编程）可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率；
- TMP 可被用来生成“基于政策选择组合”（based on combinations of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。
