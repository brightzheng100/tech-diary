# Ubuntu

## Generic

### Timezone

```text
$ timedatectl list-timezones
$ sudo timedatectl set-timezone Asia/Singapore
$ date
```

## Xenial - 16.04 LTS

### Enable dual network interfaces for VirtualBox

```text
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# which maps to "Host-only adapter"
auto enp0s3
iface enp0s3 inet dhcp

# The secondary network interface
# which maps to "NAT"
auto enp0s8
#iface enp0s8 inet dhcp
iface enp0s8 inet static
address  192.168.56.101
network  192.168.56.0
netmask  255.255.255.0
gateway  192.168.56.1
dns-nameservers  8.8.8.8  192.168.56.1
```

This setting leads to VMs provisioned on VirtualBox like this:

* It can ping Internet through NAT from Host laptop;
* It has a potential static IP if you want

```text
enp0s3    Link encap:Ethernet  HWaddr 08:00:27:d6:00:3d
          inet addr:192.168.56.101  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fed6:3d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:226 errors:0 dropped:0 overruns:0 frame:0
          TX packets:170 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:21419 (21.4 KB)  TX bytes:21035 (21.0 KB)

enp0s8    Link encap:Ethernet  HWaddr 08:00:27:44:a1:a8
          inet addr:10.0.3.15  Bcast:10.0.3.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe44:a1a8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:38 errors:0 dropped:0 overruns:0 frame:0
          TX packets:61 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:7129 (7.1 KB)  TX bytes:5521 (5.5 KB)
```

## Focal Fossa - 20.04 LTS

### Enable dual network interfaces for VirtualBox

```text
$ sudo vi /etc/netplan/00-installer-config.yaml

$ cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
  version: 2
  
$ sudo netplan try
```

This setting leads to VMs provisioned on VirtualBox like this:

* It can ping Internet through NAT from Host laptop;
* It has a potential static IP if you want

```text
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4a:d7:4d brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.105/24 brd 192.168.56.255 scope global dynamic enp0s3
       valid_lft 848sec preferred_lft 848sec
    inet6 fe80::a00:27ff:fe4a:d74d/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:36:40:11 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.15/24 brd 10.0.3.255 scope global dynamic enp0s8
       valid_lft 86048sec preferred_lft 86048sec
    inet6 fe80::a00:27ff:fe36:4011/64 scope link
       valid_lft forever preferred_lft forever
```

