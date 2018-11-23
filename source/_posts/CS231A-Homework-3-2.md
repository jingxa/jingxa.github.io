---
title: CS231A_Homework_3.2
date: 2018-10-24 15:36:00
tags:
	- SIFT
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-3:ps3_code-single-object-recognition

---

# 二、 Single Object Recognition via SIFT

本节主要使用SIFT实现识别和定位一个给定的对象从一堆测试图片中。

## 2.1 关键点匹配

给定一张图片的keypoint的descriptor ， 和另外多张图片的keypoint descriptors,实现一个算法能够检测关键点匹配

函数： `match_keypoints()`

- 计算一个点和另一幅图片所有点的最近两个点的欧式距离，如果（closed_point_distance / second_closed_point_distance）小于某个阈值，那么说明当前点和最近点closed_point为最佳匹配点

参数：
- descriptors1: 第一幅图片的 descriptors ,size:(M_1, 128),每一行是一个keypoint 的 descriptor
- descriptors2: 第二幅图片的 descriptors (M_2, 128)
- threshold: 阈值

返回值：

- matches: 匹配的descriptors数组,返回对应的关键点的序号对（N,2）


```python
def match_keypoints(descriptors1, descriptors2, threshold = 0.7):
    matches_list = []

    N = descriptors1.shape[0]
    for i in range(N):
        des1 = descriptors1[i]      # 一个点的描述符
        dist = np.sqrt(np.sum((descriptors2 - des1)**2, axis=1))        # 当前点和另一幅图片所有点的欧式距离
        index_sort = np.argsort(dist)           # 按照大小排序，返回序号

        closed = index_sort[0]
        second_closed = index_sort[1]

        if(dist[closed] < threshold * dist[second_closed]):
            matches_list.append([i, closed])

    matches = np.array(matches_list)
    return matches


```


## 2.2 优化匹配

在上述初始匹配中，有一些错误匹配对，需要移除，这里使用RANSAC算法来计算两幅图片的单应性变换，通过变换，变换到另一幅图片的点应该和对应点的距离差值应该在一个很小的阈值内，如果超出阈值，说明不是对应点

在两幅图的单应性变换中，可以得到公式如下：

