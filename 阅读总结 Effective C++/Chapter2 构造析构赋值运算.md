# 构造、析构、赋值运算

## Item5：了解 C++ 默默编写并调用了哪些函数

### 请记住

- 编译器可以暗自为 class 创建 default 构造函数、copy 构造函数、copy assignment 操作符以及析构函数。

在没有自己声明的情况下，编译器会自动为类声明一个 default 构造函数，一个 copy 构造函数，一个 copy assignment 操作符，一个析构函数。这些函数都是 public 而且 inline 的。如果声明了任意一个构造函数，那么编译器就不会提供无实参的构造函数。copy 构造函数和 copy assignment 操作符只是单纯把参数对象所有的 non-static 成员复制给当前构造的对象。

对于内置类型，编译器会复制该成员的每一个 bits 过来，而对于非内置类型则会尝试调用它的 copy 构造函数。

但是并不是所有的情况下都会生成 copy assignment 操作符，只有编译器能够理解应该如何进行复制的情况下，operator = 才会被生成，包括但不限于：

1. 该类持有 reference 成员，因为 C++ 不允许 reference 引用的目标被修改，所以编译器直接修改引用的地址是不合法的，如果编译器改成修改 reference 引用对象的值，那么其他引用该对象的 reference 或者 pointer 将在不知情的情况下被修改了指向的内容。所以编译器陷入了两难处境，干脆拒绝不默认生成 copy assignment 操作符。

2. 持有 const 成员，原因同上，修改 const 成员也是不合法的。

3. 当 base class 把自己的 copy assignment 操作符设为 private，那么编译器不会给它的 derived classes 生成 copy assignment 操作符，因为一般来说编译器认为子类应该可以处理父类的成员部分，但是他们无权调用 base class 的操作符，所以只能放弃生成 copy assignment 操作符。

## Item6： 若不想使用编译器自动生成的函数，就该明确拒绝

### 请记住

- 为了驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为 private 并且不予实现。使用像 Uncopyable 这样的 base class 也是一种做法。

这部分内容在 C++11 中已经提供了 delete 关键字帮助完成，作者说的把欲隐藏函数设为私有，并且把该类叫做 Uncopyable 给其他类继承的方法似乎已经不再需要了。

## Item7： 为多态基类声明 virtual 析构函数

### 请记住

- polymorphic（带多态性质的）base classes 应该声明一个 virtual 析构函数。如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数；
- Classes 的设计目的如果不是作为base classes 使用，或不是为了具备多态性（polymorphism），就不该声明 virtual 函数。

需要注意的是，编译器给 class 默认生成的析构函数是 non-virtual 的。

在我们声明一个 base class 后，他可以有多个 derived class，假设我们实现了一个 factory，这个 factory 提供方法提供一个 base class 的指针，实际指向的是 derived class 的对象。如果我们通过 base class 的指针来清空内存，最终得到的结果可能是只清除掉了 base class 部分的成员，但是 derived class 新增的那部分内存并没有得到释放。解决的方法也很简单，就是将析构函数声明为 virtual 即可。

如果一个 class 没有 virtual 函数，那么意味着它不打算被当做一个 base class 被继承。但是如果有需要的话，为了避免类的大小变大，最好不要给不打算当做 base class 的 class 加上 virtual。derived class 必须携带某些信息以决定那个 virtual 函数被调用，而这份信息由 vptr（virtual table pointer）指出，这个指针指向一个由函数指针指针组成的数组 vtbl（virtual table）。所以为了避免因为 vptr 造成的大小增加，不要盲目地把所有 class 的析构函数设置为 virtual。至少要有一个 virtual 函数，才应该给他的析构函数设置为 virtual。

另外，不要去继承没有 virtual 函数的类，比如说标准库中的 string 或者 STL 中的所有容器（vector、list、set等等）。

给 base class 一个 virtual 析构函数，并不是所有情况都要这样，而是只有 base class 是用于多态用途才需要这样，即需要使用 base class 接口来实际操作 derived class 时才需要这么做。

## Item8：别让异常逃离析构函数

### 请记住

- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该能够捕捉任何异常，然后吞下它们（不传播）或程序结束；
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数中）执行该操作。

C++不禁止在析构函数抛出异常，但是也并不鼓励这么做。

假设有如下代码:

```C++
class Widget:
{
    public:
    ...
        ~Widget(){...}
};
...
void doSomething()
{
    std::vector<Widget> v;
    ...
}
```

显然，在方法 doSomething 结束的时候，它创建的局部变量 v 也会被自动销毁，vector 对于销毁的实现是把内部包含的所有 Widget 都进行销毁，假设 v 中有 10 个 Widget，在析构第一个时发生异常，而后面的 9 个 Widget 仍然是要被销毁的，否则会导致内存泄漏。然而假设第二个又抛出了异常，假设第一个异常和第二个异常作用时间重叠，就会导致程序直接结束或者不明确行为。

在析构函数中捕获异常后，可以：

1. 直接结束程序（std::abort），这样可以避免后续的不明确行为，但是它直接结束了程序。

2. 捕获后记录异常数据，这样做可能导致不明确行为，但是它没有结束程序。宁愿吞下不明确行为的苦果也不能结束程序，这也是一个可选项。

