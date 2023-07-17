# C++ noexcept关键字

```C++
#include <iostream>

void func1()
{
	std::cout << "func1..." << std::endl;
}

// noexcept说明符，指定函数是否抛出异常
void func2() noexcept
{
	std::cout << "func2..." << std::endl;
}

int main(int argc, char* argv[])
{
	// noexcept运算符，进行编译时检查，如果表达式不会抛出任何异常则返回true
	std::cout << std::boolalpha << noexcept(func1()) << std::endl;    // 0
	std::cout << std::boolalpha << noexcept(func2()) << std::endl;    // 1
	return 0;
}
```
