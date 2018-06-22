---
title: 6_Epipolar_Geomotry_2
date: 2018-06-22 10:38:18
tags:
	- Epipolar_Gemotry
categories:
	- cs231A
---
> CS231A笔记
> 文章截图大多来自斯坦福CS231A课程;
---
> 本章节许多问题没有看懂，o(╥﹏╥)o
---

# 1 问题

![](https://upload-images.jianshu.io/upload_images/5361608-8cc0457621c6e56b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 两个图片平行的问题
  - 无旋转 R = I
  - 平移： 假设在x 轴上平移

那么 本质矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-208521e7cc7aff33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

计算极线的方向：

![=](https://upload-images.jianshu.io/upload_images/5361608-a2d105ea0a781cb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可知，两张图片的极线是水平的；

![](https://upload-images.jianshu.io/upload_images/5361608-9b6f663493447819.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以得知，两点是共享同一个坐标轴 v的；


# 2  Rectification:	making	two	images	“parallel”

- 平行图片的优点

![](https://upload-images.jianshu.io/upload_images/5361608-b726092d63e282c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.1 三角化
- point triangulation

![](https://upload-images.jianshu.io/upload_images/5361608-38908e5c54eec044.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 距离计算
![](https://upload-images.jianshu.io/upload_images/5361608-71094389014fe2b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-862088c7ad288ef4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.2 对应性问题

![](https://upload-images.jianshu.io/upload_images/5361608-97cac06011167479.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-c61637121427c095.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2.2.1 对应性方法

![](https://upload-images.jianshu.io/upload_images/5361608-caa07270e4d57480.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两个图片共享v轴，只在u轴上变化；

![](https://upload-images.jianshu.io/upload_images/5361608-a26a241aae147875.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种方法造成强度的改变；

![](https://upload-images.jianshu.io/upload_images/5361608-5ef88878c2c0ff5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

窗口大小不同的结果为；

![](https://upload-images.jianshu.io/upload_images/5361608-24873393b76fdbc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

存在四个不同的问题：

![](https://upload-images.jianshu.io/upload_images/5361608-11b0d26911a7843d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-364455beb480ae7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-6359e1f4480583c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-063b28dd23947fd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-9d707c9d5df7a41f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-9baeaa8d9a32d5cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



---
## 3 校正两张图片
- 通过归一化8点算法获取基本矩阵
- 极线交于极点： 通过最小二乘误差拟合
- svd计算 极点

![](https://upload-images.jianshu.io/upload_images/5361608-80295e9834fc1d02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


--- 
- 在求得极点e,e'
- 寻找一对单应性矩阵H1,H2,使得极点无穷，即两张图片平行；


---

