---
title: nginx笔记
date: 2018-09-24 23:55:12
tags:
  - nginx
---

##### 客户端nginx路径信息
配置文件路径: `/usr/local/etc/nginx/nginx.conf`
服务器默认路径: `/usr/local/var/www`
安装路径: `/usr/local/Cellar/nginx/1.13.9`

##### 启动
直接终端输入 `nginx` 启动，可以使用  `ps -ef|grep nginx` 来查看是否启动成功
![](http://ozrm3516s.bkt.clouddn.com/a6281635c5606767cbecf6528ce7ded9.jpg)
进程号为 3843
在终端输入 `kill -term  3843` 来停止进程

##### 重启

```bash
nginx -s reload
service nginx reload  #不停服务重启
```

##### 查找配置文件路径
mac下使用homebrew 安装 nginx 的路径：
`/usr/local/etc/nginx/nginx.conf`
```bash
ps -ef | grep nginx
```

###### 403 Forbidden权限问题解决

```bash
sudo chown -R $USER:$USER /root/www/blog
sudo chmod -R 755 /root/www   
```
