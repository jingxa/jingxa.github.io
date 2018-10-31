---
title: CS131-Homework-3
date: 2018-10-29 10:12:08
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw3
---

# 内容

- Harries corner detector
- RANSAC
- HoG descriptor




# 相关： Panaorama Stitching(全景拼接)

全景图片的拼接过程一般如下：

1. 使用`Harries detector`寻找关键点
2. 建立每个关键点的descriptor, 比较两幅图片的descriptor，寻找keypoints对
3. 在keypoints对上，使用 `least-squares method`求解仿设变换矩阵
4. 使用RANSAC优化矩阵，然后对多幅图片进行变换，得到全景

额外的任务：
- 实现HoG descriptor



## 1. Harris Corner Detector

harris 的基本算法分为如下几步：

```python


def harris_corners(img, window_size=3, k=0.04):
    H, W = img.shape
    window = np.ones((window_size, window_size))

    response = np.zeros((H, W))
    #第一步： 偏导数
    dx = filters.sobel_v(img)
    dy = filters.sobel_h(img)

    # 第二步： 偏导数乘积
    dxx = dx * dx
    dyy = dy * dy
    dxy = dx * dy

    # 第三步： 形成矩阵
    mxx = convolve(dxx, window)
    mxy = convolve(dxy, window)
    myy = convolve(dyy, window)      #加权计算

    # 第四步： 计算response
    for i in range(H):
        for j in range(W):
            M = np.array([[mxx[i, j], mxy[i, j]], [mxy[i, j], myy[i, j]]])
            response[i, j] = np.linalg.det(M) - k * np.trace(M) ** 2

    return response

```


## 2. Describing and Matching Keypoints 

### 2.1 Descriptors 创建

函数： `simple_descriptor()`
参数： 
-	patch: 灰度图片的一个patch，大小（H,W）

返回值：
- features： 1D的数组(H * W)

实现：
- 将H * W 的patch排列为1D，然后归一化,采用高斯核（x - u）/delta 


```python
def simple_descriptor(patch):
    feature = []

    patch = patch.reshape(-1)
    mean = np.mean(patch)       # 均值
    delta = np.std(patch)       # 标准差
    if delta > 0.0:
        patch = (patch - mean) / delta
    else:
        patch = patch - mean

    feature = list(patch)
    return feature


```

## 2.2 descriptors匹配
- 通过计算两个descriptors的欧式距离来比较
- 点A：最近点B,第二近点C
- 如果（dist(A-B)/ dist(A-C)） 小于某个阈值，说明A与B为最佳配对

函数：`match_descriptors(desc1, desc2, threshold=0.5)`
参数：
- desc1,desc2: 两个描述符数组
- thresh： 阈值

返回值：
- matches:配对点


```python
def match_descriptors(desc1, desc2, threshold=0.5):
    matches = []

    N = desc1.shape[0]
    dists = cdist(desc1, desc2)     # 每个向量的欧式距离 N * N

    idx = np.argsort(dists, axis=1)     # 从小到大对dist排序，返回序号， N * N

    for i in range(N):
        closed_dist = dists[i, idx[i, 0]]
        second_dist = dists[i, idx[i, 1]]
        if(closed_dist < threshold * second_dist):      # 比较
            matches.append([i, idx[i, 0]])

    matches = np.array(matches)
    return matches


```


## 3. Transformation Estimation

-通过匹配点来估计两个图片的仿射变换，使用最小二乘法来计算

函数： `fit_affine_matrix(p1, p2)`

参数： 
- p1, p2: 两组对应点

返回值：
- H： 仿射变换

```python


def fit_affine_matrix(p1, p2):

    assert (p1.shape[0] == p2.shape[0]),\
        'Different number of points in p1 and p2'
    p1 = pad(p1)
    p2 = pad(p2)        # 齐次矩阵

    H = np.linalg.lstsq(p2, p1)[0]
    # Sometimes numerical issues cause least-squares to produce the last
    # column which is not exactly [0, 0, 1]
    H[:,2] = np.array([0, 0, 1])
    return H


```

## 4. RANSAC 优化匹配

一般步骤为：
1. 选择随机的匹配对
2. 计算仿射变换矩阵
3. 寻找RANSAC拟合的inliers
4. 重复直到找到包含inliers最多的模型
5. 重新计算最小二乘误差 在内点


函数： `ransac(keypoints1, keypoints2, matches, n_iters=200, threshold=20)`

