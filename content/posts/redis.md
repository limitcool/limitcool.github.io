---
title: "Redis常用命令"
description: 
date: 2022-04-21T09:42:24+08:00
draft: false
slug: redis
image:
categories:
  - Database
tags:
  - Database
  - Redis
---

# 安装`Redis`

## `Debian`下安装`Redis`服务端

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```

## `Windows`下安装`Redis` 第三方`GUI`客户端

Redis (GUI)管理客户端

```bash
winget install qishibo.AnotherRedisDesktopManager
```

## `Redis`修改监听端口

```bash
vim /etc/redis/redis.conf
```

# `Redis`常用命令

## `bitMap`

使用`BitMap`实现签到,`setbit key offset value,`  `key`做为时间,`offset`做为用户`id` ,`value`做为签到状态

```shell
# 示例
setbit key offset value key
# 设置用户10086在2022/04/21进行签到
setbit check_in_2022_04_21 10086 1 
# 获取用户10086是否在2022/04/21签到
getbit check_in_2022_04_21 10086
# bitcount 获取20220421签到的用户数量 
# 可选 start和end参数
# start 和 end 参数的设置和 GETRANGE 命令类似，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位
BITCOUNT 20220421 
# BITOP 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上

# operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种：

# BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。

# BITOP OR destkey key [key ...] ，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。

# BITOP XOR destkey key [key ...] ，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。

# BITOP NOT destkey key ，对给定 key 求逻辑非，并将结果保存到 destkey 。

# 除了 NOT 操作之外，其他操作都可以接受一个或多个 key 作为输入。

BITOP AND and-result 20220421 20220420
GETBIT and-result

```

## `Redis` 消息队列

```
# LPUSH key value, Lpush用于生产并添加消息
# LPOP key,用于取出消息
```

## `Lrem`

```shell
# count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
# count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
# count = 0 : 移除表中所有与 VALUE 相等的值。
LREM key count VALUE
```

## `Pipeline`

`Redis` 使用的是客户端-服务器（`CS`）模型和请求/响应协议的 TCP 服务器。这意味着通常情况下一个请求会遵循以下步骤：

客户端向服务端发送一个查询请求，并监听 Socket 返回，通常是以阻塞模式，等待服务端响应。
服务端处理命令，并将结果返回给客户端。
管道（`pipeline`）可以一次性发送多条命令并在执行完后一次性将结果返回，pipeline 通过减少客户端与 redis 的通信次数来实现降低往返延时时间，而且 `Pipeline` 实现的原理是队列，而队列的原理是时先进先出，这样就保证数据的顺序性。

通俗点：`pipeline`就是把一组命令进行打包，然后一次性通过网络发送到Redis。同时将执行的结果批量的返回回来

```go
// 使用 go-redis
	p := Client.Pipeline()
	for _, v := range val {
		p.LRem("user:watched:"+guid, 0, v)
	}
// p.Exec()执行pipeline 请求
	p.Exec()
```



[本文参考](https://blog.csdn.net/mumuwang1234/article/details/118603697)
