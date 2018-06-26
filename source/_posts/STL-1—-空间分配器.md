---
title: STL_1—_空间分配器
date: 2018-06-26 20:56:17
tags:
	- STL
	- Allocator
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》
---

# 1. STL allocater

## 1.1 操作过程
- 对象创建
  - 内存分配： alloc:allocate()
  - 对象构造： ::construct()
- 对象释放
  - 对象析构： ::destroy()
  - 内存释放：  alloc:deallocate()

## 1.2 定义
- `<memory>`
包含以下两个头文件：
```C++
#include <stl_alloc.h>  //负责内存配置和释放
#include <stl_construct.h>  //负责对象构造和析构
```

![](https://upload-images.jianshu.io/upload_images/5361608-e5370350f3ee4f1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.3 构造和析构
- construct()
- destroy()

部分代码：
```C++
# include <new.h>  //使用placement new 

template<class T1, class T2>
inline void construct(T1* p, T2& value){
  new(p) T1(value);  // placement new ; 调用T1::T1(value);在p指向的内存上构造
}


template<class T>
inline void destroy(T& pointer){
  pointer->~T();  //调用析构 ~T()
}

//destroy 的其他版本 
···
```


## 1.4 空间的配置和释放：std::alloc
### 1.4.1 分类
- malloc()
- free()
- 内存破碎问题：
  - 第一级配置器： 直接使用malloc()和free()
  - 第二级配置器： 
    - (1) 配置区块大于128bytes，使用第一级配置器；
    - (2) 小于128bytes，视区块过小，使用memory pool 即内存池

定义：
第一级配置器： `_malloc_alloc_template`
第二级配置器：`_default_alloc_template`

- alloc不接受任何template参数；

### 1.4.2 接口
- 为了符合stl规范，包装一个接口；

```C++
template<class T， class Alloc>
class simple_aaloc{
public:
	static T* allocate(size_t n){
		return 0 == n ? 0 : (T*)Alloc::allocate(n*sizeof(T));
	}
	
	static T* allocate(void){
		return (T*) Alloc::allocate(sizeof(T));
	}
	
	static void deallocate(T* p, size_t n){
		if(0 != n)Alloc::deallocate(p, n*sizeof(T));
	}
	
	static void deallocate(T* p){
		Alloc::deallocate(p, sizeof(T));
	}
};

```

### 1.4.3  实际使用

![](https://upload-images.jianshu.io/upload_images/5361608-966b28c00474dcf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/5361608-a95bf56b52f422fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.4.5 第一级配置器 `__malloc_alloc_template`
第一级配置器直接使用`malloc`和·free·

```C++
template<int inst>  // 无template参数，“非型别参数”inst没用
class __malloc_alloc_template{

private:

// 处理内存不足的函数指针
//  oom: out of memory
	static void *oom_malloc(size_t);
	static void *oom_realloc(void*, size_t);
	static void (*__malloc_alloc_oom_handler)();
	
	
public:
//	分配内存
	static void * allocate(){
		void* result = malloc(n); 	// 直接使用malloc
		if(0 == result)result = oom_malloc(n); // 如果分配不成功，改用oom_malloc()
		return result;
	}
// 释放内存
	static void* deallocate(void* p , size_t /*n*/){
		free(p);		//直接使用free
	}
	
//... 省略
};
```

### 1.4.6 第二级配置器`__default_alloc_template`

(1) **定义**
- 如果区块：
  - 大于128bytes： 交给第一级配置器处理
  - 小于128bytes: 使用内存池（memory pool）管理

- 内存池管理（次层配置）：
  - 每次配置一大块内存，维护对应的自由链表(free-list)，然后从链表中划分内存给请求，或者回收小块内存；
  - 内存池维护16个free-lists：各自管理(8, 16,24,32, ..., 120 ,128)的小额区块；

![](https://upload-images.jianshu.io/upload_images/5361608-62692f7adc03ce2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```C++
/**
	第二配置器的部分代码
*/

enum{__ALIGN = 8};   // 小型区块的上调边界  ，分配内存不足8的补齐8
enum {__MAX_BYTES = 128};		//小型区块的上限
enum {__NFREELISTS = __MAX_BYTES / __ALIGN};  // free-lists


template<bool threads, in inst>  // 第一参数 多线程， 第二参数 无用
class __default_alloc_template{
	
private:
// round_up() 将byte上调至8的倍数
static size_t ROUND_UP(size_t bytes){
	return (((bytes) + __ALIGN -1) & ~(__ALIGN -1));
}

private:
	union obj{		// free-lists 的节点构造
		union obj * free_list_link;
		char client_data[1];  
	};
	
private:
	static obj* vaolatile free_list[__NFREELISTS];   // 16个列表
	static size_t FREELIST_INDEX(size_t bytes){
		return (((bytes) + __ALIGN -1) / __ALIGN - );
	}
	
	static void* refill(size_t n);	// 返回一个大小为n的对象，可能加入大小为n的其他区块到free list
	
	static char* chunk_alloc(size_t size, int & nobjs);		//配置一大块内存，容纳nojbs个“size”的区块
	
	
// static data members

static char* start_free;  // 内存池起始位置
static char* end_free;
static size_t heap_size;

// 成员函数
// 省略

public:
	static void* allocate(size_t n){  // n 大于0/* 后详*/}

	static void deallocate(void* p , size_t n){/*后详*/};
	
	static void* reallocate(void* p, size_t old_size, size_t new_size);
};

// 静态变量的初始化 
// 省略
```

(2) **配置器函数allocate**
- 第二级配置器分配内存的过程：

![](https://upload-images.jianshu.io/upload_images/5361608-5f57f1d1a29d7128.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```C++
	static void* allocate(size_t n){  // n 大于0
		obj* volatile * my_free_list;
		obj* result;
		
		// 大于128bytes
		if(n > (size_t)__MAX_BYTES){
			return (malloc_alloc::allocate(n));
		}
		// 寻找16个free_list中的一个
		my_free_list = free_list + FREELIST_INDEX(n);
		result = *my_free_list;
		if(0 == result){
			// 没有可用的free_list,准备填充free list
			void* r = refill(ROUND_UP(n));
			return r;
		}
		// 调整free list
		* my_free_list = result->free_list_link;  // 将result指向的区块移除，my_free_list 指向result的后续
		return result;
		
		
	}
```

(3) **配置器函数deallocate()**
- 同样，如果区块大于128bytes调用第一级配置器；
- 小于128bytes就找到相应的free list ，并将区块收回；

![](https://upload-images.jianshu.io/upload_images/5361608-db5f494e259f11b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```C++
	static void deallocate(void* p , size_t n){
		obj* q = (obj*)p;
		obj* volatile * my_free_list;
		//大于128bytes 调用第一级配置器
		if(n>(size_t)__MAX_BYTES)
		{
			malloc::deallocate(p, n);
			return;
		}
		// 寻找对应的free list
		my_free_list = free_list + FREELIST_INDEX(n);
		q->free_list_link = *my_free_list;
		*my_free_list = q;
	}
```
(4)  **重新填充 free lists**
- free list中没有可用区块的时候，调用`refill()`，为free list填充空间，新的空间取自内存池(chunk_alloc()完成)，缺省取得20个新节点，如果内存池不足，可能小于20个；
```C++

template<bool threads, int inst>
void* __default_alloc_template<threads, inst>::refill(size_t n){
	int nobjs = 20;
	// 调用chunk_alloc()，nobjs 是引用传值
	char* chunk = chunk_alloc(n, nobjs);  
	
	obj* volatile * my_free_list;
	obj* result;
	obj* current_obj, * next_obj;
	int i;
	
	// 如果只获得一个区块，直接分给调用者，free list 无新节点
	if(1 == nobjs )retun (chunk);
	
	//否则准备调整free list , 接入新节点
	my_free_list = free_list + FREELIST_INDEX(n);
	
	// 以下在chunk空间建立free list 
	result = (obj*) chunk;  // 这一块返回客户端
	
	//导引free list 指向新配置的空间
	*my_free_list = next_obj = (obj*) (chunk+n);
	
	// 从1开始，第o个返回给客端
	for（int i= 1;; i++){
		current_obj = next_obj;
		next_obj = (obj*)((char*)next_obj + n); //分块
		if(nobjs -1 == i){   // 最后一块
			current_obj->free_list_link =0;
			break;
		}else{
			current_obj ->free_list_link = next_obj;
		}
	}
	return result;
}

```
(5) **内存池(memory pool)**

- 当free list中区块不够的时候，需要从内存池中取得新的区块；
- `chunk_alloc()`完成这项任务：

```C++
// chunk_alloc
template<bool threads, in inst>
char* __default_alloc_template<threads, inst>::chunk_alloc(size_t size, int& nobjs){
	char* result;
	size_t total_bytes = size * nobjs;
	size_t bytes_left = end_free - start_free;  // 内存池剩余空间
	
	if(bytes_left >= total_bytes){
		result = start_free;
		start_free += total_bytes;
		return result;
	}else if(bytes_left >=size){
		// 剩余空间不能满足所有需求量，但足够供应一个以上的区块
		nobjs = bytes_left / size;
		total_bytes = size * nobjs;
		result = start_free;
		start_free += total_bytes;
		return result;
	}else{
		//内存池剩余空间不能满足一个区块
		size_t bytes_to_get = 2 * total_bytes + ROUND_UP(heap_size >> 4);
		
		if(bytes_left > 0){
			//内存池还有零头，线分配给free list
			//首先寻找合适的free list
			obj* volatile* my_free_list = free_list + 
				FREELIST_INDEX(bytes_left);
			// 调整free list, 将内存中的残余空间编入
			((obj*)start_free)->free_list_link = *my_free_list;
			*my_free_list = (obj*)start_free;
		}
		
		
		// 配置heap空间，用来补充内存池
		start_free = (char*)malloc(bytes_to_get);
		if(0 == start_free){
			// heap 空间不足， malloc失败
			int i;
			obj* volatile* my_free_list,*p;
			// 搜索适当的free list: 指的是“尚有未用完，而且区块够大”的free list
			for(i = size;i<__MAX_BYTES; i+=__ALIGN){
				my_free_list = free_list + FREELIST_INDEX(i);
				p = *my_free_list;
				if(0 != p){ // free_list 有未用完区块
					*my_free_list = p->free_list_link;
					start_free = (char*)p;
					end_free = start_free +i;
					//递归调用自己，为了修正nobjs
					return chunk_alloc(size, nobjs);
					// 注意： 任何残零头部都被编入到free-list 中备用
				}
			}
			end_free =0; // 如果出现意外，没有任何内存可用
			// 调用第一级配置器，看看是oom否有用
			start_free = (char*)malloc_alloc:allocate(bytes_to_get);
			// 这会导致异常，或者存不足的情况获得改善
		}
		heap_size += bytes_to_get;
		end_free = start_free + bytes_to_get；
		//递归调用自己， 为了修正nobjs
		return chunk_alloc(size, nobjs);
		
	}	
	
}

```
- `chunk_alloc()`通过`end_free - start_free`判断内存池的状态；
- 如果内存池剩余充分， 直接调出20个区块返回；
- 如果不够20个区块，返回世界能够供应的区块；
- 如果一个都不够，通过malloc() 从heap中配置内存-- 新内存块为需求量的2倍，再加上一个随着配置次数越来越大的附加量；

例子：

![](https://upload-images.jianshu.io/upload_images/5361608-dd0ce8eb28e51e1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




---

