# C++ noexcept关键字

```C++
#include <iostream>

void func1()
{
    std::cout << "func1..." << std::endl;
}

void func2() noexcept
{
    std::cout << "func2..." << std::endl;
}

int main(int argc, char *argv[])
{

    std::cout << noexcept(func1()) << std::endl;    // 0
    std::cout << noexcept(func2()) << std::endl;    // 1
    return 0;
}
```