---
title: 5_Epiplolar_Geometry
date: 2018-06-22 10:29:25
tags:
	- Epipolar_Gemotry
categories:
	- cs231A
---
> CS231A笔记
> 文章截图大多来自斯坦福CS231A课程;
---

# 1 基本概念

- 相机的旋转矩阵： 正交矩阵

- 极点
- 极线
- 极平面

- 【推荐文章】：  [对极几何（Epipolar Geometry）](http://www.cnblogs.com/clarenceliang/p/6704970.html)

- 叉乘：两个向量的叉乘可以由一个向量的反对称矩阵和另一向量的积；

![](https://upload-images.jianshu.io/upload_images/5361608-58e56509b18c3aac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.1 特点
- 极线交于一点---极点

![](https://upload-images.jianshu.io/upload_images/5361608-d9608d848346057d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







## 1.2 本质矩阵
- 相机是 标准相机：K = I

![](https://upload-images.jianshu.io/upload_images/5361608-24fd49f34c17ed5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/5361608-26aa928ebfa621b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由图中可以看出，p经过变换得到p':
![](https://latex.codecogs.com/gif.latex?p%5E%7B%27%7D%20%3D%20R%5Ccdot%20p&plus;T)
得到：
![](https://latex.codecogs.com/gif.latex?p%20%3D%20R%5E%7BT%7D%5Ccdot%20p%5E%7B%27%7D-%20R%5E%7BT%7DT)

![](https://upload-images.jianshu.io/upload_images/5361608-87f31eed4df5757f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/5361608-370b7b6e60cc5083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-7a99b066db36db54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过叉乘：可以将上个式子变成：
![](https://upload-images.jianshu.io/upload_images/5361608-dd94d10de2b0d6f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 定义： 
![](https://upload-images.jianshu.io/upload_images/5361608-a818cd6296f89153.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为本质矩阵（The Essential Matrix）：

>- 3*3的矩阵；
>- 自由度： 5
>- rank : 2 ,奇异矩阵


> 用处

1. 计算p 和 p'的极线

![](https://upload-images.jianshu.io/upload_images/5361608-5e374ad4ee35899c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
表示：p被E投射到 第二幅图像上的极线l'上；

![](https://upload-images.jianshu.io/upload_images/5361608-3a66b112ca5c0e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
表示：p’被E投射到 第一幅图像上的极线l上；


2. 极点和矩阵的联系

![](https://upload-images.jianshu.io/upload_images/5361608-320355b22790387f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.3 基本矩阵（Fundamental Matrix）
- 相机是非标准的；

![](https://upload-images.jianshu.io/upload_images/5361608-26aa928ebfa621b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对应两个相机，两个相机的投影矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-8e5e3e236a54d742.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

定义世界坐标系中的点P在两个位置的相机图片上的投影为：

![](https://upload-images.jianshu.io/upload_images/5361608-539a4452776e48a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-f33b5f801b31ac0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


根据本质矩阵得到：

![](https://upload-images.jianshu.io/upload_images/5361608-a1656321484f80b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

推导出，两个相机的位置变换：

![](https://upload-images.jianshu.io/upload_images/5361608-2687edf23d78b80f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：

![](https://upload-images.jianshu.io/upload_images/5361608-0593f0f9af9f79b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
F就被称为： `基本矩阵`

>- 3* 3 矩阵
> - 奇异矩阵  Rank :2
>- 包含R, T；
>- 包含 相机参数 K,K'
>- 自由度 ： 7

> 用处
 基本矩阵已知，已知一张图片上一点和另一张图片上的对应点， 就可以求出p和p'的对应关系；

#2  THE Eight-Point 算法
## 2.1 八点算法
- 给予两张图片，没有相机内外参数；
- 求出基本矩阵

- 方法： '_Eight-Point 算法_ '

---
1. 至少8对 对应点

![](https://upload-images.jianshu.io/upload_images/5361608-750407be55d3a189.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

则每队对对应点满足：

![](https://upload-images.jianshu.io/upload_images/5361608-39605f51776577d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

即：

![](https://upload-images.jianshu.io/upload_images/5361608-b2ab3e15ad77b5fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
展开，可以得到：

![](https://upload-images.jianshu.io/upload_images/5361608-321f94cfacb01aaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于所有对应点，使用一下公式：

![](https://upload-images.jianshu.io/upload_images/5361608-3c3ec40ba81b3477.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简写为：

![](https://upload-images.jianshu.io/upload_images/5361608-5766d9a159915312.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- W ： N*9 矩阵 ， N: 对应对的数量
- f: 基本矩阵

---
- 使用svd分解
- w: 非满秩的
- f: 真实的基本矩阵秩为2 
- #[基本矩阵的基本解法之8点算法](https://blog.csdn.net/kokerf/article/details/72630863?locationNum=2&fps=1)

![](https://upload-images.jianshu.io/upload_images/5361608-67d92e80c6b464f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
## 3.2 Normalized Eight-Point算法
标准的8点算法存在的问题：

- p点， 和通过基本矩阵将点p'映射到的极线上l= Fp'，两者的距离很大；点和极线之间的均值误差如下：

![](https://upload-images.jianshu.io/upload_images/5361608-091b3647c850b792.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- 8个对应点组成的矩阵W： 最好只有一个奇异值为0或者近似为0,其他的都为非零；这样能够使得svd效果好；
-  像素值的范围过大；如pi = (1832; 1023;)

---
解决方法：

- 归一化像素点
  - 1. 新坐标系的原点应该位于图像点的质心处（平移）；
表示为：

![](https://upload-images.jianshu.io/upload_images/5361608-6349e1ccc32f9d49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  -  其次，变换后的图像点距原点的均方距离应为2像素（缩放）

![](https://upload-images.jianshu.io/upload_images/5361608-a06c93ec30ac32aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图所示，

![](https://upload-images.jianshu.io/upload_images/5361608-446adabfc4846b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



然后用新坐标系下的图像应用8点算法计算出Fq；

![](https://upload-images.jianshu.io/upload_images/5361608-97bf6c2a7e7c0064.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后通过反归一化，获得真实的基本矩阵；




---

