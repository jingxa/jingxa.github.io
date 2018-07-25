---
title: Inside_Cplusplus_data
date: 2018-07-25 11:26:00
tags:
	- C++
	- data member 
categories:
	- C++
---
参考资料：
> 深度探索c++对象模型第三章
---

# 概述
```
class A{};
class B: public virtual A{};
class C: public virtual A{};
class D: public B, public C{ char c;};

   cout<<"A:"<<sizeof(A)<<endl;
   cout<<"B:"<<sizeof(B)<<endl;
   cout<<"C:"<<sizeof(C)<<endl;
   cout<<"D:"<<sizeof(D)<<endl;
```
结果为：

```
A:1
B:4
C:4
D:12
```
- 一个空类，`sizeof(A) == 1 byte`（这是编译器安插的一个char,是这个class对象在内存中有独一的地址）

- 当虚拟继承一个类的时候，类中存放了指向`virtual base class subobject`的相关的指针(本编译器指针为4bytes):
  - 但是，B,C为什么为4 bytes不为5bytes ? 因为，指向virtual base class的指针占据了类的开头，有了成员，则不需要编译器安插一个char；

- Alignment的限制；在32位计算机上，为了使bus的效率达到最高，一般采取内存对齐的方式；如B,C刚好4bytes，

- 对于类D:
  - 一个`virtual base class subobject`只会在派生类中存在一份实体
  - 被大家共享的A只有一个实体， 大小为1bytes;(有了成员就会忽略)
  - B和C加起来为8bytes;
  - class D中有一个`char`  1byte;
因此，D中（两个指针，一个char），对齐后为12bytes;

如图中， X:为A, Y,Z为B,C;
![image.png](https://upload-images.jianshu.io/upload_images/5361608-9d9ed4f62b0d8c5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 1. data member 的绑定

```
typedef	int length;

class Point3d{
public:
	// length 被决议为global length
	void mumble(length val){_val = val;}
	length mumble(){return _val;}
	//...
private:

	// length 的声明要放在“本class 对它的第一个参考之前”
	// 这个length会导致前面的操作 非法
	typedef float length;
	length _val;
	// ...
};
```

# 2. data member 的布局
1. Nonstatic data member在class object 的
		同一access section（public, private, protected）中的排列顺序和声明顺序一样

```
// 例子

class Point3d{
public:
	// ..
private:
	float x;
	static List<Point3d*> *freeeList;
	float y;
	static const int chunksize = 250;
	float z;
	
};

 // 等价于
class Point3d{
public:
	// ..
private:
	float x;
	static List<Point3d*> *freeeList;
	
private:
	float y;
	static const int chunksize = 250;
	
private:
	float z;
	
}; 
```


# 3. data Member 的存取

对于一个
```
	Point3d origin, *pt = &origin;
	origin.x  = 0.0;
	pt -> x = 0.0;
	
```
这两种存取方式的差异
- 如果point3d是一个派生类，在其继承结构中有一个virtual base class，并且被存取的是一个virtual base class 中继承来的member时， 必须要等待执行器才能确定pt指向哪一种class type; 


- static data member 的存取
static data member 存放在 global data segment中， 取static data member 的地址得到的是一个指向static data的指针；


# 4. 继承

## 4.1 只继承没有多态

![](https://upload-images.jianshu.io/upload_images/5361608-581623f1f3ac1e04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-294830430c13f41e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 继承中的问题：
  - 把一个class 分解成多个层继承，导致class 膨胀；


![](https://upload-images.jianshu.io/upload_images/5361608-871ea6f095a60a46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于图中的类，我们可以定义三个层级的继承；

![](https://upload-images.jianshu.io/upload_images/5361608-bd4e21e54c753ffb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

继承的布局为：

![](https://upload-images.jianshu.io/upload_images/5361608-7af278ee7d673e6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如图所示，每个类由于内存对齐，有了很多的padding;

但是，如果把padding去掉，破坏“`base class subobject`”的完整性，导致的后果如下：

![](https://upload-images.jianshu.io/upload_images/5361608-aa0c26f7447bfd19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4.2 加上多态

### 4.2.1 多态继承
- 对于虚函数，类必须产生一个vptr指向虚函数表；


```
struct no_virts{
	int d1,d2;
	
};


class has_virts: public no_virts{
public:
	virtual void foo();
	// ...
private:
	int d3;
};
```

内存布局为：
![](https://upload-images.jianshu.io/upload_images/5361608-a85603d750f45665.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 4.2.2  vptr的内存布局
vptr的布局一般放在class object 的开头或者结尾；
- 结尾：能够保留`base class C struct`的对象布局，因而能够在c程序代码中使用；

- 前端： “在多重继承下， 通过指向class members的指针调用virtual function”，会有帮助；



###  4.2.3 多重继承

![](https://upload-images.jianshu.io/upload_images/5361608-9721f7d43e638aa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- `point2d` ,`Vertex`两个类含有virtual接口
内存布局为：

![](https://upload-images.jianshu.io/upload_images/5361608-57cc458f7cbe08d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4.3 虚拟继承

![](https://upload-images.jianshu.io/upload_images/5361608-e374d8b25ef6a23a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- `Vertex`和`Point3d`虚拟继承Point2d
- ·`Vertex3d`普通继承两个类；

![](https://upload-images.jianshu.io/upload_images/5361608-9f0e2f2fc058764f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 对于`virtual base class `：

【方法一】：
  - 在每个`vritual base class` 的派生类中加入一个指针，指向`virtual base class`
- 如图3.5a

【方法二】：
  - c++在`virtual function table` 中放置`virtual base class`的offset,`virtual function table`通过正负索引来判断；
  - 索引为正值，索引到虚函数
  - 索引为负值，索引到`virtual base class `的offsets

如图所示:
![](https://upload-images.jianshu.io/upload_images/5361608-aaa028bbf0650183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---

