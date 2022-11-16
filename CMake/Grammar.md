# CMake 基本语法

## Chapter1

### 将单个源文件编译为可执行文件

1. 编写 `main.cpp`

    ```C++
    #include <cstdlib>
    #include <iostream>
    #include <string>
    std::string say_hello() { return std::string("Hello, CMake world!"); }
    int main() {
    std::cout << say_hello() << std::endl;
    return EXIT_SUCCESS;
    }
    ```

2. 编写 `CMakeLists.txt`

    ```Cmake
    cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
    project(recipe-01 LANGUAGES CXX)
    add_executable(hello-world hello-world.cpp)
    ```

4. `cmake -G "MinGW Makefiles" ..  （如此可以使用GCC编译器，否则会会自动调用VS中的MSVC编译器）`
5. `cmake --build . --config Release （可以生成release模式，默认是debug模式）`

### 切换生成器

CMake是一个构建系统生成器，可以使用单个CMakeLists.txt为不同平台上的不同工具集配置项目。您可以在CMakeLists.txt中描述构建系统必须运行的操作，以配置并编译代码。基于这些指令，CMake将为所选的构建系统(Unix Makefile、Ninja、Visual Studio等等)生成相应的指令。

用以下命令，可在平台上找到生成器名单，以及已安装的CMake版本：

```Cmake
cmake --help
```

具体实施

```Cmake
cmake -G MinGW Makefiles ..
```

### 构建和链接静态库和动态库

