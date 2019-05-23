---
title: 谷歌vpn
date: 2019-02-23 00:23:12
tags:
  - vpn
---

### 安装shadowsocks

使用的是ubuntu16.4系统,使用root用户登录后输入一下命令

```bash
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

安装完成后，脚本提示如下

```
Congratulations, Shadowsocks-python server install completed!
Your Server IP        :your_server_ip
Your Server Port      :your_server_port
Your Password         :your_password
Your Encryption Method:your_encryption_method

Welcome to visit:https://teddysun.com/342.html
Enjoy it!
```

### 卸载shadowsocks

```
./shadowsocks.sh uninstall
```

### 单用户配置
配置文件路径：/etc/shadowsocks.json
```json

{
  "server":"0.0.0.0",
  "server_port":"your_server_port",
  "local_address":"127.0.0.1",
  "local_port":1080,
  "password":"your_password",
  "timeout":300,
  "method":"your_encryption_method",
  "fast_open": false
}
```

### 多用户配置
配置文件路径：/etc/shadowsocks.json

```json
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
```

### 使用指令

启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status


### 客户端下载地址

[shadowsocks window](https://github.com/shadowsocks/shadowsocks-windows/releases)
[shadowsocks android](https://github.com/shadowsocks/shadowsocks-android/releases)
[shadowsocks mac](https://github.com/shadowsocks/ShadowsocksX-NG/releases)