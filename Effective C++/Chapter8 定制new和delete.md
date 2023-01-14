# 定制new和delete

## Item49：了解 new-handler的行为

### 请记住

- set_new_handler 允许客户指定一个函数，在内存分配无法获得满足时被调用；
- Nothrow new 是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常。

1. 什么是new-handler

    当 new 无法分配足够的内存给对象的时候，它会抛出异常，在一些旧式的编译器上它会返回 null。

    当 operator new 抛出异常以反映一个未获得满足的内存需求之前，它会先调用客户指定的错误处理函数，一个所谓的 new-handle。

    ```C++
    namespace std{
        typedef void (*new_handle)();
        new_handler set_new_handler(new_handler p) throw();
    }
    ```

    通过 typedef，定义了一个指针指向一个没有参数也不返回任何东西的函数，void方法称为 new_handle，下面 set_new_handler 的用途从名字上看也已经很明显了，是一份异常明细， throw() 表明该函数不会抛出任何异常。

    用法如下：

    ```C++
    void OutOfMem()//用来被调用的函数
    {
        std::cerr<<"Unable to satisfy request for memory\n";
        std::abort();
    }
    int main()
    {
        std::set_new_handler(outOfMem);
        int* pBigDataArray = new int[10000000000L];
    }
    ```

    那么假设系统无法给 pBigDataArray 分配那么巨大的内存，就会在输出一条语句后直接夭折。

2. new-handler 应该具备的特质

    但一个合格的 new-handler 不应该只输出一条语句，还应该有如下功能：

    1. 分配更多内存

        这会让 operator new 的下一次分配内存的操作更有可能会成功。实现的其中一个方法是，程序一开始就分配一大片的内存，当 new-hanlder 第一次调用，把这些内存释放还给程序使用。

    2. 安装另一个 new-handler

        如果当前 new-handler 无法取得更多的内存，或许它知道另外哪个方法有这样的能力。那它应该通过 set_new_handler 替换掉自身，这样下一次 new-handler 执行的时候，将会调用最新安装的new-handler。

    3. 卸除new-handler

        也就是把 null 指针传给 set_new_handler。一旦没有安装任何 new-handler，operator new 会在内存分配不成功时抛出异常。

    4. 抛出 bad-alloc（或者派生自 bad_alloc）的异常

        这样的异常不会被operator new 捕捉，因此会被传播到内存索求处。

    5. 不返回

        通常调用 abort 或者 exit。

    这些选择会让你在实现new-handler函数时拥有很大的弹性。

3. 为 class 准备专属的 new-handler

    C++ 并没有提供对 class 专属的 new-handler 的支持，不过没有关系，我们可以自己实现出这种行为。

    首先，我们需要准备一个资源管理对象，它能够通过析构函数自动释放掉自己的内存.

    ```C++
    class NewHandlerHolder {
    public:
        explicit NewHandlerHolder(std::new_handler nh)
            :handler(nh){}  // 成员初始化比构造函数更早
        ~NewHandlerHolder()
        {std::set_new_handler(handler);}    // 在析构函数中把 global-new-handler 还原
    private:
        std::new_handler handler;
        // 禁止 copying（见条款14)
        NewHandlerHolder(const NewHandlerHolder&);     
        NewHandlerHolder& operator=(const NewHandlerHolder&);
    }
    ```

    然后在对应的类中定义一个 static 的 new_handler：

    ```C++
    class Widget{
    public:
        static std::new_handler set_new_handler(std::new_handler p) throw();
        static void* operator new(std::size_t size) throw(std::bad_alloc);
    private:
        static std::new_handler currentHandler;
    }
    ```

    对于class内部的set_new_handler的实现，把它参数上的指针储存起来，并返回调用之前储存的指针，标准版set_new_handler也是这么做的。

    ```C++
    std::new_handler Widget::set_new_handler(std::new_handler p) throw()
    {
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
    }
    ```

    然后实现对应 class 的 operator new：

    ```C++
    void* Widget::operator new(std::size_t size) throw(std::bad_alloc)
    {
        NewHandlerHolder h(std::set_new_handler(currentHandler));
        return ::operator new(size);
        /*如果这里内存不足，由于 global-new-handler 被设置为currentHAandler，会执行 currentHAandler
        方法结束后调用 NewHandlerHolder 析构函数把 gloabal-new-handler 还原为之前保存的方法*/
    }
    ```

