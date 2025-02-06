# 让自己习惯 C++

## Item1：视 C++ 为一个语言联邦

### 请记住

C++ 高效编程守则是状况而变化，取决于你使用 C++ 的哪一部分。

### C++ 的四个次语言

- C
  - 没有模板（templates）
  - 没有异常（exceptions）
  - 没有重载（overloading）
- Object-Oriented C++
  - 封装（encapsulation）
  - 继承（inheritance）
  - 多态（polymorphism）
  - 虚函数（动态绑定）
- Template C++
  - 泛型编程（generic programming）
  - 模板元编程（TMP、template metaprogramming）
- STL
  - 容器（containers）
  - 迭代器（iterators）
  - 算法（algorithms）
  - 函数对象（function objects）

## Item2：尽量以 const, enum, inline 替换 #define

### 请记住

- 对于单纯常量，最好以 const 对象或 enum 替换 #define；
- 对于形似函数的宏（macros），最好改用inline函数替换 #define。

### 1. const 替换 #define 的原因

由于 #define 的代码会被预处理器进行处理，编译器看不到被替换前的代码，这恰恰可能让它导致一些问题。由于它不会被编译器看到，反而会让错误发生时调试者难以定位错误发生的原因。

例如下面的语句：

```C++
#define  ASPECT_RATIO 1.653
```

这会导致发生错误的时候，错误信息报告的是 1.653 而不是 ASPECT_RATIO，所以调试者找不到错误发生的原因，尤其是在 ASPECT_RATIO 被定义在非调试者自己编写的头文件时，调试者会被这个错误信息弄得一头雾水。

解决方法是改写成常量：

```C++
const double AspectRatio = 1.653;   // 因为全大写一般用于宏，这里改动了变量名
```

这样一来编译器就能识别该变量并且让 AspectRatio 加入记号表，错误信息就会更加准确。

### 2. const 的注意事项

1. 当这个常量是一个指针类型的时候，需要将指针本身也定义为 const，这就是为什么有时会出现一次声明有两个 const 的情况，它们指代的目标是不一样的，一个是指针本身，一个是指向的对象，例如:

    ```C++
    const char* const authorName = "Scott Meyers";
    ```

2. 定义 class 的专属常量的时候（而不是实例），需要限定常量的作用于在 class 内，所以必须声明为 static，这样这个常量就只能有至多一个实体了。例如：

    ```C++
    class GamePlayer{
    private:
        static const int NumTurns = 5;
        int scores[NumTurns];
    }
    ```

    需要注意的是，一般我们把声明写在头文件供其他文件使用，把实现写在实现文件中，而对于类的 static 成员，如果它是整型类型（包括 int，char，bool 等），可以在实现文件中不经过定义式直接使用（前提是不去取它们的地址）。但是如果是要取地址的情况或者编译器坚持需要一个定义式（尽管这是不正确的），那么必须在使用它之前先在实现文件中再提供一遍定义式，比如：

    ```C++
    const int GamePlayer::NumTurns; // 不需要设定初值，因为这只是一个定义式，值已经在声明的时候赋予了
    ```

    备注：定义式用来把一些声明式遗漏的细节补充给编译器。

### 3. the enum hack的小技巧

在某些比较旧式的编译器，可能会不允许声明式中直接赋值，所以只能把初值放在定义式上。

```C++
// 头文件
class CostEstimate{
private:
    static const double FudgeFactor ;
    
}
// 实现文件
...
const double CostEstimate::FudgeFactor = 1.36;
```

但是这只能在编译器用不着该成员对象的值的时候才能生效，例如 2. const 的注意事项上面的 GamePlayer 类，这个类的 scores 成员对象需要使用 NumTurns 来确定数组的大小。这个时候为了应对这种编译器（尽管它是错误的），需要使用叫做 the enum hack 的小技巧：

```C++
class GamePlayer{
private:
    enum
    {
        NumTurns = 5; // 类似这种把 NumTurns 变成 5 的一个记号名称的方法叫 the enum hack
    }
    int scores[NumTurns];
}
```

这种方法还有另外的一种好处：const 定义的值是可以取地址的，但 enum 不能被取地址，如果不想让常量被取地址，使用 enum 可以帮助实现这个约束。此外，enum 和 #define 不会导致非必要的内存分配。

### 4. inline 替换 #define 的原因

有时候#define会用来实现宏，比如：

```C++
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
```

上面这个宏会选择 a 和 b 当中的最大值作为f函数的入参并返回结果。

尽管上面的每个变量都加上了括号，但是这个宏仍然无法避免一些问题。

