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