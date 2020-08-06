title: 在 Ubuntu18.04 上使用 clash 部署旁路代理网关（透明代理）
date: 2019/08/20 14:10:00
categories:
- Linux
tags:
- Network
- Proxy
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200314000119.png
toc: true
---

Apple TV 4K 到货前的准备工作

<!-- more -->

最近打算购入一台 Apple TV 4K, 又因为众所周知的原因, Apple TV 4k 在没有代理的情况下是根本没法用的，所以就萌生了拿之前买的nuc来做一个旁路代理网关的想法。

在Ubuntu上部署一个透明代理网关主要是三块：安装clash, 简单配置clash， 高级配置clash, 配置iptables转发。

# 安装clash

使用clash的最主要原因是它自带redir服务，且使用go语言开发安装非常方便。

```shell
cd ~
wget https://github.com/Dreamacro/clash/releases/download/v0.15.0/clash-linux-amd64-v0.15.0.gz #下载二进制文件
gzip -d clash-linux-amd64-v0.15.0.gz #解压
mv clash-linux-amd64-v0.15.0 /usr/local/bin/clash #移动到bin
chmod +x /usr/local/bin/clash #添加执行权限
```

然后我们把clash设置成service，下面是我的`/etc/systemd/system/clash.service`

```conf
[Unit]
Description=clash service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/clash
Restart=on-failure # or always, on-abort, etc

[Install]
WantedBy=multi-user.target
```

然后我们把clash激活为开机启动

```shell
systemctl daemon-reload
systemctl enable clash
```

# 简单配置clash

在这个环节中主要是简单配置clash及其dashboard，让其能作为一个局域网的代理服务器存在。

```shell
cd ~/.config/
mkdir clash
touch config.yaml
wget https://github.com/haishanh/yacd/archive/gh-pages.zip
unzip gh-pages.zip
mv yacd-gh-pages/ dashboard/
```
然后我们给`config.yaml`加上内容：

```yaml
port: 7890
socks-port: 7891
redir-port: 7892
allow-lan: true
mode: Rule
log-level: info
external-controller: 0.0.0.0:9090
secret: ""
external-ui: dashboard
#此处内容请安装一个gui版本的clash然后在里面配置好代理然后抄过来
Proxy: 
Proxy Group:
#
Rule:
- IP-CIDR,127.0.0.0/8,DIRECT
- IP-CIDR,192.168.0.0/16,DIRECT
- FINAL,Proxy
```

然后我们执行`service clash start`，就有一个可以使用的局域网代理服务器了，甚至可以在浏览器里面访问`http://serverip:9090/ui/`来调试clash！如果是公网的话我推荐secret处加上内容以保证安全性。

# 高级配置clash

这里的配置主要是两块：

## clash dns

因为我们要拿clash做一个透明网关，那么dns服务必然是一个问题，clash自带的dns服务很神奇，我研究了一个上午，如果想了解可以看这两篇文章：[DNS污染对Clash（for Windows）的影响](https://github.com/Fndroid/clash_for_windows_pkg/wiki/DNS%E6%B1%A1%E6%9F%93%E5%AF%B9Clash%EF%BC%88for-Windows%EF%BC%89%E7%9A%84%E5%BD%B1%E5%93%8D)，[代替 Surge 增强模式——使用 KoolClash 作为代理网关](https://blog.skk.moe/post/alternate-surge-koolclash-as-gateway/)

最后我用的就是第一篇文章中所用到的方案，在clash的设置文件中加入了以下内容：

```yaml
dns:
  enable: true
  listen: 0.0.0.0:53
  enhanced-mode: redir-host
  nameserver:
    - 'tls://dns.rubyfish.cn:853'
  fallback:
    - 'tls://1.1.1.1:853'
    - 'tcp://1.1.1.1:53'
    - 'tcp://208.67.222.222:443'
    - 'tls://dns.google'
```

为什么不用fake-ip呢？因为我觉得目前的redir-host方案足够我本人使用了。

## 国内外分流

clash这个软件的一大特色就是他的分流功能，所以我想还是得用起来，不说好用，至少可以堪用。

然后我就找到了网上已经有现成的规则文件了，可以抄过来用。

如果需要分流的话就把[这个文件](https://github.com/Hackl0us/SS-Rule-Snippet/blob/master/LAZY_RULES/clash.yaml)中的rules部分抄进目前的`config.yaml`。

# 配置iptables转发

这是我们的最后一步，主要是使用iptables配置nat的转发到clash，很大一部分都是参考了 [Clash TProxy Mode](https://lancellc.gitbook.io/clash/start-clash/clash-udp-tproxy-support)，不过这里面的规则有问题，会导致 dns 的回环。

最后修复后的规则如下，这个规则网关本身是不走代理的，反正我可以用 proxychains-ng 对本机进行代理：

```
iptables -t nat -N clash
iptables -t nat -A clash -d 0.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 10.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 127.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 169.254.0.0/16 -j RETURN
iptables -t nat -A clash -d 172.16.0.0/12 -j RETURN
iptables -t nat -A clash -d 192.168.0.0/16 -j RETURN
iptables -t nat -A clash -d 224.0.0.0/4 -j RETURN
iptables -t nat -A clash -d 240.0.0.0/4 -j RETURN
iptables -t nat -A clash -d <local host ip> -j RETURN
iptables -t nat -A clash -p tcp -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.8.8 -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.4.4 -j REDIRECT --to-port 7892
iptables -t nat -A PREROUTING -p tcp -j clash

ip rule add fwmark 1 table 100
ip route add local default dev lo table 100
iptables -t mangle -N clash
iptables -t mangle -A clash -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A clash -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A clash -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A clash -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A clash -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A clash -d <local host ip> -j RETURN
iptables -t mangle -A clash -p udp -j TPROXY --on-port 7892 --tproxy-mark 1
iptables -t mangle -A PREROUTING -p udp -j clash
```

当然我们还不希望这些规则重启就没，那么我们就需要安装一些辅助工具来持久化iptables的规则：

```shell
sudo apt install iptables-persistent netfilter-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

具体可以看这里：[how-to-save-rules-of-the-iptables](https://askubuntu.com/questions/119393/how-to-save-rules-of-the-iptables)

# 尾声

那么就可以把旁路网关地址以及dns设置在需要的机器上了！我也要开始下单 Apple TV 了！
