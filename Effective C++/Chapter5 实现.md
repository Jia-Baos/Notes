# 实现

## Item26：尽可能延后变量定义式的出现时间

### 请记住

- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。

请对比以下变量定义式出现的时机：

```C++
// 这个函数过早定义变量“encrypted”，在抛出异常的情况下还是需要付出 encrypted 的构造成本和析构成本
std::string encryptPassword(const std::string& password)
{
using namespace std;
std::string encrypted;
if(password.length() < MinimumPasswordLength)
{
    throw logic_error("Password is too short");
    ...
}
return encrypted;
}
```

```C++
// 优化版本一，在抛出异常的情况下不需要付出 encrypted 的构造成本和析构成本
std::string encryptPassword(const std::string& password)
{
using namespace std;

if(password.length() < MinimumPasswordLength)
{
    throw logic_error("Password is too short");
    ...
}
std::string encrypted;
return encrypted;
}
```

```C++
// 优化版本二（考虑需要对 encrypted 进行加密）
void encrypt(std::string& s);   // 加密操作

std::string encryptPassword(const std::string& password)
{
using namespace std;

if(password.length() < MinimumPasswordLength)
{
    throw logic_error("Password is too short"); // 在抛出异常的情况下不需要付出 encrypted 的构造成本和析构成本
    ...
}

// 版本一，先执行默认构造函数，后执行 copy assigment 操作符
std::string encrypted;
encrypted = password;
encrypt(encrypted);

// 版本二 直接执行 copy 构造函数
std::string encrypted(password);
encrypt(encrypted);

return encrypted;
}
```

## Item27：尽量少做转型动作

### 请记住

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免 dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户端随后可以调用该函数，而不需将转型放进他们自己的代码内。
- 宁可使用 C++ style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

1. 旧式转型
   1. C风格转型

       ```C++
       (T)expression
       ```

   2. 函数风格转型

       ```C++
       T(expression)
       ```

    上述两种旧式转型风格没有任何区别。

2. 新式转型（C++ style）
   1. 常量性转除

        ```C++
        const_cast<T>(expression) 
        ```

        将对象的常量性移除（cast away），这也是唯一有此能力的C++操作符。

   2. 动态转型

        ```C++
        dynamic_cast<T>(expression)
        ```

        用来执行“安全向下转型”，唯一一个无法用旧式语法做到的转型方式。

   3. 重解释转型

        ```C++
        reinterpret_cast<T>(expression)
        ```

        低级转型，实际操作可能取决于编译器，这意味着它不可移植。例如把 pointer to int 转型为int，在低级代码以外非常少见。

   4. 静态转型

        ```C++
        static_cast<T>(expression)
        ```

        强制进行隐式转换，比如把 non_const 转型为 const，int 转 double 等等，但是不能把 const 转为 non_const，因为那是 const_cast 的工作。

## Item28：避免返回 handles 指向对象内部成分

### 请记住

- 避免返回 handles（包括 reference、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助 const 成员函数的行为像个 const，并将发生“虚吊号码牌”（dangling handles）的可能性降至最低。

避免返回 handle（这里的 handle 指的是 reference，指针或者迭代器）指向内部的成员，原因如下：

1. 破坏了对象的封装性，一旦你在某个访问级别较宽松的函数当中获得了一个指向内部访问级别较严苛的成员的handle，那么这个成员实际上的访问级别就跟这个函数一样宽松了。

2. 即便给返回类型限定了const让编译器禁止通过handle修改内部部分，也可能导致handle的声明周期比对象本身还要长，进而出现错误。

## Item29：为“异常安全”而努力是值得的

### 请记住

- 异常安全函数（Exception-safe functions）即使发生异常也不会泄露资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型；
- “强烈保证”往往能够以 copy-and-swap 实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义；
- 函数提供的“异常安全保证”通常最高值等于其所调用之各个函数的“异常安全保证”中的最弱者。

