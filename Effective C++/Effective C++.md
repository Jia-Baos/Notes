# Effective C++

## Item1：视C++为一个语言联邦

### C++的四个次语言

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

1. 当这个常量是一个指针类型的时候，需要将指针本身也定义为 const，这就是为什么有时会出现一次声明有两个 const 的情况，他们指代的目标是不一样的，一个是指针本身，一个是指向的对象，例如:

    ```C++
    const char* const authorName="Scott Meyers"
    ```

2. 定义 class 的专属常量的时候（而不是实例），需要限定常量的作用于在 class 内，所以必须声明为 static，这样这个常量就只能有至多一个实体了。例如：

    ```C++
    class GamePlayer{
    private:
        static const int NumTurns = 5;
        int scores[NumTurns];
    }
    ```

    需要注意的是，一般我们把声明写在头文件供其他文件使用，把实现写在实现文件中，而对于类的 static 成员，如果他是整型类型（包括int，char，bool等）,可以在实现文件中不经过定义式直接使用（前提是不去取他们的地址）。但是如果是要取地址的情况或者编译器坚持需要一个定义式（尽管这是不正确的），那么必须在使用它之前先在实现文件中再提供一遍定义式，比如：

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
const int CostEstimate::FudgeFactor = 1.36;
```

但是这只能在编译器用不着该成员对象的值的时候才能生效，例如2. const的注意事项上面的 GamePlayer 类，这个类的 scores 成员对象需要使用 NumTurns 来确定数组的大小。这个时候为了应对这种编译器（尽管它是错误的），需要使用叫做 the enum hack 的小技巧：

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

上面这个宏会选择a和b当中的最大值作为f函数的入参并返回结果。

尽管上面的每个变量都加上了括号，但是这个宏仍然无法避免一些问题。

```C++
int a=0;
int b=5;
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

### 1. 区分常量指针和指针常量

 ```C++
 const int* pb = &b;        // non-const pointer，const data 常量指针，指向“常量”的指针。在常量指针中，指针指向的内容是不可改变的，指针看起来好像指向了一个常量。
 int* const pa = &b;        // const pointer，non-const data 指针常量，指针类型的常量。在指针常量中，指针自身的值是一个常量，不可改变，始终指向同一个地址。在定义的同时必须初始化。
 const int* const pc = &c;  // const pointer，const data 指向常量的常指针。
```

### 1.const函数的返回类型

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

一个良好的用户自定义类型的特征就是避免无端的和内置类型不兼容，对于内置类型，对两数乘积赋值是无意义的，所以最好个噢诶用户自定义类型也预防这种无意义的操作。

### 2. const函数的参数

const参数可以限制函数的参数在中途进行变化，在不需要改动参数的值的情况下都应该这么做，这样可以预防类似把 == 写成 = 这样的错误，而只是付出了多打区区6个字符的代价。

### 3. const成员函数

1. 区别

    const用在成员函数上可以限制该成员函数是否能作用于const对象上面。还是以重载举例：

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

    bitwise const 阵营的人认为，只有一个成员函数不改动对象内的任何成员变量（static除外）时，才能被称之为 const，即 const 成员函数没有改变该对象的任何一个 bit。实现 bitwise const 的编译器只需要检查函数内对成员变量的赋值即可，bitwise const 也符合c++标准对const的定义。

    然而，一些可以通过 bitwise const 编译器的编译的代码实际上会导致反直觉的结果。比如说，当对象持有的只是一个指针，而不是指针指向的所有物的时候，例如仿制上文中的 TextBlock，但是不再使用 std::string 而是改用char * 代替时：

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

    //上面的定义中 operator[]符合bitwise const编译器的要求，但是会使下面的代码变得合法:

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
        bool lengthIsValid；
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
