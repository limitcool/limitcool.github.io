---
title: "手把手教你用Rust进行Dll注入"
description: 我是一个懒惰的男孩,我甚至懒的不想按键盘上的按键和挪动鼠标.可是我还是想玩游戏,该怎么做呢？通过 google 了解到我可以通过将我自己编写的dll文件注入到目标程序内,来实现这个事情.
date: 2022-09-17T15:10:26+08:00
draft: false
slug: rust-dll
image:
categories:
  - Rust
tags:
  - Rust
  - Dll
---

# 前言

我是一个懒惰的男孩,我甚至懒的不想按键盘上的按键和挪动鼠标.可是我还是想玩游戏,该怎么做呢？

通过google了解到我可以通过将我自己编写的 `dll` 文件注入到目标程序内,来实现这个事情.

将大象放在冰箱里需要几步？

答案是三步。

# `snes9x` 模拟器 `Dll` 注入实战

## 一、现在我们需要进行第一步,生成 `Dll` 文件

准确说是我们需要生成符合 `C` 标准的 `dll` 文件,如果你使用 `go` 语言,直接使用 `Cgo` 与 `C` 进行互动,即可生成符合 `C` 标准的 `dll` .

但是很明显,我要用 `Rust` 来做这件事。

由于 `Rust` 拥有出色的所有权机制,和其他语言的交互会导致 `Rust` 失去这个特性,所以这一块是属于 `Unsafe` 区域的。

`Rust` 默认生成的 `Dll` 是提供给 `Rust` 语言来调用的,而非C系语言的 `dll`.

我们现在来生成 `C` 系语言的 `Dll` 吧。

### 1.新建项目 `lib` 目录 `lib` 目录主要作为库文件以方便其他开发者调用

```bash
# 新建库项目
Cargo new --lib <project name>
Cargo new --lib joy
```

### 2.修改 `Cargo.toml` 文件 增加 `bin` 区域

```toml
[package]
name = "joy"
version = "0.1.0"
edition = "2021"

[lib]
name = "joy"
path = "src/lib.rs"
crate-type = ["cdylib"]

[[bin]]
name = "joyrun"
path = "src/main.rs"
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
```

```bash
# 为项目导入依赖ctor来生成符合c标准的dll
cargo add ctor 
```

### 3.修改 `lib.rs` 使用 `ctor`

```rust
// lib.rs
#[ctor::ctor]
fn ctor() {
    println!("我是一个dll")
}
```

#### 4.编译项目生成 `joy.dll` 以及 `joyrun.exe`

```bash
cargo build 
```

现在我们有了我们自己的 `dll` 文件,该如何将他注入到目标的进程呢？

## 二、使用 `dll-syringe` 进行dll注入

```
cargo add dll-syringe
```

### 1.修改main.rs 将刚刚编写的dll注入到目标应用

```rust
// main.rs
use dll_syringe::{Syringe, process::OwnedProcess};

fn main() {
    // 通过进程名找到目标进程
    let target_process = OwnedProcess::find_first_by_name("snes9x").unwrap();
 
    // 新建一个注入器
    let syringe = Syringe::for_process(target_process);

    // 将我们刚刚编写的dll加载进去
    let injected_payload = syringe.inject("joy.dll").unwrap();

    // do something else

    // 将我们刚刚注入的dll从目标程序内移除
    syringe.eject(injected_payload).unwrap();
}
```

### 2.运行项目

```shell
# 运行项目
cargo run 
```

此时你可能会遇到一个新问题,我的`dll`已经加载进目标程序了,为什么没有打印 "我是一个dll"

### 3.解决控制台无输出问题

这是由于目标程序没有控制台,所以我们没有看到 `dll` 的输出,接下来让我们来获取 `dll` 的输出。

此时我们可以使用 `TCP` 交互的方式或采用 `OutputDebugStringA function (debugapi.h)` 来进行打印

`OutputDebugStringA` ,需要额外开启`features` `Win32_System_Diagnostics_Debug`

```rust
// Rust Unsafe fn
// windows::Win32::System::Diagnostics::Debug::OutputDebugStringA
pub unsafe fn OutputDebugStringA<'a, P0>(lpoutputstring: P0) 
where
    P0: Into<PCSTR>, 
// Required features: "Win32_System_Diagnostics_Debug"
```

采用 `Tcp` 通信交互

```rust
// 在lib.rs 新建tcp客户端
let stream = TcpStream::connect("127.0.0.1:7331").unwrap();
```

```rust
    // 在main.rs 新建tcp服务端
    let (mut stream, addr) = listener.accept()?;
    info!(%addr,"Accepted!");
    let mut buf = vec![0u8; 1024];
    let mut stdout = std::io::stdout();
    while let Ok(n) = stream.read(&mut buf[..]) {
        if n == 0 {
            break;
        }
        stdout.write_all(&buf[..n])?
 }
```

```shell
# 运行项目
cargo run 
# 运行之后,大功告成,成功在Tcp服务端看到了,客户端对我们发起了请求。
```