1. 异常的处理方式

    具备异常安全性的函数会有如下三种对异常的处理方式。

   1. 基本承诺

        如果异常被抛出，程序的任何失误仍然保持在有效状态下，没有任何对象或者数据结构会因此败坏，所有的对象都处于一种内部前后一致的状态（比如所有的 class 约束条件都会继续获得满足）。但程序的实际状态不可预料，客户必须调用某个成员函数来得知异常抛出后对象的当前内容。

   2. 强烈保证：

        异常被抛出的话，程序的状态不会改变。调用这种函数就可以明确：程序要么完全成功，要么完全失败。

   3. 不拋掷保证（nothrow）：

        承诺该函数绝不抛出任何异常，该函数一定能完成原本承诺的功能。用于内置类型（比如 ints，指针等等）身上的所有操作都提供 nothrow 保证。

2. 实现

    一般来说，不拋掷保证的函数是理想情况。但这几乎是不可能的，任何使用动态内存的东西（例如 STL 的容器）都可能因为内存不足导致 bad_alloc 异常。所以，多数情况下函数只能在基本承诺和强烈保证当中二选一。

    当要实现强烈保证的时候，通常的伎俩是使用 copy-and-swap，先给原件做出一个副本，然后再副本上做一切必要的修改，这时候如果出现任何异常，原件仍然保持不变，直到所有改变都成功，再把修改过的那个副本和源对象在一个不抛出异常的操作中置换） swap（条款 25 中提到过 swap 是保证不抛出异常的。

    copy-and-swap 一般是管用的，但要考虑如下的情况：

    ```C++
    doSomething()
    {
        ...//copy and doxxx
        f1();
        f2();
        ...//doxxx and swap
    }
    ```

    首先，如果f1，f2只能实现基本承诺而非强烈保证，但 doSomthing 意图实现强烈保证，那么这意味着需要在 f1 之前进行 copy，f1 之后进行 swap，f2 同理。即便 f1，f2 都是强烈保证，依然会有一个问题，f1 顺利返回之后，程序状态已经被改变了，如果 f2 抛出了异常，那么必须回退到 doSomething 调用前的状态，而 f1 已经发生了，导致状态不一致。如果 f1，f2 修改的是局部状态（local state）还比较好解决，如果是非局部状态（non-local state）有连带影响时，比如数据库的改动，提供强烈保证就非常的困难。

    另外，copy-and-swap 手法进行了太多的 copy 副本的工作，耗费了时间和空间。所以，尽管强烈保证是我们所需要的特性，但并不是所有时候都值得为它付出代价，强烈保证并不是所有时候都实际的。

    函数所调用的异常安全保证不可能高于它所调用的其他函数的异常安全保证的最低等级。系统也是同理，系统中只要有一个函数是不实现异常安全保证（即连上文中的基本承诺都不提供），那么就不可以说整个系统有异常安全性。异常安全性只有“有”和“没有”的区别，没有“部分有”这种说法。

## Item30：透彻了解 inlining 的里里外外

### 请记住

- 把大多数 inlining 限制在小型的、被频繁调用的函数身上。这可以使日后的调试过程和二进制升级（binary upgradability）更加容易，也可以使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为 function templates 出现在头文件，就把它们声明为 inline。

## Item31：将文件间的编译依存关系降至最低

### 请记住

1. 编译依赖

    假设有如下类：

    ```C++
    class Person{
        public:
        Person(const std::string& name,const Date& birthday,const Address& addr);
        std::string name() const;
        std::string birthDate() const;
        std::string address() const;
        ...
        private:
        std::string theName;
        Date theBirthDate;
        Address theAddress;
    };
    ```

    如果没有 string，Date 和 Address 的定义式，这段代码无法通过编译。而这样的定义式经常由 #include 提供，所以经常来说 Person 定义文件上方应该有这些代码：

    ```C++
    #include <string>
    #include "date.h"
    #include "address.h"
    ```

    于是，这么一来 Person 定义文件和其含入的文件之间就形成了一种编译依存关系（compilation dependency）。如果这些头文件中任意一个被改变，或者他们所依赖的其他头文件有任何改变，那么所有含入 Person class 的文件也要重新编译，任何使用 Person class 的文件也必须重新编译。这会非常麻烦。

    那么为什么不使用前置声明呢？

    ```C++
    //错误
    namespace std{
        class string;
    }
    class Date;
    class Address;
    class Person{
        public:
        Person(const std::string& name,const Date& birthday,const Address& addr);
        std::string name() const;
        std::string birthDate() const;
        ...
    }
    ```

    上述代码首先就弄错了 string 不是一个 class 而是一个 typedef（定义为 basic_string\<char\>），正确的前置声明是什么并不重要，因为你本来就不应该尝试手工去声明一部分的标准程序库，而是应该采用 #inlcude。事实上，标准库的头文件不太可能成为编译的瓶颈，特别是当你的建置环境支持预编译头文件的时候。

    但是关于“前置声明每一个东西”的第二个（同时也是更加重要的一点）困难是，编译器在编译期间需要知道每一个成员的大小。

    ```C++
    int main()
    {
    int x;  // 定义一个int
    Person p(params);   // 定义一个Person
    }
    ```

    为了定义一个 int，编译器首先需要知道一个 int 要有多大，才能给 x 分配内存，当然，所有的编译器都知道一个 int 有多大。对于 Person，编译器得知这项信息的唯一方法就是访问 Person 的定义式（声明式只是表明变量的类型和名字，而定义式才是决定内存的分配，所以一个变量可以被声明多次，而只能定义一次）。