参数：
- keypoints1, keypoints2: 关键点
- matches： 对应点序号
- n_iters: 迭代次数
- thresh:  阈值

返回值：
- H : 仿射变换矩阵

实现：

- RANSAC寻找的就是使得配对点数尽可能多的H。
- 首先，随机挑选对应对，使用最小二乘求解一个变换h,计算通过当前h，能够得到的最大对应点的数量
- 然后，迭代n_iters次，选择最大对应点数量的变换矩阵矩阵H
- 另一种实现方式，可以查看[cs231A_Homework_3.2]

```python
def ransac(keypoints1, keypoints2, matches, n_iters=200, threshold=20):

    # Copy matches array, to avoid overwriting it
    orig_matches = matches.copy()
    matches = matches.copy()

    N = matches.shape[0]
    print(N)
    n_samples = int(N * 0.2)                            # 随机取样

    matched1 = pad(keypoints1[matches[:, 0]])            #第一列的序号 齐次矩阵
    matched2 = pad(keypoints2[matches[:, 1]])            # 第二列的序号

    max_inliers = np.zeros(N)
    n_inliers = 0

    # RANSAC iteration start
    for i in range(n_iters):

        temp_max = np.zeros(N, dtype=np.int32)      # 临时变量
        temp_n = 0

        idx = np.random.choice(N, n_samples, replace=False)     # 随机抽取 n_samples
        p1 = matched1[idx, :]
        p2 = matched2[idx, :]
        H = np.linalg.lstsq(p2, p1)[0]              # 临时变换 H
        H[:, 2] = np.array([0, 0, 1])

        temp_max = np.linalg.norm(matched2.dot(H) - matched1, axis=1) ** 2 < threshold      # 计算当前对应点的数量
        temp_n = np.sum(temp_max)

        if temp_n > n_inliers:          # 保存最大数量
            max_inliers = temp_max.copy()
            n_inliers = temp_n

    H = np.linalg.lstsq(matched2[max_inliers], matched1[max_inliers])[0]
    H[:, 2] = np.array([0, 0, 1])
    return H, matches[max_inliers]

```




## 5. HoG 

HoG的计算步骤一般如下：

1. 计算图片的x,y方向上的偏导数，使用`skimage.filters`
2. 将图片划分多个block,每个block划分为cells,计算每个cell的梯度直方图
3. flatten 每个block为特征向量
4. 归一化


函数：`hog_descriptor(patch, pixels_per_cell=(8,8))`
参数：
- patch : 图片(H,W)
- pixels_per_cell: 每个cell的size(M,N)

返回值：
- block : 一个block的特征向量（（H*W*n_bins） / (M*N)）
实现：
- 将patch划分为多个cells,计算每个cells的梯度，然后计算梯度直方图
- 最后归一化



```python

def hog_descriptor(patch, pixels_per_cell=(8,8)):

    assert (patch.shape[0] % pixels_per_cell[0] == 0),\
                'Heights of patch and cell do not match'
    assert (patch.shape[1] % pixels_per_cell[1] == 0),\
                'Widths of patch and cell do not match'

    n_bins = 9
    degrees_per_bin = 180 // n_bins             # 8 个方向，每个方向 20 度

    # sobel求解梯度
    Gx = filters.sobel_v(patch)
    Gy = filters.sobel_h(patch)

    # 梯度值和方向
    G = np.sqrt(Gx**2 + Gy**2)
    theta = (np.arctan2(Gy, Gx) * 180 / np.pi) % 180

    # Group entries of G and theta into cells of shape pixels_per_cell, (M, N)
    #   G_cells.shape = theta_cells.shape = (H//M, W//N)
    #   G_cells[0, 0].shape = theta_cells[0, 0].shape = (M, N)
    G_cells = view_as_blocks(G, block_shape=pixels_per_cell)            # 划分为block
    theta_cells = view_as_blocks(theta, block_shape=pixels_per_cell)
    rows = G_cells.shape[0]
    cols = G_cells.shape[1]

    # For each cell, keep track of gradient histrogram of size n_bins
    cells = np.zeros((rows, cols, n_bins))

    # Compute histogram per cell
    for i in range(rows):
        for j in range(cols):
            for m in range(pixels_per_cell[0]):
                for n in range(pixels_per_cell[1]):
                    idx = int(theta_cells[i, j, m, n] // degrees_per_bin)       # 计算当前像素点 位于n_bins 直方图的那个区间
                    if idx == 9:    # 180 度
                        idx = 8
                    cells[i, j, idx] += G_cells[i, j, m, n]             # 统计

    cells = (cells - np.mean(cells)) / np.std(cells)        # 归一化
    block = cells.reshape(-1)

    return block



```



