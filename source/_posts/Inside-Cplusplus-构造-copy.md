---
title: Inside_Cplusplus_构造_copy
date: 2018-07-22 15:23:46
tags:
	- C++
	- constructor 
categories:
	- C++
---
参考资料：
> 深度探索c++对象模型第二章
---

# 概述
- Deafault copy constructor

有三种情况，会以一个object的内容作为另一个class ojbject的初值；
- 赋值操作符

```
class x{...};
X x;

X xx = x; 
```

- 作为参数

```
void foo(X x);

X xx; 
foo(xx);   // 产生copy
```

- 函数回传

```
X foo(){
...
X xx;
 return xx;
}
```
这几种情况会导致临时object的产生；

# 1 默认以成员为单位的初始化（Default Memberwise Initialization）和位逐次拷贝(Bitwise Copy Semantics)

- 从概念上讲， 对于一个class X,“Default Memberwise Initialization”是被一个copy constructor 实现的；
- 一个良好的编译器可以为大部分object产生`bitwise copies `, 因为它们有`bitwise copy semantics`
- 因此，`default constructor` 和`copy constructor`在必要的时候才由编译器产生出来；

## 1.1 位逐次拷贝
- 如果一个class 展现`bitwise copy semantics`，就不需要合成一个默认构造函数；

```
// 类的成员为POD类型
class Word{
 public:
  Word(const char*);  
  ~Word(){delete[] str;}
...
private:
  int cnt;
  char* str;
};
```

在这个类中，并不需要合成一个默认的copy constructor， 因为上述声明占星了“default copy semantics”；


## 1.2 Default memberwise Initialization
- 当一个class 未展现出`bitwise copy semantics`时，必须合成一个copy constructor；

-  使用`default memberwise Initializatioon`方式构造； 

例子：

```
class Word{
 public:
  Word(const string&);  
  ~Word(){delete[] str;}
...
private:
  int cnt;
  string  str;
};
```

- 其中，string class 拥有explicit copy constructor;

编译器合成一个copy constructor， 以便调用string object的copy constructor;


```
// c++ 伪代码
// 编译合成版本
inline Word::Word( const Word& wd){
  str.string::string(wd.str);
  cnt = wd.cnt;
}
```

- 合成的copy constructor中， 整数，指针，数组等nonclass members 也被复制；

## 1.3 不要 Bitwise Copy Semantics
有四种情况：

- 1. 当class 内部的一个member object 含有copy constructor(不论是声明（如string）者是编译器合成(如Word)的时候);

- 2. 当class 继承自一个base class，而基类含有一个copy constructor的时候；

- 3 当class 声明了一个或多个virtual functions；

- 4. 当class 派生自一个继承链，其中有一个或多个virtual base classes；


# 2. Virtual Table 的指针


```
class ZooAnimal{
public:
	ZooAnimal();
	virtual  ~ZooAnimal();
	
	virtual void animate();
	virtual void draw();
	// ...
private:
	// animate()和draw()需要的数据
};

class Bear: public ZooAnimal{
public:
	Bear();
	virtual void animate();
	virtual void draw();
	virtual void dance();
	// ...
private:
	// 几个函数的数据
};
```

## 2.1 同类型的copy
- 当ZooAnimal利用另一个 ZooAnimal 作为初值，Bear 以另一个 Bear作为初值，都可以直接靠“ bitwise copy semantics ”完成
	
- yoqi被默认构造函数初始化， vptr被设定为Bear class 的虚函数表，而winnie 拷贝 yoqi 的vptr是安全的；

```c++
Bear yoqi;
Bear winnie = yoqi;
```

![](https://upload-images.jianshu.io/upload_images/5361608-de8bc7bf83643fb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 2.2 派生和基类的copy
- 但是，当一个基类使用派生类的object 内容做初始化操作，franny 的vptr不能指向Bear的虚函数表，（如果使用“ bitwise copy”）导致错误；
	
- 因此，合成出来的 ZooAnimal copy constructor 会明确设定object的vptr指向ZooAnimal 的虚函数表，而不是直接从 右边的class object拷贝过来；


```
ZooAnimal franny = yoqi; // 这回发生切割行为；
```

![](https://upload-images.jianshu.io/upload_images/5361608-a717ba39b03919f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2.3 virtual base class subojbect

1. 一个`class object` 以另一个object作为初值，后者拥有`virtual base class subojbect`,会使 “`bitwise copy`”失效；
	   
- 编译器都虚拟继承的承诺： 
1. 让派生类对象中的“`virtual base class subojbect`位置”在执行器准备好；
2. “Bitwise copy”会破坏这个位置，所以编译器合成的copy constructor中使用memberwise 的初始化；

![](https://upload-images.jianshu.io/upload_images/5361608-b3e4baf97ebfb19f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```

/*
	编译器产生的代码：
	1. 调用 ZooAnimal 的默认构造函数，
	2. 将 Raccon 的vptr 初始化
	3. 并且在 Raccon 中定位出 ZooAnimal subojbect的位置
	这些代码安插在 Raccon 的两个constructor 中的最前部分；
*/

class Raccon: public virtual ZooAnimal{
public:
	Raccon(){ /*设定 private的初值*/}
	Raccon(int val){/* 初始值设定*/}
	// ..
private:
	// 所有数据
};
```

- [注意]
1. 如果 Raccon object利用另一个 Raccon object 作为初始值；那么 `bitwise copy `足够；
	
2. 但是如果使用 派生类来作为基类的初值，编译器必须合成一个copy constructor，为了使vptr指向正确的位置；然后对每一个members执行必要的`memberwise`的初始化；


---

