# 多线程并发基础入门教程

## 创建线程

```C++
#include <iostream>
#include <thread>

void func()
{
        std::cout<<"Hello world~"<<std::endl;
}

int main()
{
    std::thread mythread(func);
    mythread.join();    // 阻塞主线程，等待子线程结束
    return 0;
}
```

## 互斥量

```C++
std::mutex mtx;

mtx.lock(); // 可同时对多个互斥量进行上锁
... // 此处抛出异常会使得解锁失败
mtx.unlock();

std::lock_guard<std::mutex> lock(mtx);  // 建议使用此方式进行上锁，其在构造函数中上锁、析构函数中解锁
std::unique_lock<std::mutex> lock(mtx); //相较 lock_guard 更加灵活，可记录锁的状态 
```

## 原子变量

```C++
std::atomic<int> globalVariable = 1;    // 不再需要上锁、解锁操作
```

## 条件变量
