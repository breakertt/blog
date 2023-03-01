title: Wireguard iptables rules for Oracle VPS
date: 2023/2/3 13:20:00
categories:
- Linux
toc: true
---

Becuase Oracle VPS instance has a set of quite strong initial iptables rules for the safety of server, we need to open the access from these new interafaces and ports, and also, insert these rules to the top of chains.

```
PostUp = iptables -I FORWARD 1 -i %i -o enp0s3 -j ACCEPT; iptables -I FORWARD 1 -i enp0s3 -o %i -j ACCEPT; iptables -I INPUT 1 -i enp0s3 -p udp --dport 51820 -j ACCEPT; iptables -I INPUT 1 -i %i -j ACCEPT; iptables -t nat -I POSTROUTING 1 -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o enp0s3 -j ACCEPT; iptables -D FORWARD -i enp0s3 -o wg0 -j ACCEPT; iptables -D INPUT -i enp0s3 -p udp --dport 51820 -i wg0 -j ACCEPT; iptables -D INPUT -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE
```
