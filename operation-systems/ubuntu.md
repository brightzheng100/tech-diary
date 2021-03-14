# Ubuntu

## Generic

### Timezone

```text
$ timedatectl list-timezones
$ sudo timedatectl set-timezone Asia/Singapore
$ date
```

### How to Fix ‘E: Could not get lock /var/lib/dpkg/lock’ Error in Ubuntu Linux

When trying to install components but saw this: `E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?`

This may be some `apt-get install` process is happening:

```text
$ ps aux | grep -i apt
root        7025  0.0  0.0   2608   476 ?        Ss   12:17   0:00 /bin/sh /usr/lib/apt/apt.systemd.daily install
root        7029  0.0  0.0   2608  1656 ?        S    12:17   0:00 /bin/sh /usr/lib/apt/apt.systemd.daily lock_is_held install
bright     12846  0.0  0.0   8156  2580 pts/0    S+   12:20   0:00 grep --color=auto -i ap
```

Just wait for a couple of minutes, it will be okay after the auto update process is done. To disable the auto update process:

```text
$ sudo vi /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
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

## Bionic - 18.04 LTS

### Enable dual network interfaces for VirtualBox

Ref: [https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux](https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux)

```text
$ cd /etc/netplan/

$ cat 50-cloud-init.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: yes

$ sudo vi 50-cloud-init.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: yes
    enp0s8:
      dhcp4: yes

$ sudo netplan apply

$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.107  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::a00:27ff:fe2b:4295  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:2b:42:95  txqueuelen 1000  (Ethernet)
        RX packets 64  bytes 10751 (10.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 70  bytes 10955 (10.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.3.15  netmask 255.255.255.0  broadcast 10.0.3.255
        inet6 fe80::a00:27ff:fe7a:edcf  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:7a:ed:cf  txqueuelen 1000  (Ethernet)
        RX packets 14  bytes 2232 (2.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 42  bytes 3868 (3.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1024  bytes 73520 (73.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1024  bytes 73520 (73.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
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

