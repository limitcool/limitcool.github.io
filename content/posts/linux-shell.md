---
title: "Linux Shell"
description: 
date: 2022-05-21T10:02:09+08:00
draft: false
Hidden: true
slug: linux-shell
image:
categories:
  Linux
tag:
  Linux
  Shell
---

Linux守护进程:no_good:

```bash
#!/bin/bash
# nohup.sh
while true
do 
    # -f 后跟进程名,判断进程是否正在运行
    if [ `pgrep -f <ProcessName> | wc -l` -eq 0 ];then
        echo "进程已终止"
        push
        # /dev/null 无输出日志
        nohup ./<ProcessName> > /dev/null 2>&1 &
    else
        echo "进程正在运行"
    fi
    # 每隔1分钟检查一次
    sleep 1m
done    
```

