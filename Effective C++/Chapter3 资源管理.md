# 资源管理

## Item13：以对象管理资源

### 请记住

- 为防止资源泄露，请使用RAII对象，它们在构造函数中获得资源并在析构中释放资源；
- 两个常被使用的 RAII classes 分别是 tr1::shared_ptr 和 auto_ptr。前者通常是较佳选择，因为其 copy 行为比较直观。若选择 auto_ptr，复制动作会使它（被复制物）指向 null。

假设编写了一个类：

```C++
class Investment
{...}
```

同时有一个工厂函数可以返回特定的 Investment 对象。

```C++
Investment* createInvestment();
```

为了防止内存泄漏，在C++中一般来说调用者有责任删除掉动态分配的对象。

```C++
void f
{
  Investment* pInv=createInvestment();
  ...
  delete pInv;
}
```

这一切看起来非常妥当。然而，不是所有情况下都可以很简易地释放内存的，比如说，f 函数有可能会执行不到最后一条语句，比如一个过早的 return 语句。类似的情况也发生在循环内的提前 continue 或者 break 中。总之，如果函数的提前结束导致 delete 语句没有被执行，那么内存将会泄漏。谨慎的编写程序可能会改善这一问题，但是，当代码很长的时候，保证所有动态获取的资源在每一个分支内都被释放是非常困难的，而且还要考虑抛出异常的情况。

为了避免这些情况，我们需要一个管理对象帮助我们管理资源，通过对象的析构函数自动帮助我们释放内存，典型的例子有标准程序库提供的 auto-ptr 等。

1. auto_ptr

    ```C++
    void f()
    {
    std::auto_ptr<Investment> pInv(createInvestment());
    ... // 一如既往地使用 pInv
        // 在 f 函数结束时，pInv 被销毁，经过 auto_ptr 的析构函数自动地删除 pInv
    }
    ```

    为了预防多个 auto_ptr 指向同一个对象，导致被删除多次的情况， auto_ptr 有一个与众不同的性质：如果通过 copy 构造函数或者 copy assignment 操作符去复制它，它将会变成 null，而复制获得的指针将会获得该资源的唯一使用权。但是这一特质也导致了 auto_ptr 也许不是管理资源的最佳方案，比如说 STL 容器都要求自己的元素可以发挥“正常的”复制行为，所以这些容器不能放入智能指针。

2. shared_ptr

    std::tr1::shared_ptr 是“引用计数型智慧指针”（reference-counting smart pointer： RCSP），持续追踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源，但是 shared_ptr 无法打破环状引用。

    ```C++
    void f()
    {
    std::tr1::shared_ptr<Investment> pInv(createInvestment());
    ... // 一如既往地使用 pInv
        // 在 f 函数结束时，pInv 被销毁，经过 shared_ptr的析构函数自动地删除 pInv
    }
    ```

    shared_ptr 的复制行为则像我们一般的 copy 或者 copy assignment 的预期结果，所以 STL 容器可以接纳 shared_ptr。

注意，shared_ptr 或者 auto_ptr 进行的都是 delete 而非 delete[] 操作，所以不要用于数组：

```C++
std::auto_ptr<str::string> aps(new std::string[10]);    // 馊主意，这是错误的 delete 形式
std::tr1::shared_ptr<int>spi(new int[1024]);    // 相同问题
```

管理对象的关键点在于：

1. 获得资源后立即放入管理对象（ Resource Acquision Is Initialization，即 RAII);

2. 管理对象通过析构函数释放资源。

## Item14：在资源管理类中小心 copying 行为

### 请记住

- 复制 RAII 对象必须一并复制它所管理的资源，所以资源的 copying 行为决定 RAII 对象的 copying 行为；
- 普遍而常见的 RAII class copying 行为是：抑制 copying、实行引用计数法（reference counting）。不过其他行为也都可能被实现。

假设我们使用 C_API 管理互斥锁，有 lock 和 unlock 两个函数可用：

```C++
void lock(Mutex* pm);
void unlock(mutes* pm);
```

为了保证绝不会忘记解锁一个锁住的对象，我们可能需要一个管理类来管理锁：

```C++
class Lock
{
public：
        explicit Lock(Mutex* pm):mutexPtr(pm)
    {
        lock(mutexPtr);
    }
    ~Lock()
    {
    unlock(mutePtr);
    }
private:
    Mutex *mutexPtr;
}
```

但是，如何应对对于Lock的复制呢？有如下选项：

1. 禁止复制

    有时候，禁止对管理类进行复制是合理的。

