---
title: CS131_Homework_2
date: 2018-07-14 19:07:30
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw2
---

# part 1 canny Edge Detector
- canny 算子可以分为以下五步：
  - Smoothing
  - Finding gradients
  - Non-maximum supperession
  - Double thredsholding
  - Edge tracking by hysterisis

## 1.1 Smoothing
平滑一张图片，可以使用（Gaussian kernel）高斯核来卷积；

![](https://upload-images.jianshu.io/upload_images/5361608-d9ecc61140d81a3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
def conv(image, kernel):
    Hi, Wi = image.shape
    Hk, Wk = kernel.shape
    out = np.zeros((Hi, Wi))
.
    pad_width0 = Hk // 2
    pad_width1 = Wk // 2
    pad_width = ((pad_width0,pad_width0),(pad_width1,pad_width1))
    padded = np.pad(image, pad_width, mode='edge')
    # 卷积过程
    kernel = np.flip(np.flip(kernel, 0), 1)  # 上下翻转，在左右翻转
    for i in range(Hi):
        for j in range(Wi):
            out[i, j] = np.sum(padded[i:(i + Hk), j:(j + Wk)] * kernel)
    return out


def gaussian_kernel(size, sigma):
    kernel = np.zeros((size, size))
    # size = 2k+1
    k = size // 2
    for i in range(size):
        for j in range(size):
            kernel[i, j] = np.exp(-(((i-k)**2 + (j-k)**2))/2*sigma**2) / (2 * np.pi * (sigma**2))

    return kernel
```

输出为：

![](https://upload-images.jianshu.io/upload_images/5361608-8446400e3eb433ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- [问题] What is the effect of the kernel_size and sigma?

![](https://upload-images.jianshu.io/upload_images/5361608-c58b1092a531d5d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-9aa6973c4104d788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 高斯核的sigma和size越大，就越模糊；


---
## 1.2 Finding gradients

### 1.2.1 偏导数
- 图片的偏导数可以计算为：

![](https://upload-images.jianshu.io/upload_images/5361608-bcd31befb666ccfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  对于一张图片，我们需要给图片添加一层为0 的padding:

```

def partial_x(img):
    out = None
    # Hi, Wi = img.shape
    # padd = np.zeros((Hi, Wi+2))
    # padd[:, 1:Wi+1] = img
    #
    # out = np.zeros((Hi, Wi))
    # for i in range(Wi):
    #     out[:, i] = (padd[:, i+2] - padd[:, i]) / 2

    # 使用卷积
    kernel = np.array([[1, 0, -1]]) / 2
    out = conv(img, kernel)
    return out


def partial_y(img):
    out = None
    # Hi, Wi = img.shape
    # padd = np.zeros((Hi + 2, Wi))
    # padd[1:Hi+1, :] = img
    #
    # out = np.zeros((Hi, Wi))
    # for i in range(Hi):
    #     out[i, :] = (padd[i+2, :] - padd[i, :]) / 2


    kernel = np.array([[1, 0, -1]]).T / 2
    out = conv(img, kernel)
    return out
```
结果为：

![](https://upload-images.jianshu.io/upload_images/5361608-364f1442bad2fc12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- [问题]：What is the reason for performing smoothing prior to computing the gradients?
- 在计算导数之前先进行smoothing,是为了降低噪声；

### 1.2.2 导数

![](https://upload-images.jianshu.io/upload_images/5361608-be899d6188d15932.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Hint: Use np.arctan2 to compute
- the “y-coordinate” is the first function parameter, the “x-coordinate” is the second.)

```
def gradient(img):

    G = np.zeros(img.shape)
    theta = np.zeros(img.shape)

    gx = partial_x(img)
    gy = partial_y(img)

    G = np.sqrt(gx**2 + gy**2)

    theta = np.arctan2(gy, gx)  # (-pi/2, pi/2)

    theta = (np.rad2deg(np.arctan2(gy, gx)) + 180) % 360


    return G, theta
```


![](https://upload-images.jianshu.io/upload_images/5361608-994c7f22bda0fd31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
## 1.3 Non - maximum suppression

![](https://upload-images.jianshu.io/upload_images/5361608-f8507e47aeebd8e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 在八个方向上[0,45,90,135,180,225,270,315]比较;

![](https://upload-images.jianshu.io/upload_images/5361608-ec4621ad6dad900f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
def non_maximum_suppression(G, theta):

    H, W = G.shape
    out = np.zeros((H, W))

    # Round the gradient direction to the nearest 45 degrees
    theta = np.floor((theta + 22.5) / 45) * 45  # 方向定位
    # 添加一层padding
    padd = np.zeros((H+2,W +2))
    padd[1:H+1, 1:W+1] = G
    for m in range(1, H+1):
        for n in range(1, W+1):
            # 题目定义为顺时针方向，和逆时针相反,y方向相反
            rad = np.deg2rad(theta[m-1, n-1])
            i =int(np.around(np.sin(rad)))   # 行
            j =int(np.around(np.cos(rad)))   # 列
            p1 = padd[m+i, n+j]
            p2 = padd[m-i, n-j]
            if(padd[m, n] > p1 and padd[m, n] > p2): # 一个方向上
                out[m-1, n-1] = padd[m, n]
            else:
                out[m-1, n-1] = 0
    return out
```

结果为：

```
Thetas: 0
[[0.  0.  0.6]
 [0.  0.  0.7]
 [0.  0.  0.6]]
Thetas: 45
[[0.  0.  0.6]
 [0.  0.  0.7]
 [0.4 0.5 0.6]]
Thetas: 90
[[0.4 0.  0. ]
 [0.  0.  0.7]
 [0.4 0.  0. ]]
Thetas: 135
[[0.4 0.5 0.6]
 [0.  0.  0.7]
 [0.  0.  0.6]]
```

![](https://upload-images.jianshu.io/upload_images/5361608-6f7d04485316dfa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---

## 1.4 Double Thresholding 
![](https://upload-images.jianshu.io/upload_images/5361608-632e6d73905542c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-32612e1f72d9e762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 阈值假设：
- 定义两个阈值：low High
- 如果 小于low,不是一个边
- 如果大于High, 就是一条强边
- 如果在low和High之间，弱边

使用 `np.where`:[np.where](https://blog.csdn.net/sloanqin/article/details/51612618)

```
def double_thresholding(img, high, low):

    strong_edges = np.zeros(img.shape)
    weak_edges = np.zeros(img.shape)

    strong_edges = np.where(img>=high,1,0)
    weak_edges = np.where(img>=low,1,0)
    weak_edges = weak_edges-strong_edges

    return strong_edges, weak_edges

```

---
## 1.5 Edge tracking
- strong edges : 确定的边
- weak edges: 只有和strong edges相连的边才是正确的边， 其他的就是噪点；

```
def get_neighbors(y, x, H, W):
    # 返回(y,x)周围等于(y,x)的邻居的坐标列表[(i,j),(i2,j2)..]
    neighbors = []

    for i in (y-1, y, y+1):
        for j in (x-1, x, x+1):
            if i >= 0 and i < H and j >= 0 and j < W:
                if (i == y and j == x):
                    continue
                neighbors.append((i, j))

    return neighbors

def link_edges(strong_edges, weak_edges):
    # 寻找强边像素相连的弱边像素
    H, W = strong_edges.shape
    indices = np.stack(np.nonzero(strong_edges)).T
    edges = np.zeros((H, W))

    ### YOUR CODE HERE
    edges = np.copy(strong_edges)
    for i in range(1, H-1):
        for j in range(1, W-1):
            neighbors = get_neighbors(j, i, H, W)
            if weak_edges[i, j] and np.any(edges[x, y] for x, y in neighbors):
                edges[i, j] = True
    ### END YOUR CODE

    return edges
```

![](https://upload-images.jianshu.io/upload_images/5361608-000d8d5790513651.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
## 1.6 Canny edge detector
- 实现canny算子

```
def canny(img, kernel_size=5, sigma=1.4, high=20, low=15):

    ### YOUR CODE HERE
    kernel = gaussian_kernel(kernel_size, sigma)  # 1. smoothing
    smoothed = conv(img,kernel)
    G, theta = gradient(smoothed)                 # 2. 梯度计算
    nms = non_maximum_suppression(G,theta)        # 3. non-maximum_suppression
    strong_edges, weak_edges = double_thresholding(nms, low,high) #double thresholding
    edge = link_edges(strong_edges,weak_edges)    # link
    ### END YOUR CODE

    return edge

```
![](https://upload-images.jianshu.io/upload_images/5361608-c03f9c9fc21f90d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
# Part 2 Lane Detection
![](https://upload-images.jianshu.io/upload_images/5361608-7a261fb12aceb969.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 检测平面
算法：
- 使用canny算子检测边
- 提取兴趣区域的边
- 运行Hough transform  来检测平面

## 2.1 边检测

![](https://upload-images.jianshu.io/upload_images/5361608-8f938b5f48dda00e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.2 Extracting region of interest (ROI)
- 提取兴趣区域
- 使用一个二位掩码来定义兴趣区域；然后提取兴趣区域的边；

![](https://upload-images.jianshu.io/upload_images/5361608-b234885744b5cd69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.3 Fitting lines using Hough transform
- 边检测的输出为连接点的集合；
- 使用hough变换来检测一条直线；

- 直线 y =ax + b 被表示为：(a,b),这种方式不能表示垂直线段；
- 因此，使用极方程表示：

![](https://upload-images.jianshu.io/upload_images/5361608-26385475a7a120de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-1e7e069710553bb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```


def hough_transform(img):
    """ Transform points in the input image into Hough space.

    Use the parameterization:
        rho = x * cos(theta) + y * sin(theta)
    to transform a point (x,y) to a sine-like function in Hough space.

    Args:
        img: binary image of shape (H, W)
        
    Returns:
        accumulator: numpy array of shape (m, n)
        rhos: numpy array of shape (m, )
        thetas: numpy array of shape (n, )
    """
    # Set rho and theta ranges
    W, H = img.shape
    diag_len = int(np.ceil(np.sqrt(W * W + H * H)))  # 对角线长度，np.ceil 向大取整
    rhos = np.linspace(-diag_len, diag_len, diag_len * 2.0 + 1) # 2倍，等差数列
    thetas = np.deg2rad(np.range(-90.0, 90.0))

    # Cache some reusable values
    cos_t = np.cos(thetas)
    sin_t = np.sin(thetas)
    num_thetas = len(thetas)

    # Initialize accumulator in the Hough space
    accumulator = np.zeros((2 * diag_len + 1, num_thetas), dtype=np.uint64)
    ys, xs = np.nonzero(img)

    # Transform each point (x, y) in image
    # Find rho corresponding to values in thetas
    # and increment the accumulator in the corresponding coordiate.
    ### YOUR CODE HERE
    for i, j in zip(ys, xs):
        for idx in range(thetas.shape[0]):
            r = j * cos_t[idx] + i * sin_t[idx]
            accumulator[int(r + diag_len), idx] += 1
    ### END YOUR CODE

    return accumulator, rhos, thetas

```

结果为：
![](https://upload-images.jianshu.io/upload_images/5361608-9dc4fdd0fc83ed97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 3 参考资料
1. [mikucy/CS131](https://github.com/mikucy/CS131/tree/master/hw2_release)
2. [wwdguu/CS131_homework](https://github.com/wwdguu/CS131_homework/tree/master/hw2_release)


---

