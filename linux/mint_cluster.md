
## 环境

  centos1 172.16.1.11
  centos2 172.16.1.12

## 网络配置

* 主机所在网段: 172.16.1.0/24
* nat网段: 10.0.2.0/24

## 主机克隆
  # 删除克隆主机的网卡信息
  /etc/udev/rules.d/70-persistent-net.rules
  # 修改NAME名称为网卡名称
  SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:77:89:20", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

  # 修改网卡的mac地址为/etc/udev/rules.d/70-persistent-net.rules中所对应的ATTR{address}
  /etc/sysconfig/network-scripts/ifcfg-eth0
  HWADDR=08:00:27:77:89:20


## ssh

ssh 172.16.1.11 -l root
ssh -l root 192.168.2.216 pwd

## 修改主机名
* /etc/sysconfig/network
* /etc/hosts
