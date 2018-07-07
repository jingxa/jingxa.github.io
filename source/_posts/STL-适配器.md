---
title: STL_适配器
date: 2018-07-05 21:43:51
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---


# 1 适配器 （adapter）
- "修改某物接口，形成另一个风貌者"的性质，称为adapter；

STL的适配器：
- stack
- queue


# 2 stack

- 以deque或者 list作为底层容器
- 先进后出
- stack没有迭代器

![](https://upload-images.jianshu.io/upload_images/5361608-f525a94e3884b2a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
emplate<class T, Sequence = deque<T>>
class stack{
	// __STL_NULL_TMPL_ARGS 展开为<>
	friend bool operator== __STL_NULL_TMPL_ARGS(const stack&, const stack&);
	friend bool operator< __STL_NULL_TMPL_ARGS(const stack&, const stack&);
	
public:
	typedef typename Sequence::value_type value_type;
	typedef typename Sequence:size_type 	size_type;
	typedef typename Sequence::reference	reference;
	typedef	typename Sequence::const_reference	const_reference;
	
protected:
	Sequence c;  // 底层容器
	
	
public:
	// 一下完全利用Sequence c 的操作，完成stack的操作
	bool empty()const{return c.empty();}
	size_type	size() const{return c.size();}
	reference top(){returnc.back();}
	const_reference top() const{return c.back();}
	
	// stack末端进，末端出
	void push(const value_type& x){c.push_back(x);}
	void pop(){c.pop_back();}
};


template<class T, class Sequence>
bool operator==(const stack<T,Sequence>& x, const stack<T, Sequence>& y){
	return x.c = y.c;
}

template<class T ,class Sequence>
bool operator<(const stack<T, Sequence>& x, const stack<T, Sequence>& y){
	return x.c < y.c;
}


```

# 3 queue

- 以deque或者list作为底层容器
- 先进先出的结构
- 不允许有遍历行为，没有迭代器

![](https://upload-images.jianshu.io/upload_images/5361608-c3723b2ba1434db7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
/**
	queue: 先进先出
	1. 不允许遍历
	2. deque作为底层容器
*/

template<class T, class Sequence = deque<T> >
class queue{
	// __STL_NULL_TMPL_ARGS展开为<>
	friend bool operator== __STL_NULL_TMPL_ARGS(const queue& x, const queue& y);
	friend bool operator< __STL_NULL_TMPL_ARGS(const queue& x, const queue& y);
	
public:
	typedef typename	Sequence::value_type	value_type;
	typedef	typename	Sequence::size_type		size_type;
	typedef	typename	Sequence::reference		reference;
	typedef	typename	Sequence::const_reference	const_reference;
	
protected:
	Sequence c;  // 底层容器

public:
	bool empty() const{return c.empty();}
	size_type size() const{return c.size();}
	reference front(){return c.front();}
	const_reference front() const{return c.front();}
	reference back(){return c.back();}
	const_reference back(){return c.back();}
	
	// queue 末端进，前端出
	void push(const value_type& x){c.push_back(x);}
	void pop(){c.pop_front();}
	
	
};


template<class T, class Sequence>
bool operator==(const queue<T, Sequence>& x, const queue<T,Sequence>& y){
	return x.c == y.c;
}

template<class T, class Sequence>
bool operator,(const queue<T, Sequence>& x, const queue<T,Sequence>& y){
	return x.c , y.c;
}

```


# 4 priority_queue
## 4.1  概述
- 带有权值观念的queue;
- 按照权值排列；
- 缺省情况下priority_queue利用max_heap完成；


![](https://upload-images.jianshu.io/upload_images/5361608-a4a3bd8124fc1325.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4.2 定义
- priority_queue内部以vector为底层容器；
- 使用泛型算法`push_heap`、`pop_heap`  、`mmake_heap`；


```

template<class T, class Sequence = vector<T>, 
	class Compare = less<typename Sequence::value_type> >
class priority_queue{
public:
	typedef typename Sequence::value_type  value_type;
	typedef typename Sequence::size_type 	size_type;
	typedef	typename Sequence::reference	reference;
	typedef	typename Sequence::const_reference	const_reference;
	
protected:
	Sequence c; // 底层容器
	Compare	comp;  // 元素比较器
	
pubic:
priority_queue(): c(){}
explicit priority_queue(const Compar& x):c(),comp(x){}
	
	
	// 一下用到 make_heap()，push_heap(),pop_heap()都是泛型算法
	// 注意，任一个构造函数都立刻于底层容器内产生一个隐式表述法的heap
	template<class InputIterator>
	priority_queue(InputIterator first, InputIterator last,
	const Compar& x):c(first,last), comp(x){
		make_heap(c.begin(),c.end(),comp);
	}
	
	template<class InputIterator>
	priority_queue(InputIterator first, InputIterator last)
		:c(first,last){
			make_heap(c.begin(),c.end(),comp);
		}
	
	bool empty() const{return c.empty();}
	size_type size() const{return c.size();}
	const_reference top() const{return c.front();}
	void push(const value_type& x){
		__STL_TRY{
			// push_heap 是泛型算法，先利用底层容器的push_back()将xnyrsu
			// 推入末端，在重排heap
			c.push_back(x);
			push_heap(c.begin(),c.end(),comp);  // push_heap 是泛型算法
		}
		__STL_UNWIND(c.clear())；
	}
	
	void pop(){
		__STL_TRY{
			// pop_heap 是泛型算法， 从heap内取出一个元素，它并不是真正将
			// 元素弹出，而是重排heap,然后再以底层容器pop_back()取得被弹出的元素
			pop_heap(c.begin(),c.end(),comp);
			c.pop_back();
		}
		__STL_UNWIND(c.clear());
	}
};
```

---

