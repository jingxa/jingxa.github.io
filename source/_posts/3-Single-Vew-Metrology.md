---
title: 3_Single_View_Metrology
date: 2018-06-22 10:33:38
tags:
	- Single_View_Metrology
categories:
	- cs231A
---
> CS231A笔记
> 文章截图大多来自斯坦福CS231A课程;
---

# 1  2d中的变换

## 1.1 几何变换
![](https://upload-images.jianshu.io/upload_images/5361608-4293a9fb6318994b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.2 相似变换
![](https://upload-images.jianshu.io/upload_images/5361608-f0b1c1de2d8270f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.3 仿射变换

![](https://upload-images.jianshu.io/upload_images/5361608-1e1629e3eaf9aec0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.4 投影变换

![](https://upload-images.jianshu.io/upload_images/5361608-1a1297c41268f109.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

投影变换4个共线点的交叉率：

![](https://upload-images.jianshu.io/upload_images/5361608-2d65709b5e37222b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
# 2 vanishing points and lines (灭点)

## 2.1 无穷点

![](https://upload-images.jianshu.io/upload_images/5361608-ed0a8848567585d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于两条直线的交点：
![](https://upload-images.jianshu.io/upload_images/5361608-033b9e87e41a0f47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是，如果两条平行直线的交点，在齐次坐标系进行计算：

![](https://upload-images.jianshu.io/upload_images/5361608-5d1a8ee50373adf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 在欧氏坐标系下， 这个点是无穷的；
- 两条线段的交点也是无穷的；

![](https://upload-images.jianshu.io/upload_images/5361608-ffddc1e05ba0b6f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.2 无穷线

![](https://upload-images.jianshu.io/upload_images/5361608-f0fa9ecc24c3b48e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 2.3 无穷点的投影变换
![](https://upload-images.jianshu.io/upload_images/5361608-42a7f1de720f48a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.4 无穷线的投影变换
![](https://upload-images.jianshu.io/upload_images/5361608-b58b22ded0124c7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 3 3d中的点和面

- 点
![](https://upload-images.jianshu.io/upload_images/5361608-df33547dc5b37f2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 线段
![](https://upload-images.jianshu.io/upload_images/5361608-1a498638def1cfff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 3.1 3d中的无穷点

![](https://upload-images.jianshu.io/upload_images/5361608-81e985560c732f30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.2 vanishing points
无穷点的投影变换不在是无穷的；
![](https://upload-images.jianshu.io/upload_images/5361608-01071f0955aeb371.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

给定一个相机参数K,R,T,得到灭点：
![](https://upload-images.jianshu.io/upload_images/5361608-ccbc7da86c7231c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

定义一个平行线的超集为一个平面，每对平行线的交点都是无穷点，这些无穷点组成的线段就是无穷线，这个平面的无穷线经过投影变换`H`后不再是无穷的，被称为`vanishing line 或者 horizon line`，可以通过以下计算：

![](https://upload-images.jianshu.io/upload_images/5361608-0346133f45da09b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例如：
![](https://upload-images.jianshu.io/upload_images/5361608-19aabed436274ba3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 橙色的是水平线，两个铁轨的交点在水平线，那么这两条铁轨就是水平线；


## 3.3 面
通过3d的平面法线n和对应的水平线，可以得到以下公式：

![](https://upload-images.jianshu.io/upload_images/5361608-f823386978ec4821.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5361608-f85ca24a96846ad5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如果相机的参数已知晓，那么可以得到水平线相关的平面的方向；


## 3.4 无穷面
- 定义：2个或多个无穷线组成的平面
- 在齐次坐标系下用`[0 0 0 1]T`表示

![](https://upload-images.jianshu.io/upload_images/5361608-a38031e839a04718.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设两对平行线的灭点为v1,v2, 平行线的方向分别为d1,d2，那么这两个方向的夹角为：
![](https://upload-images.jianshu.io/upload_images/5361608-3f0fee0cbccdce69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-344f019d6232aca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-f0888390d19d82e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以同时扩展到平面：
![](https://upload-images.jianshu.io/upload_images/5361608-e388a2ec9b5c1e0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3.5  投影变换 总结
- 使用投影变换，可以将无穷点投影为灭点；
- 无穷线投影为水平线；
![](https://upload-images.jianshu.io/upload_images/5361608-4d2c94fdc767a451.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-2af4fad907d2ad90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-e5c3d841693d5796.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





---

