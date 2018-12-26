---
title: CS131_Homework_5
date: 2018-12-08 16:14:42
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw5
---

# 主要内容

本节主要内容是`clustering`和`image segmentation`


### 参考资料

- [mikucy/CS131](https://github.com/mikucy/CS131/tree/master/hw5_release)
- [wwdguu/CS131_homework](https://github.com/wwdguu/CS131_homework/blob/master/hw5_release/segmentation.py)
- [Hugstar/Solutions-Stanford-cs131-Computer-Vision-Foundations-and-Application](https://github.com/Hugstar/Solutions-Stanford-cs131-Computer-Vision-Foundations-and-Application/blob/master/hw5_release/segmentation.py)


# 1. clustering algorithms

- K-Means clustering
- Hierarchical Agglomerative Clustering


## 1.1 K-Means clustering

函数：`kmeans(features, k, num_iters=100)`
参数：
- features: 特征向量 (N, a feature vector)
- k: 聚簇数量
- num_iters:迭代次数
返回值:
- assignments: cluster集合

实现步骤：
1. 随机初始化 cluster centers
2. 将每个点分给最近的中心点
3. 计算每个cluster新的中心点
4. 重复以上步骤直到没有改变

```python
def kmeans(features, k, num_iters=100):

    N, D = features.shape

    assert N >= k, 'Number of clusters cannot be greater than number of points'

    # Randomly initalize cluster centers
    idxs = np.random.choice(N, size=k, replace=False)
    centers = features[idxs]        # 1. 随机中心点
    assignments = np.zeros(N)

    for n in range(num_iters):
        ### YOUR CODE HERE
        # 2. 分类
        for i in range(N):
            dist = np.linalg.norm(features[i] - centers, axis=1)    # 每个点和中心点的距离
            assignments[i] = np.argmin(dist)        # 第i个点属于最近的中心点

        pre_centers = centers.copy()
        # 3. 重新计算中心点
        for j in range(k):
            centers[j] = np.mean(features[assignments == j], axis=0)

        # 4. 验证中心点是否改变
        if np.array_equal(pre_centers, centers):
            break
        ### END YOUR CODE

    return assignments 
 
 
```


函数： `kmeans_fast(features, k, num_iters=100)`
实现：
- 本次实现应该比上个函数快10倍
- 使用向量运算,`np.repeat`和`np.argmin`


```python

def kmeans_fast(features, k, num_iters=100):

    N, D = features.shape

    assert N >= k, 'Number of clusters cannot be greater than number of points'

    # Randomly initalize cluster centers
    idxs = np.random.choice(N, size=k, replace=False)
    centers = features[idxs]
    assignments = np.zeros(N)

    for n in range(num_iters):
        ### YOUR CODE HERE
        # 计算距离
        features_tmp = np.tile(features, (k, 1))        # (k*N, ...)
        centers_tmp = np.repeat(centers, N, axis=0)     # (N * k, ...)
        dist = np.sum((features_tmp - centers_tmp)**2, axis=1).reshape((k, N))      # 每列 即k个中心点
        assignments = np.argmin(dist, axis=0)   # 最近

        # 计算新的中心点
        pre_centers = centers
        # 3. 重新计算中心点
        for j in range(k):
            centers[j] = np.mean(features[assignments == j], axis=0)

        # 4. 验证中心点是否改变
        if np.array_equal(pre_centers, centers):
            break
        ### END YOUR CODE

    return assignments
```



## 1.2 Hierarchical Agglomerative Clustering

- 此种方法被称为HAC,每个点被初始化为一个cluster center,然后两两合并，直到剩下所需的最后的clusters

函数： `hierarchical_clustering(features, k):`
实现：
1. 将每个点作为一个cluster
2. 计算所有cluster的距离，合并两个最近的
3. 直到最后剩下所期望的clusters

`In practice, you probably do not want to use this algorithm to cluster more than 10,000 points.`

这次计算中，可以使用scipy的矩阵运算`squareform, pdist`
- squareform: 向量形式距离向量转换为矩形形式距离矩阵,反之亦然。
- pdist: 	Pairwise distances between observations in n-dimensional space.

```python

def hierarchical_clustering(features, k):

    N, D = features.shape

    assert N >= k, 'Number of clusters cannot be greater than number of points'

    # Assign each point to its own cluster
    assignments = np.arange(N)
    centers = np.copy(features)
    n_clusters = N

    while n_clusters > k:
        ### YOUR CODE HERE
        dist = pdist(centers)       # 计算相互之间的距离
        matrixDist = squareform(dist)   # 将向量形式变化为矩阵形式
        matrixDist = np.where(matrixDist != 0.0, matrixDist, 1e10)      # 将0.0的变为1e10,即为了矩阵中相同的点计算的距离去掉

        minValue = np.argmin(matrixDist)        # 最小的值的位置
        min_i = minValue // n_clusters          # 行号
        min_j = minValue - min_i * n_clusters   # 列号

        if min_j < min_i:       # 归并到小号的cluster
            min_i, min_j = min_j, min_i  # 交换一下

        for i in range(N):
            if assignments[i] == min_j:
                assignments[i] = min_i     # 两者合并

        for i in range(N):
            if assignments[i] > min_j:
                assignments[i] -= 1     # 合并了一个cluster,因此n_clusters减少一位

        centers = np.delete(centers, min_j, axis=0)  # 减少一个
        centers[min_i] = np.mean(features[assignments == min_i], axis=0)        # 重新计算中心点

        n_clusters -= 1     # 减去1

        ### END YOUR CODE

    return assignments


```


# 2. Pixel-Level Features
- 对于每个像素，最简单的clustering算法就是计算每个像素的特征，然后比较两个像素的特征值，如果类似，可以归为一个cluster

## 2.1 color features
- 例如，最简单的特征即为color

函数： `color_features(img):`

```
    Args:
        img - array of shape (H, W, C)

    Returns:
        features - array of (H * W, C)

```

实现：
- 比较两个相邻像素的颜色值即可

```python
### Pixel-Level Features
def color_features(img):
    H, W, C = img.shape
    img = img_as_float(img)
    features = np.zeros((H*W, C))

    ### YOUR CODE HERE
    features = img.reshape(H * W, C)        # color作为特征
    ### END YOUR CODE

    return features



```


## 2.2 Color and Position Features
- 将color和position联合起来作为特征，即特征为(r,g,b,x,y)
- 将颜色值为 \[0,1)的值
- 对于坐标： 归一化，0 均值和 1 方差

函数：`color_position_features(img)`

函数：
- np.mgrid： [numpy中mgrid与meshgrid的区别](https://blog.csdn.net/tymatlab/article/details/79027162)
- np.dstack : 堆栈数组按顺序深入（沿第三维）。[numpy中的stack操作：hstack()、vstack（）、stack（）、dstack（）、vsplit（）、concatenate（）](https://www.cnblogs.com/nkh222/p/8932369.html)
- np.mean and np.std: 均值和标准差 [numpy中标准差std的神坑](https://blog.csdn.net/mvpboss1004/article/details/79249142https://blog.csdn.net/mvpboss1004/article/details/79249142)

```python

def color_position_features(img):
    H, W, C = img.shape
    color = img_as_float(img)
    features = np.zeros((H*W, C+2))

    ### YOUR CODE HERE
    # 坐标
    cord = np.dstack(np.mgrid[0:H, 0:W]).reshape((H*W, 2))      # mgrid生成坐标，重新格式为（x,y）的二维
    features[:, 0:C] = color.reshape((H*W, C))      # r,g,b
    features[:, C:C+2] = cord
    features = (features - np.mean(features, axis=0)) / np.std(features, axis=0,  ddof = 0)     # 对特征归一化处理
    ### END YOUR CODE

    return features

```

和给出的答案有些不同，是计算方法有错误？


---



## 2.3 Implement Your Own Feature

略



# 3. Quantitative Evaluation

评估算法的实现，本节主要通过测试分割猫和其他背景来评估分割算法的性能。
- 将本次问题视为二分问题，将像素划分为foreground和background
- 评估：`(TP+TN)/(P+N)`

函数： `def compute_accuracy(mask_gt, mask)`
参数： 
- mask_gt: ground truth 分割,每个点为(y,x),维度为(H,W),foreground 为1
- mask: 估计值

返回值：
- accuracy： 1.0 表示完美分割

```python
### Quantitative Evaluation
def compute_accuracy(mask_gt, mask):
    accuracy = None
    ### YOUR CODE HERE
    mask_end = mask_gt - mask
    count = len(mask_end[np.where(mask_end == 0)])
    accuracy = count / (mask_gt.shape[0] * mask_gt.shape[1])
    ### END YOUR CODE

    return accuracy


```



# 总结
本次作业主要完成了聚类和分割的简单算法实现，主要是利用相邻像素的特征之间的“相似性”比较，来划分和聚合。
但是，可以从结果中看出，差异明显的图片的分割效果很好，然而，复杂的图片分割效果就很差。



