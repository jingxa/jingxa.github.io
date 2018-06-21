---
title: 1_Tensor_Learn_OPS_(2)
date: 2018-06-21 21:33:40
tags:
	- TensorFlow
categories:
	- TensorFlow
	- Standford_CS20
---

# 1. 基本概念
tensorflow不仅是一个软件库，还包括Tensorflow，Tensorboard,TensorServing;
- Tensorflow中constants,variables,operators 称为：ops;


## 1.1 Constant op

```python
tf.constant(value, dtype=None, shape=None, name='Const', verify_shape=False)

# constant of 1d tensor (vector)
a = tf.constant([2, 2], name="vector")

# constant of 2x2 tensor (matrix)
b = tf.constant([[0, 1], [2, 3]], name="matrix")

```
创建 类似numpy的结构
```
tf.zeros(shape, dtype=tf.float32, name=None)
# create a tensor of shape and all elements are zeros
# ==> [[0, 0, 0], [0, 0, 0]]
tf.zeros([2, 3], tf.int32) 

```
创建类似·input_tensor·的shape和type(除非指定)但是元素全部为0：
~~~
tf.zeros_like(input_tensor, dtype=None, name=None, optimize=True)

# input_tensor [[0, 1], [2, 3], [4, 5]]
tf.zeros_like(input_tensor)  #==> [[0, 0], [0, 0], [0, 0]]
~~~

- 创建一个tensor，所有元素都为1
```
tf.ones(shape.dtype  =  tf.float32  ,  name  =  None)


tf.ones([  2  ,   3  ],  tf  .  int32  )  
 #==>   [[  1  ,   1  ,   1  ],   [  1  ,   1  ,   1  ]]
```
- 创建一个类似input_tensor的shape结构，数据type可以自定义,所有元素为1；

![](https://upload-images.jianshu.io/upload_images/5361608-51931d934a916970.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建一个tensor ,并且使用一个标量填充；

![](https://upload-images.jianshu.io/upload_images/5361608-c5db6be102722506.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建一个连续constant，包含start和stop；

![](https://upload-images.jianshu.io/upload_images/5361608-a2fa8fa2ae007f5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 从一个数开始创建一个constant 序列，但是不包括最后limit数；

![](https://upload-images.jianshu.io/upload_images/5361608-8502e3bd5a2c5d97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-18ba5ab7b409d49b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 但是，tensor不支持python的迭代器属性：

![](https://upload-images.jianshu.io/upload_images/5361608-d11f279c7db31cc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- tensorflow的随机生成：

![](https://upload-images.jianshu.io/upload_images/5361608-3b675676bdb469d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
## 1.2 math ops

- [math_ops](https://www.tensorflow.org/api_guides/python/math_ops)

### 1.2.1 division

![](https://upload-images.jianshu.io/upload_images/5361608-1586df33c9136fcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-4ec12108b8a3f45c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2.2 add

![](https://upload-images.jianshu.io/upload_images/5361608-27d7811b4186dcf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2.3 product 

`tf.matmul` 不等于dot product：
![](https://upload-images.jianshu.io/upload_images/5361608-0ae8c3063d20cd86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2.4 ops in Python

![](https://upload-images.jianshu.io/upload_images/5361608-d8cd33fe3bf21fac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1.3 Data Type

### 1.3.1 python type
- tensorflow 适配python原始类型：
  - python中的single value(boolean,numeric value,strings) => 0-d tensor（scalars）
  - list : 1-d tensor
  - list of list : 2-d tensor

等等；

例如： 

![](https://upload-images.jianshu.io/upload_images/5361608-ddc839d123dc7831.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-7616c685ca6085bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3.2 tensor type

- [全部类型](https://www.tensorflow.org/api_docs/python/tf/DType)

![](https://upload-images.jianshu.io/upload_images/5361608-96befea16e1db0dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-4aa75cfd5df45da3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3.3 numpy Type

 - tensor 的type基于numpy的type，因此例如 `tf.int32 == np.int32` 返回true;
在tensor中可以传递numpy的type：

![](https://upload-images.jianshu.io/upload_images/5361608-d94c5e020c92428c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 **1.tensor 和numpy的boolean，numeric value相互匹配；但是string不是完全匹配，tf能够导入numpy strings，只是不要指定numpy的dtypes;**

**2.numpy  不支持gpu,tensorflow 支持；** 

**3.tensorflow能够推断数据类型状态：如integer： tf中有：8-bit,16-bit,32-bit,64bit可用，但是python不能推断使用的那种数据类型**





---

