---
title: "Hugo使用指南！"
slug: "hugo"
draft: false
date: 2022-05-20T10:23:53+08:00
description: "快速上手hugo！"
tags:
  - Go
  - Hugo
categories:
  - Go
---
查看Hugo版本号

```bash
hugo version
```

新建一个Hugo页面

```
hugo new site <siteName>
```

设置主题

```bash
cd <siteName>
git init 

# 设置为 Stack主题
git clone https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
```

部署Hugo到github
