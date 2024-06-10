# C++lambda表达式

1. C++ lambda 类类型（无名的非联合的非聚合类 类型）

2. lambda的类的 operator() 函数在默认情况下是一个 const 函数，那么 this 指针就是被 const 所修饰的，可用 mutable 去除 const 修饰。

    ```C++
   #include <iostream>

    int main(int argc, char* argv[])
    {
        auto ptr1 = []() {std::cout << 1 << std::endl; };
        ptr1();	// 1
        std::cout << "ptr1 size: " << sizeof(ptr1) << std::endl;	// 1

        // operator()被const修饰，val_不可被修改
        auto ptr2 = [val_ = 2]() mutable {std::cout << val_ << std::endl; };
        ptr2();	// 2
        std::cout << "ptr2 size: " << sizeof(ptr2) << std::endl;	// 4

        auto ptr3 = [val_ = 2]() mutable {val_ = 3; std::cout << val_ << std::endl; };
        ptr3();	// 3
        std::cout << "ptr3 size: " << sizeof(ptr3) << std::endl;	// 4

        // 利用函数指针承接lambda，生成了一个转换函数
        void(*ptr4)() = []() {std::cout << "lambda..." << std::endl; };
        (*ptr4)();
        std::cout << "ptr4 size: " << sizeof(ptr4) << std::endl;	// 8

        // 模板lambda
        auto ptr5 = []<typename T>(T val_) { std::cout << val_ << std::endl; };
        ptr5(1);	// 1
        ptr5(2.0f);	// 2
        ptr5(3.0);	// 3

        return 0;
    }
    ```

3. labmda 常见捕获方式

    1. &（以引用隐式捕获被使用的自动变量）；
    2. =（以复制隐式捕获被使用的自动变量）。
