---
title: STL_并发_线程管理简介
date: 2018-07-18 16:15:20
tags:
	- STL
	- Concurrency
categories:
	- C++
---
参考资料：

> c++ 标准库 (第二版)
> c++并发编程(第三版)
> cplusplus.com
---

# 1 概述
- `std::thread`用来启动和表现线程；

- thread对象和线程之间的关联式将一个callable objects（例如，函数，函数对象）

关联的状态：
使用`joinable()`判断：
- true ： 已经连接到一个线程
- false ： 未关联线程

在判断关联状态后，只可调用一次一下方法：
- join():等待已连接的线程返回结果
- detach(): 失去对线程的关联

# 2 操作函数
## 2.1 成员类型和成员函数
![](https://upload-images.jianshu.io/upload_images/5361608-90df511b3502839a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

部分函数的解释：

![](https://upload-images.jianshu.io/upload_images/5361608-b341ea469784d628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另一个静态函数

![](https://upload-images.jianshu.io/upload_images/5361608-cd911df5f841d07f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2.2 构造函数

声明了两个函数，第二个函数传递引用；
```
void increase_global (int n) ;
void increase_reference (std::atomic<int>& variable, int n) ;
```

(1)  默认构造函数
```
 std::vector<std::thread> threads;
```
（2) 传递给线程一个带参数的函数
- 参数作为thread的实参
```
std::thread(increase_global,1000);
```
(3) 传递给线程一个参数为引用的函数
- 使用标准库函数`std::ref()`传递引用

```
std::atomic<int> foo(0);
std::thread(increase_reference,std::ref(foo),1000);
```

上述例子中实现了thread的三种构造函数;

## 2.3 joinable(),join(),detach()
(1) `joinable()`
```
bool joinable() const noexcept;
```
- 判断线程是否已经连接
- 线程不可连接的情况：
  - 默认构造的线程
  - move过的线程
  - `join`或者`detach`后的线程；

(2) `join()`
```
void join();
```
- 函数在线程执行完成后返回；

```
// example for thread::join
#include <iostream>       // std::cout
#include <thread>         // std::thread, std::this_thread::sleep_for
#include <chrono>         // std::chrono::seconds
 
void pause_thread(int n) 
{
  std::this_thread::sleep_for (std::chrono::seconds(n));
  std::cout << "pause of " << n << " seconds ended\n";
}
 
int main() 
{
  std::cout << "Spawning 1 threads...\n";
  std::thread t1 (pause_thread,1);
  std::cout << "Done spawning threads. Now waiting for them to join:\n";
  t1.join();

  std::cout << "All threads joined!\n";

  return 0;
}
```
结果为：
```
Spawning 1 threads...
Done spawning threads. Now waiting for them to join:
pause of 1 seconds ended
All threads joined!
```
`join`让主线程必须等待其他线程返回后才能继续执行；

- 返回后的线程的`joinable`返回false,不可被连接，能够安全释放；

(3) `detach()`
```
void detach();
```
- 分离线程
- 在分离过后，线程在后台运行，thread对象不能引用，`joinable`变成不可被连接，能不安全释放；

## 2.4 转移线程的所有权
- 线程是可移动的(movable)，但是不可复制的(copyable)

![](https://upload-images.jianshu.io/upload_images/5361608-3a0087121f1a0dca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 1. 启动一个新线程关联到t1
- 2. 通过std::move()显式移动所有权，t1不在拥有相关线程；
- 3. 启动一个线程关联到临时的thread对象，然后将线程所有权移交给t1;
- 4. 默认构造一个t3;
- 5. 将t2的关联线程移交给t3;
- 6. 将t3关联的线程所有权移交给t1,但是t1已经有线程，所以会调用`std::terminate()`来终止程序；

因此，不能只通过向管理一个线程的thread对象赋值一个新的值来“舍弃”一个线程；




---

