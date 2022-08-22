---
title: "在 Ubuntu20.04 建设 openVPN"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "连接各个内网机器间的网络"    # 文章描述信息
date: 2020-08-06            # 文章编写日期
lastmod: 2020-08-06         # 文章修改日期

tags : [                    # 文章所属标签
    "网络",
    "openVPN"
]
categories : [              # 文章所属标签
    "网络",
    "openVPN",
]
keywords : [                # 文章关键词
    "openVPN",
    "Ubuntu",
    "服务器",
]

next: /tutorials/github-pages-blog      # 下一篇博客地址
prev: /tutorials/automated-deployments  # 上一篇博客地址
---
# openvpn

操作系统: Ubuntu 20.04 

参考文档：

[openVPN官方文档](https://openvpn.net/community-resources/)

[DigitalOcean官方文档](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-20-04)

## 初始服务器设置

### OpenVPN 服务器

#### 创建新用户

```
sudo adduser sammy

# 授予 sudo 权限
sudo usermod -aG sudo sammy

# 切换用户
su sammy
```

##### 创建密钥对

```
ssh-keygen

# 将公钥复制到本机
touch ~/.ssh/authorized_keys && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### CA 服务器

#### 创建新用户

```
sudo adduser sammy

# 授予 sudo 权限
sudo usermod -aG sudo sammy

# 切换用户
su sammy
```

##### 创建密钥对

```
ssh-keygen

# 将公钥复制到本机
touch ~/.ssh/authorized_keys && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 将公钥复制到 OpenVPN 服务器
cat ~/.ssh/id_rsa.pub
# 将上面命令执行结果复制到 OpenVPN 服务器的 /home/sammy/.ssh/authorized_keys 文件中

# 将 OpenVPN 服务器的公钥复制到本机，在 OpenVPN 服务器执行下面命令:
cat ~/.ssh/id_rsa.pub
# 将上面命令执行结果复制到 CA 服务器的 /home/sammy/.ssh/authorized_keys 文件中
```

#### 安装 Easy-RSA

> 版本：3.0.6。

```
sudo apt update
sudo apt install easy-rsa

# 查看版本：
dpkg -l|grep easy-rsa
ii  easy-rsa                              3.0.6-1                               all          Simple shell based CA utility
```

#### 创建公开密码匙基础建设目录

```
mkdir ~/easy-rsa

# 使用此目录创建指向前一步中安装的 easy-rsa 包文件的符号链接。
ln -s /usr/share/easy-rsa/* ~/easy-rsa/

# 确保只有所有者可以使用 chmod 命令访问它:
chmod 700 /home/sammy/easy-rsa
```

在 easy-rsa 目录中初始化 PKI:

```
cd ~/easy-rsa
./easyrsa init-pki
```

#### 创建证书颁发机构

创建并用一些默认值填充一个名为 vars 的文件。

```
vi vars
cat vars

set_var EASYRSA_REQ_COUNTRY    "CN"
set_var EASYRSA_REQ_PROVINCE   "GuangDong"
set_var EASYRSA_REQ_CITY       "ShenZhen"
set_var EASYRSA_REQ_ORG        "KLB"
set_var EASYRSA_REQ_EMAIL      "chenjinxin@chenjinxin.cn"
set_var EASYRSA_REQ_OU         "Community"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
```

为证书颁发机构创建根公钥和私钥对(不想每次与 CA 交互时都被提示输入密码，可以使用 nopass 选项运行 build-CA 命令)

```
# 按 ENTER 接受默认配置
./easyrsa build-ca nopass
```

## 安装 OpenVPN 和 Easy-RSA

### OpenVPN 服务器

更新 OpenVPN 服务器的软件包索引并安装 OpenVPN 和 Easy-RSA。

```
sudo apt update
sudo apt install openvpn easy-rsa

# 查看版本
dpkg -l|grep easy-rsa
ii  easy-rsa                              3.0.6-1                               all          Simple shell based CA utility
openvpn --version
OpenVPN 2.5.5 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Mar 22 2022
library versions: OpenSSL 3.0.2 15 Mar 2022, LZO 2.10
......
```

在 OpenVPN 服务器上创建一个新目录 ~/easy-rsa, 从 easyrsa 包安装到 ~/easy-rsa 目录中的脚本创建一个符号链接:

```
mkdir ~/easy-rsa
ln -s /usr/share/easy-rsa/* ~/easy-rsa/

# 确保目录的所有者是您的非 root sudo 用户，并使用chmod以下命令限制对该用户的访问：
sudo chown sammy ~/easy-rsa
chmod 700 ~/easy-rsa
```

## 为 OpenVPN 创建 PKI

### OpenVPN 服务器

在创建 OpenVPN 服务器的私钥和证书之前，您需要在 OpenVPN 服务器上创建一个本地公钥基础设施目录。您将使用此目录来管理服务器和客户端的证书请求，而不是直接在您的 CA 服务器上进行。

要在您的 OpenVPN 服务器上构建 PKI 目录，您需要填充一个 vars 使用一些默认值调用的文件。首先您将 cd 进入该 easy-rsa 目录，然后您将使用 vi 或您喜欢的文本编辑器创建和编辑文件。

```
cd ~/easy-rsa
vi vars
```

打开文件后，粘贴以下两行：

```
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

填充 vars 文件后，您可以继续创建 PKI 目录。为此，请 easyrsa 使用该 init-pki 选项运行脚本。

```
./easyrsa init-pki
```

## 创建 OpenVPN 服务器证书请求和私钥

### OpenVPN 服务器

生成私钥和证书签名请求

```
cd ~/easy-rsa
```

现在，您将 easyrsa 使用该 gen-req 选项后跟机器的通用名称 (CN) 来调用。CN 可以是您喜欢的任何内容，但使其具有描述性会很有帮助。在本教程中，OpenVPN 服务器的 CN 将是server. 一定要包括该nopass选项。如果不这样做，将对请求文件进行密码保护，这可能会导致以后出现权限问题。

注意：如果您选择server此处以外的名称，则必须调整下面的一些说明。例如，将生成的文件复制到/etc/openvpn目录时，您必须替换正确的名称。您还必须/etc/openvpn/server.conf稍后修改该文件以指向正确的.crt和.key文件。

```
./easyrsa gen-req server nopass
```

```
Output
Common Name (eg: your user, host, or server name) [server]:

Keypair and certificate request completed. Your files are:
req: /home/sammy/easy-rsa/pki/reqs/server.req
key: /home/sammy/easy-rsa/pki/private/server.key
```

这将为服务器创建一个私钥和一个名为server.req. 将服务器密钥复制到/etc/openvpn/server目录：

```
sudo cp /home/sammy/easy-rsa/pki/private/server.key /etc/openvpn/server/
```

## 签署 OpenVPN 服务器的证书请求

### CA 服务器

在 OpenVPN 服务器上，作为您的非 root 用户，使用 SCP 或其他传输方法将 server.req 证书请求复制到 CA 服务器进行签名：

```
# scp sammy@your_openvpn_server_ip:/home/sammy/easy-rsa/pki/reqs/server.req /tmp
scp sammy@123.125.32.26:/home/sammy/easy-rsa/pki/reqs/server.req /tmp
```

进入 ~/easy-rsa 创建 PK 的目录，然后使用easyrsa脚本导入证书请求：

```
cd ~/easy-rsa
./easyrsa import-req /tmp/server.req server
```

```
Output
...
The request has been successfully imported with a short name of: server
You may now use this name to perform signing operations on this request.
```

接下来，通过运行easyrsa带有sign-req选项的脚本来签署请求，后跟请求类型和通用名称。请求类型可以是client或server。由于我们正在处理 OpenVPN 服务器的证书请求，请务必使用server请求类型：

```
./easyrsa sign-req server server
```

在输出中，系统会提示您验证请求是否来自受信任的来源。输入yes然后按ENTER确认：

```
Output
Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 1080 days:

subject=
    commonName                = server


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /home/sammy/easy-rsa/pki/safessl-easyrsa.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Jul  6 05:46:50 2025 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/sammy/easy-rsa/pki/issued/server.crt
```

完成这些步骤后，您已经使用 CA 服务器的私钥签署了 OpenVPN 服务器的证书请求。生成的server.crt文件包含 OpenVPN 服务器的公共加密密钥，以及来自 CA 服务器的签名。签名的目的是告诉任何信任 CA 服务器的人，当他们连接到 OpenVPN 服务器时，他们也可以信任它。

要完成配置证书，请将server.crt和ca.crt文件从 CA 服务器复制到 OpenVPN 服务器：

```
# scp pki/issued/server.crt sammy@your_vpn_server_ip:/tmp
# scp pki/ca.crt sammy@your_vpn_server_ip:/tmp
scp pki/issued/server.crt sammy@123.125.32.26:/tmp
scp pki/ca.crt sammy@123.125.32.26:/tmp
```

### OpenVPN 服务器

现在回到您的 OpenVPN 服务器，将文件复制 /tmp 到 /etc/openvpn/server：

```
sudo cp /tmp/{server.crt,ca.crt} /etc/openvpn/server
```

## 配置 OpenVPN 加密材料

### OpenVPN 服务器

为了增加一层安全性，我们将添加一个额外的共享密钥，服务器和所有客户端将使用OpenVPN 的tls-crypt指令。此选项用于混淆服务器和客户端最初相互连接时使用的 TLS 证书。OpenVPN 服务器也使用它来对传入的数据包执行快速检查：如果使用预共享密钥对数据包进行签名，则服务器会对其进行处理；如果它没有签名，那么服务器知道它来自不受信任的来源并且可以丢弃它而不必执行额外的解密工作。

此选项将有助于确保您的 OpenVPN 服务器能够应对未经身份验证的流量、端口扫描和拒绝服务攻击，这些攻击会占用服务器资源。这也使得识别 OpenVPN 网络流量变得更加困难。

要生成tls-crypt预共享密钥，请在 OpenVPN 服务器上的~/easy-rsa目录中运行以下命令：

```
cd ~/easy-rsa
openvpn --genkey secret ta.key
```

结果将是一个名为ta.key. 复制到/etc/openvpn/server/目录：

```
sudo cp ta.key /etc/openvpn/server
```

将这些文件放在 OpenVPN 服务器上后，您就可以为您的用户创建客户端证书和密钥文件，您将使用它们连接到 VPN。

## 生成客户端证书和密钥对

### OpenVPN 服务器

尽管您可以在客户端计算机上生成私钥和证书请求，然后将其发送到 CA 进行签名，但本指南概述了在 OpenVPN 服务器上生成证书请求的过程。这种方法的好处是我们可以创建一个脚本，该脚本将自动生成包含所有必需密钥和证书的客户端配置文件。这让您不必将密钥、证书和配置文件传输到客户端，并简化加入 VPN 的过程。

我们将为本指南生成单个客户端密钥和证书对。如果您有多个客户，您可以对每个客户重复此过程。但请注意，您需要为每个客户端的脚本传递一个唯一的名称值。在本教程中，第一个证书/密钥对称为 client1.

首先在您的主目录中创建一个目录结构来存储客户端证书和密钥文件：

```
mkdir -p ~/client-configs/keys
```

由于您将客户的证书/密钥对和配置文件存储在此目录中，因此您现在应该锁定其权限作为安全措施：

```
chmod -R 700 ~/client-configs
```

接下来，导航回 EasyRSA 目录并 easyrsa 使用gen-req 和nopass 选项以及客户端的通用名称运行脚本：

```
cd ~/easy-rsa
./easyrsa gen-req client1 nopass
```

按 ENTER 确认常用名称。然后，将 client1.key 文件复制到 ~/client-configs/keys/ 您之前创建的目录中：

```
cp pki/private/client1.key ~/client-configs/keys/
```

### CA 服务器

现在登录到您的 CA 服务器。

接下来，将 client1.req 使用安全方法将文件传输到您的 CA 服务器：

```
# scp sammy@your_openvpn_server_ip:/home/sammy/easy-rsa/pki/reqs/client1.req /tmp
scp sammy@123.125.32.26:/home/sammy/easy-rsa/pki/reqs/client1.req /tmp
```

然后，导航到 EasyRSA 目录，并导入证书请求：

```
cd ~/easy-rsa
./easyrsa import-req /tmp/client1.req client1
```

接下来，以与您在上一步中为服务器所做的相同的方式签署请求。不过这一次，请务必指定client请求类型：

```
./easyrsa sign-req client client1
```

出现提示时，输入yes以确认您打算签署证书请求并且它来自受信任的来源：

```
Output
Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 1080 days:

subject=
    commonName                = client1


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /home/sammy/easy-rsa/pki/safessl-easyrsa.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client1'
Certificate is to be certified until Jul  6 06:18:09 2025 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/sammy/easy-rsa/pki/issued/client1.crt
```

这将创建一个名为client1.crt. 将此文件传输回服务器：

```
# scp pki/issued/client1.crt sammy@your_server_ip:/tmp
scp pki/issued/client1.crt sammy@123.125.32.26:/tmp
```

### OpenVPN 服务器

回到您的 OpenVPN 服务器，将客户端证书复制到~/client-configs/keys/目录：

```
cp /tmp/client1.crt ~/client-configs/keys/
```

接下来，将 ca.crt 和 ta.key 文件也复制到 ~/client-configs/keys/ 目录中，并为您的 sudo 用户设置适当的权限：

```
cp ~/easy-rsa/ta.key ~/client-configs/keys/
sudo cp /etc/openvpn/server/ca.crt ~/client-configs/keys/
sudo chown sammy.sammy ~/client-configs/keys/*
```

这样，您的服务器和客户端的证书和密钥都已生成并存储在 OpenVPN 服务器上的相应目录中。仍然需要对这些文件执行一些操作，但这些将在稍后的步骤中进行。现在，您可以继续配置 OpenVPN。

## 配置 OpenVPN

### OpenVPN 服务器

像许多其他广泛使用的开源工具一样，OpenVPN 有许多配置选项可用于根据您的特定需求自定义您的服务器。在本节中，我们将提供有关如何根据此软件文档中包含的示例配置文件之一设置 OpenVPN 服务器配置的说明。

首先，复制示例 server.conf 文件作为您自己的配置文件的起点：

```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
```

使用您选择的文本编辑器打开新文件进行编辑。我们将在示例中使用 vi：

```
sudo vi /etc/openvpn/server/server.conf
```

我们需要更改此文件中的几行。首先，HMAC通过搜索 tls-auth 指令找到配置的部分。默认情况下将启用此行。通过;在行的开头添加 a来注释掉它。然后在它只包含值之后添加一个新行 tls-crypt ta.key：

/etc/openvpn/server/server.conf

```
;tls-auth ta.key 0 # This file is secret
tls-crypt ta.key
```

接下来，通过查找cipher行找到有关加密密码的部分。默认值设置为AES-256-CBC，但是，AES-256-GCM密码提供更好的加密级别和性能，并且在最新的 OpenVPN 客户端中得到很好的支持。我们将通过;在此行的开头添加一个符号来注释掉默认值，然后我们将在它包含更新后的值之后添加另一行AES-256-GCM：

/etc/openvpn/server/server.conf

```
;cipher AES-256-CBC
cipher AES-256-GCM
```

在此行之后，添加一个auth指令来选择 HMAC 消息摘要算法。为此，SHA256是一个不错的选择：

/etc/openvpn/server/server.conf

```
auth SHA256
```

接下来，找到包含dh指令的行，该指令定义了 Diffie-Hellman 参数。由于我们已将所有证书配置为使用椭圆曲线密码术，因此不需要 Diffie-Hellman 种子文件。注释掉看起来像dh dh2048.pem或的现有行dh dh.pem。Diffie-Hellman 密钥的文件名可能与示例服务器配置文件中列出的不同。然后在它后面添加一行内容dh none：

/etc/openvpn/server/server.conf

```
;dh dh2048.pem
dh none
```

接下来，我们希望 OpenVPN 一旦启动就在没有特权的情况下运行，因此我们需要告诉它以用户nobody和组nogroup 运行。要启用此功能，请通过删除每行开头的符号来查找和取消注释user nobody和行：group nogroup;

/etc/openvpn/server/server.conf

```
user nobody
group nogroup
```

## 调整 OpenVPN 服务器网络配置

### OpenVPN 服务器

服务器网络配置的某些方面需要进行调整，以便 OpenVPN 可以通过 VPN 正确路由流量。其中第一个是IP 转发，这是一种确定 IP 流量应路由到何处的方法。这对于您的服务器将提供的 VPN 功能至关重要。

要调整 OpenVPN 服务器的默认 IP 转发设置，请 /etc/sysctl.conf 使用 vi 或首选编辑器打开文件：

```
sudo vi /etc/sysctl.conf
```

然后在文件底部添加以下行：

/etc/sysctl.conf

```
net.ipv4.ip_forward = 1
```

完成后保存并关闭文件。

要读取文件并加载当前会话的新值，请键入：

```
sudo sysctl -p
```

```
Output
net.ipv4.ip_forward = 1
```

现在，您的 OpenVPN 服务器将能够将传入流量从一个以太网设备转发到另一个。此设置确保服务器可以将连接到虚拟 VPN 接口的客户端的流量通过其其他物理以太网设备引导出去。此配置将通过服务器的 IP 地址路由来自客户端的所有网络流量，并且将有效地隐藏客户端的公共 IP 地址。

在下一步中，您需要配置一些防火墙规则，以确保进出 OpenVPN 服务器的流量正常流动。

## 防火墙配置(未开启防火墙可以跳过)

### OpenVPN 服务器

到目前为止，您已经在服务器上安装了 OpenVPN，对其进行了配置，并生成了客户端访问 VPN 所需的密钥和证书。但是，您尚未向 OpenVPN 提供有关从客户端发送传入 Web 流量的位置的任何说明。您可以通过建立一些防火墙规则和路由配置来规定服务器应如何处理客户端流量。

假设您遵循了本教程开始时的先决条件，您应该已经ufw在服务器上安装并运行。要允许 OpenVPN 通过防火墙，您需要启用伪装，这是一种 iptables 概念，可提供即时动态网络地址转换 (NAT) 以正确路由客户端连接。

在打开防火墙配置文件添加伪装规则之前，首先要找到自己机器的公网接口。为此，请键入：

```
ip route list default
```

您的公共接口是在此命令的输出中找到的跟在单词“dev”之后的字符串。例如，此结果显示名为 的接口eth0，它在下面突出显示：

```
Output
default via 159.65.160.1 dev eth0 proto static
```

当您拥有与默认路由关联的接口时，打开 /etc/ufw/before.rules 文件以添加相关配置：

```
sudo nano /etc/ufw/before.rules
```

UFW 规则通常使用该ufw命令添加。before.rules但是，在加载传统 UFW 规则之前，会读取文件中列出的规则并将其放置到位。在文件顶部，添加下面突出显示的行。这将为表中的POSTROUTING链设置默认策略nat并伪装来自 VPN 的任何流量。请记住eth0将以-A POSTROUTING下行中的内容替换为您在上述命令中找到的界面：

/etc/ufw/before.rules

```
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter
. . .
```

完成后保存并关闭文件。

接下来，您需要告诉 UFW 默认情况下也允许转发数据包。为此，请打开 /etc/default/ufw 文件：

```
sudo nano /etc/default/ufw
```

在里面，找到DEFAULT_FORWARD_POLICY指令并将值从 更改 DROP 为 ACCEPT ：

/etc/default/ufw

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

完成后保存并关闭文件。

接下来，调整防火墙本身以允许 OpenVPN 的流量。如果您没有更改文件中的端口和协议/etc/openvpn/server.conf，则需要打开 UDP 流量到 port 1194。如果您修改了端口和/或协议，请替换您在此处选择的值。

如果您在遵循先决条件教程时忘记添加 SSH 端口，请在此处添加：

```
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
```

注意：如果您使用不同的防火墙或自定义了 UFW 配置，则可能需要添加其他防火墙规则。例如，如果你决定隧道所有的网络流量通过VPN连接，你需要确保端口53流量允许DNS请求，而像港口80和443分别为HTTP和HTTPS流量。如果您在 VPN 上使用了其他协议，那么您还需要为它们添加规则。

添加这些规则后，禁用并重新启用 UFW 以重新启动它并从您修改的所有文件中加载更改：

```
sudo ufw disable
sudo ufw enable
```

您的服务器现已配置为正确处理 OpenVPN 流量。设置好防火墙规则后，我们可以在服务器上启动 OpenVPN 服务。

## 启动 OpenVPN

### OpenVPN 服务器

OpenVPN 作为 systemd 服务运行，因此我们可以使用 systemctl 它来管理它。我们会将 OpenVPN 配置为在启动时启动，因此只要您的服务器正在运行，您就可以随时连接到您的 VPN。为此，请将 OpenVPN 服务添加到 systemctl：

```
sudo systemctl -f enable openvpn-server@server.service
```

然后启动OpenVPN服务：

```
sudo systemctl start openvpn-server@server.service
```

使用以下命令仔细检查 OpenVPN 服务是否处于活动状态。您应该active (running)在输出中看到：

```
sudo systemctl status openvpn-server@server.service
```

```
Output
● openvpn-server@server.service - OpenVPN service for server
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2020-04-29 15:39:59 UTC; 6s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 16872 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 1137)
     Memory: 1.0M
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
             └─16872 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-version 2 --suppress-timestamps --c>
. . .
. . .
Apr 29 15:39:59 ubuntu-20 openvpn[16872]: Initialization Sequence Completed
```

我们现在已经完成了 OpenVPN 的服务器端配置。接下来，您将配置您的客户端机器并连接到 OpenVPN 服务器。

## 创建客户端配置基础架构(无特殊需求可以跳过)

### OpenVPN 服务器

为 OpenVPN 客户端创建配置文件可能有些复杂，因为每个客户端都必须有自己的配置，并且每个客户端都必须与服务器配置文件中列出的设置保持一致。这一步不是编写只能在一个客户端上使用的单个配置文件，而是概述了构建客户端配置基础结构的过程，您可以使用它来即时生成配置文件。您将首先创建一个“基本”配置文件，然后构建一个脚本，该脚本将允许您根据需要生成唯一的客户端配置文件、证书和密钥。

首先创建一个新目录，您将`client-configs`在之前创建的目录中存储客户端配置文件：

```
mkdir -p ~/client-configs/files
```

接下来，将示例客户端配置文件复制到`client-configs`目录中以用作基本配置：

```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
```

使用`nano`或您喜欢的文本编辑器打开这个新文件：

```bash
nano ~/client-configs/base.conf
```

在里面，找到`remote`指令。这将客户端指向您的 OpenVPN 服务器地址——您的 OpenVPN 服务器的公共 IP 地址。如果您决定更改 OpenVPN 服务器正在侦听的端口，您还需要更改`1194`为您选择的端口：

~/client-configs/base.conf

```
. . .
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote your_server_ip 1194
. . .
```

确保协议与您在服务器配置中使用的值匹配：

~/client-configs/base.conf

```
proto udp
```

接下来，通过删除每行开头的符号来取消对`user`and`group`指令的注释`;`：

~/client-configs/base.conf

```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```

查找设置的指示`ca`，`cert`和`key`。注释掉这些指令，因为您很快就会在文件本身中添加证书和密钥：

~/client-configs/base.conf

```
# SSL/TLS parms.
# See the server config file for more
# description. It's best to use
# a separate .crt/.key file pair
# for each client. A single ca
# file can be used for all clients.
;ca ca.crt
;cert client.crt
;key client.key
```

同样，注释掉该`tls-auth`指令，因为您将`ta.key`直接添加到客户端配置文件中（并且服务器设置为使用`tls-crypt`）：

~/client-configs/base.conf

```
# If a tls-auth key is used on the server
# then every client must also have the key.
;tls-auth ta.key 1
```

镜像您在文件中设置的`cipher`和`auth`设置`/etc/openvpn/server/server.conf`：

~/client-configs/base.conf

```
cipher AES-256-GCM
auth SHA256
```

接下来，`key-direction`在文件中的某处添加指令。您**必须**将其设置为“1”才能使 VPN 在客户端计算机上正常运行：

~/client-configs/base.conf

```
key-direction 1
```

最后，添加一些**注释掉的**行来处理基于 Linux 的 VPN 客户端将用于 DNS 解析的各种方法。您将添加两个相似但单独的注释行集。第一组用于*不*用于`systemd-resolved`管理 DNS 的客户端。这些客户端依靠该`resolvconf`实用程序来更新 Linux 客户端的 DNS 信息。

~/client-configs/base.conf

```
; script-security 2
; up /etc/openvpn/update-resolv-conf
; down /etc/openvpn/update-resolv-conf
```

现在为`systemd-resolved`用于 DNS 解析的客户端添加另一组行：

~/client-configs/base.conf

```
; script-security 2
; up /etc/openvpn/update-systemd-resolved
; down /etc/openvpn/update-systemd-resolved
; down-pre
; dhcp-option DOMAIN-ROUTE .
```

完成后保存并关闭文件。

接下来，我们将创建一个脚本，该脚本将使用相关的证书、密钥和加密文件编译您的基本配置，然后将生成的配置放在`~/client-configs/files`目录中。打开`make_config.sh`在`~/client-configs`目录中调用的新文件：

```bash
nano ~/client-configs/make_config.sh
```

在里面，添加以下内容：

~/client-configs/make_config.sh

```
#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/client-configs/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/.key \
    <(echo -e '</key>\n<tls-crypt>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-crypt>') \
    > ${OUTPUT_DIR}/.ovpn
```

完成后保存并关闭文件。

在继续之前，请确保通过键入以下内容将此文件标记为可执行文件：

```
chmod 700 ~/client-configs/make_config.sh
```

此脚本将复制`base.conf`您创建的文件，收集您为客户端创建的所有证书和密钥文件，提取它们的内容，将它们附加到基本配置文件的副本，并将所有这些内容导出到新的客户端配置文件。这意味着，不必单独管理客户端的配置、证书和密钥文件，所有需要的信息都存储在一个地方。使用此方法的好处是，如果您将来需要添加客户端，您可以运行此脚本来快速创建一个新的配置文件，并确保所有重要信息都存储在一个易于访问的单个地点。

请注意，每次添加新客户端时，您都需要为其生成新的密钥和证书，然后才能运行此脚本并生成其配置文件。您将在下一步中练习使用此脚本。

## 生成客户端配置

生成配置文件：

```
cd ~/client-configs
./make_config.sh client1
```

这将`client1.ovpn`在您的`~/client-configs/files`目录中创建一个名为的文件：

```bash
ls ~/client-configs/files
```

```
Output
client1.ovpn
```

您需要将此文件传输到您计划用作客户端的设备。例如，这可能是您的本地计算机或移动设备。

虽然用于完成此传输的确切应用程序取决于您设备的操作系统和您的个人偏好，但可靠且安全的方法是在后端使用 SFTP（SSH 文件传输协议）或 SCP（安全复制）。这将通过加密连接传输您客户端的 VPN 身份验证文件。

这是一个示例 SFTP 命令，您可以从本地计算机（macOS 或 Linux）运行该命令。这会将`client1.ovpn`我们在上一步中创建的文件复制到您的主目录：

```
# sftp sammy@openvpn_server_ip:client-configs/files/client1.ovpn ~/
sftp sammy@123.125.32.26:client-configs/files/client1.ovpn ~/
```

## 安装客户端配置

**离线安装**

```
# ubuntu20.04 系统
sudo dpkg -i pkg/ubuntu20.04/openvpn_2.4.7-1ubuntu2.20.04.4_amd64.deb
```

```
# redhat8.6 系统
sudo rpm -i pkg/redhat8.6/pkg/*
```

**在线安装**

如果您使用的是 Linux，则可以根据您的发行版使用多种工具。您的桌面环境或窗口管理器可能还包括连接实用程序。

然而，最通用的连接方式是使用 OpenVPN 软件。

在 Ubuntu 或 Debian 上，您可以像在服务器上一样通过键入以下内容来安装它：

```
sudo apt update
sudo apt install openvpn
```

在 CentOS 上，您可以启用 EPEL 存储库，然后键入以下内容进行安装：

```bash
sudo dnf install epel-release
sudo dnf install openvpn
```

#### 配置使用的客户端 `systemd-resolved`

首先`systemd-resolved`通过检查`/etc/resolv.conf`文件来确定您的系统是否用于处理 DNS 解析：

```
cat /etc/resolv.conf
```

```
Output
# This file is managed by man:systemd-resolved(8). Do not edit.
. . .

nameserver 127.0.0.53
options edns0
```

如果您的系统配置为`systemd-resolved`用于 DNS 解析，则`nameserver`选项后面的 IP 地址将为`127.0.0.53`. 文件中还应该有注释，如显示的输出，解释如何`systemd-resolved`管理文件。如果您有一个不同的 IP 地址，`127.0.0.53`那么您的系统可能没有使用`systemd-resolved`，您可以转至下一节配置具有`update-resolv-conf`脚本的Linux 客户端。

要支持这些客户端，请首先安装该`openvpn-systemd-resolved`软件包。它提供了强制`systemd-resolved`使用 VPN 服务器进行 DNS 解析的脚本。

```
sudo apt install openvpn-systemd-resolved
```

安装该软件包之一，配置客户端以使用它，并通过 VPN 接口发送所有 DNS 查询。打开客户端的 VPN 文件：

```bash
nano client1.ovpn
```

现在取消注释您之前添加的以下几行：

client1.ovpn

```
script-security 2
up /etc/openvpn/update-systemd-resolved
down /etc/openvpn/update-systemd-resolved
down-pre
dhcp-option DOMAIN-ROUTE .
```

#### 配置使用的客户端 `update-resolv-conf`

如果您的系统不`systemd-resolved`用于管理 DNS，请检查您的发行版是否包含`/etc/openvpn/update-resolv-conf`脚本：

```
ls /etc/openvpn
```

```
Output
update-resolv-conf
```

如果您的客户端包含该`update-resolv-conf`文件，请编辑您之前传输的 OpenVPN 客户端配置文件：

```bash
nano client1.ovpn
```

取消注释您添加的用于调整 DNS 设置的三行：

client1.ovpn

```
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

如果您使用 CentOS，请将`group`指令从`nogroup`to更改`nobody`为匹配发行版的可用组：

client1.ovpn

```
group nobody
```

保存并关闭文件。

**连接**

现在，您只需将`openvpn`命令指向客户端配置文件即可连接到 VPN ：

```
# 启动前注意检查 client1.ovpn 的 62 行。在 Ubuntu 系统上是 group nogroup，在 CentOS 系统上是 group nobody
sudo openvpn --config client1.ovpn
```

这应该将您连接到您的 VPN。

**注意：**如果您的客户端用于`systemd-resolved`管理 DNS，请通过运行如下`systemd-resolve --status`命令检查设置是否正确应用：

```
systemd-resolve --status tun0
```

```
Output
Link 22 (tun0)
. . .
         DNS Servers: 208.67.222.222
                      208.67.220.220
          DNS Domain: ~.
```

如果您看到您在 OpenVPN 服务器上配置的 DNS 服务器的 IP 地址，以及输出中*DNS 域*的`~.`设置，那么您已正确配置您的客户端以使用 VPN 服务器的 DNS 解析器。

## 撤销客户端证书

有时，您可能需要撤销客户端证书以防止进一步访问 OpenVPN 服务器。

### CA 服务器

```
cd ~/easy-rsa
./easyrsa revoke client1
```

这将要求您通过输入 yes 来确认撤销:

```
Output
Please confirm you wish to revoke the certificate with the following subject:

subject=
    commonName                = client-ccx


Type the word 'yes' to continue, or any other input to abort.
  Continue with revocation: yes
Using configuration from /home/sammy/easy-rsa/pki/safessl-easyrsa.cnf
Revoking Certificate D3060496FB55BF756C970DF564AA4C17.
Data Base Updated

IMPORTANT!!!

Revocation was successful. You must run gen-crl and upload a CRL to your
infrastructure in order to prevent the revoked cert from being accepted.
```

注意撤销证书行上突出显示的值。此值是要撤销的证书的唯一序列号。如果要检查本节最后一步中的撤销列表以验证证书是否在其中，则需要此值。

在确认操作之后，CA 将撤销证书。但是，依赖 CA 的远程系统无法检查是否有任何证书被撤销。用户和服务器仍然可以使用该证书，直至该证书的证书吊销列表(CRL)分发给所有依赖该证书的系统为止。

**生成一个证书吊销列表**

在 ~/easy-rsa 目录中使用 gen-CRL 选项运行 easy-rsa 命令:

```
cd ~/easy-rsa
./easyrsa gen-crl
```

发送证书吊销列表

```
# scp ~/easy-rsa/pki/crl.pem sammy@your_server_ip:/tmp
scp ~/easy-rsa/pki/crl.pem sammy@123.125.32.26:/tmp
```

### OpenVPN 服务器

使用这些说明撤销客户端证书后，您需要将生成的 crl.pem 文件复制到 /etc/openvpn/server 目录中的 OpenVPN 服务器：

```
sudo cp /tmp/crl.pem /etc/openvpn/server/
```

接下来，打开 OpenVPN 服务器配置文件:

```
sudo nano /etc/openvpn/server/server.conf
```

在文件底部，添加 crl-verify 选项，该选项将指示 OpenVPN 服务器在每次尝试连接时检查您创建的证书吊销列表：

/etc/openvpn/server/server.conf

```
crl-verify crl.pem
```

保存并关闭文件。
最后，重新启动 OpenVPN 以实现证书撤销:

```
sudo systemctl restart openvpn-server@server.service
```

客户端应该不再能够使用旧凭据成功连接到服务器。
要撤销其他客户端，请按照以下流程操作：

1. 使用 ./easyrsa revoke client_name 命令撤销证书
2. 生成新的 CRL
3. 将新的 crl.pem 文件传输到您的 OpenVPN 服务器并将其复制到 /etc/openvpn/server/ 目录以覆盖旧列表。
4. 重新启动 OpenVPN 服务。

您可以使用此过程撤销您之前为服务器颁发的任何证书。

## 配置更多客户端

**修改 client1 为你要的客户端名称**

### OpenVPN 服务器

到 EasyRSA 目录并 easyrsa 使用gen-req 和 nopass 选项以及客户端的通用名称运行脚本：

```
cd ~/easy-rsa
./easyrsa gen-req client1 nopass
```

按 ENTER 确认常用名称。然后，将 client1.key 文件复制到 ~/client-configs/keys/ 您之前创建的目录中：

```
cp pki/private/client1.key ~/client-configs/keys/
```

### CA 服务器

现在登录到您的 CA 服务器。

接下来，将 client1.req 使用安全方法将文件传输到您的 CA 服务器：

```
# scp sammy@your_openvpn_server_ip:/home/sammy/easy-rsa/pki/reqs/client1.req /tmp
scp sammy@123.125.32.26:/home/sammy/easy-rsa/pki/reqs/client1.req /tmp
```

然后，导航到 EasyRSA 目录，并导入证书请求：

```
cd ~/easy-rsa
./easyrsa import-req /tmp/client1.req client1
./easyrsa sign-req client client1
```

出现提示时，输入 ye s以确认您打算签署证书请求并且它来自受信任的来源：

```
Output
Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 1080 days:

subject=
    commonName                = client1


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /home/sammy/easy-rsa/pki/safessl-easyrsa.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client1'
Certificate is to be certified until Jul  6 06:18:09 2025 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/sammy/easy-rsa/pki/issued/client1.crt
```

这将创建一个名为 client1.crt 将此文件传输回服务器：

```
# scp pki/issued/client1.crt sammy@your_server_ip:/tmp
scp pki/issued/client1.crt sammy@123.125.32.26:/tmp
```

### OpenVPN 服务器

回到您的 OpenVPN 服务器，将客户端证书复制到~/client-configs/keys/目录：

```
cp /tmp/client1.crt ~/client-configs/keys/
```

生成配置文件：

```
cd ~/client-configs
./make_config.sh client1
```

这将`client1.ovpn`在您的`~/client-configs/files`目录中创建一个名为的文件：

```bash
ls ~/client-configs/files
```

```
Output
client1.ovpn
```

您需要将此文件传输到您计划用作客户端的设备。例如，这可能是您的本地计算机或移动设备。

```
# sftp sammy@openvpn_server_ip:client-configs/files/client1.ovpn ~/
sftp sammy@123.125.32.26:client-configs/files/client1.ovpn ~/
```

### 安装客户端配置

**离线安装**

```
# ubuntu20.04 系统
sudo dpkg -i pkg/ubuntu20.04/openvpn_2.4.7-1ubuntu2.20.04.4_amd64.deb
```

```
# redhat8.6 系统
sudo rpm -i pkg/redhat8.6/pkg/*
```

**在线安装**

如果您使用的是 Linux，则可以根据您的发行版使用多种工具。您的桌面环境或窗口管理器可能还包括连接实用程序。

然而，最通用的连接方式是使用 OpenVPN 软件。

在 Ubuntu 或 Debian 上，您可以像在服务器上一样通过键入以下内容来安装它：

```
sudo apt update
sudo apt install openvpn
```

在 CentOS 上，您可以启用 EPEL 存储库，然后键入以下内容进行安装：

```bash
sudo dnf install epel-release
sudo dnf install openvpn
```

#### 配置使用的客户端 `systemd-resolved`

首先`systemd-resolved`通过检查`/etc/resolv.conf`文件来确定您的系统是否用于处理 DNS 解析：

```
cat /etc/resolv.conf
```

```
Output
# This file is managed by man:systemd-resolved(8). Do not edit.
. . .

nameserver 127.0.0.53
options edns0
```

如果您的系统配置为`systemd-resolved`用于 DNS 解析，则`nameserver`选项后面的 IP 地址将为`127.0.0.53`. 文件中还应该有注释，如显示的输出，解释如何`systemd-resolved`管理文件。如果您有一个不同的 IP 地址，`127.0.0.53`那么您的系统可能没有使用`systemd-resolved`，您可以转至下一节配置具有`update-resolv-conf`脚本的Linux 客户端。

要支持这些客户端，请首先安装该`openvpn-systemd-resolved`软件包。它提供了强制`systemd-resolved`使用 VPN 服务器进行 DNS 解析的脚本。

```
sudo apt install openvpn-systemd-resolved
```

安装该软件包之一，配置客户端以使用它，并通过 VPN 接口发送所有 DNS 查询。打开客户端的 VPN 文件：

```bash
nano client1.ovpn
```

现在取消注释您之前添加的以下几行：

client1.ovpn

```
script-security 2
up /etc/openvpn/update-systemd-resolved
down /etc/openvpn/update-systemd-resolved
down-pre
dhcp-option DOMAIN-ROUTE .
```

#### 配置使用的客户端 `update-resolv-conf`

如果您的系统不`systemd-resolved`用于管理 DNS，请检查您的发行版是否包含`/etc/openvpn/update-resolv-conf`脚本：

```
ls /etc/openvpn
```

```
Output
update-resolv-conf
```

如果您的客户端包含该`update-resolv-conf`文件，请编辑您之前传输的 OpenVPN 客户端配置文件：

```bash
nano client1.ovpn
```

取消注释您添加的用于调整 DNS 设置的三行：

client1.ovpn

```
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

如果您使用 CentOS，请将`group`指令从`nogroup`to更改`nobody`为匹配发行版的可用组：

client1.ovpn

```
group nobody
```

保存并关闭文件。

**连接**

现在，您只需将`openvpn`命令指向客户端配置文件即可连接到 VPN ：

```
# 启动前注意检查 client1.ovpn 的 62 行。在 Ubuntu 系统上是 group nogroup，在 CentOS 系统上是 group nobody
sudo openvpn --config client1.ovpn
```

这应该将您连接到您的 VPN。

**注意：**如果您的客户端用于`systemd-resolved`管理 DNS，请通过运行如下`systemd-resolve --status`命令检查设置是否正确应用：

```
systemd-resolve --status tun0
```

```
Output
Link 22 (tun0)
. . .
         DNS Servers: 208.67.222.222
                      208.67.220.220
          DNS Domain: ~.
```

如果您看到您在 OpenVPN 服务器上配置的 DNS 服务器的 IP 地址，以及输出中*DNS 域*的`~.`设置，那么您已正确配置您的客户端以使用 VPN 服务器的 DNS 解析器。

# 安装客户端配置

**离线安装**

```
sudo rpm -i pkg/redhat8.6/pkg/*
```

> 固定客户端端口：修改 client1.ovpn 第 58 行，nobind -> port 51194

**连接**

```
sudo cp client1.ovpn /etc/openvpn/client/client1.conf
# 开机启动
systemctl enable openvpn-client@client1
# 启动
systemctl start openvpn-client@client1
# 停止
systemctl stop openvpn-client@client1
```

测试

```
systemctl status openvpn-client@client1
ping 10.10.0.1
```
# 其它设置
## 客户端固定端口
修改 client1.ovpn 第 58 行，nobind -> port 51194
## 设置 systemctl 启动
```
sudo cp client1.ovpn /etc/openvpn/client/client.conf
# 开机启动
systemctl enable openvpn-client@client
# 启动
systemctl start openvpn-client@client
# 停止
systemctl stop openvpn-client@client

# 测试
systemctl status openvpn-client@client
ping 10.8.0.1
```
## 固定客户端 IP
```
# openvpn 服务器
vi /etc/openvpn/server/server.conf
# 添加一行
client-config-dir /etc/openvpn/ccd
```
在 /etc/openvpn/ccd 创建文件，文件名为客户端的 CN 名称，内容为`ifconfig-push ${IP} ${NETMASK}`

样例：
```
sammy@OpenVPN:/etc/openvpn/ccd$ ll
total 12
drwxr-xr-x 2 root root 4096 Aug 22 15:50 ./
drwxr-xr-x 5 root root 4096 Aug 22 15:19 ../
-rw-r--r-- 1 root root   35 Aug 22 15:50 tx-cjx-master
sammy@OpenVPN:/etc/openvpn/ccd$ cat tx-cjx-master 
ifconfig-push 10.8.0.100 10.8.0.99
```
