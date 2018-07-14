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




---

