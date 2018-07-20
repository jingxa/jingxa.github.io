---
title: MTProxy：专为Telegram打造的代理工具
date: 2018-07-15 17:23:50
tags:
	- MTProxy
categories:
	- 杂乱笔记
---
> 转载自：https://lala.im/3598.html
---

> 记录一下操作

---

接着我们按键盘组合键Ctrl+C退出运行。
现在来创建一个系统服务，可以在往后更方便的管理和运行MTProxy。

新建服务文件：
```
vi /etc/systemd/system/MTProxy.service

```

写入：
```
[Unit]
Description=MTProxy
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/MTProxy
ExecStart=/root/MTProxy/objs/bin/mtproto-proxy -u nobody -p 8888 -H 2333 -S 密匙 --aes-pwd /root/MTProxy/objs/bin/proxy-secret /root/MTProxy/objs/bin/proxy-multi.conf -M 1
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

注：

1、确保WorkingDirectory以及ExecStart内正确写明mtproto-proxy可执行文件的绝对路径。
以及指定proxy-secret、proxy-multi.conf的路径也是绝对路径。如果你是按照我的这篇文章来一字不动部署的，那么可以直接看第2点注意说明。

2、“密匙”改为之前你生成的密匙。

重加载，让新的服务文件生效：
```
systemctl daemon-reload

```
现在就可以启动MTProxy了：
```
systemctl start MTProxy.service
```

查看运行状态：
```
systemctl status MTProxy.service
```

如图所示是Active就说明MTProxy运行正常：

![](https://lala.im/wp-content/uploads/2018/06/lala.im_2018-06-11-00-795.png)

把MTProxy设为开机启动：
```
systemctl enable MTProxy.service
```

停止MTProxy的运行：
```
systemctl stop MTProxy.service


```
OK，到这里，MTProxy的服务端就部署完成了，


---

