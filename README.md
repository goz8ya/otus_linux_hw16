## otus_linux_hw16
iptables -nvL --line-numbers

## Accept lo full access
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

## Accept ssh connection
iptables -A INPUT  -p tcp --dport 22 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT  -p TCP --dport=22  -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[ssh-input] "
iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[ssh-output] "

#iptables -A INPUT  -p TCP --dport=22 -m state --state NEW  -j ACCEPT
#iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

## что-то пойдет не так, машина перезагрузится.  
shutdown -r +5
## Если все пройдет успешно отменяем перезагрузку 
shutdown -c

## DROP and close everything
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP



## Accept Http Trafic
iptables -A INPUT -i eth0 -p tcp -s 0/0 --sport 1024:65535 --dport 80 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p TCP -s 0/0 --sport 1024:65535 --dport 80  -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[http-input] "
iptables -A OUTPUT -o eth0 -p tcp --sport 80 -d 0/0 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp --sport 80 -d 0/0 --dport 1024:65535 -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[http-output] "



## Accept Https Trafic
iptables -A INPUT -i eth0 -p tcp -s 0/0 --sport 1024:65535 --dport 443 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p TCP -s 0/0 --sport 1024:65535 --dport 443  -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[https-input] "
iptables -A OUTPUT -o eth0 -p tcp --sport 443 -d 0/0 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp --sport 443 -d 0/0 --dport 1024:65535 -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix="[https-output] "


# Allow incoming ICMP ping 
#Type 0 — Echo Reply
#Type 8 — Echo
iptables -A INPUT -i eth0 -p icmp --icmp-type 8 -s 0/0 -m state --state NEW,ESTABLISHED,RELATED -m limit --limit 2/sec  -j ACCEPT
iptables -A OUTPUT -o eth0  -p icmp --icmp-type 0 -d 0/0 -m state --state ESTABLISHED,RELATED -j ACCEPT


## log Drop incomming 
iptables -A INPUT -4 -m limit --limit 1/m --limit-burst 7 -j LOG --log-prefix="[default-drop-input]"
iptables -A OUTPUT -4 -m limit --limit 1/m --limit-burst 7 -j LOG --log-prefix="[default-drop-output]"

## for update, instll packets
### DNS
iptables -A OUTPUT -o eth0  -p tcp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0  -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --sport 53 -m state --state ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT

# INTERNET 443
iptables -A OUTPUT -o eth0  -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT

## Устанавливаем управление
root@server01:/home/vagrant# apt install iptables-persistent netfilter-persistent

## Сохраняем настроки
root@server01:/home/vagrant# netfilter-persistent save


## Перезагружаем, проверяем правила
cat /etc/iptables/rules.v4
*filter
:INPUT DROP [315:28936]
:FORWARD DROP [0:0]
:OUTPUT DROP [7800:748088]
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[ssh-input] "
-A INPUT -i eth0 -p tcp -m tcp --sport 1024:65535 --dport 80 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --sport 1024:65535 --dport 80 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[http-input] "
-A INPUT -i eth0 -p tcp -m tcp --sport 1024:65535 --dport 443 -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --sport 1024:65535 --dport 443 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[https-input] "
-A INPUT -i eth0 -p icmp -m icmp --icmp-type 8 -m state --state NEW,RELATED,ESTABLISHED -m limit --limit 2/sec -j ACCEPT
-A INPUT -m limit --limit 1/min --limit-burst 7 -j LOG --log-prefix "[default-drop-input]"
-A INPUT -i eth0 -p tcp -m tcp --sport 53 -m state --state ESTABLISHED -j ACCEPT
-A INPUT -i eth0 -p udp -m udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
-A OUTPUT -o lo -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[ssh-output] "
-A OUTPUT -o eth0 -p tcp -m tcp --sport 80 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --sport 80 --dport 1024:65535 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[http-output] "
-A OUTPUT -o eth0 -p tcp -m tcp --sport 443 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --sport 443 --dport 1024:65535 -m limit --limit 5/min --limit-burst 7 -j LOG --log-prefix "[https-output] "
-A OUTPUT -o eth0 -p icmp -m icmp --icmp-type 0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A OUTPUT -m limit --limit 1/min --limit-burst 7 -j LOG --log-prefix "[default-drop-output]"
-A OUTPUT -o eth0 -p tcp -m tcp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p udp -m udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
COMMIT
## Completed on Wed Apr 10 14:02:59 2024



iptables -nvL --line-numbers
Chain INPUT (policy DROP 3 packets, 333 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1       16  1704 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2       74  6121 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 state NEW,RELATED,ESTABLISHED
3        0     0 LOG        tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[ssh-input] "
4        0     0 ACCEPT     tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spts:1024:65535 dpt:80 state NEW,RELATED,ESTABLISHED
5        0     0 LOG        tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spts:1024:65535 dpt:80 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[http-input] "
6        0     0 ACCEPT     tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spts:1024:65535 dpt:443 state NEW,RELATED,ESTABLISHED
7        0     0 LOG        tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spts:1024:65535 dpt:443 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[https-input] "
8        0     0 ACCEPT     icmp --  eth0   *       0.0.0.0/0            0.0.0.0/0            icmptype 8 state NEW,RELATED,ESTABLISHED limit: avg 2/sec burst 5
9        8  2217 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            limit: avg 1/min burst 7 LOG flags 0 level 4 prefix "[default-drop-input]"
10      20  7246 ACCEPT     tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spt:53 state ESTABLISHED
11      36 11315 ACCEPT     udp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            udp spt:53 state ESTABLISHED
12     156  288K ACCEPT     tcp  --  eth0   *       0.0.0.0/0            0.0.0.0/0            tcp spt:443 state ESTABLISHED

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy DROP 8 packets, 608 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1       16  1704 ACCEPT     all  --  *      lo      0.0.0.0/0            0.0.0.0/0
2       50  6833 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp spt:22 state ESTABLISHED
3        0     0 LOG        tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp spt:22 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[ssh-output] "
4        0     0 ACCEPT     tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp spt:80 dpts:1024:65535 state ESTABLISHED
5        0     0 LOG        tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp spt:80 dpts:1024:65535 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[http-output] "
6        0     0 ACCEPT     tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp spt:443 dpts:1024:65535 state ESTABLISHED
7        0     0 LOG        tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp spt:443 dpts:1024:65535 limit: avg 5/min burst 7 LOG flags 0 level 4 prefix "[https-output] "
8        0     0 ACCEPT     icmp --  *      eth0    0.0.0.0/0            0.0.0.0/0            icmptype 0 state RELATED,ESTABLISHED
9        7   636 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            limit: avg 1/min burst 7 LOG flags 0 level 4 prefix "[default-drop-output]"
10      21  1542 ACCEPT     tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp dpt:53 state NEW,ESTABLISHED
11      36  3288 ACCEPT     udp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            udp dpt:53 state NEW,ESTABLISHED
12     100  6752 ACCEPT     tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp dpt:443 state NEW,ESTABLISHED
13       0     0 ACCEPT     tcp  --  *      eth0    0.0.0.0/0            0.0.0.0/0            tcp dpt:443 state NEW,ESTABLISHED
