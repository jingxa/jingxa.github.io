---
title: STL_非标准_slist
date: 2018-07-05 21:44:42
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---





- STL list是一个双向链表
- slist不在STL标准中，是一个单向链表；

差别：
- slist 的迭代器是单向的ForwardIterator
- list的迭代器是 双向的BidrectionalIterator


共同特点：
- 插入，移除，接合不会造成原有迭代器失效

# 1 节点结构
![](https://upload-images.jianshu.io/upload_images/5361608-c69d236d1d012e2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
// 节点

struct __slist_node_base{
	__slist_node_base* next;
	
};

// 节点结构

template<class T>
struct __slist_node : public __slist_node_base{
	T data;
};

// 全局函数
// 已知道某一节点， 插入新节点于其后
inline __slist_node_base* __slist_make_link(
			__slist_node_base* prev_node,
				__slist_node_base* new_node){
	// 令new节点的下一个节点为prev节点的下一个节点
	new_node->next = prev_node->next;
	prev_node->next = new_node;  // prev指向new节点
	return new_node;
}


// 全局函数 单向链表的大小

inline size_t __slist_size(__slist_node_base* node){
	size_t result = 0;
	for(;node !=0; node = node->next)
		++result;
	return result;
}
```

# 2 迭代器


![](https://upload-images.jianshu.io/upload_images/5361608-2d9e4831ec4892b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

/**
	slist的迭代器
*/


struct __slist_iterator_base{
	typedef size_t size_type;
	typedef	ptrdiff_t difference_type;
	typedef forward_iterator_tag  iterator_category;  // 单向链表的大小
	
	__slist_node_base* node;  // 指向节点基本结构
	__slist_node_base(__slist_node_base* x):node(x){}
	
	void incr(){
		node = node ->next;  // 前进一个节点
	}
	
	bool operator== (const __slist_iterator_base& x) const{
		return node == x.node;
	}
	
	bool operator!= (const __slist_iterator_base& x) const{
		return node != x.node;
	}

	
};


// 单向链表的迭代器结构

template<class T, class Ref , class Ptr>
struct _slist_iterator: public __slist_iterator_base{
	typedef __slist_iterator<T,T&,T*> 				iterator;
	typedef __slist_iterator<T,const T&, const T*>   const_iterator;
	typedef __slist_iterator<T,Ref,Ptr>				self;
	
	typedef T										value_type;
	typedef Ptr										pointer;
	typedef	Ref										reference;
	typedef	__slist_node<T> 						list_node;
	
	
	__slist_iterator(list_node& x):__slist_iterator_base(x){}
	// 调用slist<T>::end()会造成 __slist_iterator_base(0)
	__slist_iterator():__slist_iterator_base(0){}
	__slist_iterator(const iterator& x):__slist_iterator_base(x.node){}
	
	
	reference operator*()const{ return ((list_node*)node)->data;}
	pointer  operator->() const{ return &((operator*());}


	self& operator++(){
		incr();
		return *this;
	}
	
	self operator++(int){
		self tmp = *this;
		incr();
		return tmp;
	}
	
	// 没有实现operator-- ,因为这是一个forward iterator
};



```


# 3 数据结构 

```

/**
	slist 数据结构
	
*/


template<class T, class Alloc = alloc>
class slist{
public:
	typedef T							value_type;
	typedef value_type*					pointer;
	typedef	const value_type*			const_pointer;
	typedef	value_type& 				reference;
	typedef	const value_type&			const_reference;
	typedef size_t						size_type;
	typedef	ptrdiff_t					difference_type;
	
	
	typedef	__slist_iterator<T,T&,T*>	iterator;
	typedef	__slist_iterator<T,const T&, const T*>	const_iterator;
	
private:
	typedef	__slist_node<T>		list_node;
	typedef	__slist_node_base   list_node_base;
	typedef	__slist_iterator_base	iterator_base;
	typedef	simple_alloc<list_node, Alloc>  list_node_allocator;
	
	
	static list_node* create_node(const value_type& x){
		list_node* node = list_node_allocator::allocate(); // 配置一个节点
		__STL_TRY{
			construct(&node->data,x);
			node->next = 0;
		}
		__STL_UNWIND(list_node_allocator::deallocate(node)):
		return node;
	}
	
	
	static void destroy_node(list_node* node){
		destroy(&node->data);   // 析构元素
		list_node_allocator::deallocate(node);  // 释放空间
	}
	
	
private:

	list_node_base	 head;   // 头部，不是指针，是对象

public:
	slist(){head.next =0;}
	~slist(){clear();}
	

public:
	// 基本操作
	
	iterator begin(){return iterator((list_node*)head.next);}
	iterator end() {return iterator(0);}
	size_type size()const{return __slist_size(head.next);}
	bool empty() const {return head.next == 0;}

	
	// 两个slist互换
	
	void swap(slist& L){
		list_node_base * tmp = head.next;
		head.next = L.head.next;
		L.head.next = tmp;
	}
	
public:

	// 取头部元素
	reference front(){return ((list_node*)head.next)->data;}
	
	// 从头部插入元素
	void push_front(const value_type& x){
		__slist_make_link(&head, create_node(x));
	}
	
	// 从头部取走元素；
	void pop_front(){
		list_node* node = (list_node*) head.next;
		head.next = node->next;
		destroy_node(node);
	}
	
	...
	
};
 
```




---

