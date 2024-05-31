# VRRP 配置

## 网络拓扑与说明

![img](image/SCR-hky.png)

以下是你的学习笔记，我已经根据你提供的内容进行了重写和注释：

## 配置

### 1. 接入交换机 acsw 配置（vlan）

```shell
<Huawei> system-view // 进入系统视图
[Huawei] sysname acsw // 设置设备名称为 acsw

[acsw] vlan batch 10 // 配置 VLAN 10
[acsw] interface GigabitEthernet 0/0/1 // 配置端口 0/0/1 为 trunk 模式，并允许通过所有 VLAN
[acsw-GigabitEthernet0/0/1] port link-type trunk // 设置端口为 trunk 模式
[acsw-GigabitEthernet0/0/1] port trunk allow-pass vlan all // 允许通过所有 VLAN
[acsw-GigabitEthernet0/0/1] quit // 退出端口配置模式

[acsw] interface GigabitEthernet 0/0/2 // 配置端口 0/0/2 为 trunk 模式，并允许通过所有 VLAN
[acsw-GigabitEthernet0/0/2] port link-type trunk // 设置端口为 trunk 模式
[acsw-GigabitEthernet0/0/2] port trunk allow-pass vlan all // 允许通过所有 VLAN
[acsw-GigabitEthernet0/0/2] quit // 退出端口配置模式

[acsw] interface GigabitEthernet 0/0/3 // 配置端口 0/0/3 为 access 模式，加入 VLAN 10
[acsw-GigabitEthernet0/0/3] port link-type access // 设置端口为 access 模式
[acsw-GigabitEthernet0/0/3] port default vlan 10 // 设置端口默认 VLAN 为 10
[acsw-GigabitEthernet0/0/3] quit // 退出端口配置模式
```

### 2. 核心交换机 coresw1 配置

```shell
<Huawei> system-view
[Huawei] sysname coresw1

// 配置 VLAN 10 和 VLAN 100
[coresw1] vlan batch 10 100
// 配置端口 0/0/2 为 access 模式，加入 VLAN 100
[coresw1] interface GigabitEthernet 0/0/2
[coresw1-GigabitEthernet0/0/2] port link-type access
[coresw1-GigabitEthernet0/0/2] port default vlan 100
[coresw1-GigabitEthernet0/0/2] quit
// 配置端口 0/0/1 为 trunk 模式，并允许通过所有 VLAN
[coresw1] interface GigabitEthernet 0/0/1
[coresw1-GigabitEthernet0/0/1] port link-type trunk
[coresw1-GigabitEthernet0/0/1] port trunk allow-pass vlan all
[coresw1-GigabitEthernet0/0/1] quit
// 配置端口 0/0/3 为 trunk 模式，并允许通过所有 VLAN
[coresw1] interface GigabitEthernet 0/0/3
[coresw1-GigabitEthernet0/0/3] port link-type trunk
[coresw1-GigabitEthernet0/0/3] port trunk allow-pass vlan all
[coresw1-GigabitEthernet0/0/3] quit

// 配置 VLANIF 接口并设置 IP 地址
[coresw1] interface Vlanif 10 // 配置 VLANIF 接口并设置 IP 地址
[coresw1-Vlanif10] ip address 192.168.10.252 24 // 设置 VLANIF 10 的 IP 地址
[coresw1-Vlanif10] quit // 退出 VLANIF 接口配置模式
[coresw1] interface Vlanif 100 // 配置 VLANIF 接口并设置 IP 地址
[coresw1-Vlanif100] ip address 192.168.100.1 30 // 设置 VLANIF 100 的 IP 地址
[coresw1-Vlanif100] quit // 退出 VLANIF 接口配置模式

// 配置静态路由
[coresw1] ip route-static 0.0.0.0 0 192.168.100.2

// 配置 VRRP 主节点
[coresw1] interface Vlanif 10
[coresw1-Vlanif10] vrrp vrid 10 virtual-ip 192.168.10.254 // 配置 VRRP 虚拟路由器 ID 为 10 的虚拟 IP 地址
[coresw1-Vlanif10] vrrp vrid 10 priority 120 // 配置 VRRP 虚拟路由器 ID 为 10 的优先级为 120
[coresw1-Vlanif10] vrrp vrid 10 preempt-mode timer delay 20 // 配置 VRRP 虚拟路由器 ID 为 10 的抢占模式为定时器模式，延迟时间为 20 秒
[coresw1-Vlanif10] quit // 退出 VLANIF 接口配置模式
```

