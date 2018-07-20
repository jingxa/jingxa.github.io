---
title: STL_并发_线程间共享数据
date: 2018-07-18 16:16:01
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

# 一 概述
- 数据共享--只读 ： 没有问题
- 数据共享-- 修改： 破坏不变量

## 1.1 问题
-  修改线程之间共享数据的潜在问题： 破坏不变量
- 有问题的竞争条件：导致不变量被破坏

## 1.2 避免
- 1. 保护机制封装数据结构
- 2. 修改数据结构的设计和不变量（无锁编程）
- 3. 软件事务内存

# 二、 互斥元 概述

- c++ 提供的保护共享数据的最基本机制：互斥元
- `<mutex>`头文件

c++ 提供的锁：

![](https://upload-images.jianshu.io/upload_images/5361608-2c4eb30b79c8cd89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Mutex type: 锁的类型
- locks: 管理锁的对象
- Functions : 
  - 同时锁定多个锁 `try_lock`,`lock`, 和执行一次`call_one`

## 2.1 mutex 和lock

- `std::mutex`：创建互斥元
- `lock()` :锁定
- `unlock()`：解锁
- `std::lock_guard类模板`实现互斥元

```
std::mutex mtx;           // mutex for critical section

void print_block (int n, char c) {
  // critical section (exclusive access to std::cout signaled by locking mtx):
  mtx.lock();
  for (int i=0; i<n; ++i) { std::cout << c; }
  std::cout << '\n';
  mtx.unlock();
}
```
- 直接调用成员函数是不推荐的，所以使用上面的类模板来管理；

![](https://upload-images.jianshu.io/upload_images/5361608-17e2b2f1b2f113b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 将 lock限制在代码块中，保证lock被限制在最短周期内；

## 2.2 递归锁(Recursive_mutex)

例如，一个对象的数据结构
```

class dataAccess{
private:
	std::mutex dbMutex;
	...
	
public:
	void insertdata(...){
		std::lock_guard<std::mutex> lg(dbMutex);
		...
	}
	
	void createdata(...){
		std::lock_guard<std::mutex> lg(dbMutex);
		...
	}
	
	
	void createandInsertdata(...){
		std::lock_guard<std::mutex> lg(dbMutex);
		...
		createdata(...);   // ERROR ，错误，死锁，dbMutex已经被锁定了
	}
	
};

```
- class中创建一个普通mutex,但是成员函数中调用其他成员函数的情况，就会导致死锁；

- 第二次lock抛出：`std::system_error`

- 因此，使用`recursive_mutex`将没问题


```
class dataAccess{
private:
	std::recursive_mutex dbMutex;
	...	
public:
	void insertdata(...){
		std::lock_guard<std::recursive_mutex> lg(dbMutex);
		...
	}
	void createdata(...){
		std::lock_guard<std::recursive_mutex> lg(dbMutex);
		...
	}	
	void createandInsertdata(...){
		std::lock_guard<std::recursive_mutex> lg(dbMutex);
		...
		createdata(...);   // 没问题
	}
	
};
```

## 2.3 尝试性的lock 和 带时间的lock
### 2.3.1 try_lock
- 有时候程序想要获得一个lock，如果不成功，但是不想永远阻塞；
- `try_lock()`：成功返回true,失败返回false

```
// try_lock
// try_lock 可能假性失败，即没被使用但也返回false
/*
	std::adopt_lock : 告诉 std::lock_guard 该互斥元被锁定，
	沿用互斥元上锁的所有权
*/
std::mutex m;
while(m.try_lock() == false ){
	dosomething();
}

std::lock_guard<std::mutex> lg(m, std::adopt_lock);
```

### 2.3.2 带时间的lock
分为两种：
- `timed_lock`
- `recursive_timed_lock`
对于带时间的lock,可以使用
- `try_lock_for()`
- `try_lock_until()`
来等待某个时间段；
- 但是这两个函数在处理系统时间调整的时候，有差异；

```
std::timed_lock m;
if(m.try_lock_for(std::chrono::secondes(1))){
	std::lock_guard<std::timed_lock> lg(m, std::adopt_lock);
	...
}else{
	couldnotgetthelock();  // 不能获取lock
}
```


## 2.4 处理多个lock
### 2.4.1 使用std::lock
```
// 处理多个lock
/*
	std::adopt_lock : 告诉 std::lock_guard 该互斥元被锁定，
	沿用互斥元上锁的所有权

*/
std::mutex m1;
std::mutex m2;

...
{
	std::lock(m1,m2);  // 同时锁定成功，或者失败
	std::lock_guard<std::mutex> lockM1(m1,std::adopt_lock);
	std::lock_guard<std::mutex> lockM2(m2,std::adopt_lock);
	...
} // 自动释放所有锁

```
- 使用std::lock锁定所有mutex

### 2.4.2 使用try_lock

- "尝试取得多个lock"且“如果部分lock可用也不造成阻塞”
-std::try_lock()会在取得所有的lock情况下返回-1;否则返回失败的lock的索引（从0开始），并且成功的lock被unlock;

```
//try_lock
std::mutex m1;
std::mutex m2;
...
int idx = std::try_lock(m1,m2); 
if(idx < 0){  // 所有锁成功
	std::lock_guard<std::mutex> lockM1(m1, std::adopt_lock);
	std::lock_guard<std::mutex> lockM2(m2, std::adopt_lock);
	...
} // 自动释放所有锁
else{
	// idx是失败的lock的索引
}
```

## 2.5 注意
- 在所有例子中，使用`lock_guard`在出了程序段后自动释放；

但是如果不使用上述函数类模板,mutex为被释放；

```
// mutex仍会锁定

std::mutex m1;
std::mutex m2;

...
{
	std::lock(m1,m2);
	// no lock adopted
}
... // mutex 仍在
```


## 2.6 unique_lock
- unique_lock 和 lock_guard 提供的接口相同
- 并且能够通过`owns_lock()`和`bool()`来查询是否被锁住
- 优点：
  - 如果析构时候mutex仍被锁住，析构函数自动调用unlock();

```
// 构造函数
// 1. 传递try_to_lock,表示企图锁定mutex但不希望阻塞
std::unique_lock<std::mutex> lock(mutex, std::try_to_lock);

// 2. 传递一个时间段，表示尝试一段时间内锁定
std::unique_lock<std::mutex> lock(mutex, std::chrono::secondes(1));
if(lock){ // 判断是否锁住
	...
}
// 3. 传递defer_lock, 表示初始化这一lock对象，但尚未打算锁住mutex;
std::unique_lock<std::mutex> lock(mutex, std::defer_lock);

```

延迟锁：

```

// 稍后锁住
std::mutex m1;
std::mutex m2;

std::unique_lock<std::mutex> lockM1(m1, std::defer_lock);
std::unique_lock<std::mutex> lockM2(m2, std::defer_lock);

...
std::loc(m1,m2);  // 延迟锁住
```


## 2.7 只调用一次
有时候 某些技能被某个线程使用后，其他线程再也不需要它；
- 典型例子： lazy initialization (延迟初始化)

```
// 延迟初始化
// 单线程
bool initialized  = false;
...
if(!initialized){
	init();  // 某初始化操作
	initialized = true;
}
```
但是在多线程中，会出现问题；
c++解决方案：
```
std::once_flag oc; // 全局标志
...
std::call_once(oc, init); // 如果没有初始化 就执行
```




---

