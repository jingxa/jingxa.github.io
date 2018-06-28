---
title: STL_2_空间分配器_2
date: 2018-06-28 20:59:31
tags:
	- STL
	- Allocator
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第二章
---
# 2 内存基本处理工具
- STL有五个全局函数：作用于未初始化空间
  - construct()
  - destroy()
  - uninitialized_copy() ==>对应高层函数copy()
  - uninitialized_fill() ==> fill()
  - uninitialized_fill_n() ==> fill_n()
 
- 后面三个低层次函数：
- `<memory>`
- SGI 定义于：`<stl_uninitialized>`

- POD： plain Old Data : 标量型别或者传统的 c struct 型别：如char, int等等；
- POD: 此型别必然有用trivial ctor/dtor/ copy/ assignment函数。可以采用类似`memcpy`之类的高效率函数；
- Non-POD: 采用最保险的做法；

- __type_triats: 萃取型别的特性；
  - 如`typedef __type_traits<T>::is_POD_type is_POD` 判断T是否POD模型；
  - traits具体查看《STL源码分析》第三章;


## 2.1 uninitialized_copy

![](https://upload-images.jianshu.io/upload_images/5361608-f7bb5f4e8c3c7791.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 主要接口
```C++
// uninitialized_copy
template<class InputIterator, class ForwardIterator>
ForwardIterator uninitialized_copy(InputIterator first, InputIterator last,
	ForwardIterator result){
		retun __uninitialized_copy(first,last, result, value_type(result));
		// 利用value_type() 取出first de value_type
	}
```

- POD判别函数
```C++	
// POD  判断
template<class InputIterator, class ForwardIterator, class T>
inline ForwardIterator
__uninitialized_copy(InputIterator first, InputIterator last, 
		ForwardIterator result ,T*){
			typedef typename __type_traits<T>::is_POD_type is_POD;
			return __uninitialized_copy_aux(first, last, result, is_POD());
}
```

-  POD型别
```C++
template<class InputIterator, class ForwardIterator, class T>
inline ForwardIterator
__uninitialized_copy_aux(InputIterator first, InputIterator last,
		ForwardIterator result, __true_type){
			return copy(first, last, result);   // 调用STL算法 copy()
		}	

```
- non-POD型别
```C++ 
template<class InputIterator, class ForwardIterator, class T>
inline ForwardIterator
__uninitialized_copy_aux(InputIterator first, InputIterator last,
		ForwardIterator result, __false_type){
			ForwardIterator cur = result;
			// 异常处理省略
			for(;first != last; ++first, ++cur){
				construct(&*cur, *first);   // 必须一个一个元素构造
			}
			return cur;

		}	
```	
	
-  针对char\*, wchar_t\*的版本
```C++
inline char* uninitialized_copy(const char* first, const char* last, char* result){
	memmove(result, first, last - first);
	return result + (last - first);
} 
	
inline wchar_t* uninitialized_copy(const wchar_t* first, const wchar_t* last,
	wchar_t* result){
	memmove(result, first, sizeof(wchar_t)*(last - first));
	return result+(last - first);
}
```

## 2.2 uninitialized_fill

![](https://upload-images.jianshu.io/upload_images/5361608-89af66f445096a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  在i出调用construct(&*i, x)
```C++
template<class ForwardIterator, class T>
void uninitialized_fill(ForwardIterator first, ForwardIterator last, const T& x){
	__uninitialized_fill(first, last, x, value_type(first));
}

```
- 判断POD

```C++
template<class ForwardIterator , class T, class T1>
inline void __uninitialized_fill(ForwardIterator first, ForwardIterator last,
	const T& x, T1*){
		typedef typename __type_traits<T>::is_POD_type is_POD;
		__uninitialized_fill_aux(first, last, x, is_POD());
	}

```	
	
- POD型别
```C++
template<class ForwardIterator , class T, class T1>
inline void __uninitialized_fill(ForwardIterator first, ForwardIterator last,
	const T& x, __true_type){
		fill(first, last, x);  // 调用STL算法 fill()
		
}

```
-  non-POD型别

```C++
template<class ForwardIterator , class T, class T1>
inline void __uninitialized_fill(ForwardIterator first, ForwardIterator last,
	const T& x, __false_type){
					ForwardIterator cur = result;
			// 异常处理省略
			for(;first != last; ++first, ++cur){
				construct(&*cur, *first);   // 必须一个一个元素构造
			}
			return cur;
		
}
```

## 2.3 uninitialized_fill_n

![](https://upload-images.jianshu.io/upload_images/5361608-9fd0facaaf195893.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>
>	1. 首先使用萃取器取出first的value type,
>	2. 然后判断型别是否为POD型别（plain old data）,即标量型别，或者传统的 c struct型别；
>	3. pod必然有用无用的trivial ctor/copy/dtor/assignment函数
>	4. 对pod采用memcpy之类高效率的函数,non-pod采用保险的做法；

- 接口

```C++
template<class ForwardIterator , class Size, class T>
inline ForwardIterator uninitialized_fill_n(ForwardIterator first, Size n, const T& x){
	return __uninitialized_fill_n(first,n, x, value_type(first));
}
```

-  POD型别判断

```C++
template <class ForwardIterator , class Size, class T,class T1>
inline ForwardIterator __uninitialized_fill_n(ForwardIterator first,
	Size n , const T& x, T1*){
		typedef typename __type_traits<T1>::is_POD_type is_POD;
		return __uninitialized_fill_n_aux(first, n, x, is_POD());
	}
```
- POD 型别

```C++
template <class ForwardIterator , class Size, class T>
inline ForwardIterator __uninitialized_fill_n_aux(ForwardIterator first,
	Size n , const T& x, __true_type){
		return fill_n(first, n, x);  // 告诫函数执行
	}

```
	
-  non-POD型别

```C++

template<class ForwardIterator, class Size, class T>
inline ForwardIterator __uninitialized_fill_n_aux(ForwardIterator first,
	Size n, const T& x, __false_type){
		ForwardIterator cur = first;
		for(; n>0; --n, ++cur){
			construct(&*cur, x);
		}
		return cur;
	}
``` 

---

