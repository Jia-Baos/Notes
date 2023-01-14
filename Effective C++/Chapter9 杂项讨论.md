# 杂项讨论

## Item53：不要轻忽编译器的警告

### 请记住

- 严肃对待编译器发出的警告信息。努力在你的编译器的最高（最严苛）警告等级下争取“无任何警告”的荣誉；
- 不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原本倚赖的警告信息有可能消息。

不要习惯性的忽略编译器警告。很多人认为如果问题足够严重，编译器应该发出一个错误信息而非警告，这种观点在其他语言可能奏效，但是在 C++ 中，可以打赌编译器作者对将要发生的事情有比你更加好的领悟。比如下面这个例子：

```C++
class B{
    public:
    virtual void f() const;
};
class D:public B{
    public:
    virtual void f();
}
```

B::f是一个const函数，而D::f则不是。部分编译器可能会有如下警告：

```C++
D::f() hides virtual B::f()
```

经验不足的程序员对这个信息的反应是：“哦当然，D::f遮掩了B::f，这很正常”实际上是编译器在提醒你声明在 B 中的 f 并没有在 D 中被重新声明，而是被整个遮掩了（条款33详细说明了原因）。如果忽略这个警告，几乎肯定导致程序的错误行为，然后是很多调试行为，只是为了找出编译器其实早就侦测出来并告诉你的事情。

## Item54：让自己熟悉包括 TR1 在内的标准程序库

### 请记住

- C++ 标准库的主要机能由 STL、iostream、locales 组成。并包含 C99 标准程序库；
- TR1 添加了智能指针（例如 tr1::shared_ptr）、一般化函数指针（tr1::function）、hash-based 容器、正则表达式（regular expression）以及另外 10 个组件的支持；
- TR1 自身只是一份规范。为获得 TR1 提供的好处，你需要一份实物。一个更好的实物来源是Boost。

1. TR1 和 C++ 的历史

    C++ Standard——定义 C++ 语言及其标准程序库的规范——早在 1998 年就被标准委员会核准了。标准委员会在 2003 年发布了一个不那么重要的“错误修正版”,并且预计在 2008 年左右发布C++ Standard 2.0。日期上的不确定性让人们总是称呼下一版 C++ 为“C++0x",意指 200x 版本的 C++。

    C++0x 或许会覆盖某些有趣的语言新特性，但是大部分新技能都将会以标准程序库的扩充形式体现。如今我们已经能够知道某些新的程序库机能，因为它被详细叙述于一份称为 TR1 的文档内。TR1 代表“Technical Report 1"，那是 C++ 程序库工作小组对该份文档的称呼。标准委员会保留了 TR1 被证实铭记于 C++0x 之前的修改权，不过目前已经不可能再接受任何重大改变了。就所有意图和目标而言，TR1 宣称了一个新版 C++ 的来临，我们也许可以称之为 Standard C++ 1.1。不熟悉 TR1 技能而奢望称为一位高效率的 C++ 程序员是不可能的，因为 TR1 提供的机能几乎对每一种程序库和每一种应用程序都带来利益。

2. C++ Standard 1.0 标准程序库的组成

   1. STL（Standard Template Library，标准模板库）

       覆盖容器（如vector，stirng，map），迭代器（iterators），算法（如find,sort,transfoem），函数对象（如 less,greater），各种容器适配器（如stack，priority_queue）和函数对象适配器（如mem_fun，not1）。

   2. Iostreams

       覆盖用户自定缓冲功能、国际 化I/O，以及预先定义好的对象如 cin、cout、ceer、clog。

   3. 国际化支持

       包括多区域能力。比如 wchar_t（通常是 16 bits/char）和 wstring（由 wchar_ts 组成的 strings）等类型都对促进 Unicode 有所帮助。

   4. 数值处理

       包括复数模板（complex）和纯数值数组（valarray）。

   5. 异常阶层体系

       包括作为 base class 的 exception 以及作为 derived classes 的 logic_error 和 runtime_error，以及更深继承的各个 classes。

   6. C89标准程序库

       1989 年 C 标准程序库内的每个东西也都被覆盖于 C++ 内。

