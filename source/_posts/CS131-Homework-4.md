---
title: CS131_Homework_4
date: 2018-11-22 19:33:17
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw4
---

# 主要内容
- 本次作业主要内容是 `seam carving`


## 1. Image Reducing using Seam Carving

- basic idea: 移除不重要的像素点
- 定义 `unimportant:` == `less energy pixels`， 一般为`E = {x轴方向的梯度的绝对值} + {y 轴方向的梯度的绝对值}`

### 1.1 Energy function

函数： `energy_function(im)`

参数：
- im: 图片 (H, W, 3)

返回值：
- 每个像素点的energy (H, W)

实现：
图像梯度一般也可以用中值差分：

```
dx(i,j) = [I(i+1,j) - I(i-1,j)]/2;
dy(i,j) = [I(i,j+1) - I(i,j-1)]/2;

G(x,y) = dx(i,j) + dy(i,j);
```
- 这里可以使用`np.gradient`

```python

def energy_function(image):
    H, W, _ = image.shape
    out = np.zeros((H, W))
    gray_image = color.rgb2gray(image)

    ### YOUR CODE HERE
    dx, dy = np.gradient(gray_image)
    out = np.abs(dx) + np.abs(dy)
    ### END YOUR CODE

    return out

```

### 1.2 compute cost 

这一步从图片的顶部到低端计算每个像素的`minimal cost`,即从顶端到当前像素的最小cost路径

函数：`compute_cost(image, energy, axis=1)`
参数：
- image: 
- energy： 数组(H,W)
- axis: 计算维度(width: axis=1 或者 height: axis=0)
返回值：
- cost： 每个像素的cost，数组(H,W) 
- paths: lowest energy path图， 数组(H,W)，值为-1,0,1 (左上，正上，右上)判断当前像素是否在seam上

实现：
每个像素的cost为`M(i,j) = E(i,j) + min(M(i-1,j-1),M(i-1,j),M(i-1,j+1))`

```python

def compute_cost(image, energy, axis=1):
    energy = energy.copy()

    if axis == 0:
        energy = np.transpose(energy, (1, 0))

    H, W = energy.shape

    cost = np.zeros((H, W))
    paths = np.zeros((H, W), dtype=np.int)

    # Initialization
    cost[0] = energy[0]
    paths[0] = 0  # we don't care about the first row of paths

    ### YOUR CODE HERE
    for i in range(1, H):
        M1 = np.r_[[1e10], cost[i-1, 0:W-1]]    # 左边添加一个无穷大
        M2 = cost[i-1, :]
        M3 = np.r_[cost[i-1, 1:], [1e10]]        # 右边添加一个无穷大
        M = np.r_[M1, M2, M3].reshape(3, -1)
        cost[i] = energy[i] + np.min(M, axis=0)     # cost
        paths[i] = np.argmin(M, axis=0) - 1         # 上一层的最小值为左上角，正上方，右上角

    ### END YOUR CODE

    if axis == 0:
        cost = np.transpose(cost, (1, 0))
        paths = np.transpose(paths, (1, 0))

    # Check that paths only contains -1, 0 or 1
    assert np.all(np.any([paths == 1, paths == 0, paths == -1], axis=0)), \
           "paths contains other values than -1, 0 or 1"

    return cost, paths

```


## 2. Finding optimal seams

通过上一步计算最小能量；需要移除`optimal seam`， 即最小cost的像素点连成的线

### 2.1 Backtrack seam

函数： `backtrack_seam(paths, end)`
参数：
- paths:
- end: seam ends(H,end)
返回值：
- seam: 数组(H,), 

实现：

```
从下到上寻找最小：
        - left (value -1)
        - middle (value 0)
        - right (value 1)
```
def backtrack_seam(paths, end):
    H, W = paths.shape
    # initialize with -1 to make sure that everything gets modified
    seam = - np.ones(H, dtype=np.int)

    # Initialization
    seam[H-1] = end     # 最底层的像素点位置(H-1,end)

    ### YOUR CODE HERE
    for i in range(H-2,-1,-1):
        seam[i] = seam[i+1]+paths[i+1, seam[i+1]]       # 上层点的width坐标
    ### END YOUR CODE

    # Check that seam only contains values in [0, W-1]
    assert np.all(np.all([seam >= 0, seam < W], axis=0)), "seam contains values out of bounds"

    return seam


```python


```


### 2.2 Reduce

移除上一步计算的seam

函数： `remove_seam(image, seam)`

参数：
- seam: 每个像素点的位置为`image[i, seam[i]]`


返回值：
- out: 多维数组，(H,W,C)或者(H,W-1)

