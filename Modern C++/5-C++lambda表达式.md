# C++lambda表达式

1. C++ lambda 类类型（无名的非联合的非聚合类 类型）

2. lambda的类的 operator() 函数在默认情况下是一个 const 函数，那么 this 指针就是被 const 所修饰的，可用 mutable 去除 const 修饰

    ```C++
    #include <iostream>

    int main(int argc, char* argv[])
    {

        int x = 1;
        int y = 2;
        // 按值捕获
        auto p = [=]
        { 
            std::cout << x << " " << y << std::endl;
            std::cout << "lambda..." << std::endl; };

        p();

        return 0;
    }
    ```

3. labmda 常见捕获方式

    1. &（以引用隐式捕获被使用的自动变量）；
    2. =（以复制隐式捕获被使用的自动变量）。