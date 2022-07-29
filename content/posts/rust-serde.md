---
title: "Rust使用Serde进行序列化及反序列化"
description: 
date: 2022-07-25T14:02:22+08:00
draft: false
slug: rust-serde
image:
categories: 
  - Rust
tags:
  - Rust
  - Xml
---

# 开始之前

```toml
# 在Cargo.toml 新增以下依赖
[dependencies]
serde = { version = "1.0.140",features = ["derive"] }
serde_json = "1.0.82"
serde_yaml = "0.8"
serde_urlencoded = "0.7.1"
# 使用yaserde解析xml
yaserde = "0.8.0"
yaserde_derive = "0.8.0"
```

## `Serde`通用规则(`json`,`yaml`,`xml`)

### 1.使用`Serde`宏通过具体结构实现序列化及反序列化 

```rust
use serde::{Deserialize, Serialize};
// 为结构体实现 Serialize(序列化)属性和Deserialize(反序列化)
#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct Person {
    // 将该字段名称修改为lastname
    #[serde(rename = "lastname")]
    name: String,
    // 反序列化及序列化时忽略该字段(nickname)
    #[serde(skip)]
    nickname: String,
    // 分别设置序列化及反序列化时输出的字段名称
    #[serde(rename(serialize = "serialize_id", deserialize = "derialize_id"))
    id: i32,
    // 为age设置默认值
    #[serde(default)]
    age: i32,
    
}
```

### 2.使用`serde_json`序列化及反序列化

```rust
use serde_json::{json, Value};
let v:serde_json::Value = json!(
    {
        "x":20.0,
    	"y":15.0
    }
);
println!("x:{:#?},y:{:#?}",v["x"],v["y"]); // x:20.0, y:15.0
```

### 3.使用`Serde`宏统一格式化输入、输出字段名称

| 方法名                          | 方法效果                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| `PascalCase`                    | 首字母为大写的驼峰式命名,推荐结构体、枚举等名称以及`Yaml`配置文件读取使用。 |
| `camelCase`                     | 首字母为小写的驼峰式命名,推荐`Yaml`配置文件读取使用。        |
| `snake_case`                    | 小蛇形命名,用下划线"`_`"连接单词,推荐函数命名以及变量名称使用此种方式。 |
| `SCREAMING_SNAKE_CASE`          | 大蛇形命名,单词均为大写形式,用下划线"`_`"连接单词。推荐常数及全局变量使用此种方式。 |
| `kebab-case`(小串烤肉)          | 同`snake_case`,使用中横线"`-`"替换了下划线"`_`"。            |
| `SCREAMING-KEBAB-CAS`(大串烤肉) | 同`SCREAMING_SNAKE_CASE`,使用中横线"`-`"替换了下划线"`_`"。  |

示例:

```rust
pub struct App {
    #[serde(rename_all = "PascalCase")]
    /// 统一格式化输入、输出字段名称
    ///  #[serde(rename_all = "camelCase")]
    /// #[serde(rename_all = "snake_case")]
    /// #[serde(rename_all = "SCREAMING_SNAKE_CASE")]
    /// 仅设置
    version: String,
    app_name: String,
    host: String,
}
```

[本文参考:yaserde](https://github.com/media-io/yaserde)

[本文参考:magiclen](https://magiclen.org/rust-serde/)