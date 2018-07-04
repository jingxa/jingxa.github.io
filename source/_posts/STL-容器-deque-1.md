---
title: STL_容器_deque_1
date: 2018-07-04 16:54:10
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---

# 1 概述

![](https://upload-images.jianshu.io/upload_images/5361608-aae0b52efc3e2986.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- deque 和vector的差异：
  - deque 允许常数时间的插入或删除
  - deque 没有所谓的容量(capacity)概念
 
- deque是有分段的连续空间组合而成；
- vector的空间不足再分配情况在deque上面是不会发生的；

- deque的排序： 现将deque复制到vector，在vector排序后在复制到deque;


# 2 deque 的组成
- deque有分段的连续空间组成；

![](https://upload-images.jianshu.io/upload_images/5361608-083527a9d98f0b4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2.1 deque的中控
- 采用一块map作为主控，其中map是一块连续空间，每个元素(此处称为节点,node)都是指针，指向分段的连续空间，称为缓冲区
- 缓冲区才是存储空间主题，默认值0 表示使用512bytes缓冲区；

```
template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque{	
public:
	typedef T 						value_type;
	typedef value_type* 			pointer;
	typedef size_t					size_type;
		
protected:
	// 元素的指针的指针
	typedef pointer* 				map_pointer;
	
public:
	typedef __deque_iterator<T, T&,T*,BufSiz> iterator;
	
protected:
	//  data  member
	iterator 	start;    // 表示第一个节点
	iterator    finish;   // 表示最后一个节点
	
	map_pointer map;    // 指向map, map是连续空间，每个元素是指针，指向一块缓冲区
	size_type map_size;	// map空间大小

...
```

## 2.2 deque 的迭代器
- deque的迭代器提供了4个成员
>T*	cur;  // 迭代器指向缓冲区的current 元素
	T*  first;	// 当前指向的缓冲区的头；
	T*  last; 	// 当前指向的缓冲区的尾巴（含备用空间）
	map_pointer node;   // 指向管理中心的node


```
template<class T, class Ref, class Ptr, size_t BufSiz>
struct __deque_iterator{
	typedef __deque_iterator<T, T&,T*, BufSiz>				iterator;
	typedef	__deque_iterator<T, const T&, const T*, BufSiz> const_iterator;
	static	size_t buffer_size() {return __deque_buf_size(BufSiz, sizeof(T));}
	
	// 未继承std::iterator, 必须自行添加iterator_category
	typedef random_access_iterator_tag  			iterator_category;
	typedef T										value_type;
	typedef Ptr										pointer;
	typedef	Ref										reference;
	typedef	size_t									size_type;
	typedef	ptrdiff_t								difference_type;
	typedef T**										map_pointer;
	
	typedef	__deque_iterator						self;
	
	// 保持与容器的联结
	T*	cur;  // 迭代器指向缓冲区的current 元素
	T*  first;	// 指向缓冲区的头；
	T*  last; 	// 指向缓冲区的尾巴（含备用空间）
	map_pointer node;   // 指向管理中心

...

```

其中 `buffer_size()` 中调用全局函数：

```
/** 
	全局函数  __deque_buf_size
	1. 如果n 不为0， 传回n， bcui buffer size 有用户自定义
	2. 如果n 为0， 表示buffer size 使用默认值
		- 如果sz（元素大小，sizeof(value_type)）小于512， 传回512/sz
		- 如果sz 不小于 512 ， 传回1
*/

inline  size_t  __deque_buf_size(size_t n, size_t sz){
	return n!=0 ? n : (sz < 512 ? size_t(512/sz) : size_t(1));
}
```


## 2.3 deque的数据结构
- deque维护了一个map
- 还有start,finish 两个迭代器，指向第一个缓冲区的第一个元素和最后一个缓冲区的最后一个元素的下一个位置；
- 如果map空间不足，就重新分配更大的map;

```
template<class T, class Alloc = alloc, size_t BufSiz = 0>
class deque{	
public:
	typedef T 						value_type;
	typedef value_type* 			pointer;
	typedef size_t					size_type;
		
protected:
	// 元素的指针的指针
	typedef pointer* 				map_pointer;
	
public:
	typedef __deque_iterator<T, T&,T*,BufSiz> iterator;
	
protected:
	//  data  member
	iterator 	start;    // 表示第一个节点
	iterator    finish;   // 表示最后一个节点
	
	map_pointer map;    // 指向map, map是连续空间，每个元素是指针，指向一块缓冲区
	size_type map_size;	// map空间大小
	
protected:
	// 专属空间配置器  ,每次配置一个元素大小
	typedef simple_alloc<value_type, Alloc> data_allocator;
	// 每次配置一个指针大小
	typedef simple_alloc<pointer, Alloc> map_allocator;


...
```


# 3 deque 的操作

## 3.1 deque 基本操作

```

public:
	// 基本访问
	iterator begin(){return start;}
	iterator end(){return finish;}
	
	reference operator[](size_type n){
		return start[difference_type(n)];  // 调用__deque_iterator<>::operator[]
	}
	
	reference front(){return *start;}
	reference back() {
		iterator tmp = finish;
		--tmp;		// 调用 __deque_iterator<>::operator--
		return *tmp;
	}
	
	size_type size() const{return finish - start;}
	size_type max_size() const{return size_type(-1);} // ??不明白这个函数

	bool empty() const{return finish == start; }
```

##3.2  deque的构造和内存管理

### 3.2.1 构造
- 在前面deque的数据结构中，可以发现deque定义了个空间分配器，一个是用来分配map的空间，一个是用来分配存储空间；

deque 的一个构造函数如：

```
public:
	// 构造函数
	deque(int n, const value_type& value)
		: start(),finish(), map(0), map_size(0){
			fill_initialize(n, value);
		}
		
	
protected:
	void fill_initialize(size_type n, const value_type& value){
		create_map_and_nodes(n);   // 吧deque的结构都产生冰球安排好
		map_pointer cur;
		__STL_TRY{
			// 为每个节点的缓冲区设定初始值
			for(cur = start.node;cur< finish.node; ++cur){
				uninitialized_fill(*cur, *cur+buffer_size(),value);
			}
			// 最后一个节点的设定不同（尾部备用空间不用设定初始值）
			uninitialized_fill(finish.first,finish.cur,value);
		}catch(...){
			...
		}
	}



	
protected:
	
	//负责产生并且安排好deque的结构
	void create_map_and_nodes(size_type num_elements){
		// 需要节点数= （元素个数 / 每个缓冲区的大小） + 
		// 刚好整除，多陪一个节点
		size_type num_nodes = num_elements / buffer_size() +1;
		
		// 一个map管理的节点，最少为8个，最多是“所需节点+2”
		// 前后预留一个，扩充时使用
		map_size = max(initial_map_size(),num_nodes +2);
		map = map_allocator::allocate(map_size);
		
		// 令nstart和nfinish指向map拥有的全部节点的中间区段
		// 保持在中央，可是头尾两端的扩充能量一样大，
		map_pointer nstart = map + (map_size - num_nodes) /2;
		map_pointer nfinish = nstart + num_nodes -1;
		
		map_pointer cur;
		__STL_TRY{
			// 为map内的每个节点配置缓冲区，
			for(cur = nstart; cur <=nfinish; ++cur){
				*cur = allocate_node();
			}
		}catch(...){
			// "commit of rollback"
			...
		}
		start.set_node(nstart);
		finish.set_node(nfinish);
		start.cur  = start.first;
		finish.cur = finish.first + num_elements % buffer_size();

	}	
```
例如：创建一个deque,初始化后，插入了三个值：

![](https://upload-images.jianshu.io/upload_images/5361608-3db6dcdabc25be18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.2.2 插入
- push_front()
- push_back()

```
	
public:

	void push_back(const value_type& t){
		if(finish.cur != finish.last-1){
			// 最后缓冲有一个以上的备用空间
			construct(finish.cur,t);
			++finish.cur;	// 调整最后缓冲区的状态
		}else{  // 最后缓冲区只剩一个备用空间
			push_back_aux(t);
		}
	}
	
	void push_front(const value_type& t){
		if(start.cur != start.first){
			construct(start.cur -1, t);  // 直接在备用空间构建元素
			-- start.cur;
		}else{
			push_front_aux(t);
		}
	}
	
```
这两个函数接住了两个辅助函数：

```
	
	// push_bac_aux 先配置一整块新的缓冲区，然后设置新元素，更改finish
	void push_bac_aux(const value_type& t){
		value_type t_copy=t;
		reserve_map_at_back();  // 若符合条件重现换一个map
		*(finish.node + 1) = allocate_node(); // 新配置一个节点
		__STL_TRY{
			construct(finish.cur,t_copy);
			finish.set_node(finish.node + 1);  // 改变finish,指向新节点
			finish.cur = finish.first;
		}
		__STL_UNWIND(deallocate_node(*(finish.node + 1)));
	}
	
	// push_front_aux 第一缓冲区无备用空间
	void push_front_aux(const value_type& t){
		value_type t_copy = t;
		reserve_map_at_front();   // 若符合条件重现换一个map
		*（start.node -1) = allocate_node();  // 配置新节点
		__STL_TRY{
			start.set_node(start.node - 1);
			start.cur = start.last - 1; // 设定start的状态
			construct(start.cur, t_copy);
		}catch(...){
			// "commit or rollback"
			start.set_node(start.node + 1);
			start.cur = start.first;
			deallocate_node(*(start.node - 1);
		}
	}
	
```
如图：
![](https://upload-images.jianshu.io/upload_images/5361608-85db4fdfb5a59956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 3.2.3  map的更新
- 在插入过程中，导致创建新的缓冲区，因此map的更新：

```
protected:
	// map 重新配置
	void reserve_map_at_back()size_type node_to_add =1){
		if(node_to_add +1 > map_size - (finish.node - map)){
			// 如果map尾端的节点空间不足，重新配置更大的map空间
			reallocate_map(node_to_add,false);
		}
	}
	
	void reserve_map_at_front(size_type node_to_add =1){
		if(node_to_add > start.node - map){
			reallocate_map(node_to_add, true);
		}
	}
	
	
	// 重新配置map
	void reallocate_map(size_type node_to_add, bool add_to_front){
		size_type old_num_nodes = finish.node - start.node;
		size_type new_num_nodes = old_num_nodes + node_to_add;
		
		map_pointer new_nstart;
		if(map_size > 2*new_num_nodes){ // 原来空间足够大，中间平移
			new_nstart = map + (map_size - new_num_nodes) /2
						+ (add_to_front ? node_to_add  : 0); // 
			if(new_nstart <start.node){
				copy(start.node, finish.node + 1,new_nstart);
			}else{
				copy_backward(start.node,finish.node, new_nstart + old_num_nodes);
			}
		}else{
			size_type new_map_size = map_size + max(map_size, node_to_add) + 2);
			// 配置一块新的空间
			map_pointer new_map = map_allocator::allocate(new_map_size);
			new_nstart = new_map + (new_map_size - new_num_nodes) /2
						+ (add_to_front? node_to_add : 0);
			// 把原来map内容拷贝过来
			copy(start.node, finish.node+1, new_nstart);
			
			// 释放原来map
			map_allocator::deallocate(map, map_size);
			map = new_map;
			map_size = new_map_size;
			
		}
		// 重新设定迭代器
		start.set_node(new_nstart);
		finish.set_node(new_nstart + old_num_nodes -1);
		
	}
	
	
```


### 3.2.4 insert()

```
	
	// insert 	
	iterator isnert(iterator position, const value_type& x){
		if(position.cur == start.cur){// 插入最前端
			push_front(x);
			return start;
		}else if(position.cur = finish.cur){  // 冲入最尾部
			push_back(x);
			iterator tmp = finish;
			--tmp;
			return tmp;
			
		}else{
			return insert_aux(position, x);
		}
	}
	

```

辅助函数：

```
	// insert_aux
	iterator insert_aux(iterator pos, const value_type& x){
		difference_type index = pos - start; // 插入之间的元素个数
		value_type x_copy = x;
		if(index < size() /2 ){  // 插入之前的元素个数较少
			push_front(front());   // 在最前端加入与第一元素同值的元素
			iterator front1 = start;
			++front1;
			iterator front2 = front1;
			++front2;
			pos = start + index;
			iterator pos1 = pos;
			++pos1;
			copy(front2, pos1, front1);  // 元素移动
 		}else{  // 插入点之后元素比较少
			push_back(back());   // 尾端插入与最后一个元素一样的元素
			iterator back1 = finish;
			-- back1;
			iterator back2 = back1;
			-- back2;
 			pos = start + index;
			copy(pos, back2, back1);  // 元素移动
		}
		*pos = x_copy;
		return pos;
	}
	
	
```


---

