---
title: "如何在 Tmux 会话窗格中发送命令"
description: 本文介绍了在 Tmux 中发送命令的步骤，包括新建分离会话、发送命令至会话窗格、连接会话窗格、以及发送特殊命令。通过本文，读者将了解如何在 Tmux 中发送命令，并能够更加高效地使用 Tmux。
date: 2022-08-02T14:54:08+08:00
draft: false
slug: tmux
image:
categories:
  - Linux
tags:
  - Linux
  - Tmux
---

## 在 Tmux 会话窗格中发送命令的方法

在 `Tmux` 中，可以使用 `send-keys` 命令将命令发送到会话窗格中。以下是在 `Tmux` 中发送命令的步骤：

### 1. 新建一个分离(`Detached`)会话

使用以下命令新建一个分离会话：

```bash
tmux new -d -s mySession
```

### 2. 发送命令至会话窗格

使用以下命令将命令发送到会话窗格：

```bash
tmux send-keys -t mySession "echo 'Hello World!'" ENTER
```

这将发送 `echo 'Hello World!'` 命令，并模拟按下回车键(`ENTER`)，以在会话窗格中执行该命令。

### 3. 连接(`Attach`)会话窗格

使用以下命令连接会话窗格：

```bash
tmux a -t mySession
```

这将连接到名为 `mySession` 的会话窗格。

### 4. 发送特殊命令

要发送特殊命令，例如清除当前行或使用管理员权限运行命令，请使用以下命令：

- 清除当前行：`tmux send-keys C-c`
- 以管理员身份运行命令：`sudo tmux send-keys ...`