4. 关于 no-throw

    之前提到，当 new 时发现内存不足，有的编译器会抛出异常，但早期的编译器会返回 null。C++ 在规范确定抛出异常之前，已经有很多 C++ 程序被写出来了，C++ 标准委员会不想抛弃这些“侦测是否为 null”的群体，因此提供了另一形式的 operator new 来供应传统的“返回 null”的行为。该形式被称为 nothrow 形式，因为它们使用了 nothrow 对象（定义于头文件\<new\>）。

    ```C++
    class Widget{...};
    Widget* pw1 = new Widget;// 如果分配失败，抛出bad_alloc
    if(pw1 == 0)...//该测试一定失败
    Widget* pw2 = new (std::nothrow) Widget;// 如果分配失败，抛出bad_alloc
    if(pw2 == 0)...//该测试可能会成功
    ```

    但是 nothrow 对异常的强制保证性并不高，不建议使用。

## Item50：了解 new 和 delete 的合理替换时机

### 请记住

1. 替换的理由

    为什么要替换掉编译器自己提供的new和delete呢？有如下三种理由：

    1. 检测运用上的错误

        各式各样的编程错误可能导致数据的“overrun”（写入点在分配区块尾端之后），或者“underruns"（写入点在分配区块起点之前）。如果自定义一个 operator news，便可以超额分配内存，以额外的空间放置特定的 byte patterns（即签名，signatures）。operator delete 便可以检查上述签名是否原封不动，如果为否则表示发生了 overrun 或者 underrun，这个时候 operator delete 可以志记（log）那个事实以及那个惹是生非的指针。

    2. 强化效能

        编译器所带的 operator new 和 operator delete 用于一般目的，它们不但可以被长时间执行的程序（比如web servers）接受，也可以被执行时间少于一秒的程序接受。它们必须处理一系列需求，包括大块内存，小块内存，大小混合型内存。它们必须接纳各种分配形态，范围从程序存货期间的少量区块动态分配，到大量短命对象的持续分配和归还。它们必须考虑碎片问题（fragmentation），这会导致程序无法满足大区块内存要求，即使彼时有总量足够到那时分散为许多小区块的自由内存。

        这导致编译器自带的 operator new 和 operator delete 的表现在各种情况下都适度的好，但不对特定任何场景有最佳表现。定制版的 operator new 和 operator delete 性能一般胜过缺省版本，特定情况可能会快很多，并且需要的内存也会少最多50%。

    3. 收集使用上的统计数据

        在定制 news 和 deletes 之前，需要先收集统计数据。程序如何使用其动态内存？分配区块的大小分布如何？寿命分布如何？它们倾向于先进先出还是后进先出或者随机次序来分配和归还？它们的运用形态是否随着程序运行的时间而改变？任何时候所使用的最大动态分配量（高水位）是多少？自行定义operator new和operator delete使我们可以轻松收集到这些信息。

