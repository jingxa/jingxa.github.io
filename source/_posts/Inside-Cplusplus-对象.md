---
title: Inside_Cplusplus_对象
date: 2018-07-21 19:23:31
tags:
	- C++
	- Object 
categories:
	- C++
---
参考资料：
> 深度探索c++对象模型第一章
---
# c++对象
- 成员函数：
  - 非内联： 只会诞生一个函数实体，不在object中
  - 内联： 在每一个使用者上产生一个函数实体；

# 1 c++对象模式
data member:
- static
- non-static

成员函数：
- static
- nonstatic
- virtual

## 1.1 c++对象模型
- 指定多个access sections,内含数据：处于用一个access section的数据，必定保证其声明次序出现内存布局中；

### 1.1.1 成员数据
- nonstatic 成员数据放在每一个class object中；
- static成员数据 放在所有的class object之外；

### 1.1.2 成员函数
- static和nonstatic 都放在class object之外；

对于虚函数：
- 1).每一个class产生一个virtual function 的指针，放在表格中，这个表格就是虚函数表（virtual table）`vbtl`
- 2). 每一个class添加一个指针，指向虚函数表 : 指针为vptr(设定和重置通过构造函数和析构函数完成)


![](https://upload-images.jianshu.io/upload_images/5361608-ca46784b6a7322d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 每一个class 关联的type_info 对象放在虚函数表的第一个表格；

## 1.2 多重继承
- 继承：每一个子类中都包含一个基类实体；
- 虚拟继承： 永远只会存在一个实体；

对于一个继承：
![](https://upload-images.jianshu.io/upload_images/5361608-c735d72fc5a53677.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 每一个派生类对象中都包含一个指向基类表的指针；
- 表中的每一个表格包含一个基类对象的指针；

![](https://upload-images.jianshu.io/upload_images/5361608-1d9df8b6eea3a7d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.3 对象的内存布局
- nonstatic data member
- 支持virtual而产生的负担
- alignment的需求的填补（可能存在members之间，也可能存在边界）

# 2. 对象的差异
## 2.1 指针的转型

- 对于一个指向某地址的void\*指针，只能知道指向的地址，但是不知道通过它操作的object的类型；因此，无法通过void\*指针操作object；

- 转型： 一种编译器指令；大部分情况并不改变指针的指向的真正地址，只是影响“被指出的内存的大小和其内容”的解释方式；

## 2.2 多态的指针
对于两个类：

![](https://upload-images.jianshu.io/upload_images/5361608-120c33e99dc35995.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于以下对象b,指向Bear的指针 pb, rb，对象和指针的内存布局：

![](https://upload-images.jianshu.io/upload_images/5361608-0ddd9d7314579a04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


给予一个Bear指针和ZooAnimal指针：

![](https://upload-images.jianshu.io/upload_images/5361608-3f88acefcf9e4a62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/5361608-bd565f0e895c2b5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于对象中的虚函数：

- 运行时决定；




---

