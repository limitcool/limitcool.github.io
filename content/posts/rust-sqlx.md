---
title: "Rust Sqlx"
description: 
date: 2022-08-29T13:55:08+08:00
draft: false
slug: rust-sqlx
image:
categories:
  - 
tags:
  - 
---

# sqlx-cli

## 创建 migration

```shell
sqlx migrate add categories
```

```sql
-- Add migration script here
CREATE TABLE IF NOT EXISTS categories(
   id INT PRIMARY KEY DEFAULT AUTO_INCREMENT,
   type_id INT UNIQUE NOT NULL,
   parent_id INT NOT NULL,
   name TEXT UNIQUE NOT NULL,
  );
```

## 运行 migration

```sh
sqlx migrate run
```

