# Telnet 配置

## 需求

1. 远程登录设备需使用 telnet。
2. 使用本地 AAA 认证，用户名为 admin123，密码为 admin123。
3. 限制仅允许 192.168.1.254 地址的 telnet 登录。

## 配置

### 创建 VLAN

```shell
<Huawei> system-view
[Huawei] vlan 100
[Huawei-vlan100] quit
[Huawei] interface GigabitEthernet 0/0/1
[Huawei-GigabitEthernet0/0/1] port link-type access // 设置端口为接入模式
[Huawei-GigabitEthernet0/0/1] port default vlan 100 // 将默认 VLAN 设置为 100
[Huawei-GigabitEthernet0/0/1] quit
[Huawei] interface Vlanif 100
[Huawei-Vlanif100] ip address 192.168.1.254 24 // 设置 VLAN 接口 IP 地址
[Huawei-Vlanif100] quit
```

### 开启设备 Telnet 服务

```shell
[Huawei] telnet server enable // 开启 Telnet 服务
[Huawei] user-interface vty 0 4 // 配置 VTY 用户接口范围为 0 到 4
[Huawei-ui-vty0-4] protocol inbound telnet // 允许 Telnet 协议登录
[Huawei-ui-vty0-4] quit
```

#### 1. AAA 认证

```shell
[Huawei] user-interface vty 0 4
[Huawei-ui-vty0-4] authentication-mode aaa // 设置 AAA 认证模式
[Huawei-ui-vty0-4] quit
[Huawei] aaa
[Huawei-aaa] local-user admin123 password irreversible-cipher admin123 // 设置本地用户 admin123 的密码
[Huawei-aaa] local-user admin123 service-type telnet // 指定用户服务类型为 Telnet
[Huawei-aaa] local-user admin123 privilege level 15 // 设置用户权限等级为最高
[Huawei-aaa] quit
```

#### 2. Password 认证

```shell
[Huawei] user-interface vty 0 4
[Huawei-ui-vty0-4] authentication-mode password // 设置密码认证模式
[Huawei-ui-vty0-4] set authentication password cipher admin123 // 设置登录密码
```

### ACL 限制用户登录

```shell
[Huawei] acl 2000 // 创建 ACL 2000
[Huawei-acl-basic-2000] rule 5 deny source 192.168.1.254 0.0.0.0 // 拒绝源地址为 192.168.1.254 的访问
[Huawei-acl-basic-2000] quit
[Huawei] user-interface vty 0 4
[Huawei-ui-vty0-4] acl 2000 inbound // 将 ACL 2000 应用于入方向
```

