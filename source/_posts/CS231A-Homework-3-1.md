---
title: CS231A-Homework-3.1
date: 2018-10-22 21:05:31
tags:
	- space_carving
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-3:ps3_code-space_carving

---

# 一、 Space carving

本节内容实现一个有效的Space carving 框架的部分。
实现过程的步骤划分如下：

1. 生成初始体素网格
2. 实现一个视图下的carving
3. 多视图下的carving
4. 提高准确率
5. 验证


## 1. 生成初始体素网格：form_initial_voxels()

函数参数
- xlim:x轴大小：`[xmin, xmax]`
- ylim: y轴大小：`[ymin, ymax]`
- zlim: z轴大小：`[zmin, zmax]`
- num_voxels: voxels的数量

返回值：
- voxels: `(N,3)` 数组，返回N个体素的位置
- voxel_size: 每个voxel的size


> 计算voxel_size: 即求得整个立方体体积除去数量，但是可能由于不能除尽
> 1. 计算每个voxel的体积，然后开三次方根，得到size
> 2. 获得网格中心数组，使用`np.linspace `和`np.meshgrid`
> 注意小数问题

- `np.linspace(min,max,gap)`:在`[min,max]`区间以gap获得一个数组
- [np.meshgrid](https://blog.csdn.net/lllxxq141592654/article/details/81532855?utm_source=blogxgwz0):获得网格矩阵


```python
def form_initial_voxels(xlim, ylim, zlim, num_voxels):
    x_dim = xlim[1] - xlim[0]
    y_dim = ylim[1] - ylim[0]
    z_dim = zlim[1] - zlim[0]   # 获取三轴的长度

    total_vol = x_dim * y_dim * z_dim  # 体积
    one_vol = float(total_vol / num_voxels)
    voxel_size = np.cbrt(one_vol)                   # 三次方根

    x_voxel_num = np.round(x_dim / voxel_size)      # x轴的cube 数量,round 取整
    y_voxel_num = np.round(y_dim / voxel_size)
    z_voxel_num = np.round(z_dim / voxel_size)

    x = np.linspace(xlim[0] + 0.5 * voxel_size,
                    xlim[0] + (0.5 + x_voxel_num - 1) * voxel_size, x_voxel_num )     # 只取前 num_voxels个
    y = np.linspace(ylim[0] + 0.5 * voxel_size,
                    ylim[0] + (0.5 + y_voxel_num - 1) * voxel_size, y_voxel_num)     # 只取前 num_voxels个
    z = np.linspace(zlim[0] + 0.5 * voxel_size,
                    zlim[0] + (0.5 + z_voxel_num - 1) * voxel_size, z_voxel_num)     # 只取前 num_voxels个

    XX, YY, ZZ = np.meshgrid(x, y, z)
    voxels = np.r_[(XX.reshape(-1), YY.reshape(-1), ZZ.reshape(-1))].reshape(3, -1).T   # N *3

    return voxels, voxel_size




```

提示： 相关的函数：`np.meshgrid, np.repeat, np.tile`


## 2. 裁剪一个视图中不属于物体的体素：carve()

函数参数：
- voxels: 体素数组
- camera： 相机位置，存储数据： “silhouette” 矩阵， “image”, 和projection matrix "P"

返回：
- voxels


首先需要将3D体素投影到2D图片，移除不属于


```python

def carve(voxels, camera):
    N = voxels.shape[0]
    homo_voxels = np.c_[voxels, np.ones((N, 1))].T  # 4 * N
    voxel_index = np.arange(0, N)

    P = camera.P        # 投影矩阵 3 * 4
    img_voxels = P.dot(homo_voxels)   # 3 * N  , 投影到图片
    img_voxels /= img_voxels[2, :]   # 归一化
    img_voxels = img_voxels[0:2, :].T     # 去掉z轴 N*2

    sli = camera.silhouette  # 从camera文件中了解相关信息
    sli_idx = np.nonzero(sli)
    xmin, xmax = np.min(sli_idx[1]), np.max(sli_idx[1])     # 列
    ymin, ymax = np.min(sli_idx[0]), np.max(sli_idx[0])     # 行

    voxelX = img_voxels[:, 0]
    voxelY = img_voxels[:, 1]

    x_filter = np.all([voxelX > xmin, voxelX < xmax], axis=0)       # 一个轴上的逻辑与运算
    y_filter = np.all([voxelY > ymin, voxelY < ymax], axis=0)

    filter = np.all([x_filter, y_filter], axis=0)
    img_voxels = img_voxels[filter, :]      # 过滤大于轮廓矩阵的像素点
    voxel_index = voxel_index[filter]     # 过滤掉序号

    img_voxels = img_voxels.astype(int)     # 由于归一化，可能有小数，转为整数
    sli_filter = (sli[img_voxels[:, 1], img_voxels[:, 0]] == 1)     # (x,y)是否在轮廓矩阵中
    voxel_index = voxel_index[sli_filter]

    return voxels[voxel_index, :]



```


## 3. 获得更好的边界：get_voxel_bounds()

在初始版本中的方法中，获得voxel的边界是根据相机的位置来计算的【暂时没看明白】

如果需要获得更好的边界，可以先对初始的立方体进行初始剪切一遍，然后获取得到的voxels，取得最大和最小的点，作为边界值


```python


def get_voxel_bounds(cameras, estimate_better_bounds = False, num_voxels = 4000):
    camera_positions = np.vstack([c.T for c in cameras])

    print(camera_positions.shape)
    print("0:", camera_positions[0])

    xlim = [camera_positions[:,0].min(), camera_positions[:,0].max()]
    ylim = [camera_positions[:,1].min(), camera_positions[:,1].max()]
    zlim = [camera_positions[:,2].min(), camera_positions[:,2].max()]

    # For the zlim we need to see where each camera is looking. 
    camera_range = 0.6 * np.sqrt(diff( xlim )**2 + diff( ylim )**2)
    for c in cameras:
        viewpoint = c.T - camera_range * c.get_camera_direction()
        zlim[0] = min( zlim[0], viewpoint[2] )
        zlim[1] = max( zlim[1], viewpoint[2] )

    # Move the limits in a bit since the object must be inside the circle
    xlim = xlim + diff(xlim) / 4 * np.array([1, -1])
    ylim = ylim + diff(ylim) / 4 * np.array([1, -1])

    if estimate_better_bounds:
        voxels, voxel_size  = form_initial_voxels(xlim, ylim, zlim, num_voxels)

        for c in cameras:
            voxels = carve(voxels, c)

        min_point = np.min(voxels, axis=0) - voxel_size
        max_point = np.max(voxels, axis=0) + voxel_size

        xlim[0], ylim[0], zlim[0] = min_point[0], min_point[1], min_point[2]
        xlim[1], ylim[1], zlim[1] = max_point[0], max_point[1], max_point[2]
    return xlim, ylim, zlim
    

```



## 4. 结果

### 4.1 form_initial_voxels形成一个初始立方体

![iniital_voxels](/imgages/cs231a/ps3/space_c_a.png)

### 4.2 carving : 一个视角的裁剪

![one_carving](/imgages/cs231a/ps3/space_c_b.png)


### 4.3 没有优化边界的多视角裁剪

![muliti_carving](/imgages/cs231a/ps3/space_c_c.png)


### 4.4 优化边界的多视角裁剪

![best_carving](imgages/cs231a/ps3/space_c_d.png)




## 3. 参考文章

1. [zyxrrr/cs231a](https://github.com/zyxrrr/cs231a/blob/master/ps3/space_carving/main.py)
2. [CS231A/ps3_code/space_carving](https://github.com/mikucy/CS231A/blob/master/ps3_code/space_carving/main.py)
3. [cs231a/Homework/PS3/](https://github.com/chizhang529/cs231a/tree/master/Homework/PS3)


---

