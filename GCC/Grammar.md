# GCC 基本语法

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
