---
title: 我的新服务器配置流程(ubuntu)
date: 2017-09-20 09:07:50
keywords:
tags:
 - ubuntu
 - linux
---

# 基本设置

## 升级

```
apt update
apt upgrade
reboot
```

## 修改hostname
```
vi /etc/hostname
vi /etc/hosts
```

## 创建用户

同时加入sudoer并设置sudo免密码

> `/etc/sudoers`默认没有写权限

<!-- more -->

```
adduser myuser
chmod +x /etc/sudoers
vi /etc/sudoers
```

在`/etc/sudoers`中加入一行
```
myuser     ALL=(ALL:ALL) NOPASSWD: ALL
```

最后要去掉`/etc/suders`的写权限
```
chmod -x /etc/sudoers
```

## 设置sshkey登录

编辑`/etc/sshd/sshd_config`

```
`PubkeyAuthentication yes`
`AuthorizedKeysFile      .ssh/authorized_keys`
`GSSAPIAuthentication no`
```

重启`sshd`服务

```
service sshd restart
```

本地设置

先看一下本地`~/.ssh`路径下有没有`id_rsa.pub`，如果没有则用`ssh-keygen`生成一个，运行`ssh-keygen`之后一路回车即可

然后把`id_rsa.pub`中的内容添加到*服务器*的`/home/myuser/.ssh/authorized_keys`文件最后面(新建一行)

最后使用myuser用户登录服务器,假设服务器的ip是`192.168.1.123`

```
ssh myuser@192.168.1.123
```

此时不用密码即可登录

# 安装最新版的nginx

> 默认源中的nginx版本低

删除旧版本nginx

```
sudo apt purge nginx
```

增加nginx的ppa源

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:nginx/stable
sudo apt update
```

安装nginx

```
sudo apt install nginx
```

# 安装redis 3.2

> 默认源中的`redis`是3.0，bug很多

下载并解压缩

```
wget http://download.redis.io/releases/redis-3.2.5.tar.gz
tar xf redis-3.2.5.tar.gz
```

安装依赖

```
sudo apt-get install tcl
```

编译并安装

```
make
make test
sudo make install
```

安装系统服务

```
cd $REDIS_SOURCE/utils
sudo ./install_server.sh
```

此时`redis`安装完成，默认的系统服务名字是`redis_6379`