2. 引用计数法

    当我们希望保有资源直到最后一个使用者被销毁时，可以使用引用计数法，shared_ptr就是其中之一，不过shared_ptr默认会在无引用时释放资源内存，在本例中我们却只是希望它能够解锁该对象就好。所以需要shared_ptr的构造函数中的第二个参数deleter

    ```C++
    class Lock{
    public:
        explicit Lock(Mutex* pm)
        : mutexPtr(pm,unlock);  // 以某个 Mutex 初始化 shared_ptr，并以 unlock 函数为删除器
        {
        lock(mutexPtr.get());
        }
    private:
        std::tr1::shared_ptr<Mutex> mutexPtr;   // 使用 shared_ptr 替换 raw pointer
    }
    ```

    本例中不需要手动写 Lock 的析构函数了，因为无论编译器默认编写的析构函数还是自定义的析构函数都会自动调用所有 non_static 成员的析构函数。

3. 深度拷贝

    不仅仅是指针，连指针指向的资源都在内存中复制一份。

4. 转移底部资源的所有权

    如果某些场合你希望永远只有一个 RAII 对象指向底部资源，你可以在复制时转移底部资源的所有权，就像 auto_ptr 所做的一样。

## Item15：在资源管理类中提供对原始资源的访问

### 请记住

- APIs 往往要求提供底部资源，所以一个 RAII 类应该提供一个“取得所管理的底部资源”的方法；
- 对原始资源的访问有可能经过隐式转换或者显式转换，隐式转换更加方便，但显式转换更加安全。

由条款13可知，使用智能指针如 auto_ptr 或 tr1::shared_ptr 保存 factory 函数如 createInvestment 的调用结果：

```C++
std::tr1::shared_ptr<Investment> pInv(createInvestment());
```

假设你希望以某个函数处理 Investment 对象，像这样：

```C++
int daysHeld(const Investment* pi); // 返回投资天数
int daysHeld(pInv);                 // ！错误
```

却无法通过编译，因为 daysHeld 需要的是 Investment* 指针，你传给它的却是个类型为 std::tr1::shared_ptr\<Investment\> 的对象，因此需要将 RAII class 对象转换为其所内含之原始资源。有两个做法可以达成目标：显式转换和隐式转换。

1. 显式转换

    tr1::shared_ptr 和 auto_ptr 都提供一个 get() 函数，用来执行显式转换，也就是返回智能指针内部的原始指针（的复件）。

    ```C++
    int daysHeld(pInv.get()); // 返回投资天数
    ```

2. 隐式转换

    就像（几乎）所有只能指针一样，tr1::shared_ptr 和 auto_ptr 也重载了指针取值（pointer dereferencing）操作符（operator-> 和 operator*），它们允许隐式转换至底部原始指针：

    ```C++
    // investment 继承体系的根类
    class Investment
    {
    public:
        bool isTaxFree() const;
    ...
    };

    Investment* createInvestment(); // factory 函数
    std::tr1::shared_ptr<Investment> pi1(createInvestment());   // 令 tr1::shared_ptr 管理一笔资源
    bool taxable1 = !(pi1->isTaxFree());    // 经由 operator-> 访问资源
    ...
    std::auto_ptr<Investment> pi2(createInvestment());   // 令 auto_ptr 管理一笔资源
    bool taxable2 = !((*pi2).isTaxFree());  // 经由 operator* 访问资源
    ...
    ```

## Item16： 成对使用 new 和 delete 时要采用相同形式

### 请记住

- 如果你在 new 表达式中使用 []，必须在相应的 delete 表达式中也使用 []。如果你在 new 中不使用 []，一定不要在相应的 delete 表达式中使用 []。

尽量不要对数组使用 typedef，这可能导致编写代码时忘记使用正确的 delete。

## Item17：以独立语句将 newed 对象置入智能指针

### 请记住

- 以独立语句将 newed 对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露。

假设有如下函数：

```C++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
```

再假设使用如下方法调用：

```C++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```

在这个语句中，编译器产出 processWidget 的调用码之前，首先要确定实参，它会干三件事：

1. priority()
2. new Widget
3. 调用shared_ptr构造函数

其中 new Widget 一定发生在调用 shared_ptr 构造函数之前，因为前者是后者的参数，但是 priority() 的调用就不一定了，它可能会在两者前后甚至中间，万一 priority() 在中间且调用异常，那么之前 new 出来的 Widget 就在还没有放入 shared_ptr 之前就遗失了。

以防万一，最好把智能指针的定义单独写在一行：

```C++
std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw,priority())
```