2. 如何替换

    下面是一个促进并协助检测 overrun 或者 underrun 的被替换的 operator new。其中还有不少小问题，我们稍后完善它：

    ```C++
    static const int signature =  0xDEADBEEF；
    typedef unsigned char Byte;
    // 下面的代码还有若干小错误
    void* operator new(std::size_t size) throw(std::bad_alloc)
    {
        using namespace std;
        size_t realSize = size + 2 * sizeof(int); // 添加 2 个 int 的大小以超额分配内存
        
        void* pMem = malloc(realSize);
        if(!pMem) throw bad_alloc();
        
        // 把 signature 写入内存的最前段落和最后段落
        *(static_cast<int*>(pMem)) = signature;
        *(reinterpret_cast<int *> (static_cast<Byte*>(pMem) + realSize - sizeof(int))) = signature;
        
        return static_cast<Byte*>(pMem) + sizeof(int);
    }
    ```

    这个 operator new 最大的问题就是没有遵守接下来的条款 51 所说的在 new 中循环地调用 new_handler，即便抛开这个问题，仍然有一个关于齐位（alignment）的问题没有解决。

    许多计算机体系结构要求特定的类型必须放在特定的内存地址上面。比如它可能要求指针的地址必须是 4 倍数或者 doubles 的地址必须是 8 倍数。如果不奉行这样的约束条件，有可能导致运行期硬件异常。部分体系结构没有那么强硬，但是如果齐位条件得到满足，它们的效率会更高，比如Intel x86.

    C++ 要求所有的 operator new 返回的指针都应该适当对齐（取决于数据类型）。malloc 就是在这种要求下工作，所以 operator new 如果单纯返回一个来自 malloc 的指针时安全的。但是上述代码中返回的地址在 malloc 提供的地址基础上还偏移了一个 int 的大小，假设程序正处在一个“ints 地址必须是 4 倍数，而 double 地址必须是 8 倍数）的机器上并试图 new 一个 double 元素，就可能导致不符合预期的情况。

    像齐位这样的技术细节，正可以区分出有专业质量的内存管理器。所以很多时候写一个自定义 new 和 delete 是非必要的。某些编译器已经在它们的内存管理函数中切换到调试状态和志记状态。快速浏览一下你的编译器文档，你可能会就此消除自行撰写 new 和 delete 的需要。许多平台上已经有商业产品可以替代编译器自带的内存管理器。如果需要它们来为你的程序提高技能和改善效率，只需要进行 relink，当然前提是买下它们，或者到开源社区寻找合适的内存管理器。

## Item51：编写 new 和 delete 时需固守常规

### 请记住

- operator new 应该内涵一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用 new-handler。它也应该有能力处理 0 bytes 申请。Class 专属版本则还应该处理“比正确大小更大的（错误）申请”；
- operator delete 应该在收到 null 指针时不做任何事。Class 专属版本则还应该处理“比正确大小更大的（错误）申请”。

operator new 里面应该包含一个无穷的循环，不断尝试调用 new-handler 直到分配内存成功、申请了更多内存、handler 被替换或者被卸除、抛出 bad-alloc 异常或派生物当中其中任意一件事情发生。

C++ 规定 operator new 即便是申请 0byte 内存的情况也必须返回一个合法指针，常用的伎俩是在申请 0byte 的时候改成申请 1byte。::operator new也就是标准 operator new 有责任以某种合理方式处理 0byte 的情况。

当自定义 operator new 之后，它会被其他的 derived class 继承，比如：

```C++
class Base{
    public:
    stataic void* operator new(std::size_t size) throw(std::bad_alloc);
    ...
};
class Derived:public Base
{...};  // 假设Derived 没有声明 operator new
Derived* p = new Derived;   // 调用的是 Base::operator new
```

针对 Base 这个 class 设计的 operator new，很可能其行为只是适合大小为 size(X) 的对象而设计，而不适合其他任何的 Derived class。在这种情况下，常见的做法就是在 operator new 中检查申请内存量是不是符合预期，如果不是调用标准 operator new 来解决。

```C++
void* Base::operator new(std::size_t size) throw(std::bad_alloc)
{
    if(size != sizeof(Base))
        return ::operator new(size);    // 调用标准operator new
    ...
}
```

对于 operator new []，你唯一能做的就是分配给它一块未加工的事情。因为你甚至不知道这个数组有多少元素，也不知道每个元素的大小，因为可能申请的是 base class 的 operator new[] 有可能被继承调用，实际上装的是 derived class，而 derived class 一般来说比 base class 更大，假设每个元素大小是 sizeof(Base) 是不正确的，另外传递给 operator new[] 的 size_t 参数也未必是要分配的元素内存之和，还可能包括一个存储元素数量信息的空间。

对于 operator delete 相对比较简单，只需要知道 C++ 要求删除 null 指针永远是安全的，伪代码比如：

