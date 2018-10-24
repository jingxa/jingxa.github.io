---
title: kali linux 循环登录
date: 2018-09-25 15:24:42
tags: 
	- kali
categories:
	- linux
---

# 背景

想要安装一个kali linux学习，但是虚拟机测过，太卡了， 不能满足需求，因此，只能安装多系统主机。

- 电脑： 电脑上已经安装了win10,ubuntu18.04；

在网上找过一些安装三系统的教程，但是说的似是而非，干脆破罐子破摔，直接上！（看了几个教程都说Ultraiso好像不行，因此直接使用Win32DiskImager）


# 安装步骤

1. [清华大学软件站](https://mirrors.tuna.tsinghua.edu.cn/) 下载kali系统
2. `Win32DiskImager`制作U盘启动盘
3. 先在`win10`下进行分区，压缩出一块空间不要格式化
4. 直接重启然后选择`boot`选项，u盘启动
5. 然后将`kali`安装到预留的磁盘
6. 在安装过程中，会识别到win10，ubuntu的启动项，直接选择已经有的启动项
7. 然后重启后直接进入kali的启动项，不过win10和ubuntu的启动项都在！(没有遇到什么win10和ubuntu的启动项消失的状况)


# 问题

1. 电脑是台式机机器，安装完成后，在登录界面出现问题，账号密码正确，但是不能登录进去！

解决办法：

- 首先 `ctrl + alt + F1`，【其中F1~F6都可以进入命令终端模式】,`ctrl+alt+F7`:回到窗口模式

- 然后，需要更新系统

```bash

sudo apt-get update

sudo apt-get upgrade -f
 

```

- `reboot`即可


# 我的问题

- 本人的环境是wifi环境，使用了USB的wifi连接器，然后连接wifi.


命令模式下的wifi连接方式：

- `wpa_cli`:这个工具比较方便

简单的使用教程：


```bash
> wpa_cli			# 终端中直接输入

# 一般默认的wlan为wlan0,即无线网卡的名字

> add_network		# 这个就是创建一个新的网络连接
> 0					# 默认创建的网络连接编号


> set_network 0 ssid "mywifi"	# 通过set_network给 连接0 添加 ssid ,引号中为wifi名称

> set_network 0 psk  "12345678"  # 添加密码 psk, 密码为引号中：12345678

> enable_network 0 				# 启动连接0

# 输入 q 退出 wpa_cli

> q

# 在终端中，打开连接

~#: dhclient wlan0                 # 刚刚在wlan0上创建的连接

```

- 不知道是我的wifi信号不好还是怎么回事？几分钟后连接必然断开，因此，选择另一种做法！


1. 如果没有`wpa_cli`，可能需要安装：

```bash
sudo apt-get install wireless-tools
```
2. 配置网络：`修改/etc/network/interfaces文件`

在这个文件中添加需要的部分

```bash

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0
iface wlan0 inet dhcp
    wpa-ssid YOUR-SSID-HERE
    wpa-psk YOUR-PASSWORD-HERE

```
- 替换上面`YOUR-SSID-HERE`： 为自己的wifi名称
- `YOUR-PASSWORD-HERE`: 为自己的wifi密码



3. 重启连接

```bash
/etc/init.d/networking restart
# or: service networking restart

```
- 如果启动失败，使用`journalctl -xe`查看系统日志，我发现我使用wpa_cli的时候启动了一个进程，杀掉相关的进程就可以了！





# 参考资料

1. [Can't login in Kali Linux](https://superuser.com/questions/987969/cant-login-in-kali-linux)
2. [wpa_supplicant及wpa_cli使用方法](https://segmentfault.com/a/1190000011579147)
3. [让你的树莓派连上WiFi](http://imchao.wang/2014/01/02/make-you-raspberrypi-auto-connect-to-wifi/)
4. [Debian 9 下折腾 usb 无线网卡上网](https://www.jianshu.com/p/69807d3d4474)


---

