# Basic C++基础工具类

## noncopuable.h

#pragma once 是 C/C++ 中的编译器预处理指令，核心作用是防止头文件被重复包含（重复引入），是传统 #ifndef 头文件保护宏的简化替代方案

禁止拷贝与赋值

## Thread.h

std::function的含义是擦除可调用对象的具体类型，仅保留参数类型和返回值类型，std::function<返回值类型(参数类型1, 参数类型2, ...)>，和直接定义函数的区别是直接定义函数无法用指针指向有捕获的lambda函数

lambda函数：通过 auto Func = [a](){}捕获函数外的局部变量，a可以是类的this指针等

using 1）引入命名空间 2）引入命名空间中的单个名称 3）定义类型别名（C++11 替代 typedef）

explicit 禁止隐式转换

explicit Thread(ThreadFunc, const std::string &name = std::string()); ThreadFunc为线程执行体

进程ID：PID ，getpid()

轻量级进程ID:LWP,gettid()，linux中线程是轻量级进程，PID 标识进程整体，TID 标识进程内的单个调度单元（线程）。

pthread中的ID,pthread_self(), 仅在当前进程内唯一，不同进程的 pthread_t 可能重复

pid_t：表示进程/线程的标准类型

static std::atomic<int> numCreated_; 原子变量，操作无法被中断，static变量，所有类共享，共同计数。

## Timestamp.h

构造函数：1）如果是public，外部可以显式构造
         2）如果是private，则是单例模式，需调用public中的instance方法创建static实例保证全局唯一

localtime:将微秒级别的数据转为年月日时分数据类型

time()：返回秒级时间戳

静态工厂方法：静态工厂方法是面向对象设计模式中「创建型模式」的一种简化实现（区别于完整的 “工厂方法模式”），核心是通过类的 static 成员函数创建并返回类的实例，而非直接通过构造函数 new/() 创建。（可以用这个方式限制全局单例）


## CurrentThread.h

extern 跨文件全局变量

extern __thread int t_cachedTid;声明一个 “线程局部存储（TLS）” 的全局变量，让每个线程拥有该变量的独立副本，且能跨编译单元访问，muduo 库中获取线程 TID 需要调用 syscall(SYS_gettid)（内核系统调用），频繁调用会有性能损耗。因此线程首次获取 TID 时，将其存入 __thread t_cachedTid；后续获取 TID 直接读这个变量，无需再调用系统调用，提升性能

**muduo中分支预测优化的经典写法:核心是通过 GCC 内置函数 __builtin_expect 告诉编译器 “t_cachedTid == 0 这个条件几乎不会成立”，让编译器优化指令布局，减少 CPU 分支跳转的性能损耗。**

__builtin_expect 是 GCC/Clang 编译器提供的内置优化函数（非标准 C++），核心作用是向编译器传递 “分支预测” 信息：告诉编译器 “某个条件表达式大概率为真 / 假”；编译器根据该信息调整指令的存储顺序，让 “大概率执行的分支” 紧跟在判断指令后，减少 CPU 指令流水线的中断（跳转）。

CPU 执行指令时会采用 “流水线” 机制 —— 提前加载后续指令并行处理，但若遇到条件跳转（if/else），流水线会中断，需等待条件判断结果后再加载对应分支的指令，导致性能损耗。

CPU 有 “分支预测器”，会根据历史执行情况猜测分支走向，但如果预测错误，流水线会 “回滚”，性能损失更大。而 __builtin_expect 是程序员显式给编译器的 “预测提示”，让编译器在编译阶段就优化指令布局

sem_t: 与互斥锁相像，信号量的值是一个非负整数（>=0），表示 “可用资源数”；值为 0 时，调用 sem_wait() 的线程会阻塞；post操作：子线程获取 TID 后，释放信号量（值从 0→1），唤醒主线程继续执行。（相当于一个标志位，如果为0代表线程TID值没有获取完毕，获取完毕后才可以开启主线程）

Lambda函数：[&]以引用的方式捕获当前作用域外所有的变量

# 缓冲区与数据读写

## Buffer.h

用长度前缀法解决TCP通信的粘包问题（prependable byte）





