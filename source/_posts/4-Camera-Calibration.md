---
title: 4_Camera_Calibration
date: 2018-06-22 10:33:29
tags:
	- Single_View_Metrology
	- Camera_Calibration
categories:
	- cs231A
---
> CS231A笔记
> 文章截图大多来自斯坦福CS231A课程;
---

# 1 单视图校准

![](https://upload-images.jianshu.io/upload_images/5361608-f62ca01593756407.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 回忆一下，相机参数有5个自由度；
![](https://upload-images.jianshu.io/upload_images/5361608-e93f7fea43d1c564.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- w 为：
![](https://upload-images.jianshu.io/upload_images/5361608-b1780f2198be53b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设2,3 条件为真，那么w为：

![](https://upload-images.jianshu.io/upload_images/5361608-2193d397907e4109.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 这样就剩下了四个变量；
- 只需要三对方程就可以求得变量；

![](https://upload-images.jianshu.io/upload_images/5361608-5eb9d77e6a71dd07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-82a9d2ce14a6c8d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5361608-25e8a60c165779cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





---

