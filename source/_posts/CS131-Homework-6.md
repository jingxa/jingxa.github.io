---
title: CS131_Homework_6
date: 2018-12-08 16:14:56
tags:
	- CS131
	- homeworks
categories:
	- CS131
---
> cs131 hw6
---


# 主要内容

- image compression using SVD
- kNN methods for image recognition.
- PCA and LDA to improve kNN



# 参考资料

- [wwdguu/CS131_homework](https://github.com/wwdguu/CS131_homework/tree/master/hw6_release)
- [Solutions-Stanford-cs131-Computer-Vision-Foundations-and-Application/hw6_release/](https://github.com/Hugstar/Solutions-Stanford-cs131-Computer-Vision-Foundations-and-Application/tree/master/hw6_release)



# 1. Image  Compression


## 1.1 使用SVD压缩图片

函数： `def compress_image(image, num_values)`
参数：
- images: (H,w)
- num_values: 保留的奇异值
返回值：
- compressed_image: (H,w) 压缩后的图片
- compressed_size: 压缩图片的size

实现：
1. Get SVD of the image
2. Only keep the top `num_values` singular values, and compute `compressed_image`
3. Compute the compressed size

```python


def compress_image(image, num_values):
    compressed_image = None
    compressed_size = 0

    # YOUR CODE HERE
    H, W = image.shape
    # Steps:
    #     1. Get SVD of the image
    u, s, vt = np.linalg.svd(image)
    #     2. Only keep the top `num_values` singular values, and compute `compressed_image`
    sigular = np.diag(s[:num_values])       # 格式为对角阵
    compressed_image = u[:, :num_values].dot(sigular).dot(vt[:num_values, :])    # (H,n) * (n, n) * ( n, W)
    #     3. Compute the compressed size, 保留的数据为： (H,n), (sigular), (n, w)
    compressed_size = H * num_values + num_values + num_values * W
    pass
    # END YOUR CODE

    assert compressed_image.shape == image.shape, \
           "Compressed image and original image don't have the same shape"

    assert compressed_size > 0, "Don't forget to compute compressed_size"

    return compressed_image, compressed_size


```

# 2. k-Nearest Neighbor
使用knn算法分类，使用图片的原始特征，例如像素值；
步骤如下：
1. 计算X_train 和 X_test 的两个元素之间的L2距离
2. 将数据集划分为5个文件为了交叉验证(cross-validation)
3. 对于每个文件,对于不同的k，进行预测和评估准确度
4. 最终获得最佳的k，来评估精确度

## 2.1 compute_distance
函数： `def compute_distances(X1, X2)`
参数：
- X1，X2: (M,D)
实现：
- L2 距离就是欧式距离

```python
def compute_distances(X1, X2):

    M = X1.shape[0]
    N = X2.shape[0]
    assert X1.shape[1] == X2.shape[1]

    dists = np.zeros((M, N))

    # YOUR CODE HERE
    # (x-y)^2 == x^2 + y^2 -2xy
    x_2 = np.sum(X1**2, axis=1)[:, np.newaxis]     # 增加一个维度
    y_2 = np.sum(X2**2, axis=1)
    xy_2 = np.dot(X1, X2.T)
    dists = x_2 + y_2 - 2 * xy_2
    # END YOUR CODE

    assert dists.shape == (M, N), "dists should have shape (M, N), got %s" % dists.shape

    return dists

```


## 2.2 predict labels

函数： `def predict_labels(dists, y_train, k=1)`
参数：
- dists: (num_test, num_train)
实现：
- 对于每个test寻找最近的train图片，然后统计k个最相似图片的类别，比重占最大的类别就是test的预测类别

```python
def predict_labels(dists, y_train, k=1):

    num_test, num_train = dists.shape
    y_pred = np.zeros(num_test, dtype=np.int)

    for i in range(num_test):
        # A list of length k storing the labels of the k nearest neighbors to
        # the ith test point.
        closest_y = []
        # Use the distance matrix to find the k nearest neighbors of the ith
        # testing point, and use self.y_train to find the labels of these
        # neighbors. Store these labels in closest_y.
        # Hint: Look up the function numpy.argsort.

        # Now that you have found the labels of the k nearest neighbors, you
        # need to find the most common label in the list closest_y of labels.
        # Store this label in y_pred[i]. Break ties by choosing the smaller
        # label.

        # YOUR CODE HERE
        indices = np.argsort(dists[i])      # 最近的label,排序
        closest_y = y_train[indices[:k]]    # 选择最近的k个
        # y_train 为重复的label，本例子中有800章图片，共16个分类
        y_pred[i] = np.bincount(closest_y).argmax()     # 选择k个最相似的label中重复最多的
        # END YOUR CODE

    return y_pred


```


## 2.3 cross-validation

选择k个最相似的参数，不能确定k，通过 cross-validation 来选择k；

将所有训练集分为5个文件，
- 80% ： 训练集
- 20%%： 验证集

函数：`split_folds(X_train, y_train, num_folds)`

```python

def split_folds(X_train, y_train, num_folds):
    """Split up the training data into `num_folds` folds.

    The goal of the functions is to return training sets (features and labels) along with
    corresponding validation sets. In each fold, the validation set will represent (1/num_folds)
    of the data while the training set represent (num_folds-1)/num_folds.
    If num_folds=5, this corresponds to a 80% / 20% split.

    For instance, if X_train = [0, 1, 2, 3, 4, 5], and we want three folds, the output will be:
        X_trains = [[2, 3, 4, 5],
                    [0, 1, 4, 5],
                    [0, 1, 2, 3]]
        X_vals = [[0, 1],
                  [2, 3],
                  [4, 5]]

    Args:
        X_train: numpy array of shape (N, D) containing N examples with D features each
        y_train: numpy array of shape (N,) containing the label of each example
        num_folds: number of folds to split the data into

    jeturns:
        X_trains: numpy array of shape (num_folds, train_size * (num_folds-1) / num_folds, D)
        y_trains: numpy array of shape (num_folds, train_size * (num_folds-1) / num_folds)
        X_vals: numpy array of shape (num_folds, train_size / num_folds, D)
        y_vals: numpy array of shape (num_folds, train_size / num_folds)

    """
    assert X_train.shape[0] == y_train.shape[0]

    validation_size = X_train.shape[0] // num_folds
    training_size = X_train.shape[0] - validation_size

    X_trains = np.zeros((num_folds, training_size, X_train.shape[1]))
    y_trains = np.zeros((num_folds, training_size), dtype=np.int)
    X_vals = np.zeros((num_folds, validation_size, X_train.shape[1]))
    y_vals = np.zeros((num_folds, validation_size), dtype=np.int)

    # YOUR CODE HERE
    # Hint: You can use the numpy array_split function.
    # 例如 [1, 2, 3, 4,5]
    # [1,2,3,4: 5]
    # [2,3,4,5: 1]
    # [3,4,5,1: 2]
    # [4,5,1,2: 3]
    # [5,1,2,3: 4] 这五种组合方式
    X_num_folds = np.array(np.array_split(X_train, num_folds))
    y_num_folds = np.array(np.array_split(y_train,num_folds))

    for i in range(num_folds):
        X_trains[i] = X_num_folds[(np.arange(num_folds) != i)].reshape((-1,X_trains.shape[-1]))
        X_vals[i] = X_num_folds[i]

        y_trains[i] = y_num_folds[(np.arange(num_folds) != i)].reshape((-1, y_trains.shape[-1]))
        y_vals[i] = y_num_folds[i]

    # END YOUR CODE

    return X_trains, y_trains, X_vals, y_vals


```

# 3. PCA
PCA方法的总结：

1. 标准化数据
2. 计算特征向量和特征值 
3. 按照降序将特征值排序，选择前k个特征值和对应的特征向量，
4. 建立映射矩阵W,从选择的k个特征向量
5. 将原始的数据集X通过W降低维度为K的子空间Y

## 3,1 Eigen decomposition


类：`class PCA(object)`

函数：`def _eigen_decomp(self, X)`
返回值：
- e_vecs: 特征向量(D,D)
- e_vals: 特征值：(D,)
实现： 
- 主要功能是执行特征协方差矩阵的特征分解

```python
    def _eigen_decomp(self, X):
        N, D = X.shape
        e_vecs = None
        e_vals = None
        # YOUR CODE HERE
        # Steps: 计算步骤如下
        #     1. compute the covariance matrix of X, of shape (D, D)
        matrix = np.matmul(X.T, X) / (X.shape[0] - 1)        # 协方差矩阵 (D,D)
        #     2. compute the eigenvalues and eigenvectors of the covariance matrix
        e_vals, e_vecs = np.linalg.eig(matrix)      # 计算特征值特征向量
        #     3. Sort both of them in decreasing order (ex: 1.0 > 0.5 > 0.0 > -0.2 > -1.2)
        indices = np.argsort(-e_vals)       # 从达到小排序，序号
        e_vals = np.real(e_vals[indices])   # 返回实数
        e_vecs = np.real(e_vecs[:, indices])
        # END YOUR CODE
        # Check the output shapes
        assert e_vals.shape == (D,)
        assert e_vecs.shape == (D, D)

        return e_vecs, e_vals


```

## 3.2 SVD

函数：`   def _svd(self, X)`
返回值：
返回值：
- e_vecs: 特征向量(D,D)
- e_vals: 特征值：(D,)
实现：
- 实现svd分解

```python

    def _svd(self, X):

        vecs = None  # shape (D, D)
        N, D = X.shape
        vals = None  # shape (K,)
        # YOUR CODE HERE
        # Here, compute the SVD of X
        # Make sure to return vecs as the matrix of vectors where each column is a singular vector
        # U: X*X.T , V.T = X.T*X, S:square roots of the eigenvalues of U or V.T
        u, s, vt = np.linalg.svd(X)
        vals = s
        vecs = vt.T         # 转置
        # END YOUR CODE
        assert vecs.shape == (D, D)
        K = min(N, D)
        assert vals.shape == (K,)

        return vecs, vals


```


## 3.3 Dimensionality Reduction

对原始数据集进行降维处理

函数： `   def transform(self, X, n_components)`


```python




```



## 3.4  Reconstruction error and captured variance

将降低维度后的数据重新映射到原空间

函数：`def reconstruct(self, X_proj)`


```python
    def reconstruct(self, X_proj):
        N, n_components = X_proj.shape
        X = None

        # YOUR CODE HERE
        # Steps:
        #     1. project back onto the original space of dimension D
        X = np.zeros((N, self.W_pca.shape[1]))      # 原空间
        X[:, :n_components] = X_proj
        #     2. add the mean that we substracted in `transform`
        X = X.dot(np.linalg.inv(self.W_pca)) + self.mean
        pass
        # END YOUR CODE

        return X


```




# 4.  Fisherface: Linear Discriminant Analysis

LDA和PCA的区别：
- LDA需要labels，不是完全无监督，PCA是完全无监督的
- PCA保持 `maximum variance`
- LDA保持 `discimination between classes`

## 4.1 Dimensionality Reduction via PCA
对于本例子，N= 800, D = 4096, 为了应用LDA,需要使得D<N,使用PCA来降低维度；

## 4.2 Scatter matrices
为了分类，分别计算类内散度矩阵Sw, 和类间散度矩阵Sb

函数：`def _within_class_scatter(self, X, y)`
实现：
- 对于每个类，计算协方差矩阵
- 最后结果为协方差矩阵之和

```python

    def _within_class_scatter(self, X, y):
        _, D = X.shape
        assert X.shape[0] == y.shape[0]
        scatter_within = np.zeros((D, D))

        for i in np.unique(y):
            # YOUR CODE HERE
            # Get the covariance matrix for class i, and add it to scatter_within
            X_i = X[y == i]     # 类为i
            X_i_centered = X_i - np.mean(X_i, axis=0)
            S_i = np.matmul(X_i_centered.T, X_i_centered)       # 协方差矩阵
            scatter_within += S_i       # 结果
            # END YOUR CODE

        return scatter_within
```

函数： `def _between_class_scatter(self, X, y)`
实现： 
- 计算两个类之间的协方差矩阵

```python

    def _between_class_scatter(self, X, y):

        _, D = X.shape
        assert X.shape[0] == y.shape[0]
        scatter_between = np.zeros((D, D))

        mu = X.mean(axis=0)
        for i in np.unique(y):
            # YOUR CODE HERE
            X_i = X[y==i]           # 类别i
            mu_i = np.mean(X_i, axis=0) # 类别i的均值
            scatter_between += (len(y[y==i])) * np.matmul((mu_i - mu).T, (mu_i - mu))

            # END YOUR CODE

        return scatter_between

```


## 4.3 Solving generalized Eigenvalue problem

拟合训练数据
计算完类内散度矩阵Sw，类间散度矩阵Sb 后，计算矩阵Sw^-1Sb的特征向量和特征矩阵；


函数： `def fit(self, X, y)`

```python
    def fit(self, X, y):
        N, D = X.shape

        scatter_between = self._between_class_scatter(X, y)
        scatter_within = self._within_class_scatter(X, y)

        e_vecs = None

        # YOUR CODE HERE
        # Solve generalized eigenvalue problem for matrices `scatter_between` and `scatter_within`
        # Use `scipy.linalg.eig` instead of numpy's eigenvalue solver.
        # Don't forget to sort the values and vectors in descending order.
        e_vals, e_vecs = scipy.linalg.eig(np.linalg.inv(scatter_within).dot(scatter_between))
        # END YOUR CODE
        sorting_order = np.argsort(e_vals)[::-1]
        e_vals = e_vals[sorting_order]
        e_vecs = e_vecs[:,sorting_order]        # 从大到小排序

        self.W_lda = e_vecs

        # Check that the shape of `self.W_lda` is correct
        assert self.W_lda.shape == (D, D)

        # Each column of `self.W_lda` should have norm 1 (each one is an eigenvector)
        for i in range(D):
            assert np.allclose(np.linalg.norm(self.W_lda[:, i]), 1.0)
```

---