```C++
int a = 0;
int b = 5;
CALL_WITH_MAX(++a, b);          // a 自增了两次
CALL_WITH_MAX(++a, b +1 0);     // a 自增了一次
```

这种#define的使用方式会让错误信息难以排查问题所在。使用template inline函数可以有效避免这个问题。

```C++
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```

这个template可以实现相同的功能，并且他还有作用域和访问规则，比#define要更加优秀。此外，使用 inline 函数则可以获得宏带来的效率以及一般函数所有可预料行为和类型安全性

### 5. 宏定义的优点

1. 提高了程序的可读性，同时也方便进行修改；
2. 提高程序的运行效率：使用带参的宏定义既可完成函数调用的功能，又能避免函数的出栈与入栈操作，减少系统开销，提高运行效率；
3. 宏是由预处理器处理的，通过字符串操作可以完成很多编译器无法实现的功能。比如##连接符。

### 6. 宏定义的缺点

1. 由于是直接嵌入的，所以代码可能相对多一点；
2. 嵌套定义过多可能会影响程序的可读性，而且很容易出错；
3. 对带参的宏而言，由于是直接替换，并不会检查参数是否合法，存在安全隐患。

### 7. 补充

预编译语句仅仅是简单的值代替，缺乏类型的检测机制。这样预处理语句就不能享受C++严格的类型检查的好处，从而可能成为引发一系列错误的隐患。的确，宏定义给我们带来很多方便之处，但是必须正确使用，否则，可能会出现一些意想不到的问题。最后，引用《C陷进与缺陷》的一句话，对其进行总结：宏并不是函数，宏并不是语句，宏并不是类型定义。

## Item3：尽可能使用 const

### 请记住

- 将某些东西声明为 const 可帮助编译器侦测出错误用法。const 可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体；
- 编译器强制实施 bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）；
- 当 const 和 non-const 成员函数有着实质等价的实现时，令 non-const 版本调用 const 版本可避免代码重复。

### 1.const 函数的返回类型

const可以被用于限定函数返回一个常量：

```C++
class Rational {...};
const Rational operation* (const Rational& lhs, const Rational& rhs);
```

表面上看起来这种限定是无意义的，但是这可以避免用户写出这样的代码：

```C++
Rational a, b, c;
...
(a * b) = c;
```

一个良好的用户自定义类型的特征就是避免无端的和内置类型不兼容，对于内置类型，对两数乘积赋值是无意义的，所以最好用const 修饰用户自定义类型以预防这种无意义的操作。

### 2. const 函数的参数

const 参数可以限制函数的参数在中途进行变化，在不需要改动参数的值的情况下都应该这么做，这样可以预防类似把 == 写成 = 这样的错误，而只是付出了多打区区6个字符的代价。

### 3. const 成员函数

1. 区别

    const 用在成员函数上可以限制该成员函数是否能作用于const对象上面。还是以重载举例：

    ```C++
    class TextBlock{
    private:
        std::string text;
    public:
        ...
        //operator[] for const对象
        const char& operator[] (std::size_t position) const
        {return text[position];}
        //operator[] for non-const对象
        char& operator[] (std::size_t position)
        {return text[position];}
    }
    ```

    于是

    ```C++
    // 调用 non-const TextBlock::operator[]
    TextBlock tb("HELLO");
        std::cout << tb[0];
    // 调用 const TextBlock::operator[]
    const TextBlock ctb("World");
        std::cout << ctb[0];
    ```

    注意，上面的 const operator[] 不仅仅限制了成员函数，对返回类型也使用了 const 限定，这样一来同样的操作符用在 const 和非 const 对象上，返回的结果就会不一样。如下所示：

    ```C++
    //接上面的代码
    std::cout << tb[0];         // 正确,返回的是char&
    tb[0] = 'x';                // 正确，char& 可以更改引用的值
    std::cout << ctb[0];        // 正确，返回的是const char&
    ctb[0] = 'x';               // 不正确，const char&不可以被修改
    ```

    另外，假设 non-const operator[] 返回的是 char 而非 char& 的话，赋值操作也是不被允许的，当返回对象不是引用而是值的时候，对返回值赋值是不合法的，即便合法，被改动的也会是 text 的副本而不是编写者原本想要的结果。

