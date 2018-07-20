---
title: STL_并发_高级接口
date: 2018-07-18 16:16:26
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

# 一、 概述
- `Future`头文件中
![](https://upload-images.jianshu.io/upload_images/5361608-57a769d1502dd91d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中class和Functions的providers
- promise
- packaged_task 
- async

都是providers,能够将共享状态（shard state）的接口分享给future对象；

例如：
- `std::async()`: 让一个callable objects在后台运行，成为一个独立线程；
- `class std::future<>` ： 允许等待线程结束并且获取结果（一个返回值，或者一个异常）




# 二、 Providers
## 2.1 asnyc()

### 2.1.1 构造函数
提供了两个版本：

```
// unspecified policy (1)	
template <class Fn, class... Args>
  future<typename result_of<Fn(Args...)>::type>
    async (Fn&& fn, Args&&... args);

// specific policy (2)	
template <class Fn, class... Args>
  future<typename result_of<Fn(Args...)>::type>
    async (launch policy, Fn&& fn, Args&&... args);
```
- 第一个版本执行callable对象 fn，无需等待fn完成就可以返回
- 返回值的访问通过future对象（future::get）

第二个版本：
- 调用callable对象的时候选择一种一种策略；

| 策略| 描述|
|- | - |
| launch::async| 尝试启动fn并给予参数args,形成一个异步任务 |
| launch::deferred| 调用fn并且传递参数args，形成一个推迟任务，直到返回的future调用wait()或者get()时，任务才同步调用，否则绝不启动|
| launch::async \| launch::deferred |第一版本效果一样，即随机选择一个launch策略 |

---
## 2.2 promise
- 1.promise对象能够关联到一个shared state上，其中shared state 能够存储一个返回值或者异常(std::exception);
- 2.future对象能够检索promise对象关联的shared state
- 3.future对象要关联到一个shared state上，需要promise调用成员函数`get_value`返回的值传递给future对象；

这个时候，shared state关联到future对象和promise对象上：
- promise： 提供数据或来设置shared state
- future : 取回shared state中的数据

### 2.2.1 模板参数
```
template <class T>  promise;
template <class R&> promise<R&>;     // specialization : T is a reference type (R&)
template <>         promise<void>;   // specialization : T is void
```
### 2.2.2 构造函数
```
// default (1)	
promise();
// with allocator (2)	
template <class Alloc> promise (allocator_arg_t aa, const Alloc& alloc);
// copy [deleted] (3)	
promise (const promise&) = delete;
// move (4)	
promise (promise&& x) noexcept;
```

例子：
```
// 默认
  std::promise<int> foo;
// 分配器构造  
std::promise<int> bar = std::promise<int>(std::allocator_arg,std::allocator<int>());

// 获取关联到shared state的future对象
  std::future<int> fut = bar.get_future();
```

### 2.2.3 成员函数
![](https://upload-images.jianshu.io/upload_images/5361608-fd3927866383ab95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

promise 对象通过
- `set_value`
- `set_exception`
来向shared state中存储一个数据或者一个异常；

```
// promise example
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

void print_int (std::future<int>& fut) {
  int x = fut.get();
  std::cout << "value: " << x << '\n';
}

int main ()
{
  std::promise<int> prom;                      // create promise

  std::future<int> fut = prom.get_future();    // engagement with future

  std::thread th1 (print_int, std::ref(fut));  // send future to new thread

  prom.set_value (10);                         // fulfill promise
                                               // (synchronizes with getting the future)
  th1.join();
  return 0;
}
```


- 一旦shared state存有某值或者异常， 状态变为ready,于是可以通过future对象读取内容；

以上两个函数执行后，并不意味着执行promise 的线程结束，该线程可能还在运行；如果想要在线程结束时才设置数据或者异常，可以执行以下函数：
- `set_value_at_thread_exit()`
- `set_exception_at_thread_exit()`


### 2.2.4 操作函数 详解
![](https://upload-images.jianshu.io/upload_images/5361608-782ebdb006364b59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



---
## 2.3 packaged_task
- packaged_task 包装callable对象 ，并且允许这个callable结果异步访问；
- 类似std::function , 但是结果自动被转换为一个future对象；

packaged_task 的组成：
- stored task: 一些callable对象（例如函数指针，成员函数指针或者函数对象）并且能够利用 传递的args参数；
- shared state : 存储callable对象的结果，能够被future对象异步访问；

shared state 能够通过packaged_task调用成员函数`get_future()`关联到一个future对象；

### 2.3.1 函数模板

```
template <class T> packaged_task;     // undefined
template <class Ret, class... Args> class packaged_task<Ret(Args...)>;
```
模板参数为函数返回值类型和参数类型；

### 2.3.2 构造函数

```
//default (1)	
packaged_task() noexcept;
//initialization (2)	
template <class Fn>
  explicit packaged_task (Fn&& fn);
//with allocator (3)	
template <class Fn, class Alloc>
  explicit packaged_task (allocator_arg_t aa, const Alloc& alloc, Fn&& fn);
//copy [deleted] (4)	
packaged_task (packaged_task&) = delete;
//move (5)	
packaged_task (packaged_task&& x) noexcept;
```
例如：

```
  std::packaged_task<int(int)> foo;                          // default-constructed
  std::packaged_task<int(int)> bar ([](int x){return x*2;}); // initialized
```
模板为：
函数的返回值类型，参数类型


### 2.3.3 成员函数
![](https://upload-images.jianshu.io/upload_images/5361608-36044c6aed919a32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例子：
```
// packaged_task::get_future
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// a simple task:
int triple (int x) { return x*3; }

int main ()
{
  std::packaged_task<int(int)> tsk (triple); // package task

  std::future<int> fut = tsk.get_future();   // get future

  std::thread(std::move(tsk),33).detach();   // spawn thread and call task

  // ...

  int value = fut.get();                     // wait for the task to complete and get result

  std::cout << "The triple of 33 is " << value << ".\n";

  return 0;
}
```

### 2.3.4 操作函数详解
![](https://upload-images.jianshu.io/upload_images/5361608-8eb6a975f60f365c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
# 三、 Future


## 3.1 Future
- future对象用来表示某一操作的输出(outcome)，可能是一个返回值，或者异常；
- 输出被保留在shared state中，可能是被`std::async()`,`std::packaged_task`, `promise`创建的；

- 结果只能被取出一次。future状态：
  - 有效(valid)： 结果未被取出
  - 无效(invalid):结果被取出

### 3.1.1 模板参数和构造函数

```
template <class T>  future;
template <class R&> future<R&>;     // specialization : T is a reference type (R&)
template <>         future<void>;   // specialization : T is void
```

构造函数

```
//default (1)	
future() noexcept;
//copy [deleted] (2)	
future (const future&) = delete;
//move (3)	
future (future&& x) noexcept;
```

![](https://upload-images.jianshu.io/upload_images/5361608-45c768b706a4fd5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- future没有复制构造函数和赋值操作符，确保不会有两个object 共享一个状态；

### 3.1.2 操作函数

![](https://upload-images.jianshu.io/upload_images/5361608-80a06bc8d6f5770f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 3.2 Shared Future

- 1. shared_future  对象类似future对象，但是shared_future对象能够被复制，即能够和另一个shared_future对象共享同一个后台shared state的状态；
- 2. 当 shared state的状态一旦ready后，能够被检索多次；

- 3. shared_future对象能够从future对象转化过来，也能从future::share()获得； 这两种方法，future对象被转化为shared_future对象后，fut ure对象会无效；
- 4. shared state的生命周期会持续到最后一个相关的shared_future对象释放；
-  class shared_future<>的接口和class future<> 相同；

### 3.2.1 模板参数
```
template <class T>  shared_future;
template <class R&> shared_future<R&>;   // specialization : T is a reference type (R&)
template <>         shared_future<void>; // specialization : T is void
```

### 3.2.2 构造函数

```
// default (1)	
shared_future() noexcept;
// copy (2)	
shared_future (const shared_future& x);
// move (3)	
shared_future (shared_future&& x) noexcept;
// move from future (4)	
shared_future (future<T>&& x) noexcept;
```
- 其中move constuctors 后构造一个新的对象，但是旧的对象x不能共享shared state;

### 3.2.3 成员函数
-  class shared_future<>的接口和class future<> 相同；

![](https://upload-images.jianshu.io/upload_images/5361608-1e3de1ca5670b3b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





---

