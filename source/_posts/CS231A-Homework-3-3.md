---
title: CS231A_Homework_3.3
date: 2018-10-25 16:08:20
tags:
	- HOG
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-3:ps3_code-Histogram-of-Oriented-gradients

---

# 三、 Histogram of Oriented Gradients(方向梯度直方图)

本部分内容主要实现HoG,然后进行一个简单的应用。

1. 首先，进行梯度计算；
2. 计算直方图
3. 计算HoG特征


## 3.1 计算梯度: compute_gradient():

参数：
- im: 灰度图片 (H, W)

返回值：
- angles: 梯度角度 （H-2, W-2）
- magnitudes: 梯度 （H-2, W-2）


梯度的计算如下：
- 给定一个像素和周围8个方向的像素，

```
P1 P2 P3
P4 P5 P6
P7 P8 P9

arctan(dy/dx) = (P2 - p8)/(p4 - P6)  [可以使用np.arctan2 : 更稳定]，结果为[-180,180)
如果想要得到[0, 180],需要简单地增加180到给负的角度

梯度： sqrt((P4 - P6)^2 + (p2- P8)^ 2)

```

```python
def compute_gradient(im):
    H, W = im.shape

    angles = np.zeros((H-2, W-2))
    magnitudes = np.zeros((H-2, W-2))       # 建立两个返回值矩阵

    for i in range(1, H-1):
        for j in range(1, W-1):     # 最外圈的点不计算
            top = im[i-1, j]
            down = im[i+1, j]
            left = im[i, j-1]
            right = im[i, j+1]      # 上下左右

            angle = np.arctan2(top - down, left - right) * (180 / math.pi)      # 转化为度
            magn = np.sqrt((top-down)**2 + (left - right)**2)

            if(angle < 0):      # 加180
                angle += 180

            angles[i-1, j-1] = angle
            magnitudes[i-1, j-1] = magn

    return angles, magnitudes



```


## 3.2 生成直方图： generate_histogram()

通过给定的角度矩阵和梯度矩阵，生成角度直方图

参数：

- angles: （M, N）
- magnitudes:(M,N)
- nbins: 直方图区间数

返回值：
- histogram： 多维数组(nbins,),包含梯度角的分布

实现：

1. 每个直方图应该被划分为0到180度，nbins决定区间数
2. 迭代，将每个对应的梯度角度放到对应的直方图区间中，为了合理分配到两个相近的区间，使用如下公式：

```
# center_angle 为 区间的中心，比如 20度的区间，第一个和第二个区间的中心为10,30
	histogram[bin1] += magnitude * |angle - center_angle2 | / (180 / nbins)		
	histogram[bin2] += magnitude * |angle - center_angle1 | / (180 / nbins)
 
```

认为第1个区间和最后一个区间相邻；


```python

def generate_histogram(angles, magnitudes, nbins = 9):
    histogram = np.zeros(nbins)

    bin_size = 180 / nbins
    center_angles = np.zeros_like(histogram)

    for i in range(nbins):
        center_angles[i] = (0.5 + i) * bin_size     # 计算每个区间中心

    M, N = angles.shape

    for m in range(M):
        for n in range(N):
            angle = angles[m, n]
            magn = magnitudes[m, n]

            abs_diff = np.abs(center_angles - angle)

            # 当 angle 趋于0度
            if (180 - center_angles[-1] + angle) < abs_diff[-1]:        # 更新下最后区间的大小
                abs_diff[-1] = 180 - center_angles[-1] + angle

            # angle趋于180度
            if(180 + center_angles[0] - angle) < abs_diff[0]:
                abs_diff[0] = 180 - angle + center_angles[0]

            # 统计直方图
            bin1, bin2 = np.argsort(abs_diff)[0:2]      # 取最近的两个
            histogram[bin1] += magn * abs_diff[bin2] / (180.0 / nbins)
            histogram[bin2] += magn * abs_diff[bin1] / (180.0 / nbins)

    return histogram


```


## 3.3 计算HoG特征： compute_hog_features()

参数：
- im： 图片矩阵
- pixels_in_cell: 每个单元格的大小
- cenlls_in_block:	每个block中的cell数量
- nbins: histogram bins

返回值：
- features: hog 特征（H_blocks, W_blocks, cells_in_block \* cells_in_block \* nbins）

实现：

1. 计算每个图片的梯度和角度
2. 定义一个cell和block
3. 定义一个滑动窗口，大小为一个block的大小，滑动步长为block的一半，每个block中的cell存储一个直方图中的梯度，
每个block 特征被表示为(cells_in_block， cells_in_block, nbins),也可表示为（ cells_in_block \* cells_in_block \* nbins ）
,确保归一化
4. 返回这些所有网格的HoG特征


```python


def compute_hog_features(im, pixels_in_cell, cells_in_block, nbins):

    # 第一步： 获得角度和梯度
    angles, magnitudes = compute_gradient(im)

    # 第二步： 划分block 和cell
    cell_size = pixels_in_cell
    block_size = pixels_in_cell * cells_in_block

    # 第三步：计算滑动窗口
    H, W = angles.shape
    stride = int(block_size / 2)
    H_blocks = int((H - block_size) / stride) + 1
    W_blocks = int((W - block_size) / stride) + 1

    # 第四步： 计算每个cell的 histogram
    hog_fe = np.zeros((H_blocks, W_blocks, cells_in_block * cells_in_block * nbins))

    for h in range(H_blocks):
        for w in range(W_blocks):
            block_angles = angles[h*stride: h * stride + block_size,
                           w * stride: w*stride + block_size]      # 一个block的角度
            block_magn = magnitudes[h*stride: h*stride+block_size,
                            w*stride: w*stride+block_size]         # 梯度

            # 将一个block中的每个cell表示为一个方向直方图
            block_hog_fe = np.zeros((cells_in_block, cells_in_block, nbins))

            for i in range(cells_in_block):
                for j in range(cells_in_block):
                        cell_angles = block_angles[i*pixels_in_cell : (i+1)*pixels_in_cell,
                                        j*pixels_in_cell : (j+1)*pixels_in_cell]
                        cell_magns = block_magn[i*pixels_in_cell : (i+1) * pixels_in_cell,
                                        j*pixels_in_cell : (j+1) * pixels_in_cell]

                        cell_hist = generate_histogram(cell_angles, cell_magns, nbins)

                        block_hog_fe[i, j, :] = cell_hist

            # 归一化
            block_hog_fe = np.reshape(block_hog_fe, -1)

            block_hog_fe /= np.linalg.norm(block_hog_fe)

            hog_fe[h, w, :] = block_hog_fe


    return hog_fe

```

## 3.4 结果

![hog](https://github.com/jingxa/cs231a_my/raw/master/images/ps3/hog.png)



---

