---
title: STL_4_容器_list
date: 2018-07-02 10:55:50
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---


#  一、 list 概述

- 空间运用精准，不浪费
- 元素的插入删除， 常数时间

## 1 list的节点

```c++
template<class T>
struct __list_node{
	typedef void* void_pointer;
	void_pointer prev;  // 型别为void* ,其实可设为__list_node<T>*
	void_pointer next;
	T data;
};
```

## 2 迭代器

-  类型: 双向迭代器 --- Bidirectional iterators
-  插入(insert)和接合(splice)操作不会造成原有的迭代器失效

![](https://upload-images.jianshu.io/upload_images/5361608-8489ae0596b75ed8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现：

```c++
/**
	list 的迭代器
	类型: 双向迭代器 --- Bidirectional iterators
*/
template<class T, class Ref, class Ptr>
struct __list_iterator{
	typedef __list_iterator<T,T&,T*> iterator;
	typedef __list_iterator<T,Ref,Ptr> self;
	
	typedef bidirectional_iterator_org iterator_category;
	typedef T 				value_type;
	typedef Ptr				pointer;
	typedef Ref				reference;
	typedef __list_node<T>* link_type;
	typedef size_t 			size_type;
	typedef	ptrdiff_t		difference_type;
	
	link_type	node;		//迭代器内部的普通指针，指向list节点
	
// 构造函数

	__list_iterator(link_type x):node(x){}
	__list_iterator(){}
	__list_iterator(const iterator& x):node(x.node){}
	
	
	bool operator==(const self& x) const {return node == x.node;}
	bool operator!=(const self& x) const {return node != x.node;}
	//以下是对迭代器取值
	reference operator*() const {return (*node).data;}
	
	// 迭代器成员存取运算子的标准做法
	pointer operator->() const{reteurn &(operator*());}
	
	// 对迭代器累加1， 就是前进一个节点
	self& operator++(){
		node = (link_type)((*node).next);
		return *this;
	}
	
	self operator++(int){
		self tmp = *this;
		++*this;
		return tmp;
	}
	
	//迭代器递减1， 就是后退一个节点
	self& operator--(){
		node = (link_type)((*node).prev);
		return *this;
	}
	self operator--(int){
		self tmp = *this;
		--*this;
		return tmp;
	}

	
};

```

## 3 数据结构
![](https://upload-images.jianshu.io/upload_images/5361608-0854592f11693038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 不仅是一个双向链表
- 还是一个环状链表
- 让指针node指向刻意置于尾端的空白节点，node便能符合STL的“前闭后开”的区间要求

### 3.1 分配器

```c++
template< class T, Alloc = alloc>
class list{
protected:
	typedef __list_node<T> list_node;
	// 专属空间分配器,每次配置一个节点大小
	typedef simple_alloc<list_node,Alloc> list_node_allocator;  
	
public:
	typedef list_node* link_type;
	
protected:
	link_type node;   // 只要一个指针，就可表示整个链表

...
```
### 3.2 基本操作
- begin()
- end()
- empty()
- size()
- front()
- back()

```c++
	// node指向尾端空白节点
	iterator begin(){return (link_type)((*node).next);}
	iterator end() {return node;}
	bool empty() const{return node->next == node;}
	size_type size() const{
		size_type result =0;
		distance(begin(), end(), result);
		return result;
	}
	
	// 取头结点的内容
	reference front() const {return *begin();}
	// 取尾节点的内容
	reference back() const { return *(--end());}
```

## 4 构造和内存管理

### 4.1 构造和空间分配

list 提供了很多constructors，其中一个是default constructor

```c++
public:
	// constructers
	list(){empty_initialize();} // 产生一个空链表
	
protected:
	void empty_initialize(){
		node = get_node();
		node->next = node ;
		node->prev = node;  // 头和尾都指向自己，不设值
	}

```


- list 使用`alloc`作为空间分配器
  - 能够配置，释放，构造和销毁一个节点

```c++
template< class T, Alloc = alloc>
class list{
protected:
	typedef __list_node<T> list_node;
	// 专属空间分配器,每次配置一个节点大小
	typedef simple_alloc<list_node,Alloc> list_node_allocator; 
...
protected:
	/* 配置，释放，构造， 销毁一个节点 */
	// 配置一个节点并且返回
	link_type get_node(){reteurn list_node_allocator::allocate();}
	// 释放一节点
	void put_node(link_type p){ list_node_allocator::deallocate(p);}
	
	// 产生一个节点，带有元素值
	link_type create_node(const T& x){
		link_type p = get_node();
		construct(&p->data, x);    // 全局函数
		return p;
	}
	// 销毁一个节点
	void destroy_node(link_type p){
		destroy(&p->data);	// 全局函数
		put_node(p);
	}
...
```
## 4.2 元素操作
-  push_front()
-  push_back()
-  erase()
-  pop_front()
-  pop_back()
-  clear()
-  remove()
- unique()
- splice()
- merge()
- reverse()
- sort()



### 4.2.1 push_font()和push_back()
- 这两个函数依赖于一个函数`insert`

```c++
	// 在position 中插入一个节点，值为x
	iterator insert(iterator position, const T& x){
		link_type tmp = create_node(x);
		// 调整双指针
		tmp->next = position.node;
		tmp->prev = position.node->prev;
		(link_type(position.node->prev))->next = tmp;
		position.node->prev = tmp;
	}

```
- push_front() 和push_back()

```c++
	// 插入一个节点，作为头结点
	void push_front(const T& x){insert(begin(), x);}

	//插入一个节点，作为尾巴节点
	void push_back(const T& x){insert(end(),x);}
```

### 4.2.2 erase(), pop_font()和pop_back()
- 后面两个函数依赖于`erase()`

```c++
	// 移除 迭代器position的节点,返回下一个节点的迭代器
	iterator erase(iterator position){
		link_type next_node = link_type(position.node->next);
		link_type prev_node = link_type(position.node->prev);
		prev_node->next = next_node;
		next_node->prev = prev_node;
		destroy_node(position.node);
		return iterator(next_node);
	}

	// 移除头结点
	void pop_front(){
		erase(begin());
	}
	
	//移除尾巴节点
	void pop_back(){
		iterator tmp == end();
		erase(--tmp);
	}

```

### 4.2.3 remove()和clear(), unique()
- clear()

```c++
	// 清除所有节点
	clear(){
		link_type cur = (link_type)node->next;
		while(cur != node){
			link_type tmp = cur;
			cur = (link_type)cur->next;
			destroy_node(tmp);	// 销毁一个节点
		}
		// 恢复node原始状态
		node->next = node;
		node->prev = node;
	}

```
- remove()

```c++
	// 将数值为value的元素移除
	remove(const T& value){
		iterator first = begin();
		iterator last = end();
		while(first != last){
			iterator next = first;
			++next ;
			if(*next == value)erase(next);
			first = next;
		}
	}
```

- unique()

```c++
	// 移除数值相同的连续元素 “连续相同的元素”，移除剩余一个
	void unique(){
		iterator first = begin();
		iterator last = end();
		if(first == last) return ;  // 空链表，什么都不必做
		iterator next = first;
		while(++next != last)
			if(*first == *next){
				erase(next);
			}else{
				first = next;
				next = first;
			}
	}
```

### 4.2.4 splice()、merge()、reverse()、sort()
- 这几个函数依赖于内部函数`transfer`
![](https://upload-images.jianshu.io/upload_images/5361608-5d2569f43d9a3e18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```c++
	
protected:
	// 将[first, last) 内的元素移动到position 之前
	void transfer(iterator position, iterator first, iterator last){
		if(position != last){
			(*(link_type)((*last.node).prev)).next = position.node;  //后一个元素
			(*(link_type)((*first.node).prev)).next = last.node; // 前一个元素
			(*(link_type)((*position.node).prev).next = first.node;
			link_type tmp  = link_type(*(position.node).prev);
			(*position.node).prev = (*last.node).prev;
			(*last.node).prev = (*first.node).prev;
			(*firt.node).prev = tmp;
		}
	}
```
**（1） splice()**

![](https://upload-images.jianshu.io/upload_images/5361608-0be901f57d9f5e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```c++
// 将x接于position 之前， x必须不同于*this
void splice(iterator position, list& x){
	if(!x.empty())
		transfer(position, x.begin(), x.end());
}

// 将i所指的元素接于position 之前，position 和i可指向同一个list
void splice(iterator position, list&, iterator i){
	iterator  j = i;
	++j;
	if(position == i || position == j)return ;
	ransfer(position, i, j);
	
}

//将[first, last)内所有的元素接于position 所指位置之前
// 可为同一list的迭代器
// position 不能在区间之中
void splice(iterator position, list&, iterator first,iterator last){
	if(first != last){
		transfer(position, first, last);
	}
```


**(2) merge()**

```c++

	//  将x合并到*this中，两个list内容必须先经过递增排序
	void  merge(list<T,Alloc>& x){
		iterator first1 = begin();
		iterator last1 = end();
		iterator first2 = x.begin();
		iterator last2  = x.end();
		
		// 前提是，两个lists都已经经过递增排序
		while(first1 != last1 && first2 != last2){
			if(*first2 < *first1){
				iterator next = first2;
				transfer(first1, first2, ++next);
				first2 = next;
			}else{
				++first1;
			}
			if(first2 != last2)transfer(last2, first2, last2);
		}
	}

```

**(3) reverse()**

```c++
	//逆向重置
	void reverse(){
		// 如果空链表，不进行操作
		// size ==0 或者 size == 1
		if(node->next == node || link_type(node->next)->next == node)
			return;
		iterator first = begin();
		++ first;
		while(first != end()){
			iterator old = first;
			++ first;
			transfer(begin(), old, first);
		}
		
	}
```

**(4) sort()**

- [暂时未读明白]

```c++
	// list 不能STL算法 sort(), 必须使用自己的sort() member function,
	// stl算法 sort()只接受randomAccessIterator
	// 本函数采用quick sort
	void sort(){
		// size ==0 || size ==1
		if(node->next == node || link_type(node->next)->next == node)
			return ;
		
		// 一些新的lists,作为中介数据存放区
		list<T, Alloc> carry;
		list<T, Alloc> counter[64];
		int fill =0；
		while(!empty()){
			carry.splice(carry.begin(), *this, begin());
			int i=0;
			while( i < fill && !counter[i].empty()){
				counter[i].merge(carry);
				carry.swap(counter[i++]);
			}
			carry.swap(counter[i]);
			if(i == fill)++fill;
		}
		
		
		for(int i=1 ; i< fill; ++i){
			counter[i].merge();
		}
		swap(counter[fill - 1]);
	
	}
```






---

