---
title: "Linux部署DHCP服务"
description: Debian下使用docker镜像部署DHCP服务
date: 2022-05-23T11:11:40+08:00
draft: false
slug: linux-dhcp
image:
categories: Linux
tags:
  - Linux
  - DHCP
---

拉取`networkboot/dhcpd`镜像

```shell
docker pull networkboot/dhcpd
```

新建`data/dhcpd.conf`文件

```shell
touch /data/dhcpd.conf
```

修改`data/dhcpd.conf`文件

```
subnet 204.254.239.0 netmask 255.255.255.224 {
option subnet-mask 255.255.0.0;
option domain-name "cname.nmslwsnd.com";
option domain-name-servers 8.8.8.8;
range 204.254.239.10 204.254.239.30;
}
```

修改`/etc/network/interfaces`

```
# The loopback network interface (always required)
auto lo
iface lo inet loopback

# Get our IP address from any DHCP server
auto dhcp 
iface dhcp inet static
address 204.254.239.0
netmask 255.255.255.224

```



获取帮助命令

```shell
docker run -it --rm networkboot/dhcpd man dhcpd.conf
```

运行`DHCP`服务

```shell
docker run -it --rm --init --net host -v "/data":/data networkboot/dhcpd <网卡名称>
# 示例
docker run -it --rm --init --net host -v "/data":/data networkboot/dhcpd dhcp
```

