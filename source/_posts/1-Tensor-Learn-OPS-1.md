---
title: 11_Tensor_Learn_1
date: 2018-06-21 21:32:59
tags:
	- TensorFlow
categories:
	- TensorFlow
	- Standford_CS20
---

----
## 1.4 variable


### 1.4.1 变量的概念

constant和variable的区别：

- tf. constant is an op
- tf. V ariable is  a class with multiple ops. 

![](https://upload-images.jianshu.io/upload_images/5361608-cc093a4fa77154a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 第二点说明了常量存储在graph中，constant是memory expensive， 因此，如果graph中的constant过多，那么每次载入graph 的速度变得很慢；

- 如果想要查看graph中的存储：

![](https://upload-images.jianshu.io/upload_images/5361608-77c7034cd2acbd18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输出为：

![](https://upload-images.jianshu.io/upload_images/5361608-94e58d3c55fd6e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建一个变量：

![](https://upload-images.jianshu.io/upload_images/5361608-8a43df33a3d32b29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

老的创建变量的方式为：

![](https://upload-images.jianshu.io/upload_images/5361608-917f5fbd29b25d8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是现在不推荐使用这种方式，推荐使用：`tf.get_variable`,参数如下，当使用tf.constant作为初始器来，不需要输入shape;

![](https://upload-images.jianshu.io/upload_images/5361608-8aac8812d20daba5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-43b02007952acd16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###  1.4.2 初始化变量

- 变量在使用前需要初始化，否则抛出`FailedPreconditionError:Attempting to use unintialized value`
- 如果想获得未初始化的变量列表：如下：


![](https://upload-images.jianshu.io/upload_images/5361608-d01410fa9698f0fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(1)最简单的初始化变量的方式为：

![](https://upload-images.jianshu.io/upload_images/5361608-9136055a81ae592e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(2)初始化变量的子集：
可以使用下面的`tf.variable_initializer()`

![](https://upload-images.jianshu.io/upload_images/5361608-0cb840a6b3713f4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(3)也可以对每个变量分开初始化`tf.Variable.initializer`：

![](https://upload-images.jianshu.io/upload_images/5361608-9c8d65f4f2aaf956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(4)从一个文件中载入值来初始化变量

---
### 1.4.3 求变量值
（1） 类似tensors,获得变量值需要在一个session下；

![](https://upload-images.jianshu.io/upload_images/5361608-75a5c52f035aa654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2 ）使用`tf.Variable.eval()`

![](https://upload-images.jianshu.io/upload_images/5361608-6c7d5246165cf231.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
print(V.eal()) # 获取值
```

### 1.4.4 变量赋值
（1） `tf.Variable.assign()`

![](https://upload-images.jianshu.io/upload_images/5361608-354852da3562ae67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- W结果为10 ： 赋值操作有效，必须在session下：

![](https://upload-images.jianshu.io/upload_images/5361608-cb8c618508c2ec0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在这段代码中，没有使用初始化操作，因为初始化操作就是一个赋值操作；

- 源代码中的初始化操作：

![](https://upload-images.jianshu.io/upload_images/5361608-146474becae7166d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个例子：

![](https://upload-images.jianshu.io/upload_images/5361608-cb2f4a344122e236.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ·tf.Variable.assign_add()·和`tf.Variable.assign_sub()`不同于assign操作；这两个方法不能初始化变量，因为这两个方法依赖变量的初始值；


![](https://upload-images.jianshu.io/upload_images/5361608-bca6b384feb42500.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 因为session分别维持着值，每个session 对于一个变量有着各自的当前值：

![](https://upload-images.jianshu.io/upload_images/5361608-244b9ea35721f8c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


假如一个变量W和另一个变量U是相关的，

![](https://upload-images.jianshu.io/upload_images/5361608-d279a789b35175b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

应该使用`initialized_value()`确保这个变量W初始化在U使用这个变量前：

![](https://upload-images.jianshu.io/upload_images/5361608-043caee5b2fa2547.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
## 1.5 Interactive Session

- 交互式session和默认session不同的是不需要显式调用session；

![](https://upload-images.jianshu.io/upload_images/5361608-24ea7a500d250dc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- `tf.get_default_session()`返回当前线程的默认session；


---
## 1.6  Control Dependencies

- 有时候，如果有多个独立的op，我们想要指定op的运行顺序：
- 使用`tf.Graph.control_dependencies([control_inputs])`

![](https://upload-images.jianshu.io/upload_images/5361608-4bed84beb31387fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


---
## 1. 7 Importing Data
### 1.7.1 old way: placeholders and feed_dict

- 如果想要构建一个图，但是不知道要计算的值：

- 定义一个占位符：

![](https://upload-images.jianshu.io/upload_images/5361608-5697bb05c4dd5fe9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果这个占位符的shape为none,代表任何shape的tensor被接受；建议尽可能定义shape，不要默认；

![](https://upload-images.jianshu.io/upload_images/5361608-329d9c901d0f2eaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-496bc6ff7f0cb47f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（1） 传递单一值
但是如果运行，会得到一个错误，需要给占位符一个输入；一般传入一个字典，key为占位符，value为占位符的值：

![](https://upload-images.jianshu.io/upload_images/5361608-b67aa9d16cf42036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2）传递多个值给一个占位符
使用循环传递：

![](https://upload-images.jianshu.io/upload_images/5361608-66c6107ea8f9e3f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（3） 检查tensor是否feedable

![](https://upload-images.jianshu.io/upload_images/5361608-29c2d6ed284cb64b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.7.2 new way:tf.data

-  后面线性回归举例子；



---
## 1.8  trap of lazy loading

- 一个不是bug的bug -- 'lazy loading'

- 在tensorflow中，直到你需要一个op才会创建它；

例如：
```
x = tf.Variable(10,name = 'x')
y = tf.Variable(20,name = 'y')
z = tf.add(x,y)

with tf.Session() as sess:
  sess.run(tf.global_variables_initializer())
  writer = tf.summary.FileWriter('graphs/normal_loading',sess.graph)
  for _ in range(10):
    sess.run(z)
  writer.close()
```

在循环中：

```
x = tf.Variable(10, name='x')
y = tf.Variable(20, name='y')
with tf.Session() as sess:
  sess.run(tf.global_variables_initializer())
  writer = tf.summary.FileWriter('graphs/lazy_loading', sess.graph)
  for _ in range(10):
    sess.run(tf.add(x, y))
  print(tf.get_default_graph().as_graph_def())
  writer.close()
```
第一个只有一个add节点，结果为：
```
node {
  name: "Add"
  op: "Add"
  input: "x/read"
  input: "y/read"
  attr {
    key: "T"
    value {
      type: DT_INT32
      }
    }
}
```
而第二种有多个add节点：

![](https://upload-images.jianshu.io/upload_images/5361608-a29cca95be6409d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如果不注意，就会在graph中添加很多节点；







---

