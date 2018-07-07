---
title: STL_关联式容器_set
date: 2018-07-07 18:08:21
tags:
	- STL
	- Associative Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第五章
---


# 1 概述
- set 所有元素根据元素的键值自动被排序
- set的键值和实值都是同样的；
- 不允许两个元素有相同的键值；
- set<T>::iterator为const_iterator
- Rb_tree作为底层；

# 1. set

```
/**
	set源码摘录
*/

template<class Key, 
		class Compare = less <Key>,   // 缺省下采用递增排序
		class Alloc = alloc>
class set{
public:
	typedef Key	key_type;
	typedef	Key	value_type;
	typedef Compare	key_compare;
	typedef	Compare	value_compare;
	
private:
// 以下 identity 定义于<stl_function.h>
	/*
	template< class T>
	struct identity:public unary_function<T, T>{
		const T& operator()(const T& x){return x;}
	};
	*/
	typedef	rb_tree<key_type, value_type,identity<value_type>,key_compare,Alloc> rep_type;
	
	rep_type t;  // 采用红黑树表示set
	
public:
	typedef	 typename  rep_type::const_pointer pointer;
	typedef	 typename  rep_type::const_pointer const_pointer;
	typedef	 typename  rep_type::const_reference reference;
	typedef	 typename  rep_type::const_reference const_reference;
	typedef	 typename  rep_type::const_iterator  iterator;
	// 以上iterator定义为const，表示不允许用户执行写入操作
	
	typedef	 typename  rep_type::const_iterator	const_iterator;
	typedef	 typename  rep_type::const_reverse_iterator reverse_iterator;
	typedef  typename  rep_type::const_reverse_iterator const_reverse_iterator;
	typedef	 typename  rep_type::size_type	size_type;
	typedef	 typename  rep_type::difference_type  difference_type;
	
	// aollocattion // deallocation
	// set 使用rb_tree 的insert_unique()
	// multiset 使用 rb_tree的insert_equal()
	
    set():t(Compare()){}
	explicit set(const Compare& comp):t(comp){}
	
	
	template<class InputIterator>
	set(InputIterator first, InputIterator last):t(Compare())
	{
		t.insert_unique(first,last);
	}
	
	template<class InputIterator>
	set(InputIterator first, InputIterator last, const Compare& comp)
		:t(comp){
			t.insert_unique(first,last);
		}
	
	set(const set<Key,Compare,Alloc>& x):t(x.t){}
	
	// 赋值符
	set<Key, Compare,Alloc>& operator=(const set<Key,Compare,Alloc>& x){
		t = x.t;
		return *this;
	}
	
	
	// set 操作
	key_compare key_comp() const{return t.key_comp();}
	value_compare value_comp() const{{return t.key_comp();}
	
	iterator begin() const{return t.begin();}
	iterator end()	const{return t.end();}
	reverse_iterator rbegin() const{retur t.rbegin();}
	reverse_iterator rend() const{return t.rend();}
	bool empty() const{return t.empty();}
	size_type size() const {return t.size();}
	size_type max_size() const{return t.max_size();}
	void swap(set<Key,Compare,Alloc>&x){t.swap(x.t);}
	
	
	// 插入、 删除操作
	typedef pair<iterator,bool> pair_iterator_bool;
	pair<iterator,bool> insert(const value_type& x){
		pair<typename rep_type::iterator,bool> p = t.insert_unique(x);
		return pair<iterator,bool>(p.first,p.last);
	}
	
	
	iterator insert(iterator position, const value_type& x){
		typedef typename rep_type::iterator rep_iterator;
		return t.insert_unique((rep_iterator&)position,x);
	}
	
	template<class InputIterator>
	void insert(InputIterator first, InputIterator last){
		t.insert_unique(first, last);
	}
	
	void erase(iterator position){
		typedef typename rep_type::iterator rep_iterator;
		t.erase((rep_iterator&)position);
	}
	
	size_type erase(const key_type& x){
		return t.erase(x);
	}
	
	void erase(iterator first, iterator last){
		typedef typename rep_type::iterator rep_iterator;
		t.erase((rep_iterator&)first, (rep_iterator&)last);
	}
	
	void clear(){
		t.clear();
	}
	
	// set 操作
	iterator find(const key_type& x) const{return t.find(x);}
	size_type count(const key_type& x)const{return t.count(x);}
	iterator lower_bound(const key_type& x)const{
		return t.lower_bound(x);
	}
	iterator upper_bound(const key_type& x)const{
		return t.upper_bound(x);
	}
	pair<iterator,iterator> equal_range(const key_type& x)const{
		return t.equal_range(x);
		
	}
	
	// 以下的 __STL_NULL_TMPL_ARGS 被定义为<>
	friend bool operator==__STL_NULL_TMPL_ARGS(const set&, const set&);
	friend bool operator< __STL_NULL_TMPL_ARGS(cosnt set&, const set&);
		
};


template<class Key, class Compare, class Alloc>
inline bool operator==(const set<Key,Compare,Alloc>& x,
						const set<Key,Compare,Alloc>& y){
	return x.t == y.t;						
}



template<class Key, class Compare, class Alloc>
inline bool operator<(const set<Key,Compare,Alloc>& x,
						const set<Key,Compare,Alloc>& y){
	return x.t < y.t;						
}

```

# 2 multiset

```
/**
	multiset
*/

// set 不同之处
template<class Key, class Compare = less<Key>, class Alloc= alloc>
class multiset{
	
public:
// typedefs
... // 一样和set

//构造函数

	template<class InputIterator>
	multiset(InputIterator first,InputIterator last):t(Compare()){
		t.insert_equal(first,last);
	}
	
	template<class InputIterator>
	multiset(InputIterator first, InputIterator last, Compare& comp):t(comp){
		t.insert_equal(first,last);
	}

	...   // 与 set一样
	
	
	// 插入删除
	iterator insert(const value_type& x){
		return t.insert_equal(x);
	}
	
	iterator insert(iterator position, const value_type& x){
		typedef typename rep_type::iterator rep_iterator;
		return t.insert_equal((rep_iterator&)position,x);
	}
	
	template<class InputIterator>
	void insert(InputIterator first, InputIterator last){
		t.insert_equal(first,last);
	}
	
	... // 其他和set一样
	
};

···

---

