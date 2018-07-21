---
title: STL_并发发_条件变量
date: 2018-07-21 19:21:38
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
# 1. 概述
- 条件变量
- 头文件`<condition_variable>`

使用：
- `<mutex>` 和 `<condtion_variable>`
- 必须同时包含mutex和一个条件变量

![](https://upload-images.jianshu.io/upload_images/5361608-98e976b39dbdc9a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 条件变量分为两种：
- 第一种仅限于和std::mutex一起工作（如果没有普遍性，首选第一种）
- 后者能够和类似互斥元的任何东西一起工作；


# 2、condition_variable

- 条件变量需要使用`unique_lock`来锁住一个线程，当条件变量的wait操作的时候；
- 在wait操作的时候，不能使用`lock_guard`，因为wait()调用中，条件变量会对锁不停地加锁和解锁操作，`lock_guard`不能满足这样的操作；

## 2.1 构造函数
```
//default (1)	
condition_variable();
//copy [deleted] (2)	
condition_variable (const condition_variable&) = delete;
```
- 条件变量不能被拷贝或者move构造；

## 2.2 成员函数

![](https://upload-images.jianshu.io/upload_images/5361608-2da74d06403c1937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例子：
```
// condition_variable example
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void print_id (int id) {
  std::unique_lock<std::mutex> lck(mtx);
  while (!ready) cv.wait(lck);
  // ...
  std::cout << "thread " << id << '\n';
}

void go() {
  std::unique_lock<std::mutex> lck(mtx); // 这里可以使用lock_guard
  ready = true;
  cv.notify_all();
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(print_id,i);

  std::cout << "10 threads ready to race...\n";
  go();                       // go!

  for (auto& th : threads) th.join();

  return 0;
}
```

结果为：

```

10 threads ready to race...
thread 2
thread 0
thread 9
thread 4
thread 6
thread 8
thread 7
thread 5
thread 3
thread 1
```

其中，在print_id函数是的while循环，等价于：
```
  while (!ready) cv.wait(lck);
```
等价于：

```
cv.wait(lck,[]->bool{return ready;}) 
```
当ready条件满足的时候，就返回;如果不满足，就将线程置于阻塞或等待要求；

## 2.3 操作函数详解

![](https://upload-images.jianshu.io/upload_images/5361608-58370de18c92d484.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-aa6192275617b65a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




---

