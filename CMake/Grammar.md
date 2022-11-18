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

1. 编写`main.cpp, Message.h, Message.cpp`，三者同级
2. 编写 `CMakeLists.txt`
   1. 只生成静态链接库
   
        ```Cmake
        cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
        project(main LANGUAGES CXX)
        add_library(message
            STATIC
            Message.h
            Message.cpp
            )
        add_executable(main main.cpp)
        target_link_libraries(main message)
        ```

   2. 同时生成静态链接库和动态链接库

        ```Cmake
        cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
        project(main LANGUAGES CXX)
        add_library(message-objs
            OBJECT
            Message.h
            Message.cpp
            )

        set_target_properties(message-objs
            PROPERTIES
            POSITION_INDEPENDENT_CODE 1)

        add_library(message-shared
            SHARED
            $<TARGET_OBJECT:message-objs>
        )

        add_library(message-static
            STATIC
            $<TARGET_OBJECT:message-objs>
        )

        add_executable(main main.cpp)
        target_link_libraries(main message-static)
        ```

    3. 同时生成静态链接库和动态链接库且同名

        ```Cmake
        cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
        project(main LANGUAGES CXX)
        add_library(message-objs
            OBJECT
            Message.h
            Message.cpp
            )

        set_target_properties(message-objs
            PROPERTIES
            POSITION_INDEPENDENT_CODE 1)

        add_library(message-shared
            SHARED
            $<TARGET_OBJECT:message-objs>
        )

        add_library(message-static
            STATIC
            $<TARGET_OBJECT:message-objs>
        )

        add_executable(main main.cpp)
        target_link_libraries(main message-static)
        ```

### 使用 `install` 命令

构建好工程后，不新建 `build` 文件夹，直接执行下面的命令

1. `cmake -B build  （告诉 CMake 在一个名为 build 的目录中生成所有的文件）`
2. `cmake --build build --config Release --target install   （分别指定build的目录、Release模式、目标文件）`