2. 两种阵营

    对于 const 成员函数的定义，有两种阵营。bitwise const 阵营和 logical const 阵营。

    bitwise const 阵营的人认为，只有一个成员函数不改动对象内的任何成员变量（static 除外）时，才能被称之为 const，即 const 成员函数没有改变该对象的任何一个 bit。实现 bitwise const 的编译器只需要检查函数内对成员变量的赋值即可，bitwise const 也符合 C++ 标准对 const 的定义。

    然而，一些可以通过 bitwise const 编译器的编译的代码实际上会导致反直觉的结果。比如说，当对象持有的只是一个指针，而不是指针指向的所有物的时候，例如仿制上文中的 TextBlock，但是不再使用 std::string 而是改用 char * 代替时：

    ```C++
    class CTextBlock{
        public :
        ...
        char& operator[] (std::size_t position) const
        {
            return pText[position];
        }
        private:
        char* pText;
    }
    ```

    上面的定义中 operator[] 符合 bitwise const 编译器的要求，但是会使下面的代码变得合法：

    ```C++
    const CTextBlock cctb("Hello");
    char* pc = &cctb[0];
    *pc = 'J';    // 改动了cctb的值
    ```

    上述代码能够被执行，但是编写者在创建了一个const对象并且只对其使用const函数的情况下，还是改变了它的内容。

    这就致使了另外一派阵营 logical constness 的诞生。支持 logical constness 的人认为，const 函数应该允许修改它所处理的对象中的一些 bits，但是只有在客户端不知情的前提下才可以这么做，比如为上面的 CTextBlock 添加一个高速缓存用来存储文本长度以便询问。

    ```C++
    class CTextBlock{
        private:
        char* pText;
        bool lengthIsValid;
        std::size_t textLength;
        public :
        std::size_t CTextBlock::Length() const
        {
            if(!lengthIsValid)
            {
                lengthIsValid = true;
                textLength = std::strLen(pText);
            }
            return textLength;
        }
    }
    ```

    显然 Length 函数违反了 bitwise const 的标准，如果编译器不同意这样子赋值，但是编写者又想这样做，应该如何解决？

    答案是添加 mutable 关键字，mutable 可以释放掉 non-static 成员上面的 bitwise-const 约束：

    ```C++
    // 这些成员变量即便是在const函数内也可以被更改
    mutable bool lengthIsValid；
    mutable std::size_t textLength;
    ```

    编译器强制实施 bitwise constness，但是编写程序时应该使用“概念上的常量性”(conceptual constness).

3. 避免 const 和 non-const 的内容重复

    假设上文 TextBlock 的 operator[] 是不再是简单的返回 reference，还包括边界检验，记录，数据完善性检验等操作的话，那么：

    ```C++
    class TextBlock{
    private:
        std::string text;
    public:
        ...
        // operator[] for const对象
        const char& operator[] (std::size_t position) const
        {
            ...// 边界校验
            ...// 志记数据访问
            ...// 数据完善性检验
            return text[position];
        }
        //operator[] for non-const对象
        char& operator[] (std::size_t position)
        {
            ...// 边界校验
            ...// 志记数据访问
            ...// 数据完善性检验
            return text[position];
        }
    }
    ```

    可以发现 const operator[] 和 non-const operator[] 之间的代码几乎是重复的，当然，可以把边界检验，记录，数据完善性检验放到另外一个函数当中然后分别调用他们，但是更好的方式是一个函数调用另外一个。

    在上述代码中，const 跟 non-const 的代码几乎完全一样，除了返回的类型多了一个 const 资格修饰，这个时候把返回值的 const 转除掉是安全的，因为不论谁调用 non-const 成员函数首先要提供 non-const 对象，所以可以让 non-const 成员函数去调用 const 兄弟：

    ```C++
    class TextBlock{
    private:
        std::string text;
    public:
        ...
        // operator[] for const对象
        const char& operator[] (std::size_t position) const
        {
            ...// 边界校验
            ...// 志记数据访问
            ...// 数据完善性检验
            return text[position];
        }
        char& operator[] (std::size_t position)
        {
            return 
                const_cast<char&>( // 把op[]的返回值的const转除掉
                static_cast<const TextBlock&>(*this) // 为*this加上const
                [position]
            );
        }
    }
    ```

    注意，我们应该让 non-const 调用 const，而不是让 const 调用 non-const，因为 const 函数承诺不改变对象的逻辑状态，而 non-const 则没有这个要求。

4. const 指针

    如果const出现在*左边，说明被指向的对象是常量，出现在右边则说明指针本身是常量

    ```C++
    char greeting[] = "Hello";
    char* p = greeting;                 //non-const pointer,non-const data
    const char* p = greeting;           //non-const pointer,const data
    char* const p = greeting;           //const pointe,non-const data
    const char* const p = greeting;     //const pointer,const data
    ```

## Item4：确定对象被使用前先被初始化