2. pimpl（pointer to implement)

    那么为什么其他语言诸如 java、Smalltalk 不会出现这种问题呢？因为当我们声明一个成员对象时，它们的编译器会只分配一个指针的空间给该对象，也就是把代码视为这个样子:

    ```C++
    int main()
    {
    int x;
    Person* p;
    }
    ```

    C++ 虽然不会让编译器这么做，但我们可以手动编码完成类似的实现，一种叫做 pimpl idiot 的设计风格可以使用。那我们把 Person 分割为接口和实现类，把实现类取名为 PersonImpl，Person.h 的内容改成如下：

    ```C++
    #include <string>
    #include <memory>
    class PersonImpl;
    class Date;
    class Address;
    class Person{
        public:
        Person(const std::string& name,const Date& birthday,const Address& addr);
        std::string name() const;
        std::string birthDate() const;
        std::string address() const;
        ...
        private:
        std::tr1::shared_ptr<PersonImpl> pImpl;
    }
    ```

    现在Person中只有一个指针成员指向实现类了。于是编译的依赖关系从原来的：

    main.cpp 以及其他使用了 Person 的 cpp 依赖 Person.h，Person.cpp 依赖 Person.h

    改动 Person.h，Person.cpp main.cpp 等其他 cpp 都受影响

    变成了：

    main.cpp 及其他使用了 Person 的cpp  依赖 Person.

    Person.cpp 依赖 Person.h，PersonImpl.h（因为Person.cpp 需要调用 PersonImpl 的方法，所以需要它的定义式），PersonImpl.cpp 依赖 PersonImpl.h

    改动 PersonImpl.h ，只有 PersonImpl.cpp，Person.cpp 受到了影响，main.cpp 及其他使用了Person 的 cpp，只依赖 Person.h 而不是 Person.cpp，所以不受影响。

3. 减少编译依赖

    这一设计的关键在于把“定义的依存性”换成了“声明的依存性”，这也是编译依存性最小化的本质：尽可能让头文件自我满足，万一做不到，就让它和其他文件内的声明式（而非定义式）相依。以下的几个准则都是依据这一本质而来：

    1. 如果可以使用object reference 或者object pointer代替，那么不要使用objects

        因为指针或者引用对于编译器来说占用内存的大小是固定的，对于编译器来说该成员是一个指针而不是某个具体的 object，不需要找到定义式就可以直接使用，但是定义某个 object 的成员，就一定要用到定义式。

    2. 尽量用 class 声明式而非 class 定义式

        注意，当声明一个函数的参数或者返回类型需要某个 class 式，是不需要定义式的。只有调用这个函数的文件或者实现文件才需要使用定义式。

        ```C++
        class Date;
        Date today();
        void clearAppointments(Date d); // 两者都合法，都不需要定义式
        ```

        这样做就把“提供定义式”（通过#include）的义务从“函数声明所在”的头文件转移到“函数实现或者调用”的实现文件，就可以将“非真正必要之类型定义”和客户端之间的编译依存性去除掉。

    3. 为声明式和定义式提供不同的头文件。

        使用两个头文件分别用于声明式和定义式。当然如果有一个声明式被改变了，两个文件都得改变。这样做的话，使用者就可以总是 #include 一个声明文件而不是前置声明若干次。比如说 Date 的使用者希望声明 today 和 clearAppointments，它不应该前置声明 Date，而是应该 include 一个包含了对应的声明式的头文件。

        ```C++
        #include "datefwd.h"
        Date today();
        void clearAppointments(Date d);
        ```

        datefwd.h 中包含了 class Date 的声明式（但没有定义），其中 fwd 代表 forward declaration 即前置声明。

        标准库中类似的例子是 \<iosfwd\>，它包含 iostream 各种组件的声明式，但是它们的定义式分布在若干不同的头文件内，包括 \<sstream\>，\<streambuf\>，\<fstream\> 和 \<iostream\>。