## 6. 图片融合

当两张图片融合的时候，可以看到明显的线条，使用一种叫做**linear blending**的方法平滑消除问题。

基本步骤如下：
1. 定义 left 和 right margins
2. 定义一个 weight matrix 对于图片1：
	- 左边的权重 等于1
	- 从left margin 到 right margin， 权重 递减为 1  to 0
3. 定一个 图片2 的 weight matrix
	- 在 right margin 的右边，定义 weight = 1
	- 从right margin to left margin, 权重递减 为 0 to 1
4. 将 weight matrix 应用对应的图片上
5. 合并图片



```python
def linear_blend(img1_warped, img2_warped):
    """
    Linearly blend img1_warped and img2_warped by following the steps:

    1. Define left and right margins (already done for you)
    2. Define a weight matrices for img1_warped and img2_warped
        np.linspace and np.tile functions will be useful
    3. Apply the weight matrices to their corresponding images
    4. Combine the images

    Args:
        img1_warped: Refernce image warped into output space
        img2_warped: Transformed image warped into output space

    Returns:
        merged: Merged image in output space
    """
    out_H, out_W = img1_warped.shape # Height and width of output space
    img1_mask = (img1_warped != 0)  # Mask == 1 inside the image
    img2_mask = (img2_warped != 0) # Mask == 1 inside the image

    # Find column of middle row where warped image 1 ends
    # This is where to end weight mask for warped image 1
    # np.fliplr 左右翻转； np.argmax:最大值的下标
    right_margin = out_W - np.argmax(np.fliplr(img1_mask)[out_H//2, :].reshape(1, out_W), 1)[0] # 最大值列

    # Find column of middle row where warped image 2 starts
    # This is where to start weight mask for warped image 2
    left_margin = np.argmax(img2_mask[out_H//2, :].reshape(1, out_W), 1)[0]

    left_matrix = np.array(img1_mask,  dtype=np.float64)        # 非常重要。转换为浮点类型
    right_matrix = np.array(img2_mask, dtype=np.float64)

    # 渐进变换区域
    left_matrix[:, left_margin: right_margin] = np.tile(np.linspace(1, 0, right_margin - left_margin), (out_H, 1))
    right_matrix[:, left_margin: right_margin] = np.tile(np.linspace(0, 1, right_margin - left_margin), (out_H, 1))

    img1 = left_matrix * img1_warped
    img2 = right_matrix * img2_warped

    merged = img1 + img2

    return merged

```


上述方法存在一些问题，因为在两幅图片的margin判断的时候，可能没有取到边界的地方，导致结果图片中存在错误。

可以使用下面的一种方法，先计算


```
merged = img1_warped + img2_warped 

img1_mask = img1_mask * 1.0
img2_mask = img2_mask * 1.0
 

merged = img1_warped + img2_warped

overlap = (img1_mask * 1.0 + img2_mask)

### YOUR CODE HERE
#找到填充图像2的左边界，也就是列像素不全为0的位置
for col in range(img2_warped.shape[1]):
    if not np.all(img2_mask[:,col]==False):
        break
left_region = col 
right_region = img1.shape[1]
width_region = right_region - left_region + 1

for col in range(left_region, right_region +1):
    for row in range(img2_warped.shape[0]):
        alpha = 1 - (col - left_region)/width_region
        if img1_mask[row, col] and img2_mask[row, col]:
            merged[row, col] = alpha * img1_warped[row, col] + (1 - alpha) * img2_warped[row, col]

output = merged        
### END YOUR CODE


```





## 7. 全景融合

- 略



# 2. 结果

- [my_cs131_hw3](https://github.com/jingxa/CS131_release/blob/master/hw3_release/hw3.ipynb)

# 3. 参考资料

- [Jack-An/CS131](https://github.com/Jack-An/CS131/blob/master/hw3_release/hw3.ipynb)
- [nizihabi/Stanford_CS131_HW](https://github.com/nizihabi/Stanford_CS131_HW/blob/master/hw3_release/hw3.ipynb)
- [mikucy/CS131](https://github.com/mikucy/CS131/blob/master/hw3_release/hw3.ipynb)
- [wwdguu/CS131_homework](https://github.com/wwdguu/CS131_homework/blob/master/hw3_release/hw3.ipynb)
---

