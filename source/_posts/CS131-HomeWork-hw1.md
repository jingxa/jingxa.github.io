---
title: CS131_HomeWork_hw1
date: 2018-07-03 17:02:34
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw1
---
- 参考资料：
	- [wwdguu/CS131_homework](https://github.com/wwdguu/CS131_homework/blob/master/hw1_release/hw1.ipynb)
	- [mikucy/CS131](https://github.com/mikucy/CS131/blob/master/hw1_release/hw1.ipynb)
	
---
> python numpy
- np.var() : 方差
- np.flip() : 矩阵翻转
- np.argmax(): 	最大值的indices
	
---
	
# 一、 Convolutions
## 1.1 commutative Property
![](https://upload-images.jianshu.io/upload_images/5361608-4b4cdd9835582008.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

推导两个式子相等：

令 ：
- x = m-i ==> i = m- x
- y = m- j ==> j = m- y

带入上式子为：

![](https://upload-images.jianshu.io/upload_images/5361608-c611b3175e54daa7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后。令 x = i, y =j;即可；

## 1.2 Linear and Shift Invariance

![](https://upload-images.jianshu.io/upload_images/5361608-583e6584f429e8ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

推导：
![](https://upload-images.jianshu.io/upload_images/5361608-51354bad0e2757e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-850592471aba6fe7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.3 实现
- 卷积需要翻转卷积核；
### 1.3.1 convoution
- 使用四个循环
```python

def conv_nested(image, kernel):

    Hi, Wi = image.shape
    Hk, Wk = kernel.shape
    out = np.zeros((Hi, Wi))
    padded = zero_pad(image,Hk//2,Wk//2)
    kernel = np.flip(np.flip(kernel, 0), 1)  # 上下翻转，在左右翻转
    ### YOUR CODE HERE
    for m in range(Hi):
        for n in range(Wi):
            for i in range(Hk):
                for j in range(Wk):
                        out[m,n]+=kernel[i,j]*padded[m+i,n+j]

    return out

```


- padding添加：


```python

def zero_pad(image, pad_height, pad_width):
    H, W = image.shape
    out = np.zeros(shape=(H+pad_height*2,W+pad_width*2),dtype=np.float32)
    out[pad_height:pad_height+H,pad_width:pad_width+W]=image[:,:]
    return out

```


- 使用矩阵相乘：


```python
def conv_fast(image, kernel):
    Hi, Wi = image.shape
    Hk, Wk = kernel.shape
    out = np.zeros((Hi, Wi))
    padd_H = Hk // 2
    padd_W = Wk // 2
    img_padd = zero_pad(image, padd_H, padd_W)
    # 卷积过程
    kernel = np.flip(np.flip(kernel, 0), 1)  # 上下翻转，在左右翻转
    for i in range(Hi):
        for j in range(Wi):

            out[i, j] = np.sum(img_padd[i:(i+Hk), j:(j+Wk)] * kernel)

    return out

```

## part 2 Cross-correlation
- cross correlation和卷积一样，只是不需要翻转卷积核；

- cross correlation

```python 
def cross_correlation(f, g):
    Hi, Wi = f.shape
    Hk, Wk = g.shape
    out = np.zeros((Hi, Wi))
    padd_H = Hk // 2
    padd_W = Wk // 2
    img_padd = zero_pad(f, padd_H, padd_W)
    # 卷积过程
    for i in range(Hi):
        for j in range(Wi):
            out[i, j] = np.sum(img_padd[i:(i+Hk), j:(j+Wk)] * g)
    return out
````

- zero_mean_cross_correlation

```python
def zero_mean_cross_correlation(f, g):

    out = None
    mean = np.mean(g)
    g = np.subtract(g, mean)
    out = cross_correlation(f,g)
    return out
````

- normalized_cross_correlation

```
def normalized_cross_correlation(f, g):
    # out = None
    Hk, Wk = f.shape
    Hg, Wg = g.shape
    paddedF = zero_pad(f, Hg // 2, Wg // 2)
    out = np.zeros_like(f)
    # g = np.flip(np.flip(g, 0), 1)
    g_mean = np.mean(g)
    g_delta = np.sqrt(np.var(g))
    g_t = (g - g_mean) / g_delta

    for m in range(Hk):
        for n in range(Wk):
            conv = paddedF[m:(m+Hg), n:(n+Wg)]
            f_mean = np.mean(conv)
            f_delta = np.sqrt(np.var(conv))
            f_t = (paddedF[m:(m+Hg), n:(n+Wg)] - f_mean)/f_delta
            out[m, n] = np.sum(f_t * g_t)

    print('end')

    return out
```



---