```C++
void operator delete(void * rawMemory) throw()
{
    if(rawMemory=0) return;//如果delete一个null指针，什么都不做
    ...
}
```

这个函数的 member 版本也比较简单，和 operator new 原理类似，也需要检查请求释放的内存量是否符合预期，否则调用 ::operator delete 也就是标准 operator delete：

```C++
void Base::operator delete(void* rawMemory,std::size_t size) throw()
{
    if(rawMemory == 0) return;  // 检查null指针
    if(size != sizeof(Base)){
        ::operator delete(rawMemory);
        return;
    }
    ...
    return;
}
```

## Item52：写了 placement new 也要写 placement delete

### 请记住

- 当你写一个 placement operator new，请确定也写出了对应的 placement operator delete。如果没有这样做，你的程序可能会发生隐微儿时断时续的内存泄露；
- 当你声明 placement new 和 placement delete，请确定不要无疑是（非故意）地遮掩了它们的正常版本。

什么是 placement new 和 placement delete。

当你执行如下语句的时候：

```C++
Widget* pw = new Widget;
```

有两个语句会被执行，首先是 operator new，其次是 Widget 的 default 构造函数

假设 operator new 执行成功，default 构造函数却抛出了异常，那么 operator new 分配的内存必须得到释放，而这个释放的任务不可能交给用户，因为构造函数抛出异常后，pw 尚未被赋值，用户无法操控分配的内存，所以释放责任要落实到 C++ 运行期系统上。

运行期系统会尝试调用 operator new 对应的 operator delete 版本，但是前提是它需要知道对应的 operator delete 版本是哪一个？对于正常签名式的 operator new 和 operator delete，很容易找到对应的版本：

```C++
void* operator new(std::size_t) throw(std::bad_alloc);  // 正常operator new
void operator delete(void* rawMemory) throw();  // global 作用域中的正常签名式
void operator delete(void* rawMemory, std::size_t size) throw(); // class 作用域中典型的签名式
```

然而，当你开始声明非正常的 operator new，也就是带有其他参数的 operator new，编译器就会迷惑应该使用哪个 operator delete 才能够正确释放内存。

举个例子，假设一个类写了非正常的 operator new，同时又写了一个正常的 operator delete:

```C++
class Widget {
public:
...
    // 非正常的new
    static void* operator new(std::size_t size,std::ostream& logStream) throw(std::bad_alloc);
    // class作用域中正常的delete
    static void operator delete(void* pMemory ,std::size_t size);
}
```

可以看到 Widget 中的 operator new 额外接受了一个 std::ostream 来处理志记信息，这种除了应有的 std::size_t 之外还有其他参数的 operator new 就是所谓的 placement new。

最出名的 placement new 已经被纳入 C++ 标准程序库中，只要 #include \<new\> 即可使用它，声明如下：

```C++
void* operator new(std::size_t, void* pMemory) throw();
```

那个额外参数“指向对象该被构造之处”，这个 placement new 的用途之一是负责在 vector 的未使用空间上面创建对象，是最早的 placement new 版本。这也是为什么这种 operator new 被称作 placement new，因为它是“一个特定位置上的 new”。狭义上的 placement new 特别指的就是这个带有额外一个 void* 指针的参数，但是广义上的 placement new 指的是接受额外任意实参的 operator new。

回到上面这个例子，编译器会寻找参数和 new 额外参数相一致的 operator delete 如果没有找到，则不会执行任何 operator delete，换句话说，编译器寻找的 delete 应该是这样的：

```C++
void operator delete(void* pMemory, std::ostream& logStream) throw()
```

这种有额外参数的 operator delete 也就是 placement delete。placement delete 只会在“伴随着对应的 placement new 被触发的构造函数”异常时才会被调用，换句话说如果直接执行 delete pw，调用的仍然是正常形式的 operator delete。

条款33提到过，成员函数的名称会掩盖其外围作用域中的相同名称，你必须小心在非故意的情况下让 class 专属的 news 掩盖住可能期望的其他 news。
