---
title: "使用arm交叉编译工具并解决GLIBC版本不匹配的问题"
description: 介绍如何使用arm交叉编译工具来编译Go程序，并解决在arm平台上运行时出现GLIBC版本不匹配的问题。
date: 2022-06-10T15:00:26+08:00
draft: false
slug: go-arm
image:
categories:
  - Go
tags:
  - Arm
  - Go
  - GLIBC
---

1. 下载 ARM 交叉编译工具，可以从官方网站下载。比如，可以从如下链接下载 GNU 工具链：[https://developer.arm.com/downloads/-/gnu-a](https://developer.arm.com/downloads/-/gnu-a)

   示例:https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-mingw-w64-i686-aarch64-none-elf.tar.xz

2.  设置 Go ARM 交叉编译环境变量。具体来说，需要设置以下变量：

```ruby
$env:GOOS="linux"
$env:GOARCH="arm64"
$env:CGO_ENABLED=1 
$env:CC="D:\arm\gcc-arm-10.3-2021.07-mingw-w64-i686-aarch64-none-linux-gnu\bin\aarch64-none-linux-gnu-gcc.exe"
$env:CXX="D:\arm\gcc-arm-10.3-2021.07-mingw-w64-i686-aarch64-none-linux-gnu\bin\aarch64-none-linux-gnu-g++.exe"
```

3.  在 ARM 上运行程序时可能会出现如下错误：

```bash
./bupload: /lib/aarch64-linux-gnu/libc.so.6: version `GLIBC_2.28' not found (required by ./bupload)
./bupload: /lib/aarch64-linux-gnu/libc.so.6: version `GLIBC_2.32' not found (required by ./bupload)
./bupload: /lib/aarch64-linux-gnu/libc.so.6: version `GLIBC_2.33' not found (required by ./bupload)
```

这是因为程序需要使用较新版本的 GLIBC 库，而 ARM 上安装的库版本较旧。可以通过以下步骤来解决这个问题：

4.  查看当前系统中 libc 库所支持的版本：

```bash
strings /lib/aarch64-linux-gnu/libc.so.6 | grep GLIBC_
```

5.  备份整个 `/lib` 目录和 `/usr/include` 目录，以便稍后还原。
6.  从 GNU libc 官方网站下载对应版本的 libc 库。例如，可以从如下链接下载 2.35 版本的 libc 库：[http://ftp.gnu.org/gnu/glibc/glibc-2.35.tar.xz](http://ftp.gnu.org/gnu/glibc/glibc-2.35.tar.xz)
7.  解压 libc 库：

```
xz -d glibc-2.35.tar.xz
tar xvf glibc-2.35.tar glibc-2.35
```

8.  创建并进入 build 目录：

```bash
mkdir build
cd build
```

9.  配置 libc 库的安装选项：

```javascript
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
```

10.  编译并安装 libc 库：

```go
make -j4
make install
```

接下来是关于 `make` 报错的部分：

```yaml
asm/errno.h: No such file or directory
```

这个报错是因为 `errno.h` 文件中包含了 `asm/errno.h` 文件，但是找不到这个文件。为了解决这个问题，我们需要创建一个软链接：

```bash
ln -s /usr/include/asm-generic /usr/include/asm
```

然后又出现了另一个报错：

```bash
/usr/include/aarch64-linux-gnu/asm/sigcontext.h: No such file or directory
```

这个问题也可以通过重新安装`linux-libc-dev`后创建软链接来解决：

```bash
# find / -name sigcontext.h
sudo apt-get install --reinstall linux-libc-dev
ln -s /usr/include/aarch64-linux-gnu/asm/sigcontext.h /usr/include/asm/sigcontext.h
```

接下来，还有一个报错：

```yaml
asm/sve_context.h: No such file or directory
```

这个报错是因为最新的 Linux 内核在启用 ARM Scalable Vector Extension (SVE) 后，需要包含 `asm/sve_context.h` 文件。我们需要创建一个软链接来解决这个问题：

```bash
# find / -name sve_context.h
ln -s /usr/include/aarch64-linux-gnu/asm/sve_context.h /usr/include/asm/sve_context.h
```

最后，还需要创建一个软链接：

```bash
# find / -name byteorder.h
ln -s /usr/include/aarch64-linux-gnu/asm/byteorder.h /usr/include/asm/byteorder.h
```

完成以上步骤后，我们再次执行 `make` 命令，就应该可以顺利地编译和安装 glibc 了。
