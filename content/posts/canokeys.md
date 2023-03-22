---
title: "Canokey入门指南:2FA、OpenPGP、PIV"
description: 本文是一份Canokey入门指南，将介绍如何使用Canokey进行2FA、OpenPGP和PIV等操作。其中，2FA部分将介绍如何使用Yubikey Authenticator进行管理，OpenPGP部分将介绍如何生成GPG密钥并使用Canokey进行身份验证和加密解密，PIV部分将介绍如何在Canokey中生成PIV证书并使用其进行身份验证。
date: 2022-08-19T16:42:40+08:00
draft: false
slug: canokeys
image:
categories:
  - Linux
tags:
  - Linux
---



# 2FA

`Canokey`使用`Yubikey Authenticator`来进行管理`2FA`。

下载`Yubikey Authenticator`,以下为`Yubikey Authenticator`官方下载网址

```http
https://www.yubico.com/products/yubico-authenticator/#h-download-yubico-authenticator
```

运行`Yubikey Authenticator`

进入`custom reader`，在`Custom reader fiter`处填入 `CanoKey`

![填入CanoKey](https://upload-images.jianshu.io/upload_images/9676051-ff0cd60f38ac7334.png)

右上角`Add account` 增加`2FA`

![添加2FA](https://upload-images.jianshu.io/upload_images/9676051-1031857fe0f13d08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```yaml
Issuer: 备注 可选
Account name : 用户名 必填项
Secret Key : Hotp或Totp的key 必填项
```


# OpenPGP

## 安装GPG

Windows 用户可下载 [Gpg4Win](https://gpg4win.org/download.html)，Linux/macOS 用户使用对应包管理软件安装即可.

## 生成主密钥

```shell
gpg --expert --full-gen-key #生成GPG KEY
```

推荐使用`ECC`算法

![image-20220102223722475](https://upload-images.jianshu.io/upload_images/9676051-df42e4b958e9a238.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```shell
选择(11) ECC (set your own capabilities) # 设置自己的功能 主密钥只保留 Certify 功能，其他功能（Encr,Sign,Auth）使用子密钥
# 子密钥分成三份,分别获得三个不同的功能
# encr 解密功能
# sign 签名功能
# auth 登录验证功能
```

```shell
先选择 (S) Toggle the sign capability 
```

![image-20220102224151589](https://upload-images.jianshu.io/upload_images/9676051-c3bb19eb398419e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
之后输入q 退出
```

键入1,选择默认算法

![键入1,选择默认算法](https://upload-images.jianshu.io/upload_images/9676051-7a2c5ee8ed4800af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置主密钥永不过期

![image-20220102224451731](https://upload-images.jianshu.io/upload_images/9676051-cca6100917c2ffaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

填写信息,按照实际情况填写即可

![image-20220102224612167](https://upload-images.jianshu.io/upload_images/9676051-10430afe3aa592c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Windnows 下会弹出窗口输入密码，注意一定要保管好！！！
```

```shell

```

```shell
# 会自动生成吊销证书，注意保存到安全的地方
gpg: AllowSetForegroundWindow(22428) failed: �ܾ����ʡ�
gpg: revocation certificate stored as 'C:\\Users\\Andorid\\AppData\\Roaming\\gnupg\\openpgp-revocs.d\\<此处为私钥>.rev'
# 以上的REV文件即为吊销证书
public and secret key created and signed.
```

```shell
pub   ed25519 2022-01-02 [SC]
      <此处为Pub>
uid                      <此处为Name> <此处为email>
```

生成子密钥

```shell
 gpg --fingerprint --keyid-format long -K
```

下面生成不同功能的子密钥，其中 `<fingerprint>` 为上面输出的密钥指纹，本示例中即为 `私钥`。最后的 `2y` 为密钥过期时间，可自行设置，如不填写默认永不过期。

```shell
gpg --quick-add-key <fingerprint> cv25519 encr 2y
gpg --quick-add-key <fingerprint> ed25519 auth 2y
gpg --quick-add-key <fingerprint> ed25519 sign 2y
```

再次查看目前的私钥，可以看到已经包含了这三个子密钥。

```shell
gpg --fingerprint --keyid-format long -K
```

上面生成了三种功能的子密钥（ssb），分别为加密（E）、认证（A）、签名（S），对应 `OpenPGP Applet` 中的三个插槽。由于 `ECC` 实现的原因，加密密钥的算法区别于其他密钥的算法。

加密密钥用于加密文件和信息。签名密钥主要用于给自己的信息签名，保证这真的是来自**我**的信息。认证密钥主要用于 SSH 登录。

## 备份GPG

```shell
# 公钥
gpg -ao public-key.pub --export <ed25519/16位>
# 主密钥，请务必保存好！！！
# 注意 key id 后面的 !，表示只导出这一个私钥，若没有的话默认导出全部私钥。
gpg -ao sec-key.asc --export-secret-key <ed25519/16位>!
# sign子密钥
gpg -ao sign-key.asc --export-secret-key <ed25519/16位>!
gpg -ao auth-key.asc --export-secret-key <ed25519/16位>!
gpg -ao encr-key.asc --export-secret-key <ed25519/16位>!
```

## 导入Canokey

```shell
# 查看智能卡设备状态
gpg --card-status
# 写入GPG
gpg --edit-key <ed25519/16位> # 为上方的sec-key
# 选中第一个子密钥
key 1
# 写入到智能卡
keytocard
# 再次输入,取消选择
key 1
# 选择第二个子密钥
key 2
keytocard
key 2
key 3
keytocard
# 保存修改并退出
save

#再次查看设备状态,可以看到此时子密钥标识符为 ssb>，表示本地只有一个指向 card-no: F1D0 xxxxxxxx 智能卡的指针，已不存在私钥。现在可以删除掉主密钥了，请再次确认你已安全备份好主密钥。
gpg --card-status
```
## 删除本地密钥

```shell
gpg --delete-secret-keys <ed25519/16位> # 为上方的sec-key
```

为确保安全，也可直接删除 gpg 的工作目录：`%APPDATA%\gnupg`，Linux/macOS: `~/.gunpg`。

## 使用 Canokey

此时切换回日常使用的环境，首先导入公钥

```shell
gpg --import public-key.pub
```

然后设置子密钥指向 Canokey

```shell
gpg --edit-card
gpg/card> fetch
```

此时查看本地的私钥，可以看到已经指向了 Canokey

```
gpg --fingerprint --keyid-format long -K
```

配置gpg路径

```bash
git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe" --replace-all
```

## Git Commit 签名

首先确保 Git 本地配置以及 GitHub 中的邮箱信息包含在 `UID` 中，然后设置 Git 来指定使用子密钥中的签名（S）密钥。

```shell
git config --global user.signingkey <ed25519/16位> # 为上方的Sign密钥
```

之后在 `git commit` 时增加 `-S` 参数即可使用 gpg 进行签名。也可在配置中设置自动 gpg 签名，此处不建议全局开启该选项，因为有的脚本可能会使用 `git am` 之类的涉及到 `commit` 的命令，如果全局开启的话会导致问题。

```shell
git config commit.gpgsign true
```

如果提交到 GitHub，前往 [GitHub SSH and GPG keys](https://github.com/settings/keys) 添加公钥。此处添加后，可以直接通过对应 GitHub ID 来获取公钥：`https://github.com/<yourid>.gpg`

## PIV

首先在Web端添加自己的私钥到智能卡，之后前往 [WinCrypt SSH Agent](https://github.com/buptczq/WinCryptSSHAgent) 下载并运行，此时查看 `ssh-agent` 读取到的公钥信息，把输出的公钥信息添加到服务器的 `~/.ssh/authorized_keys`

```shell
# 设置环境池
$Env:SSH_AUTH_SOCK="\\.\pipe\openssh-ssh-agent"
# 查看ssh列表
ssh-add -L
```

此时连接 `ssh user@host`，会弹出提示输入 `PIN` 的页面，注意此时输入的是 `PIV Applet PIN`，输入后即可成功连接服务器。

```yaml
tips: 可能会出现权限不够的情况,需要禁用Windows服务OpenSSH Authentication Agent
```

最后可以把该程序快捷方式添加到启动目录 `%AppData%\Microsoft\Windows\Start Menu\Programs\Startup`，方便直接使用。
