# RIP V2路由协议配置教程（Cisco Packet Tracer 8.2.1模拟环境）

## 目录
1. [实验概述](#实验概述)
2. [实验环境准备](#实验环境准备)
3. [RIP协议技术原理](#RIP协议技术原理)
4. [详细实验步骤](#详细实验步骤)
5. [验证配置结果](#验证配置结果)
6. [排除常见问题](#排除常见问题)
7. [扩展知识](#扩展知识)

## 实验概述

### 实验名称
RIP路由协议——RIP V2配置

### 实验目的
掌握在路由器上配置RIP V2路由协议，实现不同网段之间的自动路由学习与数据转发。

### 背景描述
假设校园网通过1台三层交换机连到校园网出口路由器，路由器再和校园外的另1台路由器连接，现做适当配置，实现校园网内部主机与校园网外部主机的相互通信。

## 实验环境准备

### 实验设备
- 三层交换机S3760-01（1台）- 校园内网核心交换机
- 路由器R2632（1台）- 校园出口路由器
- 路由器R1762-01（1台）- 校园外路由器
- 路由器R1762-02（1台）- 模拟校园内PC1
- 交换机S2126G-02（1台）- 模拟校园外PC2
- V35串行线缆（1条）- 连接两台路由器（DCE端连接R1762-01）
- 直连线或交叉线（3条）- 连接其他设备

### 实验拓扑
```
+-----------------+                +------------------+                +-----------------+
|  R1762-02(PC1)  |                |    三层交换机     |                |     路由器      |
|  F1/0           |----------------|    S3760-01      |----------------|     R2632       |
| 172.16.5.11/24  |    直连线      |    VLAN 50       |    直连线      |     F1/0        |
| 网关:172.16.5.1 |                | 172.16.5.1/24    |                | 172.16.1.1/24   |
+-----------------+                |                  |                |                 |
                                   |    VLAN 10       |                |     S1/2        |
                                   | 172.16.1.2/24    |                | 172.16.2.1/24   |
                                   +------------------+                +---------+-------+
                                                                                 |
                                                                                 | V35线缆
                                                                                 |
                                                                       +---------+-------+
                                                                       |     路由器      |
                                                                       |    R1762-01     |
                                                                       |     S1/2(DCE)   |
                                                                       | 172.16.2.2/24   |
                                                                       |                 |
                                                                       |     F1/1        |
                                                                       | 172.16.3.1/24   |
                                                                       +---------+-------+
                                                                                 |
                                                                                 | 直连线
                                                                                 |
                                                                       +---------+-------+
                                                                       |  S2126G-02(PC2) |
                                                                       |  VLAN1接口      |
                                                                       | 172.16.3.22/24  |
                                                                       | 网关:172.16.3.1 |
                                                                       +-----------------+
```

### 网络地址规划
1. 校园内PC1网段：172.16.5.0/24
   - 三层交换机S3760-01的VLAN50接口：172.16.5.1/24（网关）
   - PC1(R1762-02)的F1/0接口：172.16.5.11/24

2. 三层交换机到校园出口路由器：172.16.1.0/24
   - 三层交换机S3760-01的VLAN10接口：172.16.1.2/24
   - 校园出口路由器R2632的F1/0接口：172.16.1.1/24

3. 校园路由器到校外路由器：172.16.2.0/24
   - 校园出口路由器R2632的S1/2接口：172.16.2.1/24
   - 校外路由器R1762-01的S1/2(DCE)接口：172.16.2.2/24

4. 校外PC2网段：172.16.3.0/24
   - 校外路由器R1762-01的F1/1接口：172.16.3.1/24（网关）
   - PC2(S2126G-02)的VLAN1接口：172.16.3.22/24

## RIP协议技术原理

### RIP协议概述
1. **RIP(Routing Information Protocol，路由信息协议)**：
   - 是一种应用较早、较为简单的内部网关协议(IGP)
   - 适用于小型同类网络环境
   - 属于距离矢量(Distance Vector)路由协议
   - 使用跳数(Hop Count)作为度量值(Metric)

2. **RIP协议特点**：
   - 使用UDP端口520进行消息传递
   - 以广播或组播方式发送路由更新
   - 默认30秒发送一次完整路由更新
   - 最大跳数限制为15跳（16跳表示不可达）
   - 支持水平分割、路由毒化和触发更新等稳定性机制

### RIP V1与RIP V2的区别
1. **RIP V1**：
   - 有类路由协议，不支持VLSM(变长子网掩码)
   - 以广播方式(255.255.255.255)发送路由更新
   - 不支持路由认证
   - 不携带子网掩码信息

2. **RIP V2**：
   - 无类路由协议，支持VLSM(变长子网掩码)
   - 以组播方式(224.0.0.9)发送路由更新
   - 支持明文和MD5认证
   - 路由更新中携带子网掩码信息
   - 支持路由汇总和关闭自动汇总功能(no auto-summary)

### RIP协议的工作原理
1. **路由表初始化**：
   - 路由器启动时，仅知道直连网络
   - 通过RIP进程将直连网络加入RIP路由表

2. **路由更新过程**：
   - 周期性发送更新（默认30秒）
   - 接收相邻路由器的路由更新
   - 根据接收到的信息更新自己的路由表

3. **路由选择原则**：
   - 选择跳数最少的路径
   - 如果跳数相同，保持当前路由
   - 最大跳数为15，超过15跳的网络不可达

4. **路由收敛**：
   - 网络拓扑变化时，通过路由更新逐渐达到一致状态
   - 使用触发更新加速收敛
   - 使用路由毒化、水平分割等机制防止路由环路

## 详细实验步骤

### 步骤1：创建实验拓扑

1. 启动Cisco Packet Tracer 8.2.1
2. 添加设备：
   - 放置一台三层交换机（3560或类似型号），命名为S3760-01
   - 放置两台路由器（1841或类似型号），分别命名为R2632和R1762-01
   - 再放置一台路由器(R1762-02)和一台交换机(S2126G-02)模拟PC
3. 连接设备：
   - 使用直连线连接R1762-02(PC1)的F1/0和S3760-01的F0/5
   - 使用直连线连接S3760-01的F0/1和R2632的F1/0
   - 使用V35串行线缆连接R2632的S1/2和R1762-01的S1/2（注意：DCE端连接到R1762-01）
   - 使用直连线连接R1762-01的F1/1和S2126G-02

### 步骤2：配置三层交换机S3760-01

1. 连接到S3760-01的控制台：
   - 在Packet Tracer中，点击交换机图标
   - 选择"CLI"标签页

2. 进入特权模式和全局配置模式：
   ```
   Switch> enable
   Switch# configure terminal
   Switch(config)# hostname S3760-01
   ```

3. 创建VLAN：
   ```
   S3760-01(config)# vlan 10
   S3760-01(config-vlan)# exit
   S3760-01(config)# vlan 50
   S3760-01(config-vlan)# exit
   ```

4. 将接口分配到相应VLAN：
   ```
   S3760-01(config)# interface fastEthernet 0/1
   S3760-01(config-if)# switchport access vlan 10
   S3760-01(config-if)# exit
   
   S3760-01(config)# interface fastEthernet 0/5
   S3760-01(config-if)# switchport access vlan 50
   S3760-01(config-if)# exit
   ```

5. 配置VLAN10的SVI接口（连接校园出口路由器）：
   ```
   S3760-01(config)# interface vlan 10
   S3760-01(config-if)# ip address 172.16.1.2 255.255.255.0
   S3760-01(config-if)# no shutdown
   S3760-01(config-if)# exit
   ```

6. 配置VLAN50的SVI接口（连接校园内PC）：
   ```
   S3760-01(config)# interface vlan 50
   S3760-01(config-if)# ip address 172.16.5.1 255.255.255.0
   S3760-01(config-if)# no shutdown
   S3760-01(config-if)# exit
   ```

7. 开启IP路由功能：
   ```
   S3760-01(config)# ip routing
   ```

8. 验证VLAN配置：
   ```
   S3760-01(config)# end
   S3760-01# show vlan
   ```

   您应该能看到VLAN 10和VLAN 50的信息，以及对应的端口分配。

### 步骤3：配置校园出口路由器R2632

1. 连接到R2632的控制台：
   - 在Packet Tracer中，点击路由器图标
   - 选择"CLI"标签页

2. 进入特权模式和全局配置模式：
   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R2632
   ```

3. 配置连接三层交换机的以太网接口：
   ```
   R2632(config)# interface fastEthernet 1/0
   R2632(config-if)# ip address 172.16.1.1 255.255.255.0
   R2632(config-if)# no shutdown
   R2632(config-if)# exit
   ```

4. 配置连接校外路由器的串行接口：
   ```
   R2632(config)# interface serial 1/2
   R2632(config-if)# ip address 172.16.2.1 255.255.255.0
   R2632(config-if)# no shutdown
   R2632(config-if)# exit
   ```

   > **注意**：这里不需要配置时钟频率，因为DCE端在另一台路由器R1762-01上。

5. 验证接口配置：
   ```
   R2632(config)# exit
   R2632# show ip interface brief
   ```

   确认所有配置的接口状态为"up"。

### 步骤4：配置校外路由器R1762-01

1. 连接到R1762-01的控制台：
   - 在Packet Tracer中，点击路由器图标
   - 选择"CLI"标签页

2. 进入特权模式和全局配置模式：
   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R1762-01
   ```

3. 配置连接校园路由器的串行接口（DCE端）：
   ```
   R1762-01(config)# interface serial 1/2
   R1762-01(config-if)# ip address 172.16.2.2 255.255.255.0
   R1762-01(config-if)# clock rate 64000
   R1762-01(config-if)# no shutdown
   R1762-01(config-if)# exit
   ```

   > **重要**：必须在DCE端配置时钟频率(`clock rate 64000`)，否则串行链路无法正常工作。

4. 配置连接校外PC的以太网接口：
   ```
   R1762-01(config)# interface fastEthernet 1/1
   R1762-01(config-if)# ip address 172.16.3.1 255.255.255.0
   R1762-01(config-if)# no shutdown
   R1762-01(config-if)# exit
   ```

5. 验证接口配置：
   ```
   R1762-01(config)# exit
   R1762-01# show ip interface brief
   ```

   确认所有配置的接口状态为"up"。

### 步骤5：配置模拟PC设备

1. 配置校园内PC1(R1762-02)：
   - 连接到R1762-02的控制台

   ```
   Router> enable
   Router# configure terminal
   Router(config)# hostname R1762-02
   
   R1762-02(config)# interface fastEthernet 1/0
   R1762-02(config-if)# ip address 172.16.5.11 255.255.255.0
   R1762-02(config-if)# no shutdown
   R1762-02(config-if)# exit
   
   R1762-02(config)# ip route 0.0.0.0 0.0.0.0 fastEthernet 1/0
   ```

   > **说明**：最后一条命令配置默认路由，相当于配置默认网关指向172.16.5.1。

2. 配置校外PC2(S2126G-02)：
   - 连接到S2126G-02的控制台

   ```
   Switch> enable
   Switch# configure terminal
   Switch(config)# hostname S2126G-02
   
   S2126G-02(config)# interface vlan 1
   S2126G-02(config-if)# ip address 172.16.3.22 255.255.255.0
   S2126G-02(config-if)# no shutdown
   S2126G-02(config-if)# exit
   
   S2126G-02(config)# ip default-gateway 172.16.3.1
   ```

   > **说明**：`ip default-gateway`命令配置默认网关指向校外路由器R1762-01的F1/1接口地址。

### 步骤6：配置RIP V2路由协议

1. 在三层交换机S3760-01上配置RIP V2：
   ```
   S3760-01# configure terminal
   S3760-01(config)# router rip
   S3760-01(config-router)# version 2
   S3760-01(config-router)# network 172.16.1.0
   S3760-01(config-router)# network 172.16.5.0
   S3760-01(config-router)# end
   ```

   > **说明**：三层交换机需要通告两个直连网络：172.16.1.0/24和172.16.5.0/24。

2. 在校园出口路由器R2632上配置RIP V2：
   ```
   R2632# configure terminal
   R2632(config)# router rip
   R2632(config-router)# version 2
   R2632(config-router)# network 172.16.1.0
   R2632(config-router)# network 172.16.2.0
   R2632(config-router)# no auto-summary
   R2632(config-router)# end
   ```

   > **说明**：
   > - `version 2`命令指定使用RIP版本2
   > - `network`命令指定参与RIP路由的网段
   > - `no auto-summary`命令关闭自动路由汇总，使RIP可以通告子网路由

3. 在校外路由器R1762-01上配置RIP V2：
   ```
   R1762-01# configure terminal
   R1762-01(config)# router rip
   R1762-01(config-router)# version 2
   R1762-01(config-router)# network 172.16.2.0
   R1762-01(config-router)# network 172.16.3.0
   R1762-01(config-router)# no auto-summary
   R1762-01(config-router)# end
   ```

   > **说明**：校外路由器需要通告两个直连网络：172.16.2.0/24和172.16.3.0/24。

## 验证配置结果

### 步骤1：检查RIP路由信息

1. 在三层交换机S3760-01上查看路由表：
   ```
   S3760-01# show ip route
   ```

   应该能看到类似以下输出：
   ```
   C    172.16.1.0/24 is directly connected, VLAN 10
   C    172.16.1.2/32 is local host.
   R    172.16.2.0/24 [120/1] via 172.16.1.1, 00:00:09, VLAN 10
   R    172.16.3.0/24 [120/2] via 172.16.1.1, 00:00:09, VLAN 10
   C    172.16.5.0/24 is directly connected, VLAN 50
   C    172.16.5.1/32 is local host.
   ```

   > **说明**：
   > - "C"表示直连网络
   > - "R"表示通过RIP学习的路由
   > - [120/1]中的"120"是RIP路由的管理距离，"1"是跳数

2. 在校园出口路由器R2632上查看路由表：
   ```
   R2632# show ip route
   ```

   应该能看到类似以下输出：
   ```
   C    172.16.1.0/24 is directly connected, FastEthernet 1/0
   C    172.16.1.1/32 is local host.
   C    172.16.2.0/24 is directly connected, serial 1/2
   C    172.16.2.1/32 is local host.
   R    172.16.3.0/24 [120/1] via 172.16.2.2, 00:00:26, serial 1/2
   R    172.16.5.0/24 [120/1] via 172.16.1.2, 00:00:13, FastEthernet 1/0
   ```

3. 在校外路由器R1762-01上查看路由表：
   ```
   R1762-01# show ip route
   ```

   应该能看到类似以下输出：
   ```
   R    172.16.1.0/24 [120/1] via 172.16.2.1, 00:00:09, serial 1/2
   C    172.16.2.0/24 is directly connected, serial 1/2
   C    172.16.2.2/32 is local host.
   C    172.16.3.0/24 is directly connected, FastEthernet 1/1
   C    172.16.3.1/32 is local host.
   R    172.16.5.0/24 [120/2] via 172.16.2.1, 00:00:09, serial 1/2
   ```

### 步骤2：检查RIP协议配置

1. 查看RIP协议详情：
   ```
   Router# show ip protocols
   ```

   应该能看到RIP的版本、网络、定时器和邻居等信息。

2. 查看RIP数据库：
   ```
   Router# show ip rip database
   ```

   此命令显示RIP路由协议的路由数据库。

### 步骤3：测试网络连通性

1. 从校园内PC1(R1762-02)ping校外PC2(S2126G-02)：
   ```
   R1762-02# ping 172.16.3.22
   ```

   应该看到成功的ping响应。

2. 从校外PC2(S2126G-02)ping校园内PC1(R1762-02)：
   ```
   S2126G-02# ping 172.16.5.11
   ```

   应该看到成功的ping响应：
   ```
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echos to 172.16.5.11,
   timeout is 2000 milliseconds.
   !!!!!
   
   Success rate is 100 percent (5/5)
   Minimum = 35ms Maximum = 50ms, Average = 38ms
   ```

3. 使用traceroute查看数据包经过的路径：
   ```
   S2126G-02# traceroute 172.16.5.11
   ```

   您应该能看到数据包依次经过172.16.3.1、172.16.2.1、172.16.1.2，最终到达172.16.5.11。

## 排除常见问题

### 1. 串行接口链路不通

**症状**：串行接口显示状态为"down"或无法ping通对端设备

**解决方法**：
- 确认在DCE端配置了时钟频率：
  ```
  Router(config)# interface serial 1/2
  Router(config-if)# clock rate 64000
  ```
- 检查线缆连接和类型（确保DCE/DTE端正确）
- 确认两端的接口都配置了正确的IP地址并启用（no shutdown）

### 2. RIP路由不正确

**症状**：路由表中看不到应该学习到的远程网络

**解决方法**：
- 确认RIP配置中包含了所有需要通告的网络：
  ```
  Router# show run | section router rip
  ```
- 在RIP V2中，确认使用了`no auto-summary`命令
- 检查路由器之间的连接是否正常
- 使用`debug ip rip`命令查看RIP更新的详细信息

### 3. PC无法通信

**症状**：PC可以ping通直连的网关，但无法ping通远程网络设备

**解决方法**：
- 确认PC配置了正确的默认网关
- 检查路由器和交换机的路由表是否包含所有必要的网络
- 确认中间设备的接口状态正常
- 使用`traceroute`命令确定问题发生的位置

### 4. 三层交换机无法路由

**症状**：三层交换机上的VLAN间或到外部网络的路由不通

**解决方法**：
- 确认开启了IP路由功能：`ip routing`
- 确认VLAN接口状态为up：
  ```
  Switch# show ip interface brief
  ```
- 确认RIP协议正确配置并在正确的接口上启用
- 检查VLAN配置和端口分配

## 扩展知识

### 1. RIP定时器调整

RIP有四种关键定时器，可以根据网络需求进行调整：
```
R2632(config)# router rip
R2632(config-router)# timers basic 30 180 180 240
```

- 第一个值(30)：路由更新定时器，默认30秒
- 第二个值(180)：路由失效定时器，默认180秒
- 第三个值(180)：路由抑制定时器，默认180秒
- 第四个值(240)：路由刷新定时器，默认240秒

### 2. RIP路由认证

RIP V2支持MD5认证，增强安全性：
```
R2632(config)# key chain RIPAUTH
R2632(config-keychain)# key 1
R2632(config-keychain-key)# key-string password123
R2632(config-keychain-key)# exit
R2632(config-keychain)# exit

R2632(config)# interface serial 1/2
R2632(config-if)# ip rip authentication mode md5
R2632(config-if)# ip rip authentication key-chain RIPAUTH
```

### 3. RIP路由过滤

可以使用访问控制列表和分布列表控制RIP路由的通告和接收：
```
R2632(config)# access-list 1 deny 172.16.3.0 0.0.0.255
R2632(config)# access-list 1 permit any
R2632(config)# router rip
R2632(config-router)# distribute-list 1 out serial 1/2
```

这将阻止172.16.3.0/24网段通过Serial1/2接口通告出去。

### 4. RIP路由汇总

虽然可以使用`no auto-summary`关闭自动汇总，但也可以在特定接口上手动配置汇总：
```
R2632(config)# interface serial 1/2
R2632(config-if)# ip summary-address rip 172.16.0.0 255.255.0.0
```

这会将所有172.16.x.x的子网汇总为一个172.16.0.0/16的路由通告出去。

### 5. RIP与其他路由协议比较

| 特性 | RIP | OSPF | EIGRP |
|------|-----|------|-------|
| 类型 | 距离矢量 | 链路状态 | 混合 |
| 度量值 | 跳数 | 带宽/开销 | 带宽、延迟等复合参数 |
| 最大跳数 | 15 | 无限制 | 255(默认) |
| 收敛速度 | 慢 | 快 | 很快 |
| 资源占用 | 低 | 高 | 中 |
| 应用场景 | 小型网络 | 中大型网络 | 中大型网络 |

---

通过本教程，您已经学习了如何配置RIP V2路由协议，实现不同网段之间的自动路由学习和数据转发。RIP协议虽然简单，但在小型网络中仍有其应用价值，是网络工程师必须掌握的基础技能。 