---
date: 2017-12-07
title: 如何配置服务器的 SSH 登陆
tags:
- 中文
- Linux
- Bash
- SSH
description: 本文涉及的内容比较基础，适用的场景主要包括：搭建自己的博客、搭梯子科学上网、配置机器学习的运行环境以及学习服务器端软件开发。以上这些场景涉及购买自己的服务器，因而适用于本文中描述的SSH登陆的配置。
---
# 如何配置服务器的 SSH 登陆

## 引言

笔者在工作中经常需要做一些基本的服务器配置，这里说的配置大概包括：`SSH登陆的配置`、`LNMP的安装与配置`、`购买SSL证书以及配置HTTPS`，当然还有最常见的 `Web应用的部署`。这些常见的操作对于运维和开发工程师，可以说是信手拈来。而对于那些刚接触服务器、命令行的同学来说，学习曲线还是比较陡峭的，故而也是本文的价值所在。

本文涉及的内容比较基础，适用的场景主要包括： `搭建自己的博客`、`搭梯子科学上网`、 `配置机器学习的运行环境` 以及 `学习服务器端软件开发`。以上这些场景涉及购买自己的服务器，因而适用于本文中描述的 `SSH登陆的配置`。

## 一、两种常见的SSH登陆方式

目前，大部分服务提供商（比如 DigitalOcean、Linode、Vultr）都允许用户自行选择 `密码登陆` 或者 `私钥文件登陆`，而少数服务提供商（比如 AWS）则强制用户使用 `私钥文件登陆`。使用 `私钥文件登陆` 是更加安全的做法。

如果在购买服务器后，收到的邮件中只包含 root 账号和一个随机生成的密码，那么可以确认你的服务器使用了 `密码登陆`。这种情况下，登陆服务器可以使用命令如下（注：笔者使用的是 macOS 系统，Windows 用户可以通过安装 [Git Bash for Windows](https://git-for-windows.github.io/) 模拟类似的环境）：

```bash
$ ssh -o "PubkeyAuthentication=no" root@12.34.56.78
```

其中 12.34.56.78 需要被替换成服务器的 IP 地址，执行这一命令后，输入 root 账号的默认密码即可成功登陆。如果在登陆过程中遇到 (yes/no) 相关的问题，一律输入 yes 即可。

成功登陆后，我们需要做的第一件事是修改 root 账号的登陆密码，并创建一个新的管理员账号：

```bash
#这个命令可用于修改当前用户的登陆密码
$ passwd 

#修改密码后，我们创建一个新的账号 haoxi911
$ sudo adduser haoxi911

#将新建的账号添加为系统管理员
$ usermod -aG admin haoxi911

#完成以上步骤之后，登出当前的 root 账号
$ exit

#再用刚刚新建的管理员账号重新登陆
$ ssh -o "PubkeyAuthentication=no" haoxi911@12.34.56.78

#确认登陆成功后，登出当前账号
$ exit
```

下一步，我们需要在 Windows 或者 macOS 上生成一对公钥、私钥：

```bash
#切换到 SSH 默认的目录
$ cd ~/.ssh

#生成一对名为 haoxi911 的公钥、私钥
$ ssh-keygen -o -a 100 -t ed25519 -f haoxi911

#将公钥的内容复制到剪贴板
$ cat ~/.ssh/haoxi911.pub | pbcopy
```

现在，我们重新登陆服务器，并且配置使用私钥文件登陆服务器：

```bash
#用刚才新建的管理员账号重新登陆服务器
$ ssh -o "PubkeyAuthentication=no" haoxi911@12.34.56.78

#新建 SSH 的默认文件夹
$ mkdir ~/.ssh

#修改 authorized_keys 文件，添加刚才复制的公钥
$ vim ~/.ssh/authorized_keys

# 1、按 i 键进入编辑模式
# 2、粘贴内容
# 3、按 esc 键退出编辑模式
# 4、输入 :wq 命令退出vim

#退出登陆，并重新使用私钥文件的方式登陆
$ exit
$ ssh -i ~/.ssh/haoxi911 -o "IdentitiesOnly=yes" haoxi911@12.34.56.78
```

如果登陆成功，则可以确认私钥文件的登陆已经配置成功。下一步，我们将禁止 root 用户通过 SSH 登陆，以及禁止用户通过密码登陆，这些都是提高服务器安全性的常见做法。

```bash
#登陆服务器，并且编辑 SSH 的配置文件
$ sudo vim /etc/ssh/sshd_config

#在配置文件中找到如下两行，将其改为 no
# 1、按 i 键进入编辑模式
# 2、按方向键定位到要修改的位置
# 3、将 PermitRootLogin 后面的 yes 改为 no
# 4、将 PasswordAuthentication 后面的 yes 改为 no
# 5、如果这两行默认被注释，则去掉注释符 #
# 6、按 esc 键退出编辑模式
# 7、输入 :wq 命令退出vim

#重新加载SSH服务
$ sudo service ssh restart
```

至此，我们已经成功的将 `密码登陆` 的模式修改成了更为安全的 `私钥文件登陆` 的模式。值得注意的是，如果在购买服务器时，选择并配置了 `私钥文件登陆`，服务提供商会将 `私钥文件` 通过邮件或者下载文件等方式提供给我们。这时，我们可以直接使用该私钥登陆服务器，不需要再做上述的配置：

```bash
#服务器提供商会默认创建一个管理员账号，比如 AWS 可能会使用 ubuntu，请留意
$ ssh -i ~/path/to/private_key -o "IdentitiesOnly=yes" ubuntu@12.34.56.78
```

## 二、修改本地的配置文件，简化SSH登陆的命令

上述的配置方法虽然可以正常工作，但是每次登陆都需要输入如此长的命令并不是很方便。其实，我们可以通过配置本地的 config 文件简化这一流程：

```bash
#打开本地 SSH 配置文件所在的文件夹
$ open ~/.ssh

#找到 config 文件并用自己熟悉的编辑器打开
```

配置文件的参考格式如下：

```
Host *
    UseKeychain yes

Host haoxi911
    HostName haoxi911.me
    User haoxi911
    IdentityFile /Users/Kevin/.ssh/haoxi911
    IdentitiesOnly yes
    AddKeysToAgent yes
```

将 config 文件修改并保存后，我们就可以通过如下的快捷方式登陆服务器了：

```bash
$ ssh haoxi911
```

::: tip 提示
如果有多个服务器，可以按照类似的格式添加多个 Host。值得注意的是，每个 Host 代表一个唯一的服务器，不可以重名。
:::

## 三、从本地向服务器拷贝文件

我们在第二步所做的配置，也可以被用于其他基于 SSH 的工具中，比如最常见的 SCP 工具：

```bash
#复制本地文件到服务器
$ scp ~/Images/banner.png haoxi911:/var/www/html/public/imgs

#复制本地文件夹到服务器
$ scp -r ~/Images haoxi911:/var/www/html/public/imgs
```
<ClientOnly>
  <Comment />
</ClientOnly>