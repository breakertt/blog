title: What modifications Clash made on ip route and ip rule to enable transparent proxy on TUN?
date: 2023/04/02 19:39:16
categories:
- Linux
tags:
- Network
- Proxy
toc: true
---

<!-- more -->

Before Clash start we have following ip rules and routes 

```
$ ip rule list
0:	from all lookup local
32766:	from all lookup main
32767:	from all lookup default

$ ip route list
default via 192.168.122.1 dev enp1s0 proto dhcp src 192.168.122.41 metric 100 
192.168.122.0/24 dev enp1s0 proto kernel scope link src 192.168.122.41 metric 100 
192.168.122.1 dev enp1s0 proto dhcp scope link src 192.168.122.41 metric 100 
```

After Clash start we have following ip rules and routes

```
$ ip rule list
0:	from all lookup local
9500:	not from all dport 53 lookup main suppress_prefixlength 0
9510:	not from all iif lo lookup 1970566510
9520:	from 0.0.0.0 iif lo uidrange 0-4294967294 lookup 1970566510
9530:	from 198.18.0.1 iif lo uidrange 0-4294967294 lookup 1970566510
32766:	from all lookup main
32767:	from all lookup default

$ ip route list # table main
default via 192.168.122.1 dev enp1s0 proto dhcp src 192.168.122.41 metric 100 
192.168.122.0/24 dev enp1s0 proto kernel scope link src 192.168.122.41 metric 100 
192.168.122.1 dev enp1s0 proto dhcp scope link src 192.168.122.41 metric 100 
198.18.0.0/16 dev utun proto kernel scope link src 198.18.0.1

$ ip route list table 1970566510
default dev utun proto unspec 
```

Now let us exaplsin what exactly every new lines does

```
9500:	not from all dport 53 lookup main suppress_prefixlength 0
```

This rules means sending all non-DNS query packets are sent to lookup the `main` table, but the default rule in the `main` table is ignored due to `suppress_prefixlength 0`. `suppress_prefixlength 0` means ignore rules matches more than prefixlength bits. Here we have 0 so it just ignore 0.0.0.0/0. The point of this rule is 1. allowing we can communicate to other machines under same LAN without clash; 2. send the packets with fake-ip addresses to TUN interface. However, we still need to hijack for example the gateway's DNS (192.168.122.1:53 in this case), so it is only for non-DNS query packets.

```
9510:	not from all iif lo lookup 1970566510
```

`iif lo` means input interface `lo`, this is a way of specifying rules for locally generated packets that have a specific source address. This rules filter out packets from other machines under same LAN which is being forwarded in the chains, here the machine running Clash acts as proxy-gateway, are send to the TUN interface With this rule, packtes don't have fake-ip range destinations are also transmitted to Clash in case DNS hijack not working.

```
9520:	from 0.0.0.0 iif lo uidrange 0-4294967294 lookup 1970566510
```

`from 0.0.0.0 iif lo` means packets without source addresses and generated locally. In Linux, client-initiaed packets source address can not be decided before it reaches to the interface. This rule ensures the machine running Clash itself can be proxied.

```
9530:	from 198.18.0.1 iif lo uidrange 0-4294967294 lookup 1970566510
```

I am not 100% sure about this rule. This is my guess: when the Clash responses to proxied program, it will send a packet with src_addr 198.18.0.1 first, then catch this packet and modify the src_addr to the correct mapping. Please correct me if you have better idea.

```
198.18.0.0/16 dev utun proto kernel scope link src 198.18.0.1
```
Send fake-ip range packets to TUN interface

```
default dev utun proto unspec 
```
A standalone ip table which sends all packet to TUN interface once looked up by. We already see it for several times.

# One more thing

After we understand everything, we can try to disable the transparent proxy of the machine running Clash. Can you think up how to do that?

Answer:
```
sudo ip rule del from 0.0.0.0 iif lo uidrange 0-4294967294 lookup 1970566510
```

# Refs
[1] https://utcc.utoronto.ca/~cks/space/blog/linux/IsolatedInterfacesPrinciples
