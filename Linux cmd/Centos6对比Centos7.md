# Centos6对比Centos7

#### 这里对比Centos6.5 Minimal和Centos 7.0 Minimal版本

| 区别       | Centos6                                                      | Centos7                                                      |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| iso大小    | 400M（缺少很多库，比如lzo）                                  | 900M                                                         |
| 安装       | 最后一步配置：需要自定义分区<br>（系统引导区、交换区、根分区） | 不需要                                                       |
| 网卡       | eth0                                                         | ens33                                                        |
| 配置网络   | 配置ifcfg-eth0<br>IPADDR、NETMASK、GATEWAY、DNS1             | 配置ifcfg-ens33<br/>IPADDR、NETMASK、GATEWAY、DNS1           |
| 修改主机名 | 同时修改/etc/hostname和/etc/sysconfig/network                | 修改/etc/hostname                                            |
| 防火墙     | service iptables stop，关闭防火墙<br>service iptables off,停止防火墙服务<br>chkconfig iptables off，关闭开机自启动 | systemctl stop firewalld<br>systemctl disable firewalld<br>chkconfig firewalld off |



