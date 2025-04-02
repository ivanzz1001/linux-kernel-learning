# Ubuntu22.04配置静态IP


ubuntu从17.10开始，已放弃在/etc/network/interfaces里固定IP的配置，即使配置也不会生效，而是改成netplan方式 ，网卡配置文件路径在：/etc/netplan/文件下，一般后缀名为.yaml文件

>参考: [ubuntu18.04修改IP为静态IP并能够上网](https://blog.csdn.net/IT_SoftEngineer/article/details/112794427)


### 1. 找出当前网关

<pre>
# apt install net-tools
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.180.2   0.0.0.0         UG    100    0        0 ens33
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 ens33
192.168.180.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
</pre>

### 2. 获取到当前的IP地址及网卡名称

```
# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.180.130  netmask 255.255.255.0  broadcast 192.168.180.255
        inet6 fe80::20c:29ff:fe50:d2ed  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:50:d2:ed  txqueuelen 1000  (Ethernet)
        RX packets 15288  bytes 18623933 (18.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8063  bytes 619797 (619.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1724  bytes 148156 (148.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1724  bytes 148156 (148.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 3. 修改/etc/netplan下相关文件

这里我们修改/etc/netplan/01-network-manager-all.yaml文件如下:
```
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:
      addresses: [192.168.180.130/24]
      gateway4: 192.168.180.2
      dhcp4: no
      optional: true
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4, 114.114.114.114]
        search: []
```

### 4. 配置静态IP生效

执行如下命令:
```
# netplan apply
```
