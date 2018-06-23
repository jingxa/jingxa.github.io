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

> 主题： 图片校正 Image Rectification

#1 问题

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
 校正两个图片不需要就知道相机的内参K和外参矩阵R,T，只需要通过基础矩阵计算极线；
然后计算所以极线的交点---极点；由于噪点的干扰，不可能交于一点，因此计算极点，利用最小二乘方法求得极线拟合一点；
并且在极线上的每一点：

![](https://upload-images.jianshu.io/upload_images/5361608-70ef22c82e308bcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果定义极线集合：

![](https://upload-images.jianshu.io/upload_images/5361608-2ad1cd071a0b7c4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以得到：

![](https://upload-images.jianshu.io/upload_images/5361608-80295e9834fc1d02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此整个步骤为：
- 1. 通过归一化8点算法获取基本矩阵
- 2. 极线交于极点： 通过最小二乘误差拟合
- 3. svd计算 极点

--- 
> 在求得极点e,e'后，如果极点在水平方向上不是无限的，说明图片没有平行，如果点是无限的，说明图片平行了；

因此，我们的目的：
- **寻找一对单应性矩阵H1,H2,使得极点无穷，即两张图片平行；**

---

(1). **寻找H2：令e'在水平方向上无限,即为点（f,0,0）**
- 将第二幅图的中心移动到(0,0,1)在齐次坐标系下；
这个平移矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-f15f7e8f6c1d82a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


(2). **应用一个旋转矩阵：使极点坐标旋转到水平坐标轴上，变为(f, 0, 1)**
- 如果e‘在平移后Te'的坐标为：(e1',e2',1)，旋转矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-2b8848d72443787d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-33ebdcfefbcfc87e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


(3). **将极点坐标由(f,0,1)==>(f,0,0)**
- 只需要变换矩阵：

![](https://upload-images.jianshu.io/upload_images/5361608-4cf916086e001850.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经过上面几个步骤。我们可以得到无限极点：这个单应性变换H2为：

![](https://upload-images.jianshu.io/upload_images/5361608-eceae870c337bd04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

- **为了寻找单应性矩阵H1,,需要最小化两个图片对应点的均方差之和来获得H1**

![](https://upload-images.jianshu.io/upload_images/5361608-ebe437f9682f42c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

求导后，可以得到：

!![](https://upload-images.jianshu.io/upload_images/5361608-dffe23f713cf43f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

计算式子中的未知参数：

(1). **首先计算M**
- 任意3*3的斜对称矩阵：

  ![](https://upload-images.jianshu.io/upload_images/5361608-3ca1aac7031b398e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为任何向量的叉积矩阵 是一个斜对称矩阵：

![](https://upload-images.jianshu.io/upload_images/5361608-58e56509b18c3aac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图，a向量的叉乘矩阵为斜对称矩阵；

因此，前面公式中：

![](https://upload-images.jianshu.io/upload_images/5361608-e933426be143035d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

合并后，

![](https://upload-images.jianshu.io/upload_images/5361608-cb430c0d0009a532.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果M每一列缩放e，则F也需要变化：

![](https://upload-images.jianshu.io/upload_images/5361608-c2760d1d20996d59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-b8d6728c7d829f24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(2). **计算Ha矩阵中的参数a**

- 已经知道了H2, M，如果将图片1中的点p变换到图片2中，然后经过H2变换：

![](https://upload-images.jianshu.io/upload_images/5361608-d3dd8c3950e7c971.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-b26168a6bf4b3003.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这两个点应该是最近点，然后使用最小二乘发来最小化误差：

![](https://upload-images.jianshu.io/upload_images/5361608-c45f913af4b55069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设两个点齐次坐标为：

![](https://upload-images.jianshu.io/upload_images/5361608-c7c39a32c60e82ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最小化式子为：

![](https://upload-images.jianshu.io/upload_images/5361608-03f84d774d3fede2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


因为最后一项两个点在水平线上，则为常数

![](https://upload-images.jianshu.io/upload_images/5361608-2548833df47c882f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终，这个问题变成了解决最小二乘问题：

![](https://upload-images.jianshu.io/upload_images/5361608-dd8f6953623efa08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-4b3993661018b687.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


求得a的参数后，得到了HA,H2,M三个矩阵，根据上面的式子：
最终可以求得H1；








