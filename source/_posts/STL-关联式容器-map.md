---
title: STL_关联式容器_map
date: 2018-07-07 18:08:35
tags:
	- STL
	- Associative Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第五章
---

# 1 概述
- map的元素都是pair
- map 不允许两个元素拥有相同的键值；


# 2. map
```
template< class T1, class T2>
struct pair{
	typedef T1 first_type;
	typedef T2 seconde_type;
	
	T1 first;
	T2 second;
	
	
	pair():first(T1()),second(T2()){}
	
	pair(const T1& a, const T2& b):first(a), second(b){}
	
};


template<class Key, class T,
		class Compare = less<Key>,
		class Alloc = alloc>
class map{
public:
	typedef	Key key_type;  
	typedef	T data_type;
	typedef T mapped_type;
	typedef pair<const Key, T> value_type;  // 元素型别
	typedef	Compare key_compare;
	
	// 比较函数
	class value_compare: binary_function<value_type, value_type,bool>{
		friend class  map<Key,T, Compare, Alloc>;
		protected:
			Compare comp;
			value_compare(Compare c):comp(c){}
		public:
			bool operator()(const value_type& x, const value_type& y) const{
				return comp(x.first, y.first);
			}
	};
	
	
private:
	rep_type t;
	
	
public:
	typedef	 typename  rep_type::pointer pointer;
	typedef	 typename  rep_type::const_pointer const_pointer;
	typedef	 typename  rep_type::reference reference;
	typedef	 typename  rep_type::const_reference const_reference;
	typedef	 typename  rep_type::iterator  iterator;
	// 以上iterator定义为const，表示不允许用户执行写入操作
	
	typedef	 typename  rep_type::const_iterator	const_iterator;
	typedef	 typename  rep_type::reverse_iterator reverse_iterator;
	typedef  typename  rep_type::const_reverse_iterator const_reverse_iterator;
	typedef	 typename  rep_type::size_type	size_type;
	typedef	 typename  rep_type::difference_type  difference_type;
	
	
	// 构造函数
	
	map():t(Compare()){}
	explicit map(const Compare& comp):t(comp){}
	
	template<class InputIterator>
	map(InputIterator first, InputIterator last):t(Compare()){
		t.insert_unique(first,last);
	}
	
	template<class InputIterator>
	map(InputIterator first, InputIterator last, const Compare& comp):t(comp){
		t.insert_unique(first,last);
	}
	
	map(const map<Key,T, Compare,Alloc>& x):t(x.t){}
	
	// 赋值符
	map<Key,T, Compare,Alloc>& operator=(const map<Key,T,Compare,Alloc>& x){
		t = x.t;
		return *this;
	}
	
	// 访问器
	key_compare key_comp() const(return t.key_comp(););
	value_compare value_comp() const{return value_compare(t.key_comp());}
	iterator begin() {return t.begin();}
	const_iterator begin()const{return t.begin();}
	iterator end() {return t.begin();}
	const_iterator end() const{return t.end();}
	reverse_iterator rbegin() {return t.rbegin();}
	const_reverse_iterator rbegin() const {return t.rbegin();}
	reverse_iterator rend() {return t.rend();}
	const_reverse_iterator rend() const{return t.rend();}
	
	bool empty() const {return t.empty();}
	size_type size() const{return t.size();}
	size_type max_size() const{return t.max_size();}
	
	T& operator[](const key_type& k){
		return(*((insert(value_type(k,T()))).first)).second;
	}
	
	void swap(map<Key,T,Compare,Alloc>& x){t.swap(x.t);}
	
	
	// 插入。删除
	// 插入、 删除操作

	pair<iterator,bool> insert(const value_type& x){
		return t.insert_unique(x);
	}
		
	iterator insert(iterator position, const value_type& x){
		return t.insert_unique(position,x);
	}
	
	template<class InputIterator>
	void insert(InputIterator first, InputIterator last){
		t.insert_unique(first, last);
	}
	
	void erase(iterator position){
		t.erase(position);
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
	
	
	iterator find(const key_type& x) {return t.find(x);}	
	const_iterator find(const key_type& x) const{return t.find(x);}
	size_type count(const key_type& x)const{return t.count(x);}
	iterator lower_bound(const key_type& x)const{
		return t.lower_bound(x);
	}
	iterator upper_bound(const key_type& x)const{
		return t.upper_bound(x);
	}
	pair<iterator,iterator> equal_range(const key_type& x){
		return t.equal_range(x);
		
	}
	
	pair<const_iterator,const_iterator> equal_range(const key_type& x)const{
		return t.equal_range(x);
	}
	
	// 以下的 __STL_NULL_TMPL_ARGS 被定义为<>
	friend bool operator==__STL_NULL_TMPL_ARGS(const map&, const map&);
	friend bool operator< __STL_NULL_TMPL_ARGS(cosnt map&, const map&);
};


template<class Key, class T, class Compare, class Alloc>
inline bool operator==(const map<Key,T,Compare,Alloc>& x,
						const set<Key,T, Compare,Alloc>& y){
	return x.t == y.t;						
}



template<class Key,class T, class Compare, class Alloc>
inline bool operator<(const set<Key,T,Compare,Alloc>& x,
						const set<Key,T,Compare,Alloc>& y){
	return x.t < y.t;						
}




```

# 3 multimap

```

/**

	multimap
*/

template<class Key,class T,class Compare= less<Key>
			class Alloc = alloc>
class multimap{
public:
	// typedefs 
	...  // 和set一样
	
	template<class InputIterator>
	map(InputIterator first, InputIterator last):t(Compare()){
		t.insert_equal(first,last);
	}
	
	template<class InputIterator>
	map(InputIterator first, InputIterator last, const Compare& comp):t(comp){
		t.insert_equal(first,last);
	}
	
	
	...// 和set一样
	
	// insert
	
	
	insert insert(const value_type& x){
		return t.insert_equal(x);
	}
		
	iterator insert(iterator position, const value_type& x){
		return t.insert_equal(position,x);
	}
	
	template<class InputIterator>
	void insert(InputIterator first, InputIterator last){
		t.insert_equal(first, last);
	}
	
	... // 其他和map一样
};
```


---