### 请记住

- 为内置型对象进行手工初始化，因为 C++ 不保证初始化它们；
- 构造函数最好使用成员初值列（member initialization list），而不要在构造函数本体内使用赋值操作（assignment）。初值列类出的成员变量，其排列次数应该和它们在 class 中的声明次序相同；
- 为免除“跨编译单元之初始化次序”问题，请以 local static 对象替换 non-local static 对象。

在某些语言中，不存在“无初值对象”,但是 C++ 不同。在某些平台上，读取未初始化的值会直接让程序终止运行，也有可能读取到一些“污染”的随机数据，无论如何，这都不是期望的结果。

在条款1中提到过，C++ 是 4 个子语言组成的语言联邦，如果使用的C part of C++ 这部分，那么不保证会发生初始化，但是在 non C part of C++，情况有所不同。这就是为什么 array（来自 C part of C++）不保证初始化，而 vector（来自 STL）可以保证初始化。

### 1. 初始化

最好的办法就是在使用类之前永远把他的成员们都初始化。

对于内置类型，应该手工完成初始化，比如：

```C++
int a = 0;
```

对于非内置类型，初始化的责任应该落在构造函数上。

但是需要区分的是赋值和初始化之间的区别。以下面的例子为例：

```C++
class PhoneNumber {...};
class ABEntry{
    // ABEntry是 Address Book Entry的缩写
    public:
    ABEntry(const std::stirng& name ,const std::string& address,const std::list<PhoneNumber>& phones);
    private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};
    ABEntry::ABEntry(const std::stirng& name ,const std::string& address,const std::list<PhoneNumber>& phones)
    {
        theName=name;
        theAddress=address;
        thePhones=phones;
        numTimesConsulted=0;
    }
```

这个构造函数使用起来没有任何问题，也能达到预期的效果，但是构造函数里面发生的一切并不是初始化，而是赋值。

C++ 规定，成员变量的初始化在进入构造函数前就已经发生了。在进入 ABEntry 构造函数之前，就调用了这些成员的 default 初始化函数（内置类型除外）。

一个改动方式是：

```C++
 ABEntry::ABEntry(const std::stirng& name ,const std::string& address,const std::list<PhoneNumber>& phones)
  :theName(name),theAddress(address),thePhones(phones),numTimesConsulted(0)  //这一行代码也叫成员初值列
 {}
 ```

上述代码和改动前能实现一模一样的效果，但是效率更高，因为他们没有浪费 default 初始化函数的操作去重新赋值。对于拥有 default 初始化函数的用户自定义类型，在 default 构造函数后再调用 copy assignment 赋值操作的效率低于直接调用 copy 构造函数/default 构造函数，但是对于内置类型，赋值或者初始化的成本是相同的。

当成员变量是 cosnt 或者 reference 时，它们就一定需要初值而不能再被赋值了。为了避免记住哪些变量需要初始化，哪些不需要，最好的做法就是总是使用成员初值列。

但是，如果 class 拥有多个构造函数，又有多个成员变量和 base classes，多份的成员初值列的编写将是重复繁琐的。这种情况可以把赋值表现和初始化一样好的成员拿出来改成赋值操作，放到一个（一般是private）函数当中供各个构造函数调用。

C++ 的成员初始化是按照声明的顺序进行的，在成员初值列中却可以调转顺序，这样可以通过编译但是在维护代码时可能让检查者很困惑，比如一个数组需要一个整数作为大小，但是这个整数在成员初始列中却排在数组的后面，在初值列很长时可能让检查者很困惑。

### 2. 不同编译单元内定义的 non-local static 对象的初始化次序

首先需要一点一点分析这个一连串词语组成的标题的含义。

static 对象的声明周期，从构造出来一直持续到程序结束为止。函数内定义的 static 对象被称为 local static，因为它对于函数而言是 local 的。其他被称为 non-local static。在程序结束时，static 对象的析构函数会被自动调用。

编译单元指产出单一目标文件的源码。基本上，它是单一源码文件加上其所含入的头文件（#include files）。

假设有两个源码文件，都拥有 non-local static 对象，但是其中一个的初始化使用了另外一个编译单元的 non-local static 对象。由于 C++ 没有明确规定他们的初始化顺序，导致无法保证被使用的对象已经被初始化了。

解决方案是把 non-local static 替换成 local static，也就是把这些对象放到一个函数中去，该函数返回该对象的引用，双方不再直接使用其他编译单元的 static 对象，而是改成通过对方提供的函数获得该对象的引用。很容易察觉到，这是著名的单例模式的常见实现。
