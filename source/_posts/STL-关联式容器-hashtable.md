---
title: STL_关联式容器_hashtable
date: 2018-07-08 18:44:01
tags:
	- STL
	- Associative Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第五章
---

# 1 概述

- 插入，删除，搜索都是常数时间
- 视为一种字典结构

![](https://upload-images.jianshu.io/upload_images/5361608-ab2c47b3a5e91bf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题：

1.  array数组
2.  索引
  - 索引碰撞

- 负载系数： 元素个数除以表格大小（大小为 0 ~ 1）；

2 ：解决办法：
- 线性探测：如果目标位置不可用，循序意义往下寻找，知道一个可用空间即可；
![](https://upload-images.jianshu.io/upload_images/5361608-2b6d559944a935d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 二次探测：

![](https://upload-images.jianshu.io/upload_images/5361608-f9011c6bd0983d3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 开链

 ![](https://upload-images.jianshu.io/upload_images/5361608-318b1549c961b0b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2 桶（buckets）和节点(nodes)

- hash table的表格元素为桶子；(表示可能为单个元素，或者一个list)


![](https://upload-images.jianshu.io/upload_images/5361608-fd77a7af5f9a037c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 3 hashtable 实现

## 3.1 hash table 节点 


```
/**
	hash table 节点定义
	bucket维护的linked list，不采用stl的list或slist，而是维护
	上述的hash table node;
	bucket聚合体，以vector完成；
*/

template<class Value>
struct __hashtable_node{
	__hashtable_node* next;
	Value val;

};

```

## 3.2 hashtable迭代器
- 	hashtable的迭代器器没有后退操作，也没有双向迭代器

```

/**
	hashtable 的迭代器
	hashtable的迭代器器没有后退操作，也没有双向迭代器
*/

template<class Value, class Key, class HashFcn,
		class ExtratKey, class Equalkey, class Alloc>
struct __hashtable_iterator{
	typedef hashtable<Value, Key,HashFcn,ExtratKey,Equalkey,Alloc>
		hashtable;
		
	typedef __hashtable_iterator<Value,Key,HashFcn,ExtratKey,
									Equalkey,Alloc>  iterator;
					
	typedef __hashtable_iterator<Value,Key,HashFcn,ExtratKey,
									Equalkey,Alloc>  const_iterator;
	
	typedef __hashtable_node<Value> node;
	
	typedef forward_iterator_tag  iterator_category;
	typedef	Value 				  value_type;
	typedef ptrdiff_t			  difference_type;
	typedef size_t				  size_type;
	typedef Value& 				  reference;
	typedef Value*                pointer;
	
	node* cur;   // 迭代器当前指向节点
	hashtable* ht; // 保持对容器的连接关系
	
	
	
	__hashtable_iterator(node* n, hashtable* tab):cur(n),ht(tab){}
	__hashtable_iterator(){}
	
	reference operator*()const {return cur->val;}
	pointer operator->() const {return &(operator*());}
	
	
	// hashtable的迭代器器没有后退操作，也没有双向迭代器
	// 如果存在下一节点，直接next指针就可以；
	// 如果节点为list的尾端，就跳到下一个bucket，即指向下一个list的头部节点；
	iterator oprator++(){
		const node* old = cur;
		cur = cur->next; // 如果存在， 否则进入一下if
		if(! cur){
			// 根据元素值，定位下一个bucket，起始处就为答案
			size_type bucket = ht->bkt_num(old->val);
			while(!cur && ++bucket < ht->bucket.size()){
				cur = ht->buckets[bucket];
			}
			return *this;
		}
	}
	iterator operator++(int){
		iterator tmp = *this;
		++*this;
		return tmp;
	}
	
	
	
	bool operator== (const iterator& it){return cur == it.cur;}
	bool operator!= (const iterator& it){return cur != it.cur;}
};

```

## 3.3 hashtable数据结构
### 3.3.1 模板参数
```
	hashtable的数据结构
	Value: 实值
	Key: 键值
	HashFcn: hash function类型
	ExtratKey: 从节点中取出键值的方法
	Equalkey : 判断键值相同与否的方法
	Alloc: 空间配置；
```

### 3.3.2 表格大小设计
- 开链法 
- SGI用质数来设计表格大小，先计算28 个质数的值存放起来方便直接访问；
- 提供了一个函数，用来查询“最近并大于某数”的质数

```
/**
	开链法的表格大小预先设计
	sgi stl 以质数来设计表格大小，现将28个质数计算好，用来被查询
	
*/

static const int __stl_num_primes = 28;

static const unsigned long __stl_prime_list[__stl_num_primes] = 
{
	53，97， 193， 389， 769，
	...
	...
	...  , 4294967291
}

// 一下用来找处28个质数中最近于n并大于n的质数
inline unsigned long __stl_next_prime(unsigned long n){
	const unsigned long* first = __stl_prime_list;
	const unsigned long* last = __stl_prime_list + __stl_num_primes;
	const unsigned long* pos = lower_bound(first,last,n); // 泛型算法，序列已排序
	return pos == last ? *(last -1):*pos;
}

size_type max_bucket_count() const{
	return __stl_prime_list[__stl_num_primes -1];
	// 最大值为4294967291
}
```


### 3.3.3 数据结构

```
template<class Value, class Key, class HashFcn,
			class ExtratKey,class Equalkey, class Alloc = alloc>
class hashtable{
public:
	typedef		HashFcn		hasher;   // 型别参数重新定义一个名字
	typedef		Equalkey	key_equal;	
	typedef		size_t		size_type;
	
	
private:

	// 以下三者都是函数对象 <stl_hash_fun.h>定义了数个
	// 标准型（如int,c-style string等）的hasher;
	hasher  hash;
	key_equal equals;
	ExtratKey  get_key;
	
	typedef		__hashtable_node<Value> node;
	typedef		simple_alloc<node, Alloc> node_allocator;
	
	
	vector<node*, Alloc> buckets;  // vector完成
	size_type	num_elements;
	
public:
	// bucket 个数 即 buckets vector的大小
	size_type  bucket_count() const {return buckets.size();}
	
...
```

### 3.3.4  构造函数
- hashtable没有默认构造函数,其中一个构造函数为：

```
	
protected:
	node* new_node(const value_type& obj){
		node* n = node_allocator::allocate();
		n->next = 0;
		__STL_TRY{
			construct(&n->val, obj);
			return n;
		}
		__STL_UNWIND(node_allocator::deallocate(n));
	}
	
	
	void delete_node(node* n){
		destroy(&n->val);
		node_allocator::deallocate(n);
	}
	
	// 初始构造 n个bunckets
	void iniitalize_buckets(size_type n){
		const size_type n_buckets = next_size(n);
		buckets.reserve(n_buckets);
		buckets.insert(buckets.end(),n_buckets,(node*)0);
		num_elements = 0;
	}
	
	// 就会最接近于n并且大于n的质数
	size_type next_size(size_type n) const {return __stl_next_prime(n);}
	

	
public:
	// hashtable没有默认构造函数
	hashtable<size_type n, const HashFcn& hf, const Equalkey& eql)
		:hash(hf),equals(eql),get_key(ExtratKey()),num_elements(0){
			iniitalize_buckets(n);
		}
```


### 3.3.5  插入操作和表格重建

- 当客户端插入元素的时候，会判断是否需要重建表格

- resize() ：表格重建判断

```
// 函数判断是否需要重建表示，如果不要，立刻返回，如果需要，就动手
	void resize(size_type num_elements_hint){
		// 判断方法： 元素个数和bucker vector个数比较
		// 如果前者大于后者，就重建
		// 每个list的最大容量 和bucket vector大小相同
		const size_type old_n = buckets.size();
		if(num_elements_hint > old_n){   // 需要重新配置
			const size_type n = next_size(num_elements_hint);  // 新大小
			if(n > old_n){
				vector<node*, Alloc>tmp(n, (node*)0);  // 设立新的buckets;
				__STL_TRY{
					// 处理旧的bucket
					for(size_type bucket = 0; bucket < old_n;++bucket){
						node* first = buckets[bucket];
						// 处理每个bucket的list
						while(first){
							// 寻找节点落在哪一个bucket
							size_type new_bucket =  bkt_num(first->val,n);
							// (1) 令旧的bucket 指向下一个节点
							buckets[bucket] = first->next;
							//(2)(3)将当前节点插入到新bucket中,前插法
							first->next = tmp[new_bucket];
							tmp[new_bucket] = first;
							//(4) 回到旧的bucket所指的list,准备处理下一个节点
							first = buckets[bucket];
						}
					}
					buckets.swap(tmp);  // vector::swap, 新旧两个bucket对换
					// 对换后，如果大小不一样，大的会变小，小的会变大
					// 离开后tmp释放
				}
			}
  			
		}
		
	}
	
```

- 插入操作：分为两种，一种允许重复，一种不允许重复
  - `insert_unique_noresize` ：不允许重复
  - `insert_equal_noresize` :允许节点重复

- 第一种情况

```
	// 插入元素，不允许重复
	pair<iterator, bool> insert_unique(const value_types obj){
		resize(num_elements +1);  // 判断是否需要重建表格，如果需要就重新扩充
		return insert_unique_noresize(obj);
	}
```
```
    // 在不要建表格的情况下插入新节点，键值不允许重复
	pair<iterator,bool> insert_unique_noresize(const value_type& obj){
		const size_type n = bkt_num(obj); // 决定obj的bucket位置
		node* first = buckets[n];
		
		// 如果buckets[n]已经被占用，此时first将不为0
		// 迭代到list最尾节点
		for(node* cur = first; cur; cur = cur->next){
			// 如果发现存在相同键值，就返回，不插入
			if(equals(get_key(cur->val), get_key(obj))
				return pair<iterator,bool>(iterator(cur,this),false);
		}
		
		node* tmp = new_node(obj);  // 产生新节点
		tmp->next = first;
		buckets[n] = tmp ;   // 前插法
		++num_elements;
		return pair<iterator,bool> (iterator(tmp,this),true);
	}
	
```

- 第二种情况

```
	
	// 插入元素，允许重复
	iterator insert_equal(const value_type& obj){
		resize(num_elements + 1);  // 判断是否重建表格
		return insert_equal_noresize(obj);
	}
	
```

```
	iterator insert_equal_noresize(const value_type& obj){
		const size_type n = bkt_num(obj);
		node* first = buckets[n] ; 
		
		for(node* cur = first; cur; cur = cur->next){
			if(equals(get_key(cur->val),get_key(obj)){
				// list中存在相同键值的实值，就马上插入，在返回
				node* tmp = new_node(obj);  // 新节点
				tmp->next = cur->next;
				cur->next = tmp;
				++num_elements;
				return iterator(tmp, this);  // 返回
			}
		}
		
		// 未发现重复值，到达链表尾端
		node* tmp = new_node(obj);  // 新节点
		tmp->next = first;
		buckets[n] = tmp;
		++num_elements;
		return iterator(tmp, this);  // 返回
	}
```

### 3.3.6 复制和清除

- 清除 `clear()`

```
	void clear(){  // buckets空间为释放
		// 对每一个bucket
		for(size_type i=0; i<buckets.size(); ++i){
			node* cur = bucket[i];
			// 将list每一个节点删除
			while(cur != 0){
				node* next = cur->next;
				delete_node(cur);
				cur = next;
			}
			bucket[i] = 0;  // bucket为NULL
		}
		num_elements = 0;  // 总数为0
	}
```



- 复制

```
	void copy_from(const hashtable& ht){
		// 先清除自己的buckets vector,调用vector::clear()
		buckets.clear();
		// 如果自己的空间大于对方，就不动，小于的话，增大
		buckets.reserve(ht.buckets.size());
		// 此时buckets为空，end()就是起头
		buckets.insert(buckets.end(),ht.buckets.size(), (node*)0);
		__STL_TRY{
			// 每个bucket
			for(size_type i=0 ;i<ht.buckets.size(); ++i){
				// 每个bucket 开头
				if(const node* cur = ht.bucket[i]) {
					node* cp = new_node(cur->val);
					buckets[i] = cp;
				
				
					// 每个bucket的list
					for(node* next = cur->next; next; cur =next, next = cur->next){
						cop ->next = new_node(next->val);
						cp = cp->next;
					}
				}	
			}
			num_elements = ht.num_elements;
		}
		__STL_UNWIND(clear());
	}

```


### 3.3.7 判断元素落脚

```
		
public:
	// 计算 元素的键值,即hashtable上的bucket序号
	
	// 版本1： 接受实值和buckets个数
	size_type bkt_num(const value_type& obj, size_t n) const{
		return bkt_num_key(get_key(obj),n);  // 版本4
	}
	// 版本2： 只接受实值value
	size_type bkt_num(const value_type& obj) const{
		return bkt_num_key(get_key(obj));   // 版本3
	}

	// 版本3： 只接受键值
	size_type bkt_num(const key_type& key) const{
		return bkt_num_key(key,buckets.size());   // 版本4
	}

	// 版本4： 只接受键值和buckets个数
	size_type bkt_num(const key_type& key，size_t n) const{
		return hash(key) % n; // SGI内建 hash()
	}
```



# 4. hashtable的hash function
- SGI的内置hash函数只针对了char,int,long等整数型别，而且都只是返回原值
- 只对`const char*`做了一个转换
- 因此无法处理string,double ,float等型别，
- 用户必须自定义hash function


```
/**
 hash function类型
 <stl_hash_fun.h>
*/




template<class Key>
struct hash{
	
	
};

inline size_t __stl_hash_string(const char* s){
	unsigned long  h =0;
	for(;*s;++s)
		h = 5*h + *s;
	return size_t(h);
}


// __STL_TAMPLATE_NULL 定义为template<> 在;<stl_config.h>中

__STL_TAMPLATE_NULL
struct hash<char*>{
	size_t operator()(char* s){
		return __stl_hash_string(s);
	}
}



__STL_TAMPLATE_NULL
struct hash<const char*>{
	size_t operator()(const char* s){
		return __stl_hash_string(s);
	}
}



__STL_TAMPLATE_NULL
struct hash<char>{
	size_t operator()(char s){
		return s;
	}
}


__STL_TAMPLATE_NULL
struct hash<unsigned char>{
	size_t operator()(unsigned char s){
		return s;
	}
}

__STL_TAMPLATE_NULL
struct hash<signed char>{
	size_t operator()(unsigned char s){
		return s;
	}
}

__STL_TAMPLATE_NULL
struct hash<short>{
	size_t operator()(short s){
		return s;
	}
}

__STL_TAMPLATE_NULL
struct hash<unsigned short>{
	size_t operator()(unsigned short s){
		return s;
	}
}


__STL_TAMPLATE_NULL
struct hash<int>{
	size_t operator()(int s){
		return s;
	}
}


__STL_TAMPLATE_NULL
struct hash<unsigned int>{
	size_t operator()( unsigned int s){
		return s;
	}
}


__STL_TAMPLATE_NULL
struct hash<long>{
	size_t operator()(long s){
		return s;
	}
}


__STL_TAMPLATE_NULL
struct hash<unsigned long>{
	size_t operator()( unsigned long s){
		return s;
	}
}

```

---

