# Centos

## Generic

### Enable dual network interfaces for VirtualBox

```text
$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="enp0s3"
UUID="06a57381-7efb-479a-97bd-29f1b152b896"
DEVICE="enp0s3"
ONBOOT="yes"

$ sudo cp /etc/sysconfig/network-scripts/ifcfg-enp0s3 /etc/sysconfig/network-scripts/ifcfg-enp0s8
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="enp0s8"
UUID="06a57381-7efb-479a-97bd-29f1b152b897"
DEVICE="enp0s8"
ONBOOT="yes"

$ sudo ifdown enp0s8 && sudo ifup enp0s8
```

Shutdown, reconfigure the VirtualBox network:

* Adapter 1: Host-only Adapter
* Adapter 2: NAT

Reboot:

```text
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b1:df:0f brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global noprefixroute dynamic enp0s3
       valid_lft 842sec preferred_lft 842sec
    inet6 fe80::4b38:6c82:1bf5:a68d/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7c:97:67 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.15/24 brd 10.0.3.255 scope global noprefixroute dynamic enp0s8
       valid_lft 85104sec preferred_lft 85104sec
    inet6 fe80::f138:6024:b82d:345/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

### Install network-scripts as the network manager

```text
$ sudo yum install network-scripts
$ sudo systemctl restart network

$ ls /etc/sysconfig/network-scripts/
ifcfg-enp0s3  ifdown-Team      ifdown-ippp  ifdown-routes  ifup-Team      ifup-eth   ifup-plip    ifup-sit          network-functions
ifcfg-enp0s8  ifdown-TeamPort  ifdown-ipv6  ifdown-sit     ifup-TeamPort  ifup-ippp  ifup-plusb   ifup-tunnel       network-functions-ipv6
ifcfg-lo      ifdown-bnep      ifdown-isdn  ifdown-tunnel  ifup-aliases   ifup-ipv6  ifup-post    ifup-wireless
ifdown        ifdown-eth       ifdown-post  ifup           ifup-bnep      ifup-isdn  ifup-routes  init.ipv6-global

$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="enp0s8"
DEVICE="enp0s8"
ONBOOT="yes"
```

## Centos 8



## Centos 7

### 

