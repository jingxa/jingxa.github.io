---
title: STL_容器_deque_2
date: 2018-07-04 16:54:21
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---

## 3.4 deque的元素操作
### 3.4.1 deque的前和尾部删除
- pop_back()
- pop_front()

```
	// pop 
	void pop_back(){
		if(finish.cur != finish.start){ // 最后缓冲区还有一个以上的元素
			-- finish.cur;  // 调整指针；
			destroy(finish.cur);// 最后元素析构
		}else{
			 // 最后缓冲区没有元素
			pop_back_aux(); //  释放缓冲区
		}
	}
	
	
	void pop_front(){
		if(start.cur != start.last -1){// 第一缓冲区还有多个元素
			destroy(start.cur);
			++ start.cur;
		}else{  // 第一缓冲区只有一个元素
			pop_front_aux();  // 释放缓冲区
		}
	}
	

```

这两个函数依然是借助了辅助函数：

```
	// pop_back_aux
	void pop_back_aux(){
		deallocate_node(finish.first); // 释放最后一个缓冲区
		finish.set_node(finish.node -1); // 调整finish的指向
		finish.cur = finish.last -1;
		destroy(finish.cur);   // 析构该元素
	}
	
	//pop_front_aux
	void pop_front_aux(){
		destroy(start.first);  // 第一缓冲区第一个元素析构
		deallocate_node(start.first) ;  // 释放第一缓冲区
		start.set_node(start.node + 1);
		start.cur = start.first;  // 下一个缓冲区第一个元素
	}
	
```

- 在删除元素的时候，也要考虑缓冲区的释放


### 3.4.2 clear()

```
// 清除deque，保留一个缓冲区
	void clear(){
		// 针对头和尾部中间的缓冲区，都是饱满的
		for(map_pointer node = start.node +1; node<finish.node ; ++node){
			// 析构所有元素
			destroy(*node, *node+buffer_size());
			// 释放缓冲区
			data_allocator::deallocate(*node,buffer_size());
		}
		
		if(start.node !=finish.node){  // 还有头和尾部两个缓冲区
			destroy(start.cur, start.last);   // 将头部缓冲区所有元素析构
			destroy(finish.first, finish.cur); // 尾部缓冲区析构
			// 释放掉尾部缓冲区，头部缓冲区保留
			data_allocator::deallocate(finish.start, buffer_size());
		}else{  // 只有一个缓冲区
			destroy(start.cur, finish.cur); // 所有元素析构
			finish = start; // 调整状态
		}
		
	}
	

```

### 3.4.3 erase

```
	
	// erase 操作
	iterator erase(iterator pos){
		iterator next = pos;
		++next;
		difference_type index = pos - next;  // 清除点之前的元素个数
		if(index <(size() >>1 )){
			copy_backward(start,pos, next); // 移动清除之前的元素
			pop_front();  // 消除最前的一个元素
		}else{
			copy(next, finish, pos);  // 移动清除之后的元素
			pop_back();
		}
		return start+ index;
	}
	
	
	// erase  [first, last)
	void erase(iterator first, iterator last){
		if(first == start && last == finish){  // 如果是清除整个deque
			clear();
			return finish;
		}else{
			difference_type n = last - first;
			difference_type elems_before = first - start;  // 清除区间前方的元素个数
			if(elems_before < (size() -n) /2){ // 前方元素比较少
				copy_backward(start, first, last);
				iterator new_start = start +n;   // 新起点
				destroy(start, new_start);       // 移动完毕，冗余元素析构
				for(map_pointer cur = start.node; cur<new_start.node; ++cur){
					data_allocator::deallocate(*cur, buffer_size());
				}
				start = new_start; // 设定deque的新起点
			}else{
				copy(last,finish, first);  // 向前移动后方元素
				iterator new_finish = finish - n; 
				destroy(new_finish , finish);   // 移动完成冗余析构
				
				for(map_pointer cur = new_finish.node +1;cur <=finish.node; ++cur){
					data_allocator::deallocate(*cur, buffer_size());
				}
				finish = new_finish ; // deque 新尾部
				
			}
			return start + elems_before;
			
		}
	}
	
	
```


# 4 迭代器的操作

```
/**
	deque 的迭代器
	
*/

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
	
	
public:
	void set_node(map_pointer new_node){
		node = new_node;
		first =*new_node;	// 缓冲区开头
		last = first + difference_type(buffer_size());
	}

	// 运算符重载
	reference operator*() const{return *cur;}
	
	pointer operator->() const{return &(operator*());}
	
	difference_type operator-(const self& x) const{
		return difference_type(buffer_size()) * (node - x.node -1) + 
			(cur - first) + ( x.last - x.cur);
	}
	
	self& operator++(){
		++cur;
		if(cur == last){ // 到达尾部， 切换到下一节点
			set_node(node+1);
			cur = first;
		}
		return *this;
	}
	
	self operator++(int){
		self tmp  = *this;
		++*this;
		return tmp;
	}
	
	self operator--(){
		if(cur == first){
			set_node(node -1);
			cur = last;
		}
		-- cur;
		return *this;
	}

	
	self operator--(int){
		self tmp = *this;
		-- *this;
		return tmp;
	}
	
	// 随机存取
	self& operator+= (difference_type n){
		difference_type offset = n +(cur - first);
		//  first< cur -n <= offset <= cur +n < last 
		// 目标位置在同一缓冲区
		if(offset >= && offset < difference_type(buffer_size())){ 
			cur += n;
		}else{
		// 目标的位置不在同一缓冲区
			difference_type node_offset = 
				offset > 0 ? offset / difference_type(buffer_size())
					: difference_type((-offset - 1) / buffer_size()) -1;
		
			// 切换到正确的节点
			set_node(node + node_offset);
			// 切换到正确的元素
			cur = first + (offset - node_offset * difference_type(buffer_size()))
		}
		return *this;
	}
	
	self operator+(difference_type n) const{
		self  tmp = *this;
		return tmp += n;
	}
	
	self& operator -=(difference_type n){
		return *this += -n;
	}
	
	self operator-(difference_type n ){
		self tmp = *this;
		return tmp -= n;
	}
	
	// 随机存取[]
	reference operator[] (difference_type n) const {
		return *(*this + n);
	}
		
	bool operator== (const self& x)const{
		return cur == x.cur;
	}
	bool operator!=(const self& x)const{
		return !(*this == x);
	}
	boll operator<(const self& x)const{
		return (node == x.node) ? (cur < x.cur) :(node < x.node);
	}
		
		
	
	
	
};



```


---

