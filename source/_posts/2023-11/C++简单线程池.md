---
title: C++简单线程池
date: 2023-11-17 16:40:58
hide: true
tags:
---





# 条件变量

参考文章：https://zhuanlan.zhihu.com/p/599172163



## wait函数使用



下面这段代码来自 https://en.cppreference.com/w/cpp/thread/condition_variable/wait



```C++
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <thread>

std::condition_variable cv;
std::mutex cv_m;    // This mutex is used for three purposes:
                    // 1) to synchronize accesses to i
                    // 2) to synchronize accesses to std::cerr
                    // 3) for the condition variable cv
int i = 0;

void waits(int num)
{
    std::unique_lock<std::mutex> lk(cv_m);
    std::cerr << "num:" << num << " Waiting...\n";
    cv.wait(lk, []{ return i == 1; });
    std::cerr << "num:" << num << " ...finished waiting. i == 1\n";
}

void signals()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lk(cv_m);
        std::cerr << "Notifying...\n";
    }
    cv.notify_all();

    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lk(cv_m);
        i = 1;
        std::cerr << "Notifying again...\n";
    }
    cv.notify_all();
}

int main()
{
    std::thread t1(waits, 1), t2(waits, 2), t3(waits, 3), t4(signals);
    t1.join();
    t2.join();
    t3.join();
    t4.join();
}
```

一组可能的输出为：

```bash
num:3 Waiting...
num:1 Waiting...
num:2 Waiting...
Notifying...
Notifying again...
num:3 ...finished waiting. i == 1
num:2 ...finished waiting. i == 1
num:1 ...finished waiting. i == 1
```

