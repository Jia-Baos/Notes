# C++lambda剖析

1. 如果变量满足下列条件，那么 lambda 表达式在使用它前不需要先捕获：

    1. 该变量是非局部变量，或具有静态或线程局部存储期（此时无法捕获该变量），或者
    2. 该变量是以常量表达式初始化的引用。
    3. 该变量具有 const 而非 volatile 的整型或枚举类型，并已经用常量表达式初始化，或者
    4. 该变量是 constexpr 的且没有 mutable 成员。

    ```C++
    #include <iostream>

    void f1(int, const int(&)[2] = {}) {}   // #1
    void f1(const int&, const int(&)[1]) {} // #2
    void f2(int) {}

    void test()
    {
        const int x = 17;

        auto g0 = [](auto a) { f1(x); };  // OK：调用 #1，不捕获 x
        std::cout << sizeof(g0) << std::endl;

        auto g1 = [=](auto a) { f1(x); }; // C++14 中不捕获 x，C++17 中捕获 x，捕获能被优化掉
        std::cout << sizeof(g1) << std::endl;

        auto g2 = [=](auto a)
        {
            int selector[sizeof(a) == 1 ? 1 : 2] = {};
            f1(x, selector); // OK：这是待决表达式，因此 x 被捕获
        };
        std::cout << sizeof(g2) << std::endl;

        auto g3 = [=](auto a)
        {
            typeid(a + x);  // 仅在 tpyeid 下捕获 x
        };
        std::cout << sizeof(g3) << std::endl;
    }

    int main(int argc, char* argv[])
    {
        auto p1 = [] {return 6; };  // 非捕获的lambda
        std::cout << "p1 size: " << sizeof(p1) << std::endl;    // 1

        static int a = 1;
        const int b = 2;
        constexpr int c = 3;
        // 下面情况不捕获变量
        // 该变量是非局部变量，或具有静态或线程局部存储期（此时无法捕获该变量）
        // 该变量是以常量表达式初始化的引用
        // 该变量具有 const 而非 volatile 的整型或枚举类型，并已经用常量表达式初始化
        // 该变量是 constexpr 的且没有 mutable 成员
        auto p2 = [] {
            std::cout << a << " "
                << b << " "
                << c << std::endl; };
        p2();
        std::cout << "p2 size: " << sizeof(p2) << std::endl;    // 1

        float x;
        float* ptr = &x;
        auto p3 = [=] {};
        // lambda （隐式或显式）捕获的任何实体均被该 lambda 表达式 ODR 使用
        std::cout << "p3 size: " << sizeof(p3) << std::endl;    // 1

        const int N = 10;
        auto p4 = [=] {
            int arr[N]{};
        };  // 读取编译时常量的值也不是ODR使用
        std::cout << "p4 size: " << sizeof(p4) << std::endl;    // 1

        auto p5 = [=] {
            int p = N; };   // 这里 N 的确是被捕获了
        std::cout << "p5 size: " << sizeof(p5) << std::endl;    // 4

        auto p6 = [=](auto a) {
            f2(N);
        };  // 是否捕获由捕获参数决定 size
        std::cout << "p6 size: " << sizeof(p6) << std::endl;    // 4

        test();

        return 0;
    }
    ```

2. lambda继承

    ```C++
    #include <iostream>

    // 我们继承的lambda表达式，这表示其本质上是类
    template<typename... Ts>
    struct overloaded : Ts...
    {
        using Ts::operator()...;
    };

    int main() {
        auto c = overloaded{ [](int arg) { std::cout << arg << ' '; },
                            [](double arg) { std::cout << arg << ' '; },
                            [](auto arg) { std::cout << arg << ' '; }
        };

        c(1);
        c(2.0);
        c("hello");
    }
    ```

3. lambda捕获子句展开

    ```C++
    #include <iostream>

    template<class...Args>
    void f(Args...args) {
        auto lm = [&args...] {};    // 显式捕获
        auto lm2 = [&] {};
        std::cout << sizeof(lm) << " " << sizeof(lm2) << std::endl;
    }
    int main() {
        f(1, 1.0, 1.f);
    }
    ```

4. 完美转发

    ```C++
    
    ```

5. C++14 捕获语法

    ```C++
    #include <iostream>

    struct X {
        X() { puts("X"); }
        ~X() { puts("~X"); }
        X(X&&)noexcept { puts("X&&"); }
        X(const X&) { puts("const X&"); }
    };

    int main() {
        X x1;
        auto y = [&r = std::as_const(x1)] {};   // 按引用捕获没有任何构造，使用std::as_const捕获的是const X&

        X x2;
        auto y_ = [r = std::move(x2)] {};   // 移动捕获，std::move(x2)，有移动构造和析构
    }
    ```

