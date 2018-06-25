---
title: CS231A-Homework-2.3
date: 2018-06-25 16:28:37
tags:
	- Epipolar_Geometry
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-2:ps-code2-part-3

---


# 三、 The Factorization Method

- 此方法用来解决SFM问题；


# 一、 算法过程
- Tomasi & Kanade algorithm 

- 分为两步：
  - Data centering
  - Factorization

## 1. centering step
- 居中数据： 对于一张图片中的每个点，定义新的坐标点，通过减去图片中心点：

![](https://upload-images.jianshu.io/upload_images/5361608-eb8a7a929e736702.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并且可知：

![](https://upload-images.jianshu.io/upload_images/5361608-386ec29786e6939a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上述两个式子可以得到：

![](https://upload-images.jianshu.io/upload_images/5361608-cb9796ccae27a7ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 居中后的3D点和居中后的图片点通过一个2*3矩阵A相关；

- 但是，只知道图片中的点，不知道A和X;

## 2、 建立D矩阵
通过建立一个处理矩阵D，由m个相机的n个观察点组成；

![](https://upload-images.jianshu.io/upload_images/5361608-8b77c687821f7a2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- M：2m*3
- S: 3 * n
- D = M * S 
- rank(D) = 3

## 3. 因素化
通过使用SVD方法将D分解为运动矩阵M和结构矩阵S：

![](https://upload-images.jianshu.io/upload_images/5361608-4f364584f1225c0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 因为D的秩为3， 但是由于噪声和仿射变换近似，rank(D) >3, 但是只需要取3仍然是最好的近似；

![](https://upload-images.jianshu.io/upload_images/5361608-8a2bc38962294ff1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- Tomasi and Kanade 的方法： 

 最佳因素化选择：

![](https://upload-images.jianshu.io/upload_images/5361608-be770447488c70bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

【问题】
- 怎么得到M : 2m * 3? 难道其他元素为0？
- S也是类似？
  - 因为M,S两个矩阵最大秩为3，所以两个矩阵的乘积Rank=3；实际中，D的rank大于3，但是rank为3是最佳近似；
  - 由于D有秩为3，将分解为（2m * 3）(3 * 3)(3 * n)的三个矩阵；


# 二、 python实现

## 1.因素化方法 

```Python

def factorization_method(points_im1, points_im2):
    N = points_im1.shape[0]
    points_sets = [points_im1, points_im2]      # 2* (N * 3)

    # 建立D矩阵
    D = np.zeros((4, N))
    for i in range(len(points_sets)):   # len : 2
        points = points_sets[i]        # N * 3
        # 中心化点
        centroid = 1.0 / N * points.sum(axis=0)     # 均值 (x,y),
        points[:, 0] -= centroid[0] * np.ones(N)    # x
        points[:, 1] -= centroid[1] * np.ones(N)    # y
        D[2*i:2*i+2, :] = points[:, 0:2].T    # 每一副图片的(x,y)复制到D中

    # svd分解D矩阵
    u, s, vt = np.linalg.svd(D)
    print(u.shape, s.shape, vt.shape)
    print(s)
    M = u[:, 0:3]      # Motion
    S = np.diag(s)[0:3, 0:3].dot(vt[0:3, :])        # structure
    return S, M

```
在svd分解后，直接使用u作为M运动矩阵，S = singa*vT,作为结构矩阵；其实可以选择上面最后的最佳选择；


## 2. 主函数

```Python
import numpy as np
from scipy.misc import imread
import matplotlib.pyplot as plt
import scipy.io as sio
import matplotlib.gridspec as gridspec
from epipolar_utils import *

if __name__ == '__main__':
    for im_set in ['data/set1', 'data/set1_subset']:
        # Read in the data
        im1 = imread(im_set+'/image1.jpg')
        im2 = imread(im_set+'/image2.jpg')
        points_im1 = get_data_from_txt_file(im_set + '/pt_2D_1.txt')
        points_im2 = get_data_from_txt_file(im_set + '/pt_2D_2.txt')
        points_3d = get_data_from_txt_file(im_set + '/pt_3D.txt')
        assert (points_im1.shape == points_im2.shape)

        # Run the Factorization Method
        structure, motion = factorization_method(points_im1, points_im2)

        # Plot the structure
        fig = plt.figure()
        ax = fig.add_subplot(121, projection = '3d')
        scatter_3D_axis_equal(structure[0,:], structure[1,:], structure[2,:], ax)
        ax.set_title('Factorization Method')
        ax = fig.add_subplot(122, projection = '3d')
        scatter_3D_axis_equal(points_3d[:,0], points_3d[:,1], points_3d[:,2], ax)
        ax.set_title('Ground Truth')

        plt.show()


```

# 三、结果
```
(4, 4) (4,) (37, 37)
[959.5852216  540.47613178 184.43174791  27.9151956 ]
(4, 4) (4,) (18, 18)
[264.54396508 210.06072009   7.21921783   5.12857709]
```
图对比：

![](https://upload-images.jianshu.io/upload_images/5361608-88e8a61ad5260eab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





---