![H](https://github.com/jingxa/cs231a_my/blob/master/images/ps3/h_trans.png)

[H](https://github.com/jingxa/cs231a_my/blob/master/images/ps3/h_trans.png)

因此，对应H矩阵，我们可以建立对他的9个未知量建立方程矩阵（2N*9）,其中可以得到x和y的对应等式，还有1的等式


函数：`refine_match()`
参数：
- keypoints1: 第一幅图片的 关键点descriptors数组，每一行为(u,v, scale, theta)，总大小为(M_1,4)
- keypoints2: 第二幅图片的特征数组
- matches: 初始的配对
- threshold
- num_iterations: 迭代次数

返回：
- inliers: RANSAC 拟合的对应模型的内点序列
- model: RANSAC 拟合两幅图片的变换矩阵H


```python


def refine_match(keypoints1, keypoints2, matches, reprojection_threshold = 10,
        num_iterations = 1000):

    best_model = None       # 最佳模型
    best_inliers = []       # 内点集合
    best_count = 0          # 最佳匹配对数

    sample_size = 4         # H 是 9*9矩阵，除去尺度，有八个自由度，需要4对对应点
    P = np.zeros((2 * sample_size, 9))      # 建立 2N*9的矩阵，其中每个对应对建立两个等式
    N = matches.shape[0]

    for i in range(num_iterations):
        sample_indexes = random.sample(range(0, N), sample_size)     # 随机抽取4对样本
        sample = matches[sample_indexes, :]

        for index, elem in enumerate(sample):       # 获得序列和数据
            # 取一对对应点
            p1_idx = elem[0]
            p2_idx = elem[1]

            # 转化为齐次坐标系
            point1 = keypoints1[p1_idx, 0:2]        # u,v 坐标
            point1 = np.append(point1, 1)           # (u, v, 1)

            point2 = keypoints2[p2_idx, 0:2]
            u = point2[0]
            v = point2[1]

            # 建立 P 矩阵
            P[2 * index, :] = np.reshape(np.array([point1, np.zeros(3), -u * point1]), -1)
            P[2 * index + 1, :] = np.reshape(np.array([np.zeros(3), point1, -v * point1]), -1)

        # 求解当前的H矩阵
        U, s, VT = np.linalg.svd(P)
        H = VT[-1, :].reshape(3, 3)
        H /= H[2, 2]        # 归一化

        inliers = []        # 当前对应的 内点
        count = 0

        for index, match in enumerate(matches):     # 对每一对对应点进行变换评估
            p1 = keypoints1[match[0], 0:2]      # u,v
            p1 = np.append(p1, 1)               # (u,v,1)

            p2_pred = H.dot(p1)             # p1 变换后的点
            p2_pred /= p2_pred[2]           # 归一化
            p2_pred = p2_pred[0:2]          # u,v

            p2 = keypoints2[match[1], 0:2]
            err = np.sqrt( np.sum( (p2 - p2_pred) ** 2 ) )
            if err < reprojection_threshold:        # 此对应点为正确
                count += 1
                inliers.append(index)

        # 记录最佳 H
        if count > best_count:
            best_model = H
            best_inliers = inliers
            best_count = count

    return best_inliers, best_model


```


## 2.3 get_object_region()

使用hough变换获得预测对象的边界盒子

参数：
- keypoints1: 同上
- keypoints2
- matches
- obj_bbox: (xmin,ymin,xmax, ymax)
- thresh: hough voting 阈值

返回值：
- cx: 盒子中心x坐标数组
- cy: 盒子中心y坐标数组
- w: 盒子宽度数组
- h: 盒子高度数组
- orient: 盒子方向数组


Hough Voting的思想就是将参数空间划分为子网格，然后统计子网格的落点计数，在此函数中：
1. 首先计算每一对对应点的边界盒子数据
2. 然后获得盒子的最小最大的边界值，然后将整个盒子的可能空间划分为子网格
3. 然后遍历所有的盒子数据，统计每个子网格的计数，
4. 遍历每个子网格，如果子网格的落点大于某个阈值，那么就认为这个盒子可以做对象的边界盒子。


```python
def get_object_region(keypoints1, keypoints2, matches, obj_bbox, thresh = 4,
        nbins = 4):

    cx, cy, w, h, orient = [], [], [], [], []

    # 第一步： 计算边界盒子数组
    for match in matches:
        p1_idx = match[0]
        p2_idx = match[1]

        p1 = keypoints1[p1_idx]
        p2 = keypoints2[p2_idx]

        u1, v1, s1, theta1 = p1[0], p1[1], p1[2], p1[3]     # 两个点的成员
        u2, v2, s2, theta2 = p2[0], p2[1], p2[2], p2[3]

        # 寻找对应点在img2 中的边界盒子，采用旋转平移进行计算
        xmin, ymin, xmax, ymax = obj_bbox
        xc1 = (xmin + xmax) / 2.0
        yc1 = (ymin + ymax) / 2.0       # 中心点的坐标
        w1 = (xmax - xmin) * 1.0
        h1 = (ymax - ymin) * 1.0        # 使用浮点表示

        O2 = theta2 - theta1
        xc2 = (s2/s1) * np.cos(O2) * (xc1 - u1) - (s2/s1) * np.sin(O2) * (yc1 - v1) + u2
        yc2 = (s2/s1) * np.sin(O2) * (xc1 - u1) + (s2/s1) * np.cos(O2) * (yc1 - v1) + v2
        w2 = (s2/s1) * w1
        h2 = (s2/s1) * h1       # 缩放

        # 保存到数组中
        cx.append(xc2)
        cy.append(yc2)
        w.append(w2)
        h.append(h2)
        orient.append(O2)       # 这个方向有点不理解

    # 第二步： 计算盒子的子网格划分
    cx_min, cx_max = min(cx), max(cx)
    cy_min, cy_max = min(cy), max(cy)
    w_min, w_max = min(w), max(w)
    h_min, h_max = min(h), max(h)
    orient_min, orient_max = min(orient), max(orient)

    cx_bin_size = (cx_max - cx_min) / float(nbins)
    cy_bin_size = (cy_max - cy_min) / float(nbins)
    w_bin_size = (w_max - w_min) / float(nbins)
    h_bin_size = (h_max - h_min) / float(nbins)
    orient_bin_size = (orient_max - orient_min) / float(nbins)


    # 第三步： 统计每个子网格的计数
    bins = defaultdict(list)        # 由于nbins为4，那么就只计算4个参数
    N = matches.shape[0]
    for n in range(N):
        x_center = cx[n]
        y_center = cy[n]
        w_center = w[n]
        orient_center = orient[n]

        for i in range(nbins):
            for j in range(nbins):
                    for k in range(nbins):
                            for l in range(nbins):
                                if(cx_min + i * cx_bin_size <= x_center
                                        and x_center <= cx_min +(i+1) * cx_bin_size):       # x坐标
                                            if(cy_min + j * cy_bin_size <= y_center
                                                and y_center <= cy_min + (j+1) * cy_bin_size):
                                                    if(w_min + k * w_bin_size <= w_center
                                                        and w_center <= w_min + (k+1)*w_bin_size):
                                                            if(orient_min + l*orient_bin_size <= orient_center
                                                                and orient_center <= orient_min + (l+1) * orient_bin_size):
                                                                    bins[(i, j, k, l)].append(n)
    # 第四步： 统计
    cx0, cy0, w0, h0, orient0 = [], [], [], [], []

    for bin_idx in bins:
        indices = bins[bin_idx]
        votes = len(indices)

        if(votes >= thresh):
            cx0.append(np.sum(np.array(cx)[indices]) / votes)       # 平均数
            cy0.append(np.sum(np.array(cy)[indices]) / votes)
            w0.append(np.sum(np.array(w)[indices]) / votes)
            h0.append(np.sum(np.array(w)[indices]) / votes)
            orient0.append(np.sum(np.array(orient)[indices]) / votes)

    return cx0, cy0, w0, h0, orient0

```



## 2.4 结果

- [cs231a/ps3](https://github.com/jingxa/cs231a_my/tree/master/ps3_code/code_ps3)



## 2.5 待解决问题

1. 在第二个函数中，关键点的成员包含[u, v, scale, theta],后面两个是怎么计算的？

2. 在边界盒子的子网格统计中，使用了4个参数，但是在5个参数中，我使用h参数替换掉orient参数，结果不对，为什么呢？





















---

