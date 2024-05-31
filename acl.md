# ACL 配置

## 基本 ACL 配置 2000-2999

```shell
[Router] time-range workday 8:30 to 18:00 working-day  # 配置一个时间范围 "workday"，从8:30到18:00，适用于工作日
[Router] acl 2000  # 创建基本ACL 2000
[Router-acl-basic-2000] rule 5 permit source 192.168.1.10 0 time-range workday  # 允许来自192.168.1.10的流量，但仅在 "workday" 时间范围内生效 0是反掩码，0表示精确ip地址而不是网段
[Router-acl-basic-2000] quit  # 退出ACL配置模式
```

## 高级 ACL 配置 3000-3999

```shell
[Router] acl 3000  # 创建高级ACL 3000
[Router-acl-adv-3000] rule 5 permit ip source 192.168.2.0 0.0.0.255 destination 192.168.3.100 0  # 允许来自192.168.2.0/24的IP流量到达192.168.3.100
[Router-acl-adv-3000] rule 10 deny ip source 192.168.1.0 0.0.0.255 destination 192.168.3.100 0  # 拒绝来自192.168.1.0/24的IP流量到达192.168.3.100
[Router-acl-adv-3000] rule 15 deny ip source any destination 192.168.3.100 0  # 拒绝所有其他源的IP流量到达192.168.3.100
[Router-acl-adv-3000] quit  # 退出ACL配置模式
```

## DHCP 配置

```shell
[Router] dhcp enable  # 开启DHCP功能
[Router] ip pool 102  # 系统视图下创建IP地址池102
[Router-ip-pool-102] network 192.168.102.0 mask 255.255.255.0  # 配置地址池范围为192.168.102.0/24
[Router-ip-pool-102] gateway-list 192.168.102.254  # 设置默认网关为192.168.102.254
[Router-ip-pool-102] dns-list 8.8.8.8  # 设置DNS服务器为8.8.8.8
[Router-ip-pool-102] excluded-ip-address 192.168.102.101 192.168.102.253  # 排除地址范围192.168.102.101到192.168.102.253
[Router-ip-pool-102] lease 10  # 设置租约时间为10天
[Router-ip-pool-102] quit  # 退出IP地址池配置模式
[Router] interface GigabitEthernet 0/0/2  # 进入GigabitEthernet 0/0/2接口配置模式
[Router-GigabitEthernet0/0/2] dhcp select global  # 接口使用全局地址池进行DHCP分配
[Router-GigabitEthernet0/0/2] quit  # 退出接口配置模式
```

### 禁止 Telnet 登录

```shell
[Router] user-interface vty 0 4  # 进入VTY 0到4的用户界面视图
[Router-ui-vty0-4] acl 2000 inbound  # 应用ACL 2000到VTY用户界面的入方向，禁止Telnet登录
```

### 禁止访问

```shell
[Router] interface GigabitEthernet 3/0/0  # 进入GigabitEthernet 3/0/0接口配置模式
[Router-GigabitEthernet3/0/0] traffic-filter outbound acl 2000  # 应用ACL 2000到接口的出方向，禁止访问
```
