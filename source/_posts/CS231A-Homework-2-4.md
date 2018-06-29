---
title: CS231A_Homework_2.4
date: 2018-06-29 16:22:58
tags:
	- Epipolar_Geometry
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-2:ps-code2-part-4

---

# 四、Triangulation
## 1 计算过程
在一个SFM问题中，
![](https://upload-images.jianshu.io/upload_images/5361608-6963a262144aef6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.1 变换矩阵
- 透视投影中，变换矩阵共有11个自由度；

![](https://upload-images.jianshu.io/upload_images/5361608-3edbf7feab88f62d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.1.1 模糊性
- 投影变换中，可以用一个任意的4*4矩阵来表达模糊性

![](https://upload-images.jianshu.io/upload_images/5361608-b04ee09379b657e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.1.2 问题
- 对于M张图片， n个3D点，存在变换：

![](https://upload-images.jianshu.io/upload_images/5361608-25be029e4a34cfa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 因此，SFM问题==> 从m*n的观察点中，估计m个3  * 4的矩阵和n个位置点；

- 上面可知：已知道的参数为：m张图片；
  - 如果相机未被校准，3d点也未知，不能计算 从3d点到2d的变换，也就无法求得相机矩阵；
  - 因此，【采用一种4 * 4投影矩阵，现将点投影过后进行变换】（这个没有搞懂）

-等式的需求：
  - 11*m + 3*n -15 个未知数需要 2m*n个等式

> 不懂为什么要减去15

---
## 2 计算方法

### 2.1 线性法（通过基础矩阵）
步骤为：
![](https://upload-images.jianshu.io/upload_images/5361608-6baddccdc7b64486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(1) **计算基础矩阵F**
- 使用投影变换H,使得：

![](https://upload-images.jianshu.io/upload_images/5361608-ca41e3160aebf191.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么对于，对应观察点：

![](https://upload-images.jianshu.io/upload_images/5361608-a068e0b2e384d123.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其中，在叉乘中，

![](https://upload-images.jianshu.io/upload_images/5361608-6245e42a46cbd158.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出基础矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-37c11d1916365451.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **F: 使用八点算法计算**

(3) **使用F估计投影相机**

在得到基础矩阵后，
那么，可以得出两个观察点之间变换的参数b的计算方法：

![](https://upload-images.jianshu.io/upload_images/5361608-d056a708c90daee4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，A矩阵的计算方法：

![](https://upload-images.jianshu.io/upload_images/5361608-c800177439cf3529.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参数b的性质符合下面：

![](https://upload-images.jianshu.io/upload_images/5361608-57da4b0fb203a2a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此，b是一个极点！
那么最终得到的矩阵为：

![](https://upload-images.jianshu.io/upload_images/5361608-d23d4cf324b53352.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


(3) **估计 3d点**

![](https://upload-images.jianshu.io/upload_images/5361608-7aa3144bd8758b69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 3 python 实现
### 3.1 估计两幅图片的变换矩阵RT
- 通过 本质矩阵求 R,T

- 在标准相机和非标准相机中，存在：

![](https://upload-images.jianshu.io/upload_images/5361608-581c6327efd6eaac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-0494233d51db29c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-47bdfea9649897c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-2b94f27a4cb512bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 因此可以通过分解本质矩阵来求得 R,T;

计算过程为：

![](https://upload-images.jianshu.io/upload_images/5361608-ca3aaeab50e1a5da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


【代码】

```python
def estimate_initial_RT(E):
    # SVD分解
    u, s, vt = np.linalg.svd(E)

    # 计算R
    z = np.array([
        [0, 1, 0],
        [-1, 0, 0],
        [0, 0, 0]
    ])

    w = np.array([
        [0, -1, 0],
        [1, 0, 0],
        [0, 0, 1]
    ])

    M = u.dot(z).dot(u.T)
    Q1 = u.dot(w).dot(vt)
    R1 = np.linalg.det(Q1) * Q1

    Q2 = u.dot(w.T).dot(vt)
    R2 = np.linalg.det(Q2) * Q2

    # 计算 T,T为u的第三列
    T1 = u[:, 2].reshape(-1,1)
    T2 = - T1

    # R 有两种， T有两种，一共四中可能
    R_set = [R1, R2]
    T_set = [T1, T2]
    RT_set = []
    for i in range(len(R_set)):
        for j in range(len(T_set)):
            RT_set.append(np.hstack( (R_set[i], T_set[j]) ))

    RT = np.zeros((4, 3, 4))
    for i in range(RT.shape[0]):
        RT[i, :, :] = RT_set[i]


    return RT


```


### 3.2 不同图片中的对应点

![](https://upload-images.jianshu.io/upload_images/5361608-34352a4e92bbf406.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 对应3D点和图片上的点，存在上`p = MP'`,在齐次坐标系下，得到上式；

![](https://upload-images.jianshu.io/upload_images/5361608-4f0424a16bb4daa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此，A矩阵为（）：

![](https://latex.codecogs.com/gif.latex?%5Cbegin%7Bbmatrix%7D%20m_%7B1%7D-u_%7B1%7Dm_%7B3%7D%20%5C%5C%20m_%7B2%7D-v_%7B1%7Dm_%7B3%7D%5C%5C%20...%5C%5C%20m_1-u_%7Bn%7Dm_%7B3%7D%5C%5C%20m_%7B2%7D-v_%7Bn%7Dm_%7B3%7D%20%5Cend%7Bbmatrix%7D)

- 这里， M是已知的，将 3D点P作为未知数；
- 使用svd分解；

【代码】

```python

def linear_estimate_3d_point(image_points, camera_matrices):
    # 传入两个点， 两个相机矩阵
    N = image_points.shape[0]   # N == 2
    # 建立系数矩阵
    A = np.zeros((2*N, 4))  # 每一点建立两个等式
    A_set = []

    for i in range(N):
        pi = image_points[i]
        Mi = camera_matrices[i]
        x = pi[0] * Mi[2] - Mi[0]
        y = pi[1] * Mi[2] - Mi[1]
        A_set.append(x)
        A_set.append(y)


    for i in range(A.shape[0]):
        A[i] = A_set[i]

    u, s, vt = np.linalg.svd(A)
    p = vt[-1]      # 取最后一列
    p = p / p[-1]       # 齐次坐标，最后一个为1
    p = p[:3]       # 取前三个，3D坐标

    return p

```


### 3.3 优化：计算对应点的重投影误差
- 给一张图片i ,给定一个相机矩阵 Mi, 还有3D 的点P,得到：

![](https://upload-images.jianshu.io/upload_images/5361608-28a616f7ee46a342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-a241e0b53d76f2d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

投影误差为：

![](https://upload-images.jianshu.io/upload_images/5361608-c5c14c63438ac4bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 每个e都是(2 * 1)的向量，可以对e求导，有K个相机位置，那么雅克比矩阵为（ 2K * 3）：

![](https://upload-images.jianshu.io/upload_images/5361608-ca2844b513ce5a5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 本题提供一个真实3d点：
- 对一个点求导得：

![](https://upload-images.jianshu.io/upload_images/5361608-dea44473877e5b3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 一个投影点有(x,y)，得到两个等式，可得两个偏导数

代码为：

```python
def reprojection_error(point_3d, image_points, camera_matrices):
    # 图像点的数量    3d点：1， 2d点： 2， 相机矩阵 2
    N = image_points.shape[0]
    # 建立其次坐标系
    point_3d_homo = np.hstack((point_3d, 1))

    error_set = []
    # 计算误差
    for i in range(N):
        pi = image_points[i]
        Mi = camera_matrices[i]
        Yi = Mi.dot(point_3d_homo)  # 3d点的投影

        pi_prime = 1.0 / Yi[2] * np.array([Yi[0], Yi[1]])       # 转变为2d点
        error_i = pi_prime - pi
        error_set.append(error_i[0])        # x 误差
        error_set.append(error_i[1])        # y 误差

    # 变换成 array
    error = np.array(error_set)
    return error


def jacobian(point_3d, camera_matrices):
    # 一个 3D点， 2个相机矩阵
    K = camera_matrices.shape[0]

    J = np.zeros((2 * K, 3))        # 2*k, 3

    #  齐次坐标系
    point_3d_homo = np.hstack((point_3d, 1))

    J_tmp = []
    # 计算 雅克比矩阵
    for i in range(K):
        Mi = camera_matrices[i]
        pi = Mi.dot(point_3d_homo)      # 投影点
        Jix =(pi[2]*Mi[0] - pi[0] * Mi[2] ) / pi[2]**2      # x
        Jiy =(pi[2]*Mi[1] - pi[1] * Mi[2]) / pi[2]**2       # y
        J_tmp.append(Jix)
        J_tmp.append(Jiy)

    for i in range(J.shape[0]):
        J[i] = J_tmp[i][:3]         # J_tmp 每一行包含齐次坐标，去掉最后一列
    return J


```


### 3.4  高斯牛顿法
- 寻找一个近似最小化重投影误差；

![](https://upload-images.jianshu.io/upload_images/5361608-6c176df1512910f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


代码为：

```python
def nonlinear_estimate_3d_point(image_points, camera_matrices):
    # 计算初始估计的真实点
    P = linear_estimate_3d_point(image_points, camera_matrices)

    # 迭代10 次
    for i in range(10):
        e = reprojection_error(P, image_points, camera_matrices)
        J = jacobian(P, camera_matrices)
        P -= np.linalg.inv(J.T.dot(J)).dot(J.T).dot(e)

    return P
```

### 3.5 估计 R, T
- 估计最有可能的R,T变换；
- 在第一个问题中，从本质矩阵中估计了4个 最有可能的RT矩阵，本次从中选择最优的一个；

1. 首先， 对每一对R,T ， 计算3D点的对应位置；
2. 正确的R,T从图片中估计的3D点拥有正向的深度值，即z值；
3. 正向的值是相对于其相对的相机位置；所以，3d点先要变换到当前相机坐标系下；

```python
def estimate_RT_from_E(E, image_points, K):
    # 获得四队RT
    RT = estimate_initial_RT(E)
    count = np.zeros((1, 4))        # 每一对的估计得分
    I0 = np.array([[1.0, 0, 0, 0],
                   [0, 1.0, 0, 0],
                   [0, 0, 1.0, 0]])     # [I 0] 设第一个相机为初始位置

    M1 = K.dot(I0)      # 第一个相机的变换矩阵

    camera_matrices = np.zeros((2, 3, 4))       # 建立两个相机变换矩阵
    camera_matrices[0] = M1


    for i in range(RT.shape[0]):        # 对每一个RT进行测试
        rt = RT[i]
        M2_i = K.dot(rt)
        camera_matrices[1] = M2_i       # 临时第二个相机变换矩阵
        for j in range(image_points.shape[0]):      # 估计 3D点
            point_j_3d = nonlinear_estimate_3d_point(image_points[j], camera_matrices)
            pj_homo = np.vstack((point_j_3d.reshape(3, 1), [1]))      # 转换为齐次坐标
            pj_prime = camera1tocamera2(pj_homo, rt)      # 将3d点变换到相对的相机坐标系下
            if pj_homo[2] > 0 and pj_prime[2] > 0:
                count[0, i] += 1     # 3D点面向相机 ，z值为正， 计算正确


    maxIdx = np.argmax(count)
    maxRT = RT[maxIdx]
    return maxRT


def camera1tocamera2(P, RT):
    P_homo = np.array([P[0], P[1], P[2], 1.0])
    A = np.zeros((4, 4))
    A[0:3, :] = RT
    A[3, :] = np.array([0, 0, 0, 1.0])

    P_prim_homo = A.dot(P_homo.T)
    P_prim_homo /= P_prim_homo[3]
    P_prime = P_prim_homo[0:3]

    return P_prime


```

## 4. 结果
```
Part A: Check your matrices against the example R,T
--------------------------------------------------------------------------------
Example RT:
 [[ 0.9736 -0.0988 -0.2056  0.9994]
 [ 0.1019  0.9948  0.0045 -0.0089]
 [ 0.2041 -0.0254  0.9786  0.0331]]
Estimated RT:
 [[[ 0.98305251 -0.11787055 -0.14040758  0.99941228]
  [-0.11925737 -0.99286228 -0.00147453 -0.00886961]
  [-0.13923158  0.01819418 -0.99009269  0.03311219]]

 [[ 0.98305251 -0.11787055 -0.14040758 -0.99941228]
  [-0.11925737 -0.99286228 -0.00147453  0.00886961]
  [-0.13923158  0.01819418 -0.99009269 -0.03311219]]

 [[ 0.97364135 -0.09878708 -0.20558119  0.99941228]
  [ 0.10189204  0.99478508  0.00454512 -0.00886961]
  [ 0.2040601  -0.02537241  0.97862951  0.03311219]]

 [[ 0.97364135 -0.09878708 -0.20558119 -0.99941228]
  [ 0.10189204  0.99478508  0.00454512  0.00886961]
  [ 0.2040601  -0.02537241  0.97862951 -0.03311219]]]
--------------------------------------------------------------------------------
Part B: Check that the difference from expected point 
is near zero
--------------------------------------------------------------------------------
Difference:  0.0029243053036863698
--------------------------------------------------------------------------------
Part C: Check that the difference from expected error/Jacobian 
is near zero
--------------------------------------------------------------------------------
Error Difference:  8.301299988565727e-07
Jacobian Difference:  1.817115702351657e-08
--------------------------------------------------------------------------------
Part D: Check that the reprojection error from nonlinear method
is lower than linear method
--------------------------------------------------------------------------------
Linear method error: 98.73542356894183
Nonlinear method error: 95.59481784846034
--------------------------------------------------------------------------------
Part E: Check your matrix against the example R,T
--------------------------------------------------------------------------------
Example RT:
 [[ 0.9736 -0.0988 -0.2056  0.9994]
 [ 0.1019  0.9948  0.0045 -0.0089]
 [ 0.2041 -0.0254  0.9786  0.0331]]
Estimated RT:
 [[ 0.97364135 -0.09878708 -0.20558119  0.99941228]
 [ 0.10189204  0.99478508  0.00454512 -0.00886961]
 [ 0.2040601  -0.02537241  0.97862951  0.03311219]]
--------------------------------------------------------------------------------
Part F: Run the entire SFM pipeline
--------------------------------------------------------------------------------
```

![](https://upload-images.jianshu.io/upload_images/5361608-cd68806911f3d3b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

