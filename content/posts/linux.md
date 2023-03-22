---
title: "Linux"
description: 
date: 2022-09-08T15:19:00+08:00
draft: true
slug: linux
image:
categories:
  - Linux
tags:
  - Linux
---

```bash
# 使用cd 进入到上一个目录
cd -
```

复制和粘贴

```bash
ctrl + shift + c
ctrl + shift + v 
```



快速移动

```bash
# 移动到行首
ctrl + a
# 移动到行尾
ctrl + e
```

快速删除

```bash
# 删除光标之前的内容
ctrl + u 
# 删除光标之后的内容
ctrl + k
# 恢复之前删除的内容
ctrl + y
```

不适用cat

```
使用less 查看 顶部的文件
less filename
```

使用alt+backspace删除,以单词为单位

```
 tcpdump host 1.1.1.1
```

```
# 并行执行命令 Parallel
find . -type f -name '*.html' -print | parallel gzip
```

