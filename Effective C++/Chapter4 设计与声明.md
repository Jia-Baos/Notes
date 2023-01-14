# 设计与声明

## Item18：让接口更容易被正确使用，不易被误用

### 请记住

- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质；
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容；
- “阻止误用”的办法包括建立新类型、限制类型上的操作、束缚对象值，以及消除客户的资源难管理责任；
- tr1::shared_ptr 支持定制型删除器（custom deleter）。这颗防范 DLL 问题，可被用来自动解除互斥锁（mutexes；见条款14）等等。

## Item19：设计 class 犹如设计 type

### 请记住

- class 的设计就是 type 的设计。在定义一个新的 type 之前，请确定你已经考虑过本条款覆盖的所有讨论主题。

和其他面向对象的语言一样，每当设计出一个新的 class，同时也意味着生成了一个新的 type，要注意如下几点：

1. 新的 type 的对象如何生成和销毁？如何设计构造函数和析构函数？
2. 初始化和赋值的差别
3. 新的对象如果被 pass by value 意味着什么？注意，copy 构造函数决定了一个 type 的 pass by value 如何实现。
4. 什么是新 type 的“合法值”
5. 继承，注意 virtual 函数和 non-virtual 函数，尤其是在析构函数上面。
6. 什么样的操作符和函数对于新 type 是合理的？
7. 哪些标准函数应该驳回？（比如 copy 函数和 copy assignment 操作符）
8. 哪些成员是私有，哪些是公有？
9. “未声明接口”
10. 如果要实现的是一整个 type 家族，那么不应该设计一个 class，而是要设计一个新的 class template
11. 如果能依靠 derived class 就能实现目标，那么就不必大费周章设计一个新的 class

## Item20：宁以 pass-by-reference-to-const 替换 pass-by-value

### 请记住 

- 尽量以 pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可避免切割问题（slicing problem）。
- 以上规则并不适用于内置类型，以及 STL 的迭代器和函数对象。对它们而言，pass-by-value往往比较适当。

pass-by-value是c++继承自C的一种传递方式，考虑一下代码：

```C++
bool validateStudent(Student s);    // 函数以by value的方式接受学生
Student plato;
bool platoIsOK = validateStudent(plato);    // 调用函数
```

先调用 Student 的 copy 构造函数，以 plato 为蓝本将 s 初始化。当 validateStudent 返回时，s 被销毁。因此对此函数而言，参数的传递成本是“一次 Student copy 构造函数的调用，加上一次 Student 析构函数调用”。

1. 传递参数时调用了对象的 copy 构造函数，然后在函数结束的时候还要调用该对象的析构函数。copy 构造函数实际上还把对象拥有的成员变量都调用了对应的 copy构造函数，同理析构函数也调用了所有成员变量的析构函数，成本很高。
2. 当一个derived class被视为一个base class对象传递的时候，调用的是base class 的copy函数而不是derived class 的copy函数。
3. 因此尽量以pass-by-reference-to-const来代替pass-by-value，除非正在传递内置类型，STL的迭代器以及函数对象。

## Item21：必须返回对象时，别妄想返回其 reference

### 请记住

- 绝不要返回 pointer 或 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象。条款4 已经为“在单线程环境中合理返回 reference 指向一个 local static 对象”提供了一份设计实例。

不要让方法返回一个 reference 或 pointer 指向一个 local 对象，因为方法结束时 local 对象就已经通过析构函数摧毁了。

返回 reference 或 pointer 指向 heap-allocated 的对象同样是不可取的，因为没有函数去负责释放它的内存。

同理 reference 或 pointer 指向 static local 也可能导致错误，尤其是在同一语句中多次使用该方法并将返回值作为实参传递到其他方法去时。

## Item22：将成员变量声明为 private

### 请记住

- 切记将成员变量声明为 private，这可赋予客户访问输的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性。
- 成员的封装性和“当内容改变时可能造成的代码破坏量成反比”。
- proteced 并不比 public 更具备封装性。

参考下面的代码：

```C++
class AccessLevels
{
public:
...
int getReadOnly() const
{return readOnly;}
void setReadWrite(int value)
{readWrite = value;}
int getReadWrite() const
{return readWrite;}
void setWriteOnly(int value)
{writeOnly = value;}
}
...
private:
int noAccess;       // 对此 int 无任何访问动作
int readOnly;       // 对此 int 做只读访问（read-only access）
int readWrite;      // 对此 int 做读写访问（read-write access）
int writeOnly;      // 对此 int 做惟写访问（write-only access）
```

