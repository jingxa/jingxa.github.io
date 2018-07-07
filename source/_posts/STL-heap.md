---
title: STL_heap
date: 2018-07-05 21:44:16
tags:
	- STL
	- Container
categories:
	- STL源码分析
---

> 本文章内容来源于《STL源码分析》第四章
---
- heap不归属于STL容器组件，扮演priority-quque的助手，即priority queue的底层；

# 1 概述
## 1.1 binary heap
- 一种 complete binary tree (完全二叉树)的存储方式
- 使用vector作为存储空间

![](https://upload-images.jianshu.io/upload_images/5361608-00622adba11ef4f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.2 heap的隐式表述法(implicit representation)
- 将数组的#0元素保留
- 那么在数组中，位于数组i处的节点的左子节点为2i，右子节点为(2*i +1)
-【SGI STL 并未使用这个方法】

## 1.3 max-heap
- STL实现的是大根堆

# 2 heap算法

## 2.1 push_heap 算法
- 新加入的元素位于最下一层作为叶节点，也就是插入到vector的end()处；
- 通过和父节点进行比较，如果值比父节点大，就父子对换位置，一直上溯，直到不需要对换或者到根节点为止；


![](https://upload-images.jianshu.io/upload_images/5361608-58668b0e78d9e076.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- push_heap

```
// push_heap
template<class RandomAccessIterator>
inline void push_heap(RandomAccessIterator first, RandomAccessIterator last){
	// 此函数调用的时候 新元素已经置于容器底部
	__push_heap_aux(first,last,distance_type(first),value_type(first));
}

```

- 使用__push_heap_aux 和 __push_heap作为内部函数；

```

template<class RandomAccessIterator, class Distance, class T>
inline void __push_heap_aux(RandomAccessIterator first,
				RandomAccessIterator last, Distance*, T*){
	// 新值在容器最尾端，位置： （last -first）-1
	// 隐式表述法 ,根节点编号从0开始
	__push_heap(first, Distance(last - first)-1, Distance(0), T(*(last -1)));
}


// 一下这组push_back()不允许指定'大小比较标准'
template<class RandomAccessIterator, class Distance, class T>
void __push_heap(RandomAccessIterator first, Distance holeIndex, 
			Distance topIndex, T value){
	
	Distance parent = (holeIndex -1) /2; // 找出父节点
	while(holeIndex > topIndex && *(first + parent) < value){  // 大根堆调整
		// 尚未到达顶端并且父节点小于新值
		*(first + holeIndex) = *(first + parent); // 令洞位置为父节点
		holeIndex = parent; // 调整洞号， 向上提升
		parent = (holeIndex - 1) /2;   // 新父节点	
	}   // 调整完成
	*(first + holeIndex) = value;  // 赋值
}
```

## 2.2 pop_heap算法

- 先将根节点和最下一层的最右边的叶节点对换位置；然后调整除了最后一个节点的vector的first,last-1]的堆；

![](https://upload-images.jianshu.io/upload_images/5361608-f729a93c0e6b8d49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- pop_heap

```
/**
	pop_heap
	1. 取走根节点
	2. 满足完全二叉树的条件，将最下一层的最右边的叶节点拿掉
*/
template<class RandomAccessIterator>
inline void pop_heap(RandomAccessIterator first, RandomAccessIterator last){
	
	__pop_heap_aux(first, last, value_type(first));
	
}

```

- 使用了__pop_heap_aux作为内部函数：

```
// 根据隐式表述法， pop操作的结果为底部容器的第一个元素
// 首先将首值和尾节点互换，然后重新调整[first, last -1)
template<class RandomAccessIterator , class T>
inline void __pop_heap_aux(RandomAccessIterator first, RandomAccessIterator last,
							T*){
	__pop_heap(first,last-1, last-1, T(*(last - 1), distance_type(first));							
}

template<class RandomAccessIterator, class T, class Distance>
inline void __pop_heap(RandomAccessIterator first, 
						RandomAccessIterator last,
						RandomAccessIterator result, 
						T value, Distance*)
{
	*result = * first;   // 设定尾值为首值 
					     //然后后面可以由容器pop_back()取出
	// 重新调整heap
	__adjust_heap(first,Distance(0), Distance(last -first), value);
	
}

```

- heap的调整算法：`__adjust_heap`
```

template<class RandomAccessIterator, class Distance, class T>
void __adjust_heap(RandomAccessIterator first, Distance holeIndex, 
					Distance len, T value){
	
	Distance topIndex = holeIndex; 
	Distance secondChild = 2*holeIndex +2;  // 洞节点的右子节点
	while(secondChild < len){
		// 比较洞节点的两个子节点， 然后secondChild代替较大子节点
		if(*(first + secondChild) < *(first(secondChild -1))){
			secondChild --;
		}
		*(first + holeIndex ) = *(first + secondChild);  // 替换父节点和子节点
		holeIndex = secondChild;
		secondChild = 2*(secondChild +1);  // 下一层
	}
	if(secondChild == len){  // 没有右子节点 ，只有左节点
		// 令左子值为洞值，在令洞号移动左子节点处
		*(first + holeIndex ) = *(first +(secondChild -1));
		holeIndex = secondChild -1;
	}
	// 将要调整的洞号填入目前的洞号，
	// 此时已经满足次序性
	// 可直接为*（first + holeIndex) = value;
	
	__push_heap(first, holeIndex, topIndex,value);
	
	
}
```


## 2.3 sort_heap 算法

- pop_heap每次将最大元素放到尾端，那么通过持续调用pop_heap算法就可以得到逆序序列；

```
/**
	sort_heap
	1. pop_heap 每次取出一个最大值，持续pop，可得逆序序列；
*/


template<class RandomAccessIterator>
void sort_heap(RandomAccessIterator first, 
				RandomAccessIterator last){
	// 没执行一次pop_heap(),极值放在尾端
	while(last - first >1){
		pop_heap(first,last--); // 执行一次，操作范围缩小一点
	}
}
```

## 2.4 make_heap算法

- 将一段现有数据转化一个heap

```
/**
	make_heap: 将一段数据转化为堆
*/
template<clss RandomAccessIterator>
inline void make_heap(RandomAccessIterator first, RandomAccessIterator last){
	__make_heap(first,last, value_type(first),distance_type(first));
}

template<class RandomAccessIterator, class T, class Distance>
void __make_heap(RandomAccessIterator first,
				RandomAccessIterator	last, T*, Distance*)
{
	if(last - first < 2) return ; // 长度0或1 不用重新排列
	Distance len = last - first;
	
	// 找出第一个需要重新排列的子树头部，以parent标出，
	Distance parent = (len -2 ) /2;
	
	while(true){
		// 重新排列parent为首的子树，len为了让__adjust_heap()判断操作范围
		__adjust_heap(first,parent,len,T(*(first+parent)));
		if(parent == 0) return ; // 走完根节点就结束；
		parent --; // 重排子树的头部向前一个节点；
	}
	
}
```

---