### 3. 核心交换机 coresw2 配置

```shell
<Huawei> system-view
[Huawei] sysname coresw2

// 配置 VLAN 10 和 VLAN 200
[coresw2] vlan batch 10 200
// 配置端口 0/0/2 为 access 模式，加入 VLAN 200
[coresw2] interface GigabitEthernet 0/0/2
[coresw2-GigabitEthernet0/0/2] port link-type access
[coresw2-GigabitEthernet0/0/2] port default vlan 200
[coresw2-GigabitEthernet0/0/2] quit
// 配置端口 0/0/1 为 trunk 模式，并允许通过所有 VLAN
[coresw2] interface GigabitEthernet 0/0/1
[coresw2-GigabitEthernet0/0/1] port link-type trunk
[coresw2-GigabitEthernet0/0/1] port trunk allow-pass vlan all
[coresw2-GigabitEthernet0/0/1] quit
// 配置端口 0/0/3 为 trunk 模式，并允许通过所有 VLAN
[coresw2] interface GigabitEthernet 0/0/3
[coresw2-GigabitEthernet0/0/3] port link-type trunk
[coresw2-GigabitEthernet0/0/3] port trunk allow-pass vlan all
[coresw2-GigabitEthernet0/0/3] quit

// 配置 VLANIF 接口并设置 IP 地址
[coresw2] interface Vlanif 10 // 配置 VLANIF 接口并设置 IP 地址
[coresw2-Vlanif10] ip address 192.168.10.253 24 // 设置 VLANIF 10 的 IP 地址
[coresw2-Vlanif10] quit // 退出 VLANIF 接口配置模式
[coresw2] interface Vlanif 200 // 配置 VLANIF 接口并设置 IP 地址
[coresw2-Vlanif200] ip address 192.168.200.1 30 // 设置 VLANIF 200 的 IP 地址
[coresw2-Vlanif200] quit // 退出 VLANIF 接口配置模式

// 配置静态路由
[coresw2] ip route-static 0.0.0.0 0 192.168.200.2

// 配置 VRRP 备份节点
[coresw2] interface Vlanif 10 // 配置 VRRP 备份节点
[coresw2-Vlanif10] vrrp vrid 10 virtual-ip 192.168.10.254 // 配置 VRRP 虚拟路由器 ID 为 10 的虚拟 IP 地址
[coresw2-Vlanif10] vrrp vrid 10 priority 100 // 默认 100
[coresw2-Vlanif10] quit // 退出 VLANIF 接口配置模式
```

### 4. 出口路由 router 配置

```shell
<Huawei> system-view // 进入系统视图
[Huawei] sysname router // 设置设备名称为 router

// 配置接口 IP 地址
[router] interface GigabitEthernet 0/0/0 // 配置接口 IP 地址
[router-GigabitEthernet0/0/0] ip address 100.1.1.2 30 // 设置接口 GigabitEthernet 0/0/0 的 IP 地址为 100.1.1.2，子网掩码为 30
[router-GigabitEthernet0/0/0] quit // 退出接口配置模式
[router] interface GigabitEthernet 0/0/1 // 配置接口 IP 地址
[router-GigabitEthernet0/0/1] ip address 192.168.100.2 30 // 设置接口 GigabitEthernet 0/0/1 的 IP 地址为 192.168.100.2，子网掩码为 30
[router-GigabitEthernet0/0/1] quit // 退出接口配置模式
[router] interface GigabitEthernet 0/0/2 // 配置接口 IP 地址
[router-GigabitEthernet0/0/2] ip address 192.168.200.2 30 // 设置接口 GigabitEthernet 0/0/2 的 IP 地址为 192.168.200.2，子网掩码为 30
[router-GigabitEthernet0/0/2] quit // 退出接口配置模式

// 配置静态路由
[router] ip route-static 0.0.0.0 0 100.1.1.1 // 配置静态路由
[router] ip route-static 192.168.10.0 24 192.168.100.1 // 配置静态路由，将目的网络 192.168.10.0/24 下一跳设为 192.168.100.1
[router] ip route-static 192.168.10.0 24 192.168.200.1 // 配置静态路由，将目的网络 192.168.10.0/24 下一跳设为 192.168.200.1
```

### 5. internet 配置

```shell
[internet] ip router-static 0.0.0.0 0 100.1.1.2 // 配置默认路由，将所有流量发送到 IP 地址 100.1.1.2
```

这些已经重写并添加了注释，如果你还需要进一步的帮助或者有其他问题，随时告诉我！
