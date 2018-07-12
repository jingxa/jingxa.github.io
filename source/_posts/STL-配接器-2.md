---
title: STL_适配接器_2
date: 2018-07-12 21:36:37
tags:
	- STL
	- Iterator Adapter
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第八章
---

# 1 概述
- bind 
- negate
- compose
- 修饰(使他成为一个仿函数)

接口：
- `<functional>`
- SGI STL`<stl_function.h>`

【注意】：
- 所有期待获得配节能力的组件，本身必须是可配接的；
- 一元仿函数必须继承自`unary_function`
- 二元仿函数必须继承自`binary_function`
- 成员函数必须以`mem_fun`处理过
- 一般函数必须以`ptr_fun`处理过


![](https://upload-images.jianshu.io/upload_images/5361608-9496865ef7ffb463.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-c1f1605f6072ba0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---


# 2 实现

## 2.1 返回值得逻辑否定not1,not2

```
/**
	1. 返回值得逻辑否定not1,not2
	--- not1: 仿函数只有一个参数
	--- not2: 仿函数有两个参数
*/

template<class Predicate>
class unary_negate: public unary_function<typename Predicate::argument_type,bool>{
protected:
	Predicate pred;  // 内部成员；
public:
	explicit unary_negate(const Predicate& x):pred(x){}
	
	bool operator()(const typename Predicate::argument_type& x)const{
		return !pred(x);
	}
	
};

// 辅助函数
template<class Predicate>
inline unary_negate<Predicate> not1(const Predicate& pred){
	return unary_negate<Predicate>(pred);
}



// Binary Predicate 的逻辑负值


template<class Predicate>
class binary_negate: public binary_function<typename Predicate::first_argument_type,
											typename Predicate::second_argument_type,
											bool>{
protected:
	Predicate pred;  // 内部成员；
public:
	explicit binary_negate(const Predicate& x):pred(x){}
	
	bool operator()(const typename Predicate::first_argument_type& x,
					const typename Predicate::second_argument_type& y)const{
		return !pred(x,y);
	}
	
};

// 辅助函数
template<class Predicate>
inline binary_negate<Predicate> not2(const Predicate& pred){
	return binary_negate<Predicate>(pred);
}


```

## 2.2 bind1st, bind2nd对参数进行绑定

```
/**
	2. bind1st, bind2nd对参数进行绑定
	--- bind1st(...)会产生一个binder1st<operation>(...)对象；	
	--- bind2nd(...)会产生一个binder2nd<operation>(...)对象；
*/


// binder1st
template<class Operation>
class binder1st: public unary_function<typename Operation::second_argument_type,
										typename Operation::result_type>{
protected:
	Operation op;  // 内部成员
	typename Operation::first_argument_type value; // 内部成员
	
public:
	// ctor
	binder1st(const Operation& x, 
		const typename Operation::first_argument_type& y):op(x),value(y){
			
		}
		
	typename Operation::result_type
		operator()(const typename Operation::second_argument_type& x)const{
			return op(value,x);
		}											
};

// 辅助函数
template<class Operation , class T>
inline binder1st<Operation> bind1st(const Operation&operation& op, const T& x){
	typedef typename Operation::first_argument_type arg1_value;
	// 把 x转为第一参数类型
	return binder1st<Operation>(op, arg1_value(x));
}



// binder2nd
template<class Operation>
class binder2nd: public unary_function<typename Operation::first_argument_type,
										typename Operation::result_type>{
protected:
	Operation op;  // 内部成员
	typename Operation::second_argument_type value; // 内部成员
	
public:
	// ctor
	binder1st(const Operation& x, 
		const typename Operation::second_argument_type& y):op(x),value(y){
			
		}
		
	typename Operation::result_type
		operator()(const typename Operation::first_argument_type& x)const{
			return op(x,value);
		}											
};

// 辅助函数 bind2nd<Op>
template<class Operation , class T>
inline binder2nd<Operation> bind2nd(const Operation&operation& op, const T& x){
	typedef typename Operation::second_argument_type arg1_value;
	// 把 x转为第二参数类型
	return binder2nd<Operation>(op, arg1_value(x));
}

```

## 2.3 用于函数合成： compose1, compose2

```
/**
	3. 用于函数合成： compose1, compose2

*/

template<class Operation1, class Operation2>
class unary_compose:public unary_function<typename Operation2:argument_type,
										typename Operation1::result_type>{
protected:
	Operation1 op1;
	Operation2 op2;  // 内部成员
public:
	// ctor
	unary_compose(const Operation1& x, const Operation2& y)
		:op1(x),op2(y){}
		
	typename Operation1::result_type
		operator()(const typename Operation::argument_type& x)const{
			return op1(op2(x)); // 函数合成
		}
};

// 辅助函数
template<class Operation1, Operation2>
inline unary_compose<Operation1,Operation2>
compose1(const Operation1& op1, const Operation2& op2){
	return unary_compose<Operation1,Operation2>(op1,op2);
}




// binary_compose
template<class Operation1, class Operation2,class Operation3>
class binary_compose:public unary_function<typename Operation2:argument_type,
										typename Operation1::result_type>{
protected:
	Operation1 op1;
	Operation2 op2;  // 内部成员
	Operation3 op3;
public:
	// ctor
	binary_compose(const Operation1& x, const Operation2& y,Operation& z)
		:op1(x),op2(y),op3(z){}
		
	typename Operation1::result_type
		operator()(const typename Operation::argument_type& x)const{
			return op1(op2(x), op3(x)); // 函数合成
		}
};

// binary_compose<Op1,Op2,Op3>
template<class Operation1, Operation2,Operation3>
inline binary_compose<Operation1,Operation2,Operation3>
compose2(const Operation1& op1, const Operation2& op2,const Operation3& op3){
	return binary_compose<Operation1,Operation2,Operation3>(op1,op2,op3);
}


```

## 2.4 用于函数指针：ptr_fun

```
/**
	4. 用于函数指针：ptr_fun
	--- pointer_to_unary_function
	--- pointer_to_binary_function
*/
// pointer_to_unary_function
template<class Arg, class Result>
class pointer_to_unary_function：public unary_function<Arg,Result>
{
protected:
	Result(*ptr)(Arg); // 内部成员，一个函数指针
public:
	// ctors
	pointer_to_unary_function(){}
	
	explicit pointer_to_unary_function(Result(*x)(Arg)):ptr(x){}
	
	// 通过函数指针执行函数
	Result operator()(Arg x)const{return ptr(x);}
};


// 辅助函数
template<class Arg,class Result>
inline pointer_to_unary_function<Arg,Result>
ptr_fun(Result (*x)(Arg)){
	return pointer_to_unary_function<Arg,Result>(x);
}





// pointer_to_binary_function
template<class Arg1,Arg2, class Result>
class pointer_to_binary_function:public binary_function<Arg1,Arg2,Result>
{
protected:
	Result(*ptr)(Arg1, Arg2); // 内部成员，一个函数指针
public:
	// ctors
	pointer_to_binary_function(){}
	
	explicit pointer_to_binary_function(Result(*x)(Arg1,Arg2)):ptr(x){}
	
	// 通过函数指针执行函数
	Result operator()(Arg1 x, Arg2 y)const{return ptr(x, y);}
};


// 辅助函数
template<class Arg1,class Arg2,class Result>
inline pointer_to_binary_function<Arg1,Arg2,Result>
ptr_fun(Result (*x)(Arg1,Arg2)){
	return pointer_to_binary_function<Arg1,Arg2,Result>(x);
}

```

## 2.5 用于成员函数指针：mem_fun, mem_fun_ref

```
/**
	5. 用于成员函数指针：mem_fun, mem_fun_ref
	-- 对于每个Adapter function objects：每个对象有三种变化：
	(1) 无任何参数 vs "一个参数"
	(2) “通过pointer调用” vs “通过reference 调用”
	(3) "const 成员函数" vs "non-const 成员函数"
	// 所有复杂都在对象内部，直接调用上层函数 mem_fun  和 mem_fun_ref
*/

// 1.无任何参数，通过pointer调用， “non-const 成员函数”
template<class S, class T>
class mem_fun_t: public unary_function<T* , S>
{
public:
	explicit mem_fun_t(S (T::*pf()):f(pf){} // 记录下来
	S operator()(T* p) {return (p->*f)();}  // 调用”
	
private:
	S (T::*f)();  // 内部成员 ， 成员对象函数 指针
	
};

// 2.无任何参数， 通过 pointer调用， const 成员函数
template<class S, class T>
class const_mem_fun_t: public unary_function<T* , S>
{
public:
	explicit mem_fun_t(S (T::*pf() const):f(pf){} // 记录下来
	S operator()(const T* p) const{return (p->*f)();}  // 调用”
	
private:
	S (T::*f)() const;  // 内部成员 ， 成员对象函数 指针
	
};

// 3.无任何参数， 通过reference调用， non-const成员函数

template<class S, class T>
class mem_fun_ref_t: public unary_function<T , S>
{
public:
	explicit mem_fun_ref_t(S (T::*pf()):f(pf){} // 记录下来
	S operator()( T& r) {return (r.*f)();}  // 调用”
	
private:
	S (T::*f)() ;  // 内部成员 ， 成员对象函数 指针
	
};


// 4.无任何参数， 通过reference调用， const成员函数

template<class S, class T>
class const_mem_fun_ref_t: public unary_function<T , S>
{
public:
	explicit const_mem_fun_ref_t(S (T::*pf() const):f(pf){} // 记录下来
	S operator()( const T& r) const{return (r.*f)();}  // 调用”
	
private:
	S (T::*f)() const;  // 内部成员 ， 成员对象函数 指针
	
};


// 5.有一个参数，通过pointer调用， “non-const 成员函数”
template<class S, class T,class A>
class mem_fun1_t: public unary_function<T*,A , S>
{
public:
	explicit mem_fun1_t(S (T::*pf(A)):f(pf){} // 记录下来
	S operator()(T* p,A x){return (p->*f)(x);}  // 调用”
	
private:
	S (T::*f)(A);  // 内部成员 ， 成员对象函数 指针
	
};




// 6. 有一个参数，通过pointer调用， “const 成员函数”

template<class S, class T,class A>
class const_mem_fun1_t: public unary_function<T*,A , S>
{
public:
	explicit const_mem_fun1_t(S (T::*pf(A)):f(pf){} // 记录下来
	S operator()(const T* p, A x) const{return (p->*f)(x);}  // 调用”
	
private:
	S (T::*f)(A) const;  // 内部成员 ， 成员对象函数 指针
	
};



// 7. 有一个参数，通过reference调用， “non-const 成员函数”
template<class S, class T,class A>
class mem_fun1_ref_t: public unary_function<T, A , S>
{
public:
	explicit mem_fun1_ref_t(S (T::*pf(A)):f(pf){} // 记录下来
	S operator()(T& r,A x){return (r.*f)(x);}  // 调用”
	
private:
	S (T::*f)(A);  // 内部成员 ， 成员对象函数 指针
	
};


// 8.有一个参数，通过reference调用， “const 成员函数”
template<class S, class T,class A>
class const_mem_fun1_ref_t: public unary_function<T*,A , S>
{
public:
	explicit const_mem_fun1_ref_t(S (T::*pf(A)):f(pf){} // 记录下来
	S operator()(const T& r, A x) const{return (r.*f)(x);}  // 调用”
	
private:
	S (T::*f)(A) const;  // 内部成员 ， 成员对象函数 指针
	
};



// 辅助函数

// mem_fun mem_fun_ref
template<class S, class T>
inline mem_fun_t<S,T> mem_fun(S T::*f()){
	return mem_fun_t<S,T>(f);
}

template<class S, class T>
inline const_mem_fun_t<S,T> mem_fun(S T::*f() const){
	return const_mem_fun_t<S,T>(f);
}

template<class S, class T>
inline mem_fun_ref_t<S,T> mem_fun_ref(S T::*f() ){
	return mem_fun_ref_t<S,T>(f);
}

template<class S, class T>
inline const_mem_fun_ref_t<S,T> mem_fun_ref(S T::*f() const){
	return const_mem_fun_ref_t<S,T>(f);
}



// 带参数
template< class S, class T, class A>
inline mem_fun1_t<S,T,A> mem_fun(S (T::*f)(A)){
	return mem_fun1_t<S,T,A>(f);
}


template< class S, class T, class A>
inline const_mem_fun1_t<S,T,A> mem_fun(S (T::*f)(A) const){
	return const_mem_fun1_t<S,T,A>(f);
}

template< class S, class T, class A>
inline mem_fun1_ref_t<S,T,A> mem_fun(S (T::*f)(A)){
	return mem_fun1_ref_t<S,T,A>(f);
}


template< class S, class T, class A>
inline const_mem_fun1_ref_t<S,T,A> mem_fun(S (T::*f)(A) const){
	return const_mem_fun1_ref_t<S,T,A>(f);
}


```


