```python
def remove_seam(image, seam):

    # Add extra dimension if 2D input
    if len(image.shape) == 2:
        image = np.expand_dims(image, axis=2)   # 增加一个维度

    out = None
    H, W, C = image.shape
    ### YOUR CODE HERE
    out = np.zeros((H, W - 1, C), dtype=image.dtype)        # 返回值，每一行删除一个像素
    for i in range(H):
        out[i, :seam[i], :]=image[i, :seam[i], :]
        out[i, seam[i]:, :]=image[i, seam[i]+1:, :]

    ### END YOUR CODE
    out = np.squeeze(out)  # remove last dimension if C == 1

    # Make sure that `out` has same type as `image`
    assert out.dtype == image.dtype, \
       "Type changed between image (%s) and out (%s) in remove_seam" % (image.dtype, out.dtype)

    return out


```


函数：`reduce(image, size, axis=1, efunc=energy_function, cfunc=compute_cost)`

参数：
- size:根据axis移除像素知道axis的大小值为size

实现：
- 每一次移除一条最小的seam，最终移除size次


```python

def reduce(image, size, axis=1, efunc=energy_function, cfunc=compute_cost):

    out = np.copy(image)
    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    H = out.shape[0]
    W = out.shape[1]

    assert W > size, "Size must be smaller than %d" % W

    assert size > 0, "Size must be greater than zero"

    ### YOUR CODE HERE
    while out.shape[1] > size:
        energy = efunc(out)         # 首先重新计算energy
        cost, paths = cfunc(out, energy)        # 第二步计算cost map
        seam = backtrack_seam(paths, np.argmin(cost[-1]))       #计算optimal seam
        out = remove_seam(out, seam)                            # 移除
    ### END YOUR CODE

    assert out.shape[1] == size, "Output doesn't have the right shape"

    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    return out
```






## 3. Image Enlarging

### 3.1 Enlarge naive
扩大图片，可以复制seam

函数： 

#### `enlarge_naive(image, size, axis=1, efunc=energy_function, cfunc=compute_cost)`

增大图片某axis到size大小
Use functions:
	- efunc
	- cfunc
	- backtrack_seam
	- duplicate_seam
	
返回值： out数组(size, W, c)如果axis=0, 或 (H, size, c) 如果axis=1


#### `enlarge(image, size, axis=1, efunc=energy_function, cfunc=compute_cost)`
寻找多条lowest energy seam
Use functions:
	- find_seams
	- duplicate_seam
	
返回值： out数组(size, W, c)如果axis=0, 或 (H, size, c) 如果axis=1	
	

```python


def enlarge_naive(image, size, axis=1, efunc=energy_function, cfunc=compute_cost):


    out = np.copy(image)
    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    H = out.shape[0]
    W = out.shape[1]

    assert size > W, "size must be greather than %d" % W

    ### YOUR CODE HERE
    while out.shape[1] < size:
        energy = efunc(out)         # 首先重新计算energy
        cost, paths = cfunc(out, energy)        # 第二步计算cost map
        seam = backtrack_seam(paths, np.argmin(cost[-1]))       #计算optimal seam
        out = duplicate_seam(out, seam)                            # 复制
    ### END YOUR CODE

    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    return out

	
	
def enlarge(image, size, axis=1, efunc=energy_function, cfunc=compute_cost):

    out = np.copy(image)
    # Transpose for height resizing
    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    H, W, C = out.shape

    assert size > W, "size must be greather than %d" % W

    assert size <= 2 * W, "size must be smaller than %d" % (2 * W)

    ### YOUR CODE HERE
    seams = find_seams(out, size - W)       # 寻找size - W 条 seam
    seams = np.expand_dims(seams, axis=2)   # (H,W)
    for i in range(size - W):
        out = duplicate_seam(out, np.where(seams == i+1)[1])        # 复制当前 seam
        seams = duplicate_seam(seams, np.where(seams == i+1)[1])    # seam也复制
    ### END YOUR CODE

    if axis == 0:
        out = np.transpose(out, (1, 0, 2))

    return out	

```



## 4. Faster reduce

- 【暂时没有搞明白 ， 使用别人的代码】

函数： `reduce_fast(image, size, axis=1, efunc=energy_function, cfunc=compute_cost)`


## 5. Forward Energy

函数：`compute_forward_cost(image, energy)`

实现：

```
M1: M(i-1,j-1)
M2: M(i-1,j)
M3: M(i-1,j+1)
CL(i,j): |I(i,j+1) - I(i,j-1)| + |I(i-1,j) - I(i,j-1)|
CL(i,j): |I(i,j+1) - I(i,j-1)| + |I(i-1,j) - I(i,j+1)|
CV(i,j): |I(i,j+1) - I(i,j-1)|
M(i,j) = min{{M1+CL(i,j)}, {M2 + Cv(i,j)}, {M3+CR(i,j)}}


```








---