## Item23：宁以 non-member、non-friend 替换 member 函数

### 请记住

- 宁可拿 non-member non-friend 函数替换 member 函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性。

如果要在一个 member 函数和一个 non-member,non-friend 函数之间做抉择，并且两者提供相同的技能，那么导致较大封装性的是 non-member，non-friend 函数，因为它并不增加“能够访问 class 中的 private 成分”的函数数量。因为 member 函数不仅仅可以访问 class 中的 private 数据，也可以取用 private 函数，enums，typedefs等等，但是non-member，non-friend函数则无法访问上述的任何资源。

另外，只因在意封装性而让函数成为 class 的 non-member，并不意味着它“不可以是另外一个 class 的 member”。比如我们可以把对应方法写成某工具类中的一个 static member 函数，只要它不是目标对象的 member 或者 friend，就没有影响目标对象的封装性。应该让尽量少的函数可以接触到目标对象的成员。

C++ 中比较自然的做法：

```C++
namespace WebBrowserStuff
{
    class WebBrowser {...};
    void clearBrowser(WebBrowserStuff& wb);
    ...
}
```

备注：namespace可跨越多个源码文件，C++ 标准库就是采用此方式将不同的库封装在 std 命名空间中。

## Item24：若所有参数皆需类型转换，请为此采用 non-member 函数

### 请记住

- 如果你需要为某个函数的所有参数（包括被 this 指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个 non-member。

假设有 Rational class：

```C++
class Rational
{
public:
    Rational(int numerator = 0, int denominator = 0);   // 构造函数可以不为 explicit，允许 int-to-Rational 隐式转换
    const Rational operator* (const Rational& rhs) const;   // 乘法实现
    int numerator() const;
    int denominator() const;
private:
...
};

int main()
{
    Rational oneEighth(1, 8);
    Rational oneHalf(1, 2);
    Rational result = oneHalf * oneEight;   // 正确
    result = oneHalf * 2;   // 正确
    result = 2 * oneHalf;   // 错误
    result = oneHalf.operator*(2);  // 正确
    result = 2.operator*(oneHalf);  // 错误
}
```

隐式类型转换（implicit type conversion）。编译器知道你正在传递一个 int，而函数需要的是 Rational；但它也知道只要调用 Rational 构造函数并赋予你所提供的 int， 就可以编出一个适当的 Rational 来。于是它就那样做了。需要注意的是，只有当参数被列于参数列（parameter list）内，这个参数才是隐式类型转换的合格参与者。

```C++
const Rational temp(2); // 根据 2 建立一个临时性的 Rational 对象
result = oneHalf * temp;    // 等同于 oneHalf.operator*(temp)
```

如果想要支持混合式算数运算，可采用下面的方法：
```C++
class Rational {...};
...
const Rational operator* (const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.numerator()*rhs.numerator(), lhs.denominator()*rhs.denominator());
}
```

## Item25：考虑写出一个不抛出异常的 swap 函数

### 请记住

- 当 std::swap 对你的类型效率不高时，提供一个 swap 成员函数，并确定这个函数不抛出异常；
- 如果你提供一个 member swap，也该提供一个 non-member swap 用来调用前者。对于 classes（而非 templates），也该特化 std::swap。
- 调用 swap 时应针对 std::swap 使用 using 声明式，然后调用 swap 并且不带任何“命名空间资格修饰符”。
- 为“用户定义类型”进行 std templates 全特化是好的，但千万不要尝试在 std 内加入某些对 std 而言全新的东西。

标准库提供了一个默认的 swap 方法，绝大多数情况下它是非常好用的。

```C++
namespace std
{
    template<typename T>
    void swap(T& a,T& b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
```

只要你的类支持 copy 构造和 copy assignment 操作符，这个函数就可以使用。但是，这个方法调用了 1 次 copy 函数，2 次 copy assignment 操作符，有些时候这是没有必要的，比如说当你的类只是含有一个指针时。

```C++
class WidgetImpl    // 针对Widget数据而设计的class
{
    public:
    ...// 细节不重要
    private:
    int a,b,c;  // 可能有许多数据
    std::vector<double> v;  // 这意味着复制的时间会很长
}
class Widget // 这个class采用 pimpl（Pointer-to-implementation） 手法
{
    public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        ...
           *pImpl = *(rhs.pImpl); 
        ...
    }
    ...
    private:
    WidgetImpl* pImpl; // 指针，所致对象内含有 Widget 数据
}
```

