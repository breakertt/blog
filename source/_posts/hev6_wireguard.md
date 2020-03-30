title: 使用 VPS 把 HE IPv6 地址 WireGuard 分配给 Windows 客户端使用
date: 2020/03/14 14:43:30
categories:
- Network
tags:
- Linux
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200314132717.png
---

把 HE 给你的 IPv6 通过 VPS 带回家吧！

<!-- more -->

# 前言

最近呢，发现自己其实对网络一窍不通。之前玩了很久的应用层和传输层，但是对网络层和接口着实是捉襟见肘。前两天看到了 Dr.Cai 发的[BGP速成](https://www.91yunbbs.com/discussion/641/drcais-noob-bgplayer-in-1hour-绝赞速成班-逃#latest)，打算也来搞个玩玩。

但是我的想法是，并没有要一下去就去上 BGP 玩，正好 HE 的 6in4 隧道会分配一个 /64 甚至是 /48 的 v6 段，那不如先拿这个来玩玩。搞定了这个再去搞那些和 BGP 有关系的广播等高级操作。

不过要说一句的是，本人目前在墙外，国内我不知道是否可以这样搞！

# 注册 tunnelbroker

这个有很多博文讲过如何操作了，就不赘述了,可以参考[这篇](https://www.91yunbbs.com/discussion/549/让affman失业系列菜鸟教学-第一篇-用6in4-隧道来替代-socks5代理)。

# VPS(CentOS 7) 配置 IPv6 隧道

## 解除 IPv6 限制（可选）

这里我们要在`/etc/sysctl.conf`加入这几行。

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

## 设置隧道用的 interface

新建`/etc/sysconfig/network-scripts/ifcfg-sit1`，编辑如下内容：

```
DEVICE=sit1
BOOTPROTO=none
ONBOOT=yes
IPV6INIT=yes
IPV6TUNNELIPV4=<Remote IPv4>
IPV6TUNNELIPV4LOCAL=<Local(VPS) IPv4>
IPV6ADDR=<Client IPv6(with mask)>
IPV6_DEFAULTGW=<Server IPv6(without mask)>
```

这边再提供一个虚拟样例：
```
DEVICE=sit1
BOOTPROTO=none
ONBOOT=yes
IPV6INIT=yes
IPV6TUNNELIPV4=216.66.88.98
IPV6TUNNELIPV4LOCAL=185.177.111.111
IPV6ADDR=2001:470:1111:510::2/64
IPV6_DEFAULTGW=2001:470:1111:510::1
```

如果本机没有原生 IPv6 的话，我们就可以在这里```ifup sit1```，然后 ```curl http://v6.ipv6-test.com/api/myip.php``` 看看自己的 v6 是否已经通了。
如果有原生 IPv6 的话，看一下后面我的解决。

## 原生 v6 的追加配置

### 设置默认 v6 出口为 sit1

```
/etc/sysconfig/network

NETWORKING=yes
IPV6_DEFAULTDEV=sit1
```

这边我遇到了一个小坑，我的 VPS 商用了 cloud-init 导致每次重启会重写这个文件，解决方法是运行如下命令：

```
touch /etc/cloud/cloud-init.disabled
```

### 关闭原生 interface 的 v6 功能

在 `/etc/sysctl.conf` 中加入 `net.ipv6.conf.<your default interface>.disable_ipv6 = 1` 也许是个不错的办法。

# 配置WireGuard

## 服务端

### 一键安装

这边我们就先直接用一键脚本进行 WireGurad 的安装，执行如下命令：

```
yum install -y wget && wget https://raw.githubusercontent.com/atrandys/wireguard/master/wireguard_install.sh && chmod +x wireguard_install.sh && ./wireguard_install.sh
```

如果要换内核的话请先换内核在进行安装。

这个脚本自动配置了客户端和服务端的密码以及配置，讲道理是可以直接用的，但是这并不是我们想要的。我们这边只是要本地分配到 IPv6 然后 v6 地址通过 WireGuard 访问，所以还要进一步配置。

首先我们启用一波自动 WireGuard 的自动开机啥的：

```
systemctl enable wg-quick@wg0.service ### 自动开机
systemctl start wg-quick@wg0.service  ### 起一下服务看看能不能跑
systemctl stop wg-quick@wg0.service   ### 停掉，开始配置
```

### 编辑 `wg0.conf`

编辑 `/etc/wireguard/wg0.conf` 为如下：

```
[Interface]
Address = 10.0.0.1/24
MTU = 1420
SaveConfig = false
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o net0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o net0 -j MASQUERADE
ListenPort = 56056
PrivateKey = <>

[Peer]
PublicKey = <>
AllowedIPs = 10.0.0.2/32, 2001:470:1111:510::1/128
```

![](https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200314142004.png)

这里的地址前缀中取一个地址出来写到 AllowedIPs 里面，因为我们就用一台电脑登陆，`::1/128` 就是个不错的选择。至于 PublicKey 和 PrivateKey 就按照原来就好。

然后我们就可以用 `wg-quick up wg0` 来暂时开启 WireGuard 服务并配置客户端。

## Windows 客户端

这边我选择了 TunSafe 作为我的客户端，下载地址：https://tunsafe.com/download 。

安装完成之后，我们点一下主界面的 Edit Config，然后先把服务器端自动生成在 `/etc/wireguard/client.conf` 中的内容拷贝过来。

然后，把 `Interface` 中的 `Address` 改为 `10.0.0.2/32, <Routed /64>::1/128` ; `Peer` 中的 `AllowedIPs` 改为 `10.0.0.2/24, ::0/0`, 这样的话我们 v4 除了 WireGuard 内网都走本地， v6 全部走 WireGuard。

最终配置如下：

```
[Interface]
PrivateKey = <>
Address = 10.0.0.2/32, 2001:470:1111:510::1/128
DNS = 8.8.8.8
MTU = 1420

[Peer]
PublicKey = <>
Endpoint = 185.177.111.111:56056
AllowedIPs = 10.0.0.2/24, ::0/0
PersistentKeepalive = 25
```
然后尝试链接，用浏览器打开 `https://ipv6-test.com/` 看看有没有得到自己的 v6 段下的地址。

## 收尾工作

在服务端运行如下脚本，进行长期运行：

```
wg-quick up wg0 ### 关闭临时服务
systemctl start wg-quick@wg0.service  ### 开启守护服务
```

# 结语

之后的打算还有不少，就先把 flag 立起来吧。

- 给 R6300v2 安装 OpenWrt 并用 WireGuard 分配整个网段的 IPv6 给本地局域网的设备
- 找一种比 WireGuard 更好的组网方式（可以透过GFW)
- 注册ASN，宣告自己的 IP 地址并用之前的方法把这些 IP 带回家
 


# 参考资料
1. [how-to-get-rid-of-cloud-init](https://askubuntu.com/questions/539277/how-to-get-rid-of-cloud-init)
2. [科学上网指南(10)——wireguard](https://eddyemma.com/blog/2018/08/26/科学上网指南-wireguard/)
3. [将宣告的 subnet 通过 wireguard 分配给客户端使用](https://blog.ni-co.moe/public/571.html)
4. [CentOS 安装最新版的Wireguard](https://kotori.net/2018/10/21/centos-%E5%AE%89%E8%A3%85%E6%9C%80%E6%96%B0%E7%89%88%E7%9A%84wireguard/)
5. [CentOS7一键脚本安装WireGuard
](https://www.atrandys.com/2018/886.html)