title: Baidupcs-web 在 apache2 上的反向代理配置
date: 2019/08/20 16:10:00
categories:
- Linux
tags:
- Network
- Proxy
thumbnail: https://raw.githubusercontent.com/imaginebreake/imaginebreake.github.io/pic/20200313235940.png
toc: true
---

此处省略一万句脏话。

<!-- more -->

最近被安利了一个webui版本的baidupcs-go，叫做BiaduPCS-Web，据说不错我就也来用用。

安装很容易，使用了GitHub上的一键脚本：https://github.com/user1121114685/baidupcsweb

但是服务起来之后用的是5299端口，不太优雅对吧？那我就想着做个反代好了，也就是本文的重点了。

一开始我是想配一个类似于"http://server/pan/", 这样的子目录的反向代理的，但是这个软件不支持baseURL的配置以及路径都是写死的，那就只能单开一个域名给这个软件了。如果您使用的http服务器是nginx请看这里：[请问如何用nginx进行反向代理](https://github.com/liuzhuoling2011/baidupcs-web/issues/20)，caddy我也不太清楚，反正这篇文章讲的只有apache的。

这个软件有部分链接用的不是http，而是websocket，这就是问题所在了，查了不少资料，我们需要先开启用几个插件：

```shell
a2enmod proxy
a2enmod proxy_http
a2enmod rewrite
sudo systemctl restart apache2
```

然后添加如下的新站点，只开了443的，80请自己改，内容如下：

```
<VirtualHost *:443>
    ServerName pan.abc.com ######

    SSLCertificateFile  /etc/apache2/ssl/pan/cert.pem ######
    SSLCertificateKeyFile /etc/apache2/ssl/pan/key.pem ######
    SSLCertificateChainFile /etc/apache2/ssl/pan/chain.pem ######

    SSLEngine on

    ProxyPreserveHost On
    ProxyRequests Off
    RewriteEngine On

    RewriteCond %{HTTP:Upgrade} = websocket [NC]
    RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
    RewriteRule /ws(.*)           ws://localhost:5299/ws$1 [P,L]
    RequestHeader set X-Forwarded-Proto "https"

    <Location /ws>
        Require all granted
        ProxyPassReverse ws://127.0.0.1:5299/ws
    </Location>

    <Location />
        Require all granted
        ProxyPass http://localhost:5299/
        ProxyPassReverse /
    </Location>

</VirtualHost>
```

简单讲一下，这边把当http请求需要升级为ws请求时，把反代的头改成了ws，然后就可以成功访问了。