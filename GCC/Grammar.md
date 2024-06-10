# GCC 基本语法

## 编译中的参数说明

1. `-g  （在编译的时候，产生调试信息）`
2. `-I  （指定头文件路径，建议相对路径）`
3. `-i  （指定头文件名字，一般不使用）`
4. `-L  （指定链接的动态库或静态库的路径，建议相对路径）`
5. `--Wl:rpath （运行时库路径）`
6. `-l  （指定连接的库的名字，例：libc.a -> -lc）`
7. `-w  （关闭警告信息）`
8. `-Wall   （打印警告信息）`
9. `-O3 （指定优化等级，0~3，3最高）`

## Makefiles文件常用宏

1. `$@: 表示目标文件`
2. `$^: 表示所有的依赖文件`
3. `$+: 也是所有依赖目标的集合,只是它不去除重复的依赖目标`
4. `$<: 表示第一个依赖文件`
5. `$?: 表示比目标还要新的依赖文件列表`

## 编译、执行C++脚本（单个脚本）

1. `g++ test.cpp -o test.exe`
2. `./test.exe* 或者 ./test.exe`

## 预处理、编译、汇编、链接、执行C++脚本（详细）

* `-E Preprocess only, do not compile, assemble or link`
* `-S Compile only, do not assemble or link`
* `-c Compile and assemble, but not link`
* `-o <file> Place the output into <file>`

1. `g++ -E test.cpp -o test.i`
2. `g++ -S test.i -o test.s`
3. `g++ -c test.s -o test.o`
4. `g++ test.o  （把二进制目标文件链接成一个整体）`
    1. `./a.out*    （执行.out文件）`
    2. `g++ test.o -o test.exe`

## g++ 简单工程编译

有以下三个文件：

* a.h
* a.cpp
* main.cpp

那么编译可以有以下两种方式：

1、分开编译：

1. g++ -c a.cpp
2. g++ -c mian.cpp
3. g++ a.o main.o -o test.out
4. 然后执行./test.out即可

2、一起编译：

1. g++ -o test.o a.cpp main.cpp