实际上，我们在交换 Widget 的时候，只需要交换两个 pImpl 的指针指向，但是默认的 swap 会复制三个 WidgetImpl，这非常低效。我们希望告诉std::swap()，交换的类型是 Widget 的时候，只需要交换指针就好。

```C++
namespace std
{
template<>  // 编译暂时不能通过
void swap<Widget>(Widget & a, Widget& b)
{
swap(a.pImpl, b.pImpl);
}
}
```

该函数一开始的 template<> 表明它是 swap 的一个特化版本，只有 T 为 Widget 时会执行此版本 swap 方法，否则执行默认的 swap，但是这个方法访问了a、b的 private 成员，所以它不能通过编译。

为了让 swap 可以最终访问 private 成员，我们可以给 Widget 补充一个 public 函数，并且让 std::swap 调用它。

```C++
class Widget
{
    public：
        ...
        void swap(Widget& other)
    {
        using std::swap; //稍后解释这个声明的必要性
        swap(pImpl,other.pImpl);
    }
    ...
};
namespace std{
    template<>
    void swap<Widget>(Widget& a,Widget& b)
    {
        a.swap(b);
    }
}
```

这个做法是可行的，而且和 STL 容器的处理方式一致，STL 的容器也提供了 std::swap 的特化版本和自己的 public swap 方法。

但是，假设 Widget 不是一个 class 而是一个 class Template：

```C++
template<typename T>
class WidgetImpl{...};
template<typename T>
class Widget{...};
```

当想要继续特化 std::swap 时却出现了问题。

```C++
namespace std{
template<typename T>
void swap<Widget<T>> (Widget<T>& a,Widget<T>& b)//不合法
{
a.swap(b); 
}
}
```

这是因为对于 C++ 的偏特化而言，只允许对 class template 偏特化，而不能对 function template 偏特化。

如果要类似的效果，可以试着重载函数。

```C++
namespace std{
template<typename T>
void swap(Widget<T>& a,Widget<T>& b)
{
a.swap(b);
}
}
```

然而，std 作为标准库原则上不能被添加新的 template（特化不算添加），class 或者方法，所以如果要避免未定义行为，最好避免这种写法。其中一个解决方案是，放弃对 std::swap 的特化，而是声明在自己的命名空间内。

```C++
namespace WidgetStuff{
... 
template<typename T>
class Widget{...};
...
template<typename T>
void swap(Widget<T>& a,Widget<T>& b)
{
a.swap(b);
}
}
```

现在，任何地方的任何代码如果打算交换两个 Widget，C++ 的名称查找法则（name lookup rules，更准确的说是 argument-dependent-rules )会找到 WidgetStuff 中的 swap 版本并使用它。当然如果不加上这个命名空间 WidgetStuff，在 global 命名空间中依然会拥有这些模板，也不影响使用，但这会让 global 命名空间太过拥挤。

那么，当想要进行交换的时候，假设我们正在写一个 function tmeplate，需要置换其中两个值：

```C++
template<typename T>
void doSomething(T& obj1,T& obj2)
{
    ...
        swap(obj1,obj2);
    ...
}
```

我们所希望做的是，尝试调用 T 类型的 swap 特化版本，并且在该方法不存在的情况下，调用 std 中的一般化版本。

所以应该这么做：

```C++
template<typename T>
void doSomething(T& obj1,T& obj2)
{
    ...
    using std::swap;
    swap(obj1,obj2);
    ...
}
```

一旦 swap 被调用，C++ 基于实参的名称查找规则会查找 global 命名空间或者 T 类型所在命名空间任何 T 专属的 swap，如果没有才会使用 std 的 swap，这就是为什么需要在函数内使用 using std::swap；即便如此编译器还是更喜欢 std::swap 的 T 专属特化，假设你已经针对 T 把 std::swap 特化，特化版将会被编译器挑选中。

记得不要用命名空间固定死 swap 的命名空间，这样名称查找规则就失效了。

```C++
std::swap(obj1, obj2);  // 错误
```

注意这个在自己命名空间定义的 non-member 的 swap 不应该抛出异常，因为 swap 函数应该具备强烈的异常安全性。
