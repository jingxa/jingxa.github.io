---
title: STL_仿函数
date: 2018-07-11 20:47:56
tags:
	- STL
	- Function Object
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第七章
---

![](https://upload-images.jianshu.io/upload_images/5361608-e4607d5a9010612b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 1 概述

- 操作数的个数
  - 一元
  - 二元

- 功能
  - 算法运算
  - 关系运算
  -  逻辑运算

- `<stl_finction.h>` 


# 2 基类

```

/**
	一元仿函数： unary_function
*/

template<class Arg, class Result>
struct unary_function{
	typedef Arg argument_type;
	typedef Result result_type;
};


/**
	二元仿函数： binary_function
*/
template<class Arg1,class Arg2,class Result>
struct binary_function{
	typedef Arg1 first_argument_type;
	typedef Arg2 seconde_argument_type;
	typedef Result return_type;
};

```


# 3 算术运算


```
/**

	一、 算术类
	--- 支持加法，减法，乘法，除法，模数和否定运算
	-- 加法：plus<T>
	-- 减法：minus<T>
	-- 乘法： multiplies<T>
	-- 除法： divides<T>
	-- 模取： modulus<T>
	-- 否定： negate<T>
*/


template<class T>
struct plus : public binary_function<T,T,T>{
	T operator()(const T& x, const T& y) const{
		return x + y;
	}
};

template<class T>
struct minus: public binary_function<T,T,T>{
	T operator()(const T& x, const T& y) const {
		return x - y;
	}
};


template<class T>
struct multiplies: public binary_function<T,T,T>{
	T operator()(const T& x,const T& y) const{
		return x * y;
	}
};



template<class T>
struct divides: public binary_function<T,T,T>{
	T operator()(const T& x, const T& y) const{
		return x / y;
	}
};

template<class T>
struct modulus: public binary_function<T,T,T>{
	T operator()(const T& x, const T& y) const {
		return x % y;
	}
};


template<class T>
struct negate: public unary_function<T,T>{
	T operator()(const T& x) const{
		return -x;
	}
};


```

# 4 关系运算


```

/**

	二、关系运算
	-- 等于： equal_to<T
	-- 不等于： not_equal_to<T
	-- 大于： greater<T>
	-- 大于或等于： greater_equal<T>
	-- 小于： less<T>
	-- 小于等于： less_equal<T>
	
*/
template< class T>
struct equal_to: public binary_function<T,T, bool> {
	bool operator()(const T& x, const T& y)const{
		return x==y;
	}
};

template<class T>
struct not_equal_to: public binary_function<T,T,T>{
	bool oeperator()(const T& x, const T& y) const{
		return x!= y;
	}
};



template< class T>
struct greater: public binary_function<T,T,T> {
	bool operator()(const T& x, const T& y) const{
		return x>y;
	}
};

template< class T>
struct less: public binary_function<T,T,T> {
	bool operator()(const T& x,const T& y) const{
		return x < y;
	}
};

template< class T>
struct greater_equal: public binary_function<T,T,T>{
	bool operator()(const T& x, const T& y) const{
		sdreturn x>=y;
	}
};

template< class T>
struct less_equal: public binary_function<T,T,T>{
	bool operator()(const T& x,const T& y)const{
		return x <= y;
	}
};



```

# 5 逻辑运算

```

/**
	三、 逻辑运算
	-- And : logical_and
	-- Or  : logical_or
	__ Not:  logical_not
*/

template<class T>
struct logical_and:public binary_function<T,T,T>{
	bool operator()(const T& x, const T& y) const{
		return x && y;
	}
};


template<class T>
struct logical_or:public binary_function<T,T,T>{
	bool operator()(const T& x, const T& y) const{
		return x || y;
	}
};



template<class T>
struct logical_not:public unary_function<T,T>{
	bool operator()(const T& x) const{
		return !x ;
	}
};






```



# 6 证同（identity）、选择（select）、 投射（project）

## 6.1 证同
### 6.1.1 仿函数的证同
 证同元素(identity element)
- 数值A若与该元素做op运算，会得到A自己；
- 加法的证同元素为0
- 乘法的证同元素为1 

```
/**

 3.2 证同元素(identity element)
- 数值A若与该元素做op运算，会得到A自己；
- 加法的证同元素为0
- 乘法的证同元素为1 
*/

template<class T>
inline T identity_element(plus<T>){
	return T(0);
}


template< class T>
inline T identity_element(multiplies<T>){
	return T(1);
}

```

### 6.1.2 证同函数
```

struct identity: public unary_function<T,T>{
	const T& operator()(const T& x)const{
		return x;
	}
};
```

## 6.2 选择


```

// 选择函数： 接受一个pair,传回第一个元素
// 运用于<stl_map.h> ，用来指定rb_tree所需要的KeyOfValue op;
template< class Pair>
struct select1st:public unary_function<Pair,typename Pair::first_type>
{
	const typename Pair::first_type& operator()(const Pair& x) const{
		return x.first;
	}
};

// 选择函数，返回第二个元素
// SGI stl 未使用此式子
template< class Pair>
struct select2nd:public unary_function<Pair,typename Pair::second_type>
{
	const typename Pair::second_type& operator()(const Pair& x) const{
		return x.second;
	}
};
```


## 6.3 投射

```

// 投射函数：传回第一参数，忽略第二参数
// SGI STL 未使用此式子
template<class Arg1,class Arg2>
struct project1st:public binary_function<T,T,T>{
	Arg1 operator()(const Arg1& x, const Arg2&) const{
		return x;
	}
};



// 投射函数：传回第二参数，忽略第一参数
// SGI STL 未使用此式子
template<class Arg1,class Arg2>
struct project1st:public binary_function<T,T,T>{
	Arg2 operator()(const Arg1&, const Arg2& y) const{
		return y;
	}
};

```
---