为了保险起见，可以把那些可能导致异常的代码单独封装到一个函数去，供客户调用，尽管在析构函数中，依然要检查对应的成员是否已经被销毁，如果没有销毁，最后还是要进入上文提到的“二选一”的抉择中。但是这至少给了用户一个自己处理异常的机会，尽可能让异常在客户自己的函数中被处理而不是在析构函数中，在用户没有手动调用的情况下，析构函数也提供了一个“第二道防线”，如果在析构函数中导致了程序结束或者不明确行为，客户也没有立场抱怨，因为他们没有第一手处理掉可能发生的异常。

## Item9： 绝不要在构造和析构过程中调用 virtual 函数

### 请记住

- 在构造和析构期间不要调用 virtual 函数，因为这类调用从不下降至 derived class（比起当前执行构造函数和析构函数的那层）

尤其是从 JAVA 或者 C# 转来的程序员需要注意这件事。C++ 和这些语言有着不同之处。

由于在执行 derived class 的构造函数之前，首先执行的是 base class 的构造函数，如果在 base class 的构造函数调用 virtual 函数，客户实际上想要调用的可能是 derived class 的版本，但是最终执行的是 base class 的构造函数。这么做是因为编译器认为 derived class 重写的函数几乎必定会调用 local 变量，但是这些属于 derived class 的变量此时都还没有初始化，此时很可能导致未定义行为，所以最后编译器不会让客户调用 derived class 的版本，而是 base class 的版本。更本质的一个原因是，在 base class 构造期间，对象的类型就是 base class 而不是 derived class，在这期间使用运行时类型信息，也都会显示对象的类型是 base class。析构函数也是同样的道理。

如果要实现类似的功能，可以去掉该方法的 virtual，derived class 不是重写该方法，而是在成员初值列把自定义的信息传递给父类处理，构造自定义信息可以使用一个静态方法以防使用了未初始化变量。

## Item10： 令 operator= 返回一个 reference to *this

### 请记住

- 令赋值（assignment）操作符返回一个 reference to *this

对于赋值操作，应该允许连锁赋值。

```C++
int x, y, z;
x = y = z = 15;
// 赋值采用右结合律相当于
x = ( y = ( z = 15 ) );
```

为了实现连续赋值，赋值操作符必须返回一个 reference 指向左侧的实参。这是为classes 实现赋值操作符所应该实现的一项约定（尽管它不是强制的），这个约定不仅仅适用于标准赋值符，也适用于各种赋值的形式。比如：

```C++
class Widget{
public:
...
Widget& operator+=(const Widget& rhs)   // 当然 -=，*=也是一样的
{
...
return *this;
}
Widget& operator=(int rhs) // 即便它的参数类型不符合协定，还是返回reference to *this
{
...
return *this;
}
}
```

## Item11：在 operator= 中处理 “自我赋值”

### 请记住

- 确保当前对象自我赋值时 operator= 有良好行为。其中技术包括比较“来源对象”和“目标对象”的的hi之、精心周到的语句顺序、以及 copy-and-swap；
- 确定任何函数如果操作一个以上对象，而其中对各对象是同一对象时，其行为仍然正确。

C++ 是允许自我赋值的，即允许把自己赋值给自己。但在此过程中，必须确保左右两边的参数实际指向同一个对象时，该函数依然能够保持正确合理的运行，尤其是在需要 delete 指针释放内存的时候。保证自我赋值安全性的常用方法有：

1. “正同测试”（identity test）

    ```C++
    class Bitmap{...};

    class Widget
    {
    ...
    private:
        Bitmap* pb;
    }

    Widget& Widget::operator=(const Widget& rhs)
    {
        if(this == &rhs) return *this;
        else
        {
            delete this->pb;
            this->pb = new Bitmap(*rhs.pb);
            return *this;
        }
    }
    ```

2. 精心周到的语句
 
   ```C++
   Widget& Widget::operator=(const Widget& rhs)
    {
        Bitmap* pOrig = this->pb;
        this->pb = new Bitmap(*rhs.pb);
        delete this->pOrig;
        return *this;
    }
    ```

3. copy and swap

    ```C++
    class Widget
    {
        ...
        void swap(Widget& rhs); // 交换 *this 和 rhs 的数据
        ...
    }

    Widget& Widget::operator=(const Widget& rhs)
    {
        Widget temp(rhs);   // 为 rhs 数据制作一份复件（副本）
        swap(temp);         // 将 *this 数据和上述复件的数据交换
        return *this;
    }    
    ```

## Item12： 复制对象时勿忘其每一个成分

### 请记住

- Copying 函数应该确保复制“对象内所有成员变量”及“ base class 成分”；
- 不要尝试以某个 copying 函数实现另一个 copying 函数，应该将共同机能放进第三个函数中，并由两个 copying 函数共同调用。

Copying函数的时候，当你继承了一个 base class，记得在成员初值列也把 base class 的 Copying 函数补充进去。

不要用 Copying 函数调用 Copy assignment 函数，或者用 copy assignment 函数调用 Copying 函数，因为 Copying 函数是构造函数，Copy assignment函数是赋值函数，不能给一个已经构造的函数构造，或者给一个未构造的函数赋值。
