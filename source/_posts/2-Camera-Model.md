---
title: 2_Camera_Model
date: 2018-06-22 10:16:24
tags:
	- Camera
categories:
	- cs231A
---
> CS231A笔记
> 文章截图大多来自斯坦福CS231A课程;
---

- 干脆 明亮的图片

1.  镜头
- 使用lense 透镜来解决上述问题；

- 透镜的问题：
   - pincushion distortion 枕形畸变
  - barrel distortion 桶形畸变


# 3. 3d到2d的投影变换
![image.png](https://upload-images.jianshu.io/upload_images/5361608-03d37c544f3260a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5361608-f22aaa6b85b5c1c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对于某些平移和缩放：

![image.png](https://upload-images.jianshu.io/upload_images/5361608-aa9051feb0ad3ba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是在公式中，缺少两个重要参数，相机制造误差：
- skewness: 倾斜
- distortion： 形变

![](https://upload-images.jianshu.io/upload_images/5361608-cd8d34757e165d97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此形成5个自由度的相机内参矩阵K;

世界坐标到相机坐标系的变换为：

![](https://upload-images.jianshu.io/upload_images/5361608-e8d619979d82849a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

综上，变换矩阵由两个部分组成：
- instrinsic : K : 5个自由度
- extrinsic ： T ,R : 各自3个自由度
总共11个自由度；

# 4. Camera Calibration（相机校准）

在投影变换中，
![](https://upload-images.jianshu.io/upload_images/5361608-2de9c0e396c134df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原理为：
![](https://upload-images.jianshu.io/upload_images/5361608-56bd38497c0aa26a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 4.1 投影类型

参考资料：
-[ 摄像机透视投影近似模型](https://blog.csdn.net/jyc1228/article/details/4209870)
### 4.1.1 弱透视投影

>弱透视：weak perpective, 如果摄像机的视场比较小，而且物体表面深度变化相对其到摄像机的距离很小的话，物体上个点的深度可以用一个固定的深度值z0近似，这个值一般取物体质心的深度。这种近似可以看做是两次投影的合成。

![](https://upload-images.jianshu.io/upload_images/5361608-10876044e7f7900a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>首先，整个物体按平行于光轴的方向正投影到经过物体质心并与图像平面平行的平面上；

![](https://upload-images.jianshu.io/upload_images/5361608-bf0b8cfedaf66c57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>然后，再按透视模型将上述物体平面的图像投影到摄像机的图像平面上，这一步实际是全局的放缩。因此，弱透视也叫放缩正投影（scaled orthographic projection）。


![](https://upload-images.jianshu.io/upload_images/5361608-d6e80555a72c7dfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 4.1.2 正投影：orthographic projection
![](https://upload-images.jianshu.io/upload_images/5361608-68c0aa0e15865b30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>这种近似完全忽略了深度信息



## 4.2 校准目标

> 估计内部参数和外部参数；

![](https://upload-images.jianshu.io/upload_images/5361608-819d27eb90c59245.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-b2b94ebd08e64e3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-734a190121f0af84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于11自由度的参数：只要2n>11,满足的n个点就可以得到很好的结果；
![](https://upload-images.jianshu.io/upload_images/5361608-4f0424a16bb4daa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-52f254e482337ca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

得到一个方程组：

![](https://upload-images.jianshu.io/upload_images/5361608-63d1cb18425bbd9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

求解一个线性方程：
- 0 为解
- 非零解： 

![](https://upload-images.jianshu.io/upload_images/5361608-314c82c970c265c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 参考资料： [奇异值分解(SVD)原理与在降维中的应用](http://www.cnblogs.com/pinard/p/6251584.html)


![](https://upload-images.jianshu.io/upload_images/5361608-38920e7d3beea99c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



---
### 4.2.2 Radial Distortion

- 参考资料：[透镜畸变及校正模型](https://blog.csdn.net/dcrmg/article/details/52950141)

![](https://upload-images.jianshu.io/upload_images/5361608-fdc05c03cc05a7f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





---

