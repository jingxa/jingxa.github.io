---
title: Inside_Cplusplus_构造_default
date: 2018-07-21 19:23:57
tags:
	- C++
	- constructor 
categories:
	- C++
---
参考资料：
> 深度探索c++对象模型第二章
---

# 1. 默认构造函数
- 默认构造函数在被需要的时被编译器产出来；
- 编译器需要的时候，才产生默认构造函数；
  - 编译器合成的构造函数只执行编译器所需要的行动，不会将data成员初始化；

【注意】：当class包含多个constructor，但是没有default constructor，编译器不会产生default constructor，而是扩展每个constructor；


编译器产生的默认构造函数分为：

- 是(trivial)平凡的，即没有任何用处的；
- non-trivial:有用的


一下讨论产生non-trivial 默认构造函数的情况：

## 1.1 带有“默认构造函数”的member class 对象
- 如果一个class，包含一个`member object` ， 后者有`默认构造函数`，那么这个class的合成的默认构造函数是`non-trivial`的；

例子：
```
/****************** 例子1 ****************/
class Foo {
public:
	Foo();
	Foo(int);
	...
};

class Bar{
public:
	Foo foo;
	char *str;
	
};


// 编译器合成的版本
// 伪代码
Bar::Bar(){
		foo.Foo::FOO();  // 编译器代码
}


// 为了代码正确执行
// str 必须初始化， 程序员的代码
Bar::Bar(){
	str =0;  // explicit user code
}

// 编译器扩展版本
Bar::Bar(){
	foo.Foo::FOO();  // 编译器代码
	str =0;  // explicit user code
}



/************* 例子2 ：   ***************/
class Dopey{
public:
	Dopey();
	...
};

class Sneezy{
public:
	Sneezy();
	Sneezy(int);
	...
};

class Bashful{
public:
	Bashful();
	...
};

Class Snow_white{
public:
	Dopey dopey;
	Sneezy sneezy;
	Bashful bashful;
	...

private:
	int mumble;
};


// 如果未定义默认函数
// 编译器合成版本
Snow_white::Snow_white(){
	dopey.Dopey::Dopey();
	sneezy.Sneezy::Sneezy();
	bashful.Bashful::Bashful();
	
}

/* 程序员的代码 */
Snow_white::Snow_white():sneezy(1024){
	mumble = 2028;
}

// 编译器扩充为：
/* 程序员的代码 */
Snow_white::Snow_white():sneezy(1024){
	
	// 插入代码
	dopey.Dopey::Dopey();
	sneezy.Sneezy::Sneezy(1024);
	bashful.Bashful::Bashful();
	mumble = 2028;
}

```

##  1.2 带有 “default constructor ”的Base class 

	如果一个没有任何构造函数的class 派生带有“default constructor”
	的base class。这个派生类的合成默认构造函数为non-trivial

## 1.3 "带有一个virtual Function"的 Class 
	- 1. class  声明或者继承一个 virtual Function;
	- 2. class 派生子一个继承串链，其中有一个或者多个 virtual base classes;

	如果程序员没有声明constructor， 编译器就会合成一个
	default constructor;
	
	【编译器动作】
	- 1. virtual function table(vtpl)会被编译器产生出来，
		内放	virtual function  的地址；
	- 2. 在每一个class object中，一个额外的pointer member（vptr)
		会被编译器合成出来，内含相关的class vbtl的地址；

## 1.4 "带有一个 virtual base class" 的class
```
编译器会安插“允许每一个virtual base class 的执行器存取操作”的代码；
```

## 1.5 总结

	1. 在合成的default constructor中，只有base class subobjects和
		member class objects被初始化
	2. 其他的non-static data member, 如整数，整数指针，数组等不会被
		初始化，应该是程序员提供；


---

