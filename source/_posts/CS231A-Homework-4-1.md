---
title: CS231A_Homework_4.1
date: 2018-11-06 20:19:00
tags:
	- Face Detection
	- homeworks
categories:
	- cs231A
---

> cs231A Homework-4: Face Detection

---

# 1. Face Detection with HoG
本节作业主要利用HoG来计算特征，并且作为目标检测的一部分。

## 1.1 运行SVM训练输出边界盒子

函数：`run_detector(im, clf, window_size, cell_size, block_size, nbins, thresh=1)`
参数：
- im: 图片
- clf: svm对象，使用`decision_function()`确定对象是否为face
- window_size: 滑动窗口的size数组
- cell_size: (cell_size, cell_size)包含pixels
- block_size: （block_size, block_size)包含的cells
- nbins： 直方图的bins

返回值：
- bboxes: (D * 4) 的边界盒子数组【xmin, ymin,width, height】
- scores: 每个边界盒子的SVM scores

实现：
1. 使用`compute_hog_features()`计算HoG特征
2. 对滑动窗口计算score,每次滑动的步长为n pixels，步长stride = (block_size * cell_size / 2)
3. SVM得到的score大于1，就认为窗口为边界盒子

```python

def run_detector(im, clf, window_size, cell_size, block_size, nbins, thresh=1):
    H, W = im.shape
    win_H, win_W = window_size
    stride = int(block_size * cell_size / 2)

    # 返回值
    bboxes = []
    scores = []

    # 计算每个窗口是否为face
    for i in range(0, W - win_W, stride):
        for j in range(0, H - win_H, stride):
            bbox = [i, j, win_W, win_H]     # 窗口 [xmin ymin width height]
            im_bbox = im[j:j+win_H, i: i+win_W]
            feature_im = compute_hog_features(im_bbox, cell_size, block_size, nbins)        # HoG
            score_im = clf.decision_function(feature_im.flatten().reshape(1,-1))        # 先将HoG特征变为vector,然后进行判断是否为face特征

            if score_im > thresh:
                scores.append(score_im)
                bboxes.append(bbox)

    # 变换为numpy类型
    bboxes = np.array(bboxes)
    scores = np.array(scores)

    return bboxes, scores



```



## 1.2 非最大化约束

在上一步执行的face窗口检测中，可能存在一个face侦测多个窗口的情况，对于每个窗口，都有一个score，因此对于同一个face的窗口，比较score,留下最高的窗口，移除剩下的窗口。

函数: `non_max_suppression(bboxes, confidences)`

参数： 
- `bboxes:` 窗口 (N, 4)
- confidences: 每个窗口的SVM confidence信度 （N,1），即score

返回值：
- nmss_bboxes: 非重叠的窗口，即移除弱特征的窗口

实现：
1. 首先sort 窗口confidence
2. 然后移除重叠的窗口
 

```python
def non_max_suppression(bboxes, confidences):
    nms_bboxs = []   # 返回窗口

    # 对confidences 排序
    con_idx = np.argsort(-confidences.flatten())          # 从大到小排列

    N = bboxes.shape[0]

    for i in range(N):
        bbox = bboxes[con_idx[i], :]        # 一个窗口,[xmin, ymin, width, height]

        if i == 0:      # 第一个窗口，加入结果序列
            nms_bboxs.append(bbox)
        else:           # 查看当前窗口是否和已有窗口重叠

            # 计算窗口中心
            cx = (2 * bbox[0] + bbox[2]) / 2
            cy = (2 * bbox[1] + bbox[3]) / 2

            isOverlap = False

            for j in range(len(nms_bboxs)):
                xmin, ymin, w, h = nms_bboxs[j][0], nms_bboxs[j][1],\
                                   nms_bboxs[j][2], nms_bboxs[j][3]

                xmax, ymax = (xmin + w), (ymin + h)

                if xmin <= cx <= xmax and ymin <= cy <= ymax:     # 两个窗口重叠
                    isOverlap = True
                    break

            if not isOverlap:       # 没有重叠
                nms_bboxs.append(bbox)

    nms_bboxs = np.array(nms_bboxs)
    return nms_bboxs



```

## 1.3 结果

![ps4_a_1](https://raw.githubusercontent.com/jingxa/cs231a_my/master/images/ps4/ps4_a_1.png)
![ps4_a_2](https://raw.githubusercontent.com/jingxa/cs231a_my/master/images/ps4/ps4_a_2.png)
 
 
---

