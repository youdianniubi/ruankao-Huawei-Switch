# DHCP

## 配置

### DHCP 接口地址池配置

```shell
[Router] dhcp enable // 开启 DHCP 功能
[Router] interface GigabitEthernet 0/0/1
[Router-GigabitEthernet0/0/1] ip address 192.168.101.1 24 // 配置接口 IP 地址和子网掩码
[Router-GigabitEthernet0/0/1] dhcp select interface // 将接口 IP 地址作为网关
[Router-GigabitEthernet0/0/1] dhcp server dns-list 8.8.8.8 // 设置 DNS 服务器地址
[Router-GigabitEthernet0/0/1] dhcp server excluded-ip-address 192.168.101.101 192.168.101.253 // 排除的 IP 地址范围
[Router-GigabitEthernet0/0/1] dhcp server lease day 10 // 设置 IP 地址租期为 10 天
[Router-GigabitEthernet0/0/1] quit
```

### DHCP 全局地址池配置

```shell
[Router] dhcp enable // 开启 DHCP 功能
[Router] ip pool 102 // 在系统视图下创建 IP 地址池 102
[Router-ip-pool-102] network 192.168.102.0 mask 255.255.255.0 // 配置地址池的网段和子网掩码
[Router-ip-pool-102] gateway-list 192.168.102.254 // 设置网关地址
[Router-ip-pool-102] dns-list 8.8.8.8 // 设置 DNS 服务器地址
[Router-ip-pool-102] excluded-ip-address 192.168.102.101 192.168.102.253 // 排除的 IP 地址范围
[Router-ip-pool-102] lease 10 // 设置 IP 地址租期为 10 天
[Router-ip-pool-102] quit
[Router] interface GigabitEthernet 0/0/2
[Router-GigabitEthernet0/0/2] dhcp select global // 使用全局地址池的 DHCP
[Router-GigabitEthernet0/0/2] quit
```
