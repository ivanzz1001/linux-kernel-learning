# Centos7配置静态IP

参看:

- [CentOS 7 设置静态 IP](https://blog.csdn.net/nyist_zxp/article/details/131338441)


### 1. 找出当前IP地址及网卡名称
```
# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.180.140  netmask 255.255.255.0  broadcast 192.168.180.255
        inet6 fe80::8d17:f8ed:eb74:5941  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:46:68:b6  txqueuelen 1000  (Ethernet)
        RX packets 38  bytes 7027 (6.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 117  bytes 14165 (13.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 252  bytes 21672 (21.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 252  bytes 21672 (21.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:fe:35:91  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

### 2. 修改配置文件
修改网卡对应的配置文件: /etc/sysconfig/network-scripts/ifcfg-ens33 
```
# static IP
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static 
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=2de148e1-945e-4744-bc6d-18394e985967
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.180.140 
NETMASK=255.255.255.0 
GATEWAY=192.168.180.2
DNS1=8.8.8.8
DNS2=8.8.4.4
DNS3=114.114.114.114

```

之后执行如下命令重启网络:
<pre>
# service network restart
# ping www.baidu.com
</pre>

### 3. 附录：安装Openssh

>ps: 参考https://blog.csdn.net/DT_FlagshipStore/article/details/126051811

- 安装openssh-server
<pre>
# yum install openssh-server
</pre>

- 修改配置: /etc/ssh/sshd_config
<pre>
PermitRootLogin yes
PasswordAuthentication yes

# 配置文件底部增加如下
HostKeyAlgorithms=+ssh-rsa,ssh-dss
KexAlgorithms=+diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
</pre>

- 重启openssh服务
<pre>
# service sshd restart
</pre>