4. 实现

    像上文中的 Person 这样的 pimpl 设计的实现一般被称为 Handle class，应该如何实现它呢，一般有两种方法。

    1. 转交给实现类来实现

        ```C++
        //Person.cpp
        #include "Person.h"
        #include "PersonImpl.h"//注意，因为需要使用PersonImpl的成员函数，所以需要它的定义式
        //构造函数
        Person::Person(const std::string & name,const Date& birthday,const Address& addr):pImpl(new PersonImpl(name,birthday,addr))
        {}

        std::string Person::name() const
        {
            return pImpl->name();
        }
        ```

    2. 把 Handle class 当做抽象基类

        另一种方法是把 Person 当做特殊的 abstract base class，称为 Interface class。这种 class 的目的是详细意义描述 derived classes 的接口，因此它不带有成员变量，也没有构造函数，只有一个 virtual 析构函数（见条款7）以及一组 pure virtual 函数。

        这种 Interface class 的射击类似 Java 和 .NET 的 Interfaces，但是 C++ 的 Interface 并不需要承担 Java 和 .NET 的 Interface 需要承担的责任。比如说 Java 和 .NET 不允许在 Interfaces 实现成员变量或者成员函数（其实 Java 已经可以了...），但是 C++ 并不禁止。

        ```C++
        class Person
        {
            public:
            virtual ~Person();
            virtual std::string name() const=0;
            virtual std::string birthDate() cosnt=0;
            virtual std::string address() const=0;
            ...
        };
        ```

        这个 class 的客户必须以 Person 的 pointers 或者 references 来编写应用程序。因为这个类不可能为内含 pure virtual 函数的 Person classed 具现出实体，但是却有可能为派生自 Person 的 classes 具现出实体。除非 Interface class 的接口被修改，否则其客户不需要重新编译。

        但是客户必须要有办法给这种 class 创建新的对象。他们通常调用一个特殊函数。这个函数扮演“真正将要被具现化”的那个 derived classes 的构造函数的角色，又叫工厂函数或者 virtual 构造函数。它们返回指针（最好是智能指针），指向动态分配所得的对象，而该对象支持 Interface class 的接口，该方法一般是 static 的：

        ```C++
        class Person{
            public:
            ...
                static std::tr1::shared_ptr<Person> create(const std::string & name,const Date& birthday,const Address& addr);
            ...
        }
        ```

        客户这样使用他们：

        ```C++
        std::string anme;
        Date dateofBirth;
        Address address;
        ...
        std::tr1::shared_ptr<Person> pp(Person::create(name,dateOfBirth,address));
        ...
            std::cout<<pp->name()
                    << "was born on"
                    <<pp->birthDate()
                    <<"and now lives at"
                    <<pp->address();
        ```

        然后我们需要一个 Interface class 的真正的具象类（concrete class），这个类提供接口的 virtual 函数的具体实现，而且要有一个构造函数。

        ```C++
        class RealPerson:public Person{
        public:
        RealPerson(const std::string& name,const Date& birthday,const Address& addr)
        :theName(name),theBirthDate(birthday),theAddress(addr)
        {}
        virtual ~RealPerson() {}
        std::string name() cosnt {...};
        std::string birthDate() cosnt {...};
        std::string address() const {...};
        private:
        std::string theName;
        Date theBirthDate;
        Address theAddress;
        }
        ```

        然后再实现 Person::create 这个工厂函数：

        ```C++
        std::tr1::shared_ptr<Person> Person::create(const std::string& name,const Date& birthday,const Address& addr)
        {
            return std::tr1::shared_ptr<Person>(new RealPerson(name,birthday,addr));
        }
        ```

    无论是哪种实现会在运行时付出额外的代价，换取编译时的便利，所以需要在这之间做出取舍。