3. TR1 的组成

    1. 智能指针（smart pointers）

        1. tr1::shared_ptr

            如同内置指针一样工作，但会记录有多少个 tr1::shared_ptr 指向同一个对象，一旦最后一个这样的指针被销毁，即某对象的引用次数变成 0 时，自动销毁该对象。但是对于环形数据结构依然会造成资源泄漏

        2. tr1::weak_ptr

            不参与引用计数的计算，也就是说当最后一个 tr1::shared_ptr 被销毁，即便还有若干个 tr1::weak_ptr 指向它，该对象依然会被删除。

    2. tr1::function

        用来表示任何 callable entity（可调用物，任何函数或者函数对象），只要其签名符合目标。比如说我们像注册一个 callback 函数，该函数接受一个 int 并且返回一个 string，我们可以这么写：

        ```C++
        void  registerCallback(std::string func(int));
        ```

        其中参数名称 func 是可有可无的，上述代码也可以这么声明：

        ```C++
        void registerCallback(std::string (int));
        ```

        此处的 std::string (int)是一个函数签名,tr1::function 使上述的 RegisterCallback 有可能更加富弹性的接受任何可调用物，只要这个可调用物接受一个 int 或者任何可被转换为 int 的东西。

        ```C++
        void registerCallback(std::tr1::function<std::string(int)> func)
        ```

        剩下的tr1组件可以被分为两组。第一组提供彼此互不相干的独立机能:

    3. tr1::bind

        它能够做 STL 绑定器（binders），bind1st 和 bind2nd 所做的每一件事情，而且还可以做的更多。和前任绑定器不同的是 tr1::bind 可以和 const 及  non-const 成员函数共同协同运作，可以和 by-reference 参数协同运作。而且它不需要特殊协助就看可以处理函数指针，所以我们调用 tr1::bind 之前不必再被 ptr_fun，mem_fun 或者 mem_fun_ref 搞得一团糟了。简单来说 tr1::bind 是第二代绑定工具，并且比前一代好用多了。

    4. Hash tables

        用来实现 sets，multisets，maps 和 multi-maps。每个新容器的接口都以其前任（TR1之前的）对应容器塑膜而成.如它们的名称所示：

        ```C++
        tr1::unordered_set;
        tr1::unordered_multiset;
        tr1::unordered_map;
        tr1::unordered_multimap;
        ```

        这些名称强调了以 hash 为基础的这些 tr1 容器内部并没有任何可预期的次序。

    5. 正则表达式

        包括以正则表达式为基础的字符串查找和替换，或者从某个匹配字符串到另一个匹配字符串的注意迭代等等。

    6. Tuples

        这是标准程序库中 pair 的新一代制品。pair 只能持有两个对象，但是 tr1::tuple 可以持有任意个数的对象。

    7. tr1::array

        本质上是一个“STL 化”数组，也就是支持成员函数比如 begin 和 end 的数组。不过它的大小是固定的，并不适用动态内存

    8. tr1::mem_fn

        语句构造上与成员函数指针一致的东西。如同 tr1::bind 纳入并扩充了 C++98 的 bind1st 和 bind2nd 的能力一样，tr1::mem_fn 纳入并扩充了 C++98 的 mem_fun 和 mem_fun_ref 的能力

    9. tr1::reference_wrapper

        一个“让 references 的行为更像对象”的设施。它可以造成容器“犹如持有 references”，而你知道容器实际上只能持有对象或者指针。

    10. 随机数生成工具

        它大大超越了 C++ 从 C 标准程序库继承而来的 rand。

    11. 数学特殊函数

        包括 Laguerre 多项式，Bessel 函数，完全椭圆积分等等更多数学函数。

    12. C99 兼容扩充

        这是一大堆函数和模板（templates），用来把许多新的 C99 程序库特性带进 C++。

        第二组 TR1 组件由更加精巧的 template 技术（包括template metaprogramming，也就是模板元编程构成）。

    13. type traits

        见条款 47，可以指出 T 是否是个内置类型，是否提供 virtual 函数，是否是个 empth class，是否可以隐式转换成类型 U...等等，也可以显现给该给定类型之适当齐位，这对于定制型内存分配器的编写人员是十分关键的信息。

   14. tr1::result_of

        这是一个 template，用于推到函数调用的返回类型。当我们编写 templates 时，能够指涉（refer to）函数或者函数把调用动作所返回的对象的类型往往很重要，但是该类型有可能以复杂的方式取决于函数的参数类型。tr1::result_of 是的“指涉函数返回类型”变得失分容易。它自己也被 TR1 自身的若干组件采用。

## Item55：让自己熟悉 Boost

### 请记住

- Boost 是一个社群，也是一个网站。致力于免费、源码开放、同僚复审的 C++ 程序库开发。Boost 在 C++ 标准化过程中扮演深具影响力的角色；
- Boost 提供许多 TR1 组件实现品，以及其他许多程序库。

Boost 是一个 C++ 开发者集结的社群，也是一个可自由下载的 C++ 程序库群。它的网址是 Boost C++ Libraries。现在你应该把它设为你的桌面书签之一。

对比其他 C++ 组织和网站，Boost 有两件事是其他任何组织不能相比的。第一，它和 C++ 标准委员会之间有独一无二的密切关系，并且对委员会深具影响力。因为 Boost 由委员会成员创设。所以 Boost 成员和委员会成员有很大的重叠。Boost 的目标是作为一个“可做加入标准 C++ 各种功能”的测试场。以 TR1 提案进入标准 C++ 的 14 个新程序库中，超过三分之二奠基于 Boost 的工作成果。第二，它接纳程序库的过程要经过“讨论、琢磨、再次提交”的重重检查。
