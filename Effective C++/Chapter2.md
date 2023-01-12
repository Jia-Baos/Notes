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
