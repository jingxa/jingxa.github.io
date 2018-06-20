---
title: cs231a-homework-1
date: 2018-06-19 22:05:44
tags: 
- cs231A 
- homeworks
categories: 
- cs231A
---
# 一、affine camera Calibration

## 1.1 相机参数 计算

![](https://upload-images.jianshu.io/upload_images/5361608-03fce6331a4ddc1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-2ff53ebf6a86b559.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 小块大小： 50 mm * 50 mm
- 小块间隔： 30mm
- 总大小： 450 mm * 450 mm

---
- 计算思路：
- 查看ps1_code中的三个npy文件的shape
- real_XY:(12,2)
- front:(12,2)
- back:(12,2)
说明选取了十二个点作为测量点，对于3D到2D的变换：
- `p' = M P`

在相机校准中，相机的参数矩阵为：(3 * 4),一共有11个自由度；

- 但是本题，是affine camera， 是弱透视投影，因此，参数为：

![](https://upload-images.jianshu.io/upload_images/5361608-88c8a13eac856914.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 只需要计算8个自由度；

![](https://upload-images.jianshu.io/upload_images/5361608-b8ec2e79577d7b2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- m3为[0 0 0 1]

![](https://upload-images.jianshu.io/upload_images/5361608-a91e9bee8aaa4231.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 则m3Pi = 1

- 因此，可以建立上图中的P矩阵为：(2n * 8)的矩阵，n为参考点个数；

- m 为： (8 * 1),省略m3,到时候添加上一行[0 0 0 1]就可以

- 因此线性方程变为： Pm=0 ==>Am=b

使用最小二乘，求解：

![](https://upload-images.jianshu.io/upload_images/5361608-3c782966f307b92b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####在我们的方程中：AM =b
- A： 2n * 8
- m: 8 * 1
- b = 2n * 1



![](https://latex.codecogs.com/gif.latex?minimize%20%5Cleft%20%5C%7C%20Am%20-b%20%5Cright%20%5C%7C%5E%7B2%7D%20%2C)

对m求导，得到 

![](https://latex.codecogs.com/gif.latex?%5Cleft%20%5C%7C%20Am-b%20%5Cright%20%5C%7C%5E%7B2%7D%20%3D%20A%5E%7B2%7Dm%5E%7B2%7D-2Amb%20&plus;%20b%5E%7B2%7D%20%5C%5C%20%3D%3E%202A%5E%7B2%7Dm%20-%202A%5E%7BT%7Db%20%3D0%20%5C%5C%20%3D%3E%20m%20%3D%20%28A%5E%7B2%7D%29%5E%7B-1%7DA%5E%7BT%7Db%20%5C%5C%20%3D%3E%20m%20%3D%20%28A%5E%7BT%7DA%29%5E%7B-1%7DA%5E%7BT%7Db)


因此，计算 affine camera的矩阵为：

```python

def compute_camera_matrix(real_XY, front_image, back_image):
    img_num1 = front_image.shape[0] 
    img_num2 = back_image.shape[0]

    # 建立真实XYZ的齐次矩阵
    ones = np.ones((img_num1, 1))
    front_z = np.zeros((img_num1, 1))
    front_scence = np.c_[real_XY, front_z, ones]  # 12 * 4

    back_z =150 * np.ones((img_num2,1))
    back_scence = np.c_[real_XY, back_z, ones]  # 12 * 4

    # 合并两个真实场景
    M_scene = np.r_[front_scence, back_scence]  # 24 * 4

    # 系数矩阵 A  2n * 8
    n = img_num1 + img_num2
    A = np.zeros((2*n, 8))
    for i in range(0, A.shape[0], 2):
        idx = int(i/2)
        A[i, :] = np.hstack((M_scene[idx, :], [0, 0, 0, 0]))
        A[i+1, :] = np.hstack(([0, 0, 0, 0], M_scene[idx, :]))

    # 图片对应点矩阵 2n * 1 
    # b = [U1,V1, U2,V2,..., Un,Vn]
    b = front_image[0].T
    for i in range(1, img_num1, 1):
        b = np.hstack((b, front_image[i].T))
    for j in range(img_num2):
        b = np.hstack((b, back_image[j].T))
    b = np.reshape(b, (2 * n, 1))

    # 计算矩阵，添加最后一行
    # p = np.linalg.lstsq(A, b, rcond=None)  # 直接计算 AM= b ==> M = A^(-1)*b
    # camera_matrix = p[0]

    camera_matrix = np.linalg.inv(A.T.dot(A)).dot(A.T).dot(b)   # 使用最小二乘计算结果
    camera_matrix = np.reshape(camera_matrix, (2, -1)) # m(8,1) ==> m(2,4)
    camera_matrix = np.vstack((camera_matrix, [0, 0, 0, 1])) # 添加最后一列 ==> m（3,4）
    return camera_matrix
```
- 其中，使用两中方法：
- 计算 m = A^(-1)*b
- 计算最小二乘法

两个的结果为；

- 方法一：
```
 [[ 5.31276507e-01 -1.80886074e-02  1.20509667e-01  1.29720641e+02]
 [ 4.84975447e-02  5.36366401e-01 -1.02675222e-01  4.43879607e+01]
 [ 0.00000000e+00  0.00000000e+00  0.00000000e+00  1.00000000e+00]]
```
- 方法二：

```
[[ 5.31276507e-01 -1.80886074e-02  1.20509667e-01  1.29720641e+02]
 [ 4.84975447e-02  5.36366401e-01 -1.02675222e-01  4.43879607e+01]
 [ 0.00000000e+00  0.00000000e+00  0.00000000e+00  1.00000000e+00]]
```

- 两者之间的差为：

```
[[ 3.55271368e-15  5.19723153e-15 -1.38777878e-17  0.00000000e+00]
 [ 2.20060081e-13 -1.89404048e-13  3.06282777e-14  2.84217094e-14]
 [ 0.00000000e+00  0.00000000e+00  0.00000000e+00  0.00000000e+00]]

```
可以看出每一项的差很小,说明两种方法近似；

---
## 1.2 RMS 计算

- RMS : 

![](https://upload-images.jianshu.io/upload_images/5361608-53ce720a360065b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```python
def rms_error(camera_matrix, real_XY, front_image, back_image):
    img_num1 = front_image.shape[0]
    img_num2 = back_image.shape[0]

    ones = np.ones((img_num1,1))
    # 建立图像XYZ的齐次矩阵
    front_img = np.c_[front_image, ones]     # front_image : n * 3
    back_img = np.c_[back_image, ones]   # back_image: n * 3
    img = np.r_[front_img, back_img]
    img = img.T  # img : 3 * 2n

    # 建立真实XYZ的齐次矩阵
    front_z = np.zeros((img_num1, 1))
    front_scence = np.c_[real_XY, front_z, ones]  # n * 4

    back_z =150 * np.ones((img_num2,1))
    back_scence = np.c_[real_XY, back_z, ones]  # n * 4

    # 合并两个真实场景
    M_scene = np.r_[front_scence, back_scence]  # 2n * 4
    M_scene = M_scene.T  # real : 4 * 2n

    M_scene_trans = camera_matrix.dot(M_scene)  # 变换
    diff_sqr = (M_scene_trans - img)**2 # 平方差
    diff_sum = np.sum(np.sum(diff_sqr,axis=0))  # 平方差的和，先行相加，在列相加
    diff_sum /= (img_num1 + img_num2)   # 求均值

    rms_error = np.sqrt(diff_sum)
    return rms_error

```

## 1.3 主函数
- python 3.6
这两个函数的主函数：

```python
import numpy as np
if __name__ == '__main__':
    # Load the example coordinates setup.
    real_XY = np.load('real_XY.npy')
    front_image = np.load('front_image.npy')
    back_image = np.load('back_image.npy')

    camera_matrix = compute_camera_matrix(real_XY, front_image, back_image)
    rmse = rms_error(camera_matrix, real_XY, front_image, back_image)
    print ("Camera Matrix:\n", camera_matrix)
    print()
    print ("RMS Error: ", rmse)
```

![](https://upload-images.jianshu.io/upload_images/5361608-18036e390c98df9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.4 使用两个位置平面的原因
- 如果只使用一个平面：
  - 线性方程的系数矩阵： rank < n; 非满秩矩阵，没有唯一解，

![](https://upload-images.jianshu.io/upload_images/5361608-7bbbfc7ee4db2e68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个方阵是奇异矩阵，没有可逆矩阵；

- 因此，需要至少两个平面；

---
# 二、单视图几行

## 2.1 灭点

- 相机 no-skew ：没有偏斜
- square pixels with   no distortion : 没有扭曲

- 在3d中平行的线投影到2D中，相交于一点---灭点；因此，通过两线相交求得灭点；


```python
'''
COMPUTE_VANISHING_POINTS
Arguments:
    points - 四对点，前两对为一条直线，后两对为一条，两条线是平行的；
Returns:
    灭点
'''
def compute_vanishing_point(points):
    # 获取四对点
    x1 = points[0, 0]
    y1 = points[0, 1]   # (x1,y1)
    x2 = points[1, 0]
    y2 = points[1, 1]   # (x2,y2)
    x3 = points[2, 0]
    y3 = points[2, 1]   # （x3,y3）
    x4 = points[3, 0]
    y4 = points[3, 1]   # (x4,y4)

    # 计算两条直线的参数
    a1 = (y2 - y1)/(x2 - x1)
    b1 = y1 - a1 * x1
    print("line1: ", a1, b1)

    a2 = (y4 - y3)/(x4 - x3)
    b2 = y3 - a2 * x3
    print("line2:", a2, b2)

    # 计算交点
    x = (b1 - b2)/(a2 - a1)
    y = a1 * x + b1
    vanish_point = np.array([x, y])
    return vanish_point
```

## 2.2 计算相机内参矩阵K
- 使用三对灭点计算内部参数
- 无穷线的投影变成了水平线：

因此将一对3d平行线的无穷交点投影成一个灭点：

![](https://upload-images.jianshu.io/upload_images/5361608-ccbc7da86c7231c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设两对平行线的灭点为v1,v2, 两对平行线的方向分别为d1,d2，那么这每一对平行线的夹角为：

![](https://upload-images.jianshu.io/upload_images/5361608-3f0fee0cbccdce69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-344f019d6232aca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-f0888390d19d82e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于一个标准相机,可以将w简化为：

![](https://upload-images.jianshu.io/upload_images/5361608-2af4fad907d2ad90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-0d9e8622d8de703b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 得4个未知参数，然而w是可以缩放的，因此w6最后缩放为1,只需要3个未知变量；

- 因此，只需要三对灭点就可以求得矩阵K;

![](https://upload-images.jianshu.io/upload_images/5361608-d37dfb8754ca5585.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 每一个式子可以求解一个未知解；

![](https://latex.codecogs.com/gif.latex?%5Bx_%7B1%7D%2Cy_%7B1%7D%2C1%5D*%5Cbegin%7Bbmatrix%7D%20%26w_%7B1%7D%20%260%20%26w_%7B4%7D%5C%5C%20%260%20%26w_%7B1%7D%20%26w_%7B5%7D%20%5C%5C%20%26w_%7B4%7D%20%26w_%7B5%7D%20%26w_%7B6%7D%20%5Cend%7Bbmatrix%7D*%5Bx_%7B2%7D%2Cy_%7B2%7D%2C1%5D%20%3D%200)

化简为：

![](https://latex.codecogs.com/gif.latex?%28x_%7B1%7Dx_%7B2%7D&plus;y_%7B1%7Dy_%7B2%7D%29w_%7B1%7D%20&plus;%20%28x_%7B1%7D%20&plus;%20x_%7B2%7D%29w_%7B4%7D&plus;%28y_%7B1%7D%20&plus;%20y_%7B2%7D%29w_%7B5%7D&plus;w_%7B6%7D%3D0)

w6 最小化1；

- 说明w是一个对称矩阵，并且是正定矩阵，那么可以在最后，使用Cholesky分解法进行分解；

- ## [cholesky分解](https://blog.csdn.net/billbliss/article/details/78559387)

因此，通过三对灭点构建3*4系数矩阵;
代码如下：

```python
'''
COMPUTE_K_FROM_VANISHING_POINTS
Arguments:
    vanishing_points - a list of vanishing points

Returns:
    K - the intrinsic camera matrix (3x3 matrix)
'''
def compute_K_from_vanishing_points(vanishing_points):
    v1 = vanishing_points[0]
    v2 = vanishing_points[1]
    v3 = vanishing_points[2]

    # 构建系数矩阵
    A = np.zeros((3, 4))
    A[0] = np.array([(v1[0]*v2[0] + v1[1]*v2[1]), (v1[0] + v2[0]), (v1[1] + v2[1]), 1])
    A[1] = np.array([(v1[0]*v3[0] + v1[1]*v3[1]), (v1[0] + v3[0]), (v1[1] + v3[1]), 1])
    A[2] = np.array([(v2[0]*v3[0] + v2[1]*v3[1]), (v2[0] + v3[0]), (v2[1] + v3[1]), 1])

    # print("A:\n",A)
    # SVD分解
    U, s, vT = np.linalg.svd(A, full_matrices=True)

    # print('SVD:\n',U,U.shape)
    # print('s:\n',s, s.shape)
    # print('v:\n',vT,vT.shape)
    # print()

    w = vT[-1, :]   # 取最后一行，最为最优解
    print('w:\n',w, w.shape)
    omega = np.array([  # 建立w矩阵
        [w[0], 0, w[1]],
        [0, w[0], w[2]],
        [w[1], w[2], w[3]]
    ], dtype=np.float64)

    print()
    # 使用cholesky 分解得到K
    kT_inv = np.linalg.cholesky(omega)  # w = (k*k.T)^-1 ==> 分解为 k.T^-1
    k = np.linalg.inv(kT_inv.T) 
    k /= k[2, 2]    # 最小化
    return k

```
【注意】
-  三对灭点： 需要正交
- 为了减小误差，可以使用超过两条以上的平行线计算灭点，然后计算这个坐标的平均值作为最终灭点；

## 2.3  计算两个平面的夹角

- 通过上面的公式，通过灭点计算灭线，然后计算角度；
- 【问题】
  - 不明白为什么通过求点的叉积来获得灭线向量，感觉没有意义啊？？

```python
'''
COMPUTE_K_FROM_VANISHING_POINTS
Arguments:
    vanishing_pair1 - a list of a pair of vanishing points computed from lines within the same plane
    vanishing_pair2 - a list of another pair of vanishing points from a different plane than vanishing_pair1
    K - the camera matrix used to take both images

Returns:
    angle - the angle in degrees between the planes which the vanishing point pair comes from2
'''
def compute_angle_between_planes(vanishing_pair1, vanishing_pair2, K):
    omega_inv = K.dot(K.T)

    # a set of vanishing points on one plane
    v1 = np.hstack((vanishing_pair1[0], 1))
    v2 = np.hstack((vanishing_pair1[1], 1))

    # another set of vanishing points on the other plane
    v3 = np.hstack((vanishing_pair2[0], 1))
    v4 = np.hstack((vanishing_pair2[1], 1))

    # find two vanishing lines
    L1 = np.cross(v1.T, v2.T)  # 为什么如此？
    L2 = np.cross(v3.T, v4.T)

    # find the angle between planes
    costheta = (L1.T.dot(omega_inv).dot(L2)) / (np.sqrt(L1.T.dot(omega_inv).dot(L1)) * np.sqrt(L2.T.dot(omega_inv).dot(L2)))
    theta = (np.arccos(costheta) / math.pi) * 180

    return theta
```


## 2.4 计算旋转矩阵

![](https://upload-images.jianshu.io/upload_images/5361608-d09bf4c264e3b2d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-ccbc7da86c7231c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 假设相机只经过旋转，没有平移；
- 通过灭点求解的真实平行线的方向；
- 两幅图片的真实平行线是不变的，通过真实平行线的矩阵变换求得相机的旋转矩阵；


```python
'''
COMPUTE_K_FROM_VANISHING_POINTS
Arguments:
    vanishing_points1 - a list of vanishing points in image 1
    vanishing_points2 - a list of vanishing points in image 2
    K - the camera matrix used to take both images

Returns:
    R - the rotation matrix between camera 1 and camera 2
'''
def compute_rotation_matrix_between_cameras(vanishing_points1, vanishing_points2, K):

    ones = np.ones((vanishing_points1.shape[0], 1))
    # 建立齐次矩阵
    v1 = np.hstack((vanishing_points1, ones)).T
    v2 = np.hstack((vanishing_points2, ones)).T

    # 计算真实平行线的方向
    # d = K^-1 * v
    k_inv = np.linalg.inv(K)
    D1 = k_inv.dot(v1) / np.linalg.norm(k_inv.dot(v1),axis=0)   # 按列计算norm范数
    D2 = k_inv.dot(v2) / np.linalg.norm(k_inv.dot(v2),axis=0)

    # d2 = R * d1, d2.T = d1.T * R.T
    R = np.linalg.lstsq(D1.T, D2.T,rcond = None)[0].T

    return R
```


---

