---
title: CS231A-Homework-2.2
date: 2018-06-25 10:32:43
tags:
	- Epipolar_Geometry
	- homeworks
categories:
	- cs231A
---

# 2 图片校正 之 单应性匹配
## 2.1 求解过程
 校正两个图片不需要就知道相机的内参K和外参矩阵R,T，只需要通过基础矩阵计算极线；
然后计算所以极线的交点---极点；由于噪点的干扰，不可能交于一点，因此计算极点，利用最小二乘方法求得极线拟合一点；
并且在极线上的每一点：

![](https://upload-images.jianshu.io/upload_images/5361608-041355bcb93319ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

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


---
## 2.2 python 实现

在经过上面的了解：

### 一、 计算极点

```Python
def compute_epipole(points1, points2, F):
    # F.Tp2 = l, 求得p2到p1面上的映射直线
    line = F.T.dot(points2.T)  # 3 * N
    lineT = line.T      # N * 3
    u, s, vt = np.linalg.svd(lineT)
    e = vt[-1, :]       # 最优解
    e /= e[2]       # 齐次坐标（x, y, 1）

    return e
```

### 二、 计算 H1,H2

```Python
def compute_matching_homographies(e2, F, im2, points1, points2):
    # 首先计算H2
    W = im2.shape[1]
    H = im2.shape[0]

    # 平移矩阵T
    T = np.identity(3)
    T[0, 2] = -1.0 * W /2
    T[1, 2] = -1.0 * H /2

    # 旋转矩阵R
    e = T.dot(e2)   # 极点平移过后形成（x',y',1）
    e_x = e[0]
    e_y = e[1]
    if e_x >= 0:
        alpha = 1.0
    else:
        alpha = -1.0    # 旋转矩阵alpha 正负判断

    R = np.identity(3)
    R[0, 0] = alpha * e_x / np.sqrt(e_x**2 + e_y**2)
    R[0, 1] = alpha * e_y / np.sqrt(e_x**2 + e_y**2)
    R[1, 0] = -alpha * e_y / np.sqrt(e_x**2 + e_y**2)
    R[1, 1] = alpha * e_x / np.sqrt(e_x**2 + e_y**2)

    # 矩阵G
    f = R.dot(e)[0]     # R变换之后，e==>(f,0,1)
    G = np.identity(3)
    G[2, 0] = -1.0 / f

    H2 = np.linalg.inv(T).dot(G.dot(R.dot(T)))     # H2 = T^-1 G R T

    # ===============================
    # 计算H1 , H1 = HaH2M

    # 首先计算M
    e_p = np.zeros((3, 3))
    e_p[0, 1] = - e2[2]
    e_p[0, 2] = e2[1]
    e_p[1, 0] = e2[2]
    e_p[1, 2] = - e2[0]
    e_p[2, 0] = - e2[1]
    e_p[2, 1] = e2[0]   # skew-symmetric 矩阵

    v = np.array([1, 1, 1])
    M = e_p.dot(F) + np.outer(e2, v)    # e2*VT = (3,3)


    # 计算Ha
    p1_hat = H2.dot(M.dot(points1.T)).T     # p1_hat = H2Mp  3 * N,转置之后 N * 3
    p2_hat = H2.dot(points2.T).T            # pe_hat = H2p' , 3 * N ,转置之后 N * 3
    W = p1_hat / p1_hat[:, 2].reshape(-1, 1)    # 齐次坐标系
    b = (p2_hat / p2_hat[:, 2].reshape(-1, 1))[:, 0]

    # 最小二乘问题
    a1, a2, a3 = np.linalg.lstsq(W, b, rcond=None)[0]
    HA = np.identity(3)
    HA[0] = np.array([a1, a2, a3])

    H1 = HA.dot(H2).dot(M)      # H1 = HaH2M

    return H1, H2

```

### 三、 主函数
```Python
if __name__ == '__main__':
    # Read in the data
    im_set = 'data/set1'
    im1 = imread(im_set+'/image1.jpg')
    im2 = imread(im_set+'/image2.jpg')
    points1 = get_data_from_txt_file(im_set+'/pt_2D_1.txt')
    points2 = get_data_from_txt_file(im_set+'/pt_2D_2.txt')
    assert (points1.shape == points2.shape)

    F = normalized_eight_point_alg(points1, points2)
    e1 = compute_epipole(points1, points2, F)
    e2 = compute_epipole(points2, points1, F.transpose())
    print("e1", e1)
    print("e2", e2)



    # Find the homographies needed to rectify the pair of images
    H1, H2 = compute_matching_homographies(e2, F, im2, points1, points2)
    print("H1:\n", H1)
    print
    print("H2:\n", H2)

    # Transforming the images by the homographies
    new_points1 = H1.dot(points1.T)
    new_points2 = H2.dot(points2.T)
    new_points1 /= new_points1[2,:]
    new_points2 /= new_points2[2,:]
    new_points1 = new_points1.T
    new_points2 = new_points2.T
    rectified_im1, offset1 = compute_rectified_image(im1, H1)
    rectified_im2, offset2 = compute_rectified_image(im2, H2)
    new_points1 -= offset1 + (0,)
    new_points2 -= offset2 + (0,)

    # Plotting the image
    F_new = normalized_eight_point_alg(new_points1, new_points2)
    plot_epipolar_lines_on_images(new_points1, new_points2, rectified_im1, rectified_im2, F_new)
    plt.show()
```

### 四、 结果

```
e1 [-1.30071143e+03 -1.42448272e+02  1.00000000e+00]
e2 [1.65412463e+03 4.53021078e+01 1.00000000e+00]
H1:
 [[-1.20006316e+01 -4.15501447e+00 -1.23476881e+02]
 [ 1.41006481e+00 -1.48704147e+01 -2.84177469e+02]
 [-9.21889298e-03 -2.19184511e-03 -1.23033440e+01]]
H2:
 [[ 8.09798131e-01 -1.22036874e-01  7.99331183e+01]
 [-3.00186699e-02  1.01581538e+00  3.63604348e+00]
 [-6.99360915e-04  1.05393946e-04  1.15205554e+00]]
```

调整过后的图片为：

![](https://upload-images.jianshu.io/upload_images/5361608-d76d7018232d0b04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




---

