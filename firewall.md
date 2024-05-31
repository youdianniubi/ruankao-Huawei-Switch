# Firewall 配置

## 网络拓扑与说明

![img|600](image/SCR-xkd.png)

## 配置要求

1. 根据拓扑图，配置防火墙接口的IP地址，并将接口划入相应的安全区域。
2. 允许内网主机PC1主动访问Internet，但不允许Internet主动访问PC1。
3. 在出口防火墙上进行NAT，使用NAT公网地址池为100.1.1.10-100.1.1.20。
4. 允许Internet通过公网地址100.1.1.100/24访问内部Web服务的目的地址为192.168.2.100/24。

## 总结

1. 防火墙的主要功能是实现内部信任网络与公共不可信任网络的隔离，并进行访问控制。
2. 部署方式包括**串行部署**和旁路部署。串行部署可以检测所有通过防火墙的流量，而旁路部署只能检测通过策略路由等方式引流通过防火墙的流量。另外，IPS（入侵防御系统）和IDS（入侵检测系统）也是防火墙的一部分，它们分别可以阻断攻击和检测安全威胁。
3. 华为防火墙默认有四个安全域：信任区域、非信任区域、非军事化区域和本地区域。其中，Trust用于连接内部网络，Untrust用于连接Internet互联网，DMZ用于连接对外提供访问的服务器，Local指防火墙自身。
4. 注意事项包括：
   - local、trust、DMZ、untrust这四个安全域是系统自带的，不能删除。除了这四个域之外，还可以自定义域。
   - 安全域等级为Local > Trust > DMZ > Untrust，自定义的域的优先级可以自行调节。
   - 如果域与域之间没有配置策略，则默认是deny，即任何数据如果没有策略是无法通过的。在同一区域内的数据相当于二层交换机一样直接转发。
   - **优先级低的域向优先级高的域方向是inbound，反之是outbound**。

## 配置

### 1. 防火墙 Firewall 配置

```shell
[Firewall] interface GigabitEthernet 1/0/1  # 配置内部接口IP地址
[Firewall-GigabitEthernet0/0/1] ip address 192.168.1.254 24  # 设置IP地址为192.168.1.254/24
[Firewall-GigabitEthernet0/0/1] quit

[Firewall] interface GigabitEthernet 1/0/2  # 配置DMZ接口IP地址
[Firewall-GigabitEthernet0/0/2] ip address 192.168.2.254 24  # 设置IP地址为192.168.2.254/24
[Firewall-GigabitEthernet0/0/2] quit

[Firewall] interface GigabitEthernet 1/0/3  # 配置出口接口IP地址
[Firewall-GigabitEthernet0/0/3] ip address 100.1.1.1 24  # 设置IP地址为100.1.1.1/24
[Firewall-GigabitEthernet0/0/3] quit

// 将防火墙接口加入相应安全域
[Firewall] firewall zone trust  # 将内部接口加入信任区域
[Firewall-zone-trust] add interface GigabitEthernet1/0/1
[Firewall-zone-trust] quit

[Firewall] firewall zone dmz  # 将DMZ接口加入非军事化区域
[Firewall-zone-dmz] add interface GigabitEthernet1/0/2
[Firewall-zone-dmz] quit

[Firewall] firewall zone untrust  # 将出口接口加入非信任区域
[Firewall-zone-untrust] add interface GigabitEthernet1/0/3
[Firewall-zone-untrust] quit

// 配置安全策略，允许信任区域（192.168.1.0/24）访问Internet
[Firewall] security-policy  # 进入防火墙安全策略配置模式
[Firewall-policy-security] rule name trust_to_untrust  # 创建一个名为trust_to_untrust的安全策略规则
[Firewall-policy-security-rule-trust_to_untrust] source-zone trust  # 设置源区域为信任区域
[Firewall-policy-security-rule-trust_to_untrust] source-address 192.168.1.0 mask 255.255.255.0  # 指定源地址为192.168.1.0/24
[Firewall-policy-security-rule-trust_to_untrust] destination-zone untrust  # 设置目的区域为不可信任区域（即Internet）
[Firewall-policy-security-rule-policy_sec_deny1] action permit  # 允许流量通过
[Firewall-policy-security-rule-policy_sec_deny1] quit  # 退出安全策略规则配置模式

// 配置 NAT 地址池，开启端口转换。
[Firewall] nat address-group addressgroup1  # 进入NAT地址池配置模式并创建名为addressgroup1的地址组
[Firewall-address-group-addressgroup1] mode pat  # 设置地址组模式为PAT（端口地址转换）
[Firewall-address-group-addressgroup1] section 0 100.1.1.10 100.1.1.20  # 指定地址范围为100.1.1.10至100.1.1.20
[Firewall-address-group-addressgroup1] quit  # 退出地址组配置模式

// 配置源 NAT 策略，实现私网指定网段访问 Internet 时自动进行源地址转换。
[Firewall] nat-policy  # 进入NAT策略配置模式
[Firewall-policy-nat] rule name policy_nat1  # 创建一个名为policy_nat1的NAT策略规则
[Firewall-policy-nat-rule-policy_nat1] source-zone trust  # 指定源区域为信任区域
[Firewall-policy-nat-rule-policy_nat1] destination-zone untrust  # 指定目的区域为不可信任区域（Internet）
[Firewall-policy-nat-rule-policy_nat1] source-address 192.168.1.0 24  # 指定源地址为192.168.1.0/24
[Firewall-policy-nat-rule-policy_nat1] destination-address any  # 指定目的地址为任意地址
[Firewall-policy-nat-rule-policy_nat1] action source-nat address-group addressgroup1  # 对源地址进行地址组转换
[Firewall-policy-nat-rule-policy_nat1] quit  # 退出NAT策略规则配置模式

// 放行信任区域到DMZ区域的流量
[Firewall] security-policy  # 进入安全策略配置模式
[Firewall-policy-security] rule name trust_to_dmz  # 创建一个名为trust_to_dmz的安全策略规则
[Firewall-policy-security-rule-trust_to_dmz] source-zone trust  # 指定源区域为信任区域
[Firewall-policy-security-rule-trust_to_dmz] destination-zone dmz  # 指定目的区域为DMZ区域
[Firewall-policy-security-rule-trust_to_dmz] action permit  # 允许流量通过

// 配置 NAT Server 功能，将内网 Web 服务映射到公网地址。
[Firewal] nat server policy_web protocol tcp global 100.1.1.100 80 inside 192.168.2.100 80  # 将内网Web服务（192.168.2.100:80）映射到公网地址100.1.1.100:80

[Firewall] display firewall session table  # 显示防火墙会话表，用于查看当前活动的会话信息
```

### 2. Internet 配置

```shell
[Internet] interface GigabitEthernet 0/0/0  # 配置与Internet连接的接口
[Internet-GigabitEthernet0/0/0] ip address 100.1.1.2 24  # 设置接口IP地址为100.1.1.2/24
[Internet-GigabitEthernet0/0/0] quit
```
