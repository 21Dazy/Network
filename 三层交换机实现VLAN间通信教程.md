# 三层交换机实现VLAN间通信教程（Cisco Packet Tracer 8.2.1模拟环境）

## 目录
1. [实验概述](#实验概述)
2. [实验环境准备](#实验环境准备)
3. [技术原理](#技术原理)
4. [详细实验步骤](#详细实验步骤)
5. [验证配置结果](#验证配置结果)
6. [排除常见问题](#排除常见问题)
7. [扩展知识](#扩展知识)

## 实验概述

### 实验名称
VLAN/802.1Q - VLAN间通信

### 实验目的
通过三层交换机实现不同VLAN之间的互相通信。

### 背景描述
某企业有两个主要部门：销售部和技术部。销售部的计算机系统分散连接在两台交换机上，销售部内部需要相互通信，同时销售部和技术部也需要进行相互通信。我们需要在交换机上做适当配置来实现这一目标。

## 实验环境准备

### 实验设备
- 三层交换机S3760-01（1台）- 作为SwitchA
- 二层交换机S2126G-01（1台）- 作为SwitchB
- 路由器R1762-02（1台）- 模拟销售部PC1
- 路由器R1762-03（1台）- 模拟技术部PC2
- 路由器R2632（1台）- 模拟销售部PC3
- 直连线（3条）

### 实验拓扑
```
                          +------------------+
                          |    路由器R2632   |
                          |   (销售部PC3)    |
                          |   F1/1           |
                          |   192.168.10.2/24|
                          +--------+---------+
                                   |
                                   | 
                                   |
                          +--------+---------+
                          |                  |
               +----------+  二层交换机      |
               |          |  S2126G-01      |
               |F0/1      |  (SwitchB)      |
               |          |                  |
               |          +--------+---------+
               |                   |
               |                   |F0/24 (Trunk)
               |                   |
               |          +--------+---------+
               |          |                  |       +------------------+
               |          |  三层交换机      |F0/5   |    路由器R1762-02|
               |          |  S3760-01       +-------+   (销售部PC1)    |
               |          |  (SwitchA)      |       |   F1/0           |
               |          |  VLAN10 SVI     |       |   192.168.10.1/24|
               |          |  192.168.10.254 |       +------------------+
               |          |  VLAN20 SVI     |
               |          |  192.168.20.254 |       +------------------+
               |          |                  |F0/15  |    路由器R1762-03|
               |          +------------------+-------+   (技术部PC2)    |
                                                     |   F1/0           |
                                                     |   192.168.20.1/24|
                                                     +------------------+
```

### 规划VLAN分配
- VLAN 10: 销售部
  - SwitchA的F0/5端口 - 连接R1762-02(销售部PC1)
  - SwitchB的F0/1端口 - 连接R2632(销售部PC3)
- VLAN 20: 技术部
  - SwitchA的F0/15端口 - 连接R1762-03(技术部PC2)

### 网络地址规划
- 销售部(VLAN 10): 192.168.10.0/24
  - R1762-02 (PC1): 192.168.10.1/24
  - R2632 (PC3): 192.168.10.2/24
  - SVI接口: 192.168.10.254/24 (网关)
- 技术部(VLAN 20): 192.168.20.0/24
  - R1762-03 (PC2): 192.168.20.1/24
  - SVI接口: 192.168.20.254/24 (网关)

## 技术原理

### VLAN间通信原理

1. **VLAN隔离**：
   - 在交换网络中，通过VLAN对一个物理网络进行了逻辑划分
   - 不同VLAN之间在二层是隔离的，无法直接通信
   - 需要三层设备(路由器或三层交换机)才能实现不同VLAN之间的通信

2. **三层交换机的作用**：
   - 三层交换机同时具备二层交换和三层路由能力
   - 能够识别数据包的IP地址，查找路由表进行选路转发
   - 通过SVI(Switched Virtual Interface，交换虚拟接口)实现VLAN间路由

3. **SVI(交换虚拟接口)**：
   - 为VLAN创建的虚拟三层接口
   - 可以配置IP地址，作为该VLAN内所有设备的默认网关
   - 使得不同VLAN之间能够通过IP路由实现通信

4. **直连路由**：
   - 直连路由是指：为三层设备的接口配置IP地址并激活该端口
   - 三层设备会自动产生该接口IP所在网段的直连路由信息
   - 三层交换机通过直连路由实现不同VLAN之间的互访

5. **VLAN间路由过程**：
   - 源主机发送数据包到默认网关(SVI接口)
   - 三层交换机查询路由表，确定目标网络的出口接口
   - 交换机通过出口接口将数据包转发到目标VLAN
   - 目标主机接收数据包

## 详细实验步骤

### 步骤1：搭建网络拓扑

1. 启动Cisco Packet Tracer 8.2.1
2. 添加设备：
   - 放置一台三层交换机(3560或类似型号)，命名为S3760-01(SwitchA)
   - 放置一台二层交换机(2960或类似型号)，命名为S2126G-01(SwitchB)
   - 放置三台路由器模拟PC，分别命名为R1762-02、R1762-03和R2632
3. 使用直连线连接设备：
   - S3760-01的F0/5端口连接到R1762-02的F1/0端口
   - S3760-01的F0/15端口连接到R1762-03的F1/0端口
   - S3760-01的F0/24端口连接到S2126G-01的F0/24端口
   - S2126G-01的F0/1端口连接到R2632的F1/1端口

### 步骤2：在三层交换机S3760-01(SwitchA)上创建VLAN 10并配置端口

1. 连接到三层交换机S3760-01的控制台：
   - 在Packet Tracer中，点击交换机图标
   - 选择"CLI"标签页

2. 进入特权模式：
   ```
   Switch> enable
   ```

3. 进入全局配置模式：
   ```
   Switch# configure terminal
   ```

4. 设置交换机名称：
   ```
   Switch(config)# hostname S3760-01
   ```

5. 创建VLAN 10并命名为sales：
   ```
   S3760-01(config)# vlan 10
   S3760-01(config-vlan)# name sales
   S3760-01(config-vlan)# exit
   ```

6. 将F0/5端口配置为VLAN 10的访问端口：
   ```
   S3760-01(config)# interface fastEthernet 0/5
   S3760-01(config-if)# switchport mode access
   S3760-01(config-if)# switchport access vlan 10
   S3760-01(config-if)# exit
   ```

7. 验证VLAN 10的配置：
   ```
   S3760-01(config)# end
   S3760-01# show vlan id 10
   ```

   您应该看到类似以下的输出：
   ```
   VLAN Name                             Status    Ports
   ---- -------------------------------- --------- -------------------------------
   10   sales                            STATIC    Fa0/5
   ```

### 步骤3：在三层交换机S3760-01上创建VLAN 20并配置端口

1. 进入全局配置模式：
   ```
   S3760-01# configure terminal
   ```

2. 创建VLAN 20并命名为technical：
   ```
   S3760-01(config)# vlan 20
   S3760-01(config-vlan)# name technical
   S3760-01(config-vlan)# exit
   ```

3. 将F0/15端口配置为VLAN 20的访问端口：
   ```
   S3760-01(config)# interface fastEthernet 0/15
   S3760-01(config-if)# switchport mode access
   S3760-01(config-if)# switchport access vlan 20
   S3760-01(config-if)# exit
   ```

4. 验证VLAN 20的配置：
   ```
   S3760-01(config)# end
   S3760-01# show vlan id 20
   ```

   您应该看到类似以下的输出：
   ```
   VLAN Name                             Status    Ports
   ---- -------------------------------- --------- -------------------------------
   20   technical                        STATIC    Fa0/15
   ```

### 步骤4：配置两台交换机之间的Trunk链路

1. 在三层交换机S3760-01上配置F0/24端口为Trunk模式：
   ```
   S3760-01# configure terminal
   S3760-01(config)# interface fastEthernet 0/24
   S3760-01(config-if)# switchport mode trunk
   S3760-01(config-if)# exit
   ```

2. 在二层交换机S2126G-01上配置F0/24端口为Trunk模式：
   - 切换到S2126G-01交换机的控制台
   
   ```
   Switch> enable
   Switch# configure terminal
   Switch(config)# hostname S2126G-01
   S2126G-01(config)# interface fastEthernet 0/24
   S2126G-01(config-if)# switchport mode trunk
   S2126G-01(config-if)# exit
   ```

### 步骤5：在二层交换机S2126G-01上创建VLAN 10并配置端口

1. 创建VLAN 10并命名为sales：
   ```
   S2126G-01(config)# vlan 10
   S2126G-01(config-vlan)# name sales
   S2126G-01(config-vlan)# exit
   ```

2. 将F0/1端口配置为VLAN 10的访问端口：
   ```
   S2126G-01(config)# interface fastEthernet 0/1
   S2126G-01(config-if)# switchport mode access
   S2126G-01(config-if)# switchport access vlan 10
   S2126G-01(config-if)# exit
   ```

3. 验证VLAN 10的配置：
   ```
   S2126G-01(config)# end
   S2126G-01# show vlan id 10
   ```

   您应该看到类似以下的输出：
   ```
   VLAN Name                             Status    Ports
   ---- -------------------------------- --------- -------------------------------
   10   sales                            active    Fa0/1
   ```

### 步骤6：配置路由器(模拟PC)的IP地址

1. 配置路由器R1762-02(销售部PC1)的IP地址和默认路由：
   - 切换到R1762-02路由器的控制台
   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R1762-02
   
   R1762-02(config)# interface fastEthernet 1/0
   R1762-02(config-if)# ip address 192.168.10.1 255.255.255.0
   R1762-02(config-if)# no shutdown
   R1762-02(config-if)# exit
   
   R1762-02(config)# ip route 0.0.0.0 0.0.0.0 fastEthernet 1/0
   R1762-02(config)# exit
   ```

2. 配置路由器R1762-03(技术部PC2)的IP地址和默认路由：
   - 切换到R1762-03路由器的控制台
   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R1762-03
   
   R1762-03(config)# interface fastEthernet 1/0
   R1762-03(config-if)# ip address 192.168.20.1 255.255.255.0
   R1762-03(config-if)# no shutdown
   R1762-03(config-if)# exit
   
   R1762-03(config)# ip route 0.0.0.0 0.0.0.0 fastEthernet 1/0
   R1762-03(config)# exit
   ```

3. 配置路由器R2632(销售部PC3)的IP地址和默认路由：
   - 切换到R2632路由器的控制台
   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R2632
   
   R2632(config)# interface fastEthernet 1/1
   R2632(config-if)# ip address 192.168.10.2 255.255.255.0
   R2632(config-if)# no shutdown
   R2632(config-if)# exit
   
   R2632(config)# ip route 0.0.0.0 0.0.0.0 fastEthernet 1/1
   R2632(config)# exit
   ```

### 步骤7：在三层交换机上启用路由功能并配置SVI接口

1. 切换回三层交换机S3760-01的控制台

2. 确保IP路由功能已开启：
   ```
   S3760-01# configure terminal
   S3760-01(config)# ip routing
   ```

3. 为VLAN 10创建SVI接口并配置IP地址：
   ```
   S3760-01(config)# interface vlan 10
   S3760-01(config-if)# ip address 192.168.10.254 255.255.255.0
   S3760-01(config-if)# no shutdown
   S3760-01(config-if)# exit
   ```

4. 为VLAN 20创建SVI接口并配置IP地址：
   ```
   S3760-01(config)# interface vlan 20
   S3760-01(config-if)# ip address 192.168.20.254 255.255.255.0
   S3760-01(config-if)# no shutdown
   S3760-01(config-if)# exit
   ```

5. 保存配置：
   ```
   S3760-01(config)# end
   S3760-01# write memory
   ```

## 验证配置结果

### 步骤1：验证SVI接口状态

1. 在三层交换机S3760-01上查看IP接口状态：
   ```
   S3760-01# show ip interface brief
   ```

   确认VLAN 10和VLAN 20接口处于"up"状态。

2. 查看详细的接口信息：
   ```
   S3760-01# show ip interface
   ```

   应该能看到VLAN 10和VLAN 20的接口状态为UP，并显示正确的IP地址。

### 步骤2：验证路由表

1. 查看三层交换机的路由表：
   ```
   S3760-01# show ip route
   ```

   应该能看到两个直连网络：
   - 192.168.10.0/24，通过VLAN 10接口直连
   - 192.168.20.0/24，通过VLAN 20接口直连

### 步骤3：测试同一VLAN内的连通性

1. 从R1762-02(PC1)ping R2632(PC3)：
   - 两者都属于VLAN 10(销售部)
   - 切换到R1762-02的控制台
   ```
   R1762-02# ping 192.168.10.2
   ```

   应该看到成功的ping响应：
   ```
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echoes to 192.168.10.2, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
   ```

### 步骤4：测试不同VLAN之间的连通性

1. 从R1762-02(PC1, VLAN 10)ping R1762-03(PC2, VLAN 20)：
   ```
   R1762-02# ping 192.168.20.1
   ```

   应该看到成功的ping响应：
   ```
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echoes to 192.168.20.1, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
   ```

2. 从R2632(PC3, VLAN 10)ping R1762-03(PC2, VLAN 20)：
   - 切换到R2632的控制台
   ```
   R2632# ping 192.168.20.1
   ```

   应该看到成功的ping响应：
   ```
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echoes to 192.168.20.1, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
   ```

### 步骤5：检查交换机配置

1. 查看三层交换机S3760-01的配置：
   ```
   S3760-01# show running-config
   ```

   重点关注：
   - VLAN配置
   - 端口VLAN分配
   - SVI接口配置
   - IP路由功能是否开启

2. 查看二层交换机S2126G-01的配置：
   ```
   S2126G-01# show running-config
   ```

   重点关注：
   - VLAN配置
   - 端口VLAN分配
   - Trunk端口配置

## 排除常见问题

如果遇到VLAN间通信问题，请检查以下几个方面：

### 1. 检查物理连接

- 确认所有设备之间的连接线缆正确连接到指定端口
- 在Packet Tracer中，确认连接线显示为绿色（表示链路正常）

### 2. 检查VLAN配置

- 确认两台交换机上都创建了相应的VLAN
- 使用`show vlan`命令验证端口是否正确分配到指定VLAN
- 使用`show interfaces trunk`命令验证Trunk链路状态

### 3. 检查三层交换机的路由功能

- 确认`ip routing`命令已配置
- 使用`show ip route`命令检查路由表是否包含到各VLAN网段的路由

### 4. 检查SVI接口状态

- 使用`show ip interface brief`命令检查VLAN接口状态
- 确保所有SVI接口显示为"up/up"状态
- 如果接口显示为"down"，检查相应VLAN是否存在并且有至少一个活动端口

### 5. 检查PC配置

- 确认所有PC(路由器)配置了正确的IP地址和子网掩码
- 确认默认网关配置正确（指向相应VLAN的SVI接口地址）
- 使用`ping`命令测试到网关的连通性

### 6. 重启问题接口

如果特定接口有问题，可以尝试重启该接口：
```
interface vlan 10
shutdown
no shutdown
```

## 扩展知识

### 1. VLAN间路由的方法比较

| 方法 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| 路由器单臂路由 | 配置简单，设备成本低 | 端口利用率低，性能受限 | 小型网络，流量较小 |
| 三层交换机SVI | 性能高，路由在硬件中完成 | 设备成本较高 | 中大型网络，需要高性能 |
| 多路由器多接口 | 隔离性好，安全性高 | 配置复杂，成本高 | 安全要求高的网络 |

### 2. VLAN接口状态控制

VLAN接口(SVI)的状态取决于几个因素：

1. VLAN接口的管理状态（由`shutdown`/`no shutdown`命令控制）
2. VLAN本身是否存在（在VLAN数据库中定义）
3. 该VLAN中是否有至少一个处于up状态的接口

如果要使VLAN接口处于up状态，需要：
- 配置`no shutdown`
- 确保VLAN在VLAN数据库中存在
- 确保该VLAN中至少有一个活动端口

### 3. 使用ACL限制VLAN间访问

可以在SVI接口上配置访问控制列表(ACL)来限制VLAN间的访问：

```
S3760-01(config)# access-list 101 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
S3760-01(config)# access-list 101 permit ip any any
S3760-01(config)# interface vlan 10
S3760-01(config-if)# ip access-group 101 out
```

这将阻止VLAN 10中的设备访问VLAN 20的设备。

### 4. 使用策略路由控制VLAN间通信路径

可以使用策略路由(PBR)来影响VLAN间通信的路径和行为：

```
S3760-01(config)# access-list 102 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
S3760-01(config)# route-map VLAN_POLICY permit 10
S3760-01(config-route-map)# match ip address 102
S3760-01(config-route-map)# set ip next-hop 192.168.30.1
S3760-01(config-route-map)# exit
S3760-01(config)# interface vlan 10
S3760-01(config-if)# ip policy route-map VLAN_POLICY
```

这将使VLAN 10到VLAN 20的流量通过192.168.30.1这个下一跳。

---

通过本教程，您已经学习了如何配置三层交换机实现不同VLAN之间的通信。您创建了VLAN、配置了Trunk链路、设置了SVI接口，并最终实现了不同VLAN间的互访。这些技能在企业网络环境中非常重要，能帮助您构建高效、安全的分段网络。 