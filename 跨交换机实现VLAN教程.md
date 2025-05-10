# 跨交换机实现VLAN教程（Cisco Packet Tracer 8.2.1模拟环境）

## 目录
1. [实验概述](#实验概述)
2. [实验环境准备](#实验环境准备)
3. [VLAN跨交换机技术原理](#VLAN跨交换机技术原理)
4. [详细实验步骤](#详细实验步骤)
5. [故障排除与问题解决](#故障排除与问题解决)
6. [验证配置结果](#验证配置结果)
7. [常见问题和注意事项](#常见问题和注意事项)
8. [扩展知识](#扩展知识)

## 实验概述

### 实验名称
跨交换机实现VLAN

### 实验目的
理解并掌握跨交换机之间VLAN的配置方法和特点。

### 背景描述
某企业有两个主要部门：销售部和技术部，其中销售部门的个人计算机系统分散连接在两台交换机上，他们之间需要相互进行通信，但为了数据安全起见，销售部和技术部需要进行相互隔离，需要在交换机上做适当配置来实现这一目标。

## 实验环境准备

### 实验拓扑
```
                      Trunk链路
   +-------------+    F0/24    +-------------+
   |             |-------------|             |
   |  交换机1    |             |   交换机2   |
   | S3760-01    |             |  S2126G-01  |
   |             |             |             |
   +-------------+             +-------------+
          |                           |
          | F0/5                      | F0/1
          |                           |
     +---------+                 +---------+
     |  R1762-02|                 |  R2632  |
     | 192.168.10.1|               | 192.168.10.2|
     +---------+                 +---------+
     (销售部PC1)                (销售部PC2)
          
          | F0/15    
          |          
     +---------+      
     |  R1762-03|      
     | 192.168.10.11|    
     +---------+      
     (技术部PC1)                
```

### 实验设备
- 交换机1：S3760-01（1台）
- 交换机2：S2126G-01（1台）
- 主机：R1762-02、R1762-03、R2632（3台路由器模拟PC）
- 直连线（4条）

## VLAN跨交换机技术原理

当需要在多台交换机间实现VLAN时，需要使用Trunk（干道）链路连接交换机，Trunk链路可以传输多个VLAN的数据。

关键概念：

1. **VLAN标签（Tag）**：
   - 遵循IEEE 802.1Q协议标准
   - 在以太网帧中添加4字节的标签信息，标识该帧属于哪个VLAN
   - 使交换机能够识别并正确转发不同VLAN的数据

2. **端口类型**：
   - **Access端口**：只属于一个VLAN，连接终端设备，不添加VLAN标签
   - **Trunk端口**：允许多个VLAN通过，连接交换机与交换机，添加VLAN标签

3. **VLAN传播**：
   - VLAN信息需要在交换机之间手动配置一致
   - 通过Trunk链路，同一VLAN的设备可以跨交换机通信
   - 不同VLAN的设备即使连接到不同交换机，也保持隔离状态

## 详细实验步骤

### 步骤1：创建拓扑

1. 启动Cisco Packet Tracer
2. 添加2台交换机（S3760-01和S2126G-01）
3. 添加3台路由器（R1762-02、R1762-03、R2632）作为PC使用
4. 使用直连线连接设备：
   - R1762-02连接到S3760-01的F0/5端口
   - R1762-03连接到S3760-01的F0/15端口
   - R2632连接到S2126G-01的F0/1端口
   - S3760-01的F0/24端口连接到S2126G-01的F0/24端口

### 步骤2：在交换机S3760-01上创建VLAN并分配端口

1. 配置销售部VLAN 10：
   ```
   S3760-01>enable
   S3760-01#configure terminal
   S3760-01(config)#vlan 10
   S3760-01(config-vlan)#name sales
   S3760-01(config-vlan)#exit
   ```

2. 将端口F0/5分配到VLAN 10：
   ```
   S3760-01(config)#interface fastEthernet 0/5
   S3760-01(config-if)#switchport mode access
   S3760-01(config-if)#switchport access vlan 10
   S3760-01(config-if)#exit
   ```

3. 配置技术部VLAN 20：
   ```
   S3760-01(config)#vlan 20
   S3760-01(config-vlan)#name technical
   S3760-01(config-vlan)#exit
   ```

4. 将端口F0/15分配到VLAN 20：
   ```
   S3760-01(config)#interface fastEthernet 0/15
   S3760-01(config-if)#switchport mode access
   S3760-01(config-if)#switchport access vlan 20
   S3760-01(config-if)#exit
   ```

5. 查看VLAN配置情况：
   ```
   S3760-01#show vlan brief
   ```

### 步骤3：在交换机S2126G-01上创建VLAN并分配端口

1. 配置销售部VLAN 10：
   ```
   S2126G-01>enable
   S2126G-01#configure terminal
   S2126G-01(config)#vlan 10
   S2126G-01(config-vlan)#name sales
   S2126G-01(config-vlan)#exit
   ```

2. 将端口F0/1分配到VLAN 10：
   ```
   S2126G-01(config)#interface fastEthernet 0/1
   S2126G-01(config-if)#switchport mode access
   S2126G-01(config-if)#switchport access vlan 10
   S2126G-01(config-if)#exit
   ```

3. 查看VLAN配置情况：
   ```
   S2126G-01#show vlan brief
   ```

### 步骤4：配置两台交换机之间的Trunk链路

1. 配置S3760-01的Trunk端口：
   ```
   S3760-01#configure terminal
   S3760-01(config)#interface fastEthernet 0/24
   S3760-01(config-if)#switchport mode trunk
   S3760-01(config-if)#exit
   ```

2. 配置S2126G-01的Trunk端口：
   ```
   S2126G-01#configure terminal
   S2126G-01(config)#interface fastEthernet 0/24
   S2126G-01(config-if)#switchport mode trunk
   S2126G-01(config-if)#exit
   ```

### 步骤5：配置三台"PC"（路由器）的IP地址

1. 配置R1762-02（销售部PC1）：
   ```
   R1762-02>enable
   R1762-02#configure terminal
   R1762-02(config)#interface fastEthernet 1/0
   R1762-02(config-if)#ip address 192.168.10.1 255.255.255.0
   R1762-02(config-if)#no shutdown
   R1762-02(config-if)#exit
   ```

2. 配置R1762-03（技术部PC1）：
   ```
   R1762-03>enable
   R1762-03#configure terminal
   R1762-03(config)#interface fastEthernet 1/0
   R1762-03(config-if)#ip address 192.168.10.11 255.255.255.0
   R1762-03(config-if)#no shutdown
   R1762-03(config-if)#exit
   ```

3. 配置R2632（销售部PC2）：
   ```
   R2632>enable
   R2632#configure terminal
   R2632(config)#interface fastEthernet 1/1
   R2632(config-if)#ip address 192.168.10.2 255.255.255.0
   R2632(config-if)#no shutdown
   R2632(config-if)#exit
   ```

## 故障排除与问题解决

如果设备无法ping通，请检查以下几个方面：

### 1. 检查IP地址配置

确保所有设备的IP地址正确配置：
```
R1762-02#show ip interface brief
R1762-03#show ip interface brief
R2632#show ip interface brief
```

### 2. 检查VLAN配置

查看两台交换机上的VLAN配置：
```
S3760-01#show vlan brief
S2126G-01#show vlan brief
```

确保：
- VLAN 10已在两台交换机上创建
- R1762-02连接的F0/5端口分配到VLAN 10
- R2632连接的F0/1端口分配到VLAN 10
- R1762-03连接的F0/15端口分配到VLAN 20

### 3. 检查Trunk配置

查看两台交换机的Trunk端口配置：
```
S3760-01#show interfaces fastEthernet 0/24 switchport
S2126G-01#show interfaces fastEthernet 0/24 switchport
```

确保：
- 两端口的模式都是"trunk"
- 允许传输的VLAN包含VLAN 10和VLAN 20
- 原生VLAN匹配（通常是VLAN 1）

### 4. 常见解决方案

1. **问题：Trunk端口配置不正确**

   解决方法：确保两端都配置为trunk模式，并且允许所有VLAN通过：
   ```
   S3760-01(config)#interface fastEthernet 0/24
   S3760-01(config-if)#switchport mode trunk
   S3760-01(config-if)#switchport trunk allowed vlan all
   
   S2126G-01(config)#interface fastEthernet 0/24
   S2126G-01(config-if)#switchport mode trunk
   S2126G-01(config-if)#switchport trunk allowed vlan all
   ```

2. **问题：VLAN不一致**

   解决方法：确保两台交换机上的VLAN ID和名称一致：
   ```
   S3760-01#show vlan id 10
   S2126G-01#show vlan id 10
   ```

3. **问题：端口状态问题**

   解决方法：检查端口是否处于up状态：
   ```
   S3760-01#show interfaces status
   S2126G-01#show interfaces status
   ```

4. **问题：IP地址错误**
   
   在您的实验中，IP地址可能有错误。确保所有设备使用正确的IP地址：
   ```
   R1762-02(config)#interface fastEthernet 1/0
   R1762-02(config-if)#ip address 192.168.10.1 255.255.255.0

   R1762-03(config)#interface fastEthernet 1/0
   R1762-03(config-if)#ip address 192.168.10.11 255.255.255.0

   R2632(config)#interface fastEthernet 1/1
   R2632(config-if)#ip address 192.168.10.2 255.255.255.0
   ```

## 验证配置结果

### 1. 验证交换机VLAN配置

查看S3760-01上的VLAN配置：
```
S3760-01#show vlan brief
```

应看到类似以下输出：
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, ...(其他端口)
10   sales                            active    Fa0/5
20   technical                        active    Fa0/15
```

查看S2126G-01上的VLAN配置：
```
S2126G-01#show vlan brief
```

应看到类似以下输出：
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/3, ...(其他端口)
10   sales                            active    Fa0/1
```

### 2. 验证Trunk端口配置

查看S3760-01上的Trunk端口：
```
S3760-01#show interfaces fastEthernet 0/24 switchport
```

应看到类似以下输出：
```
Name: Fa0/24
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Trunking VLANs Enabled: ALL
```

### 3. 验证连通性

从销售部PC2 (R2632) 尝试ping销售部PC1 (R1762-02)：
```
R2632#ping 192.168.10.1
```

应该成功：
```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/10 ms
```

从销售部PC2 (R2632) 尝试ping技术部PC1 (R1762-03)：
```
R2632#ping 192.168.10.11
```

应该失败（因为不同VLAN）：
```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.11, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

## 常见问题和注意事项

1. **交换机间的Trunk配置**：
   - 两台交换机连接的端口必须都配置为Trunk模式
   - 确保Trunk端口允许所需VLAN通过

2. **VLAN一致性**：
   - 两台交换机上的VLAN ID和名称应保持一致
   - 端口VLAN分配要与预期的网络设计匹配

3. **IP地址配置**：
   - 同一VLAN中的设备应使用相同网段的IP地址
   - 不同VLAN中的设备即使IP地址相同也无法直接通信

4. **隔离验证**：
   - 不同VLAN之间互相ping应该失败，这证明隔离有效
   - 同一VLAN的设备即使连接在不同交换机上也应能互相通信

5. **注意事项**：
   - Trunk接口在默认情况下允许所有VLAN通过
   - 可以使用`switchport trunk allowed vlan`命令限制特定VLAN通过
   - 原生VLAN上的流量不会打标签，确保两端配置一致

## 扩展知识

### VLAN标准和兼容性

1. **IEEE 802.1Q**：
   - 行业标准的VLAN标记协议
   - 允许多个VLAN的流量在同一物理链路上传输
   - 通过在以太网帧中添加标签来实现VLAN隔离

2. **ISL（Inter-Switch Link）**：
   - Cisco专有的VLAN封装协议
   - 现代网络中已较少使用，大多采用802.1Q

### VLAN间路由

如果需要不同VLAN间通信，可以采用以下方法：

1. **三层交换机**：
   - 配置SVI（交换虚拟接口）
   - 启用IP路由功能
   - 为每个VLAN配置一个SVI接口作为网关

2. **Router-on-a-Stick**：
   - 使用路由器通过单个物理接口连接到交换机的Trunk端口
   - 在路由器上配置子接口，每个子接口对应一个VLAN
   - 为每个子接口配置不同的IP地址作为相应VLAN的网关

### VTP（VLAN中继协议）

VTP可以帮助在多台交换机之间同步VLAN配置：

1. **VTP模式**：
   - Server：创建、修改和删除VLAN，并将更改通告给其他交换机
   - Client：接收并应用来自服务器的VLAN更改，不能本地修改VLAN
   - Transparent：不参与VTP，但会转发VTP通告

2. **VTP配置**：
   ```
   Switch(config)#vtp domain domain-name
   Switch(config)#vtp password password
   Switch(config)#vtp mode {server|client|transparent}
   ```

3. **VTP版本**：
   - VTP v1和v2对VLAN 1-1005有效
   - VTP v3增加了对扩展VLAN (1006-4094)的支持

---

通过本教程，您已经学习了如何在Cisco Packet Tracer中配置跨交换机VLAN，以及如何排查和解决常见的连接问题。正确配置后，同一VLAN的设备应该能够跨交换机通信，而不同VLAN的设备应该保持隔离。 