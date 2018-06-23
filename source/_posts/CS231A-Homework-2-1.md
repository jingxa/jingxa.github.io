---
title: CS231A-Homework-2.1
date: 2018-06-23 17:35:13
tags:
	- Epipolar_Geometry
	- homeworks
categories:
	- cs231A
---
> CS231A-Homework-2:ps-code2-part-1

# 一、 从对应点估计基础矩阵

- 本题要求从对应点估计基础矩阵，使用二种方法：
  - 线性最小二乘法的八点算法
  - 归一化八点算法

- 笔记：
    -  [对极几何](https://jingxa.github.io/2018/06/22/5-Epiplolar-Geometry/)
## 1.1 八点算法-- 基础矩阵求解过程

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

- 这里进行了两次SVD分解；
- 首先获得F矩阵的近似；
- 第二次获得RANK为2的F;

```
# points1, points2 都是np.array
def lls_eight_point_alg(points1, points2):
    len = points1.shape[0]

    W = np.zeros((len, 9))   # 37 * 9 齐次矩阵
    for i in range(len):
        u1 = points1[i, 0]
        v1 = points1[i, 1]
        u2 = points2[i, 0]
        v2 = points2[i, 1]
        W[i] = np.r_[u1*u2, u2*v1, u2, v2*u1, v1*v2, v2, u1, v1, 1]

    # SVD
    U, S, VT = np.linalg.svd(W, full_matrices=True)
    f = VT[-1, :]   # 最后一行为最优解
    F_hat = np.reshape(f, (3, 3))    # 最小二乘的近似

    # 计算rank =2 的F
    U, S_hat, VT = np.linalg.svd(F_hat, full_matrices=True)
    s = np.zeros((3, 3))   # sigma 矩阵
    s[0, 0] = S_hat[0]
    s[1, 1] = S_hat[1]  # sigma 的Rank为2
    F = np.dot(U, np.dot(s, VT))

    return F
```

## 2. 归一化八点算法
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

----
整个算法分为三个步骤：

- 归一化，对每个点进行缩放
- 八点算法计算Fq,
- 反归一化

(1) 归一化： 先平移到新坐标系，在缩放
![](http://latex.codecogs.com/gif.latex?%5Cbegin%7Bbmatrix%7D%20%26s_%7Bx%7D%20%260%20%26s_%7Bx%7Dt_%7Bx%7D%5C%5C%20%260%20%26s_%7By%7D%20%26s_%7By%7Dt_%7By%7D%5C%5C%20%260%20%260%20%261%20%5Cend%7Bbmatrix%7D%20*%20%5Cbegin%7Bbmatrix%7D%20x%5C%5C%20y%5C%5C%201%20%5Cend%7Bbmatrix%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20s_%7Bx%7D%28x&plus;t_%7Bx%7D%29%5C%5C%20s_%7By%7D%28y&plus;t_%7By%7D%29%5C%5C%201%20%5Cend%7Bbmatrix%7D)

其中，缩放系数为：
![](http://latex.codecogs.com/gif.latex?%5Csqrt%7B2/%28%5Csum_%7Bi%3D0%7D%5E%7Bn%7D%28x_%7Bi%7D%20-%20%5Coverline%7Bx%7D%29%5E%7B2%7D%20/%20N%29%20%7D)

代码为：
```
def normalized_eight_point_alg(points1, points2):
    N = points1.shape[0]
    points1_uv = points1[:, 0:2]
    points2_uv = points2[:, 0:2]    # 取x,y 坐标
    #
    # 取坐标均值
    points1_mean = np.mean(points1_uv, axis=0)
    points2_mean = np.mean(points2_uv, axis=0)

    # 点集的到中心的差
    points1_new = points1_uv - points1_mean
    points2_new = points2_uv - points2_mean

    # 计算缩放参数
    scale = np.sqrt(np.sum(points1_new**2)/N)
    scale1 = np.sqrt(2 / (np.sum(points1_new**2)/N * 1.0))
    scale2 = np.sqrt(2 / (np.sum(points2_new**2)/N * 1.0))

    # 归一化矩阵
    T1 = np.array([
        [scale1, 0, -points1_mean[0] * scale1],
        [0, scale1, -points1_mean[1] * scale2],
        [0, 0, 1]
    ])

    T2 = np.array([
        [scale2, 0, -points1_mean[0] * scale1],
        [0, scale2, -points1_mean[1] * scale2],
        [0, 0, 1]
    ])

    # 对坐标点变换
    q1 = T1.dot(points1.T).T    # N * 3
    q2 = T2.dot(points2.T).T    # N * 3

    # 八点算法
    Fq = lls_eight_point_alg(q1, q2)

    #反归一化
    F = T2.T.dot(Fq).dot(T1)

    return F


```


## 3. 计算平均距离
- 通过公式将p2的点映射成为p1面上的一条直线---极线

![](https://upload-images.jianshu.io/upload_images/5361608-47c66c9adca14dc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 可以通过点到直线的距离公式计算平均距离：

![](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=930595087,3467127615&fm=58)

代码为：

```
def compute_distance_to_epipolar_lines(points1, points2, F):
    # F.Tp2 = l, 求得p2到p1面上的映射直线
    line = F.T.dot(points2.T)  # 3 * N

    dis_sum = 0
    N = points1.shape[0]

    for i in range(N):
        x = points1[i, 0]
        y = points1[i, 1]
        A = line[0, i]
        B = line[1, i]
        C = line[2, i]
        dis_sum += np.abs(A*x + B*y + C) / np.sqrt(A**2 + B**2)

    return dis_sum / N      # 平均距离
```

## 4. 画出极线

```
def plot_epipolar_lines_on_images(points1, points2, im1, im2, F):
    plt.subplot(1, 2, 1)  # 建立1*2 的图
    line1 = F.T.dot(points2.T)   # p2到p1面上的极线
    N1 = line1.shape[1]     # 极线的数量
    for i in range(N1):
        plt.plot([0, im1.shape[1]],
                 [-line1[2][i] * 1.0 / line1[1][i], -(line1[2][i] + line1[0][i] * im1.shape[1]) * 1.0 / line1[1][i]], 'r')
        plt.plot([points1[i][0]], [points1[i][1]], 'b*')
    plt.imshow(im1, cmap='gray')

    plt.subplot(1, 2, 2)
    line2 = F.dot(points1.T)
    N2 = line2.shape[1]
    for i in range(N2):
        plt.plot([0, im2.shape[1]], [-line2[2][i] * 1.0 / line2[1][i], -(line2[2][i] + line2[0][i] * im2.shape[1]) / line2[1][i]],
                 'r')
        plt.plot([points2[i][0]], [points2[i][1]], 'b*')
    plt.imshow(im2, cmap='gray')
```

结果为：
```
Set: data/set1
--------------------------------------------------------------------------------
Fundamental Matrix from LLS  8-point algorithm:
 [[ 1.55218081e-06 -8.18161523e-06 -1.50440111e-03]
 [-5.86997052e-06 -3.02892219e-07 -1.13607605e-02]
 [-3.52312036e-03  1.41453881e-02  9.99828068e-01]]
Distance to lines in image 1 for LLS: 28.025662937533877
Distance to lines in image 2 for LLS: 25.162875800036915
p'^T F p = 0.03156399064220228
Fundamental Matrix from normalized 8-point algorithm:
 [[ 5.93261511e-07 -5.08492255e-06  8.76427688e-05]
 [-4.66834735e-06 -3.20108624e-07 -6.12207138e-03]
 [-7.74714403e-04  8.42028676e-03  1.25311400e-01]]
Distance to lines in image 1 for normalized: 0.9431072572196602
Distance to lines in image 2 for normalized: 0.8719800541568359
--------------------------------------------------------------------------------
Set: data/set2
--------------------------------------------------------------------------------
Fundamental Matrix from LLS  8-point algorithm:
 [[-5.63087200e-06  2.74976583e-05 -6.42650411e-03]
 [-2.77622828e-05 -6.74748522e-06  1.52182033e-02]
 [ 1.07623595e-02 -1.22519240e-02 -9.99730547e-01]]
Distance to lines in image 1 for LLS: 9.701438829435915
Distance to lines in image 2 for LLS: 14.568227190498229
p'^T F p = 0.03149037056281445
Fundamental Matrix from normalized 8-point algorithm:
 [[-1.53880961e-07  2.46528633e-06 -1.57563630e-04]
 [ 3.50323566e-06  3.08159735e-07  6.82243058e-03]
 [ 2.42265054e-04 -8.27925885e-03 -4.08002117e-03]]
Distance to lines in image 1 for normalized: 0.8955997529976532
Distance to lines in image 2 for normalized: 0.8959928005846117
```

![](https://upload-images.jianshu.io/upload_images/5361608-3ce26c6c5b0b7826.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




---

