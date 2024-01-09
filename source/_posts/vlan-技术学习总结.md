---
title: VLAN 技术学习总结
date: 2022-04-09 03:59:31
updated: 2022-04-09 03:59:31
tags:
mathjax: true
---

## VLAN技术学习总结

### 基本概念

虚拟局域网技术（Virtual Local Area Network 简写 VLAN）将一个**交换式LAN**划分成多个相互独立的虚拟物理网络，这些逻辑上虚拟的物理网络称为VLAN，每一个VLAN就是一个广播域，不同VLAN之间在逻辑上相互隔离，不同VLAN成员不能在二层直接通信，VLAN之间的通信只能通过三层或者更高层实现。

划分VLAN是为了隔离广播域，其标准为IEEE 802.1Q。

<!--more-->

### VLAN标识符

一个VLAN由一个VLAN ID（VLAN Identifier）唯一标识。VLAN ID 简写 VID。

VID值由一个12比特的无符号数表示，其理论有效范围为 1 到 4096 （$2^{12}$)，具体设备支持的VID范围可能更小。

默认VID值为1，用户不能指派他用。当没有划分VLAN时，所有站点都属于VLAN1，通常VLAN 1作为管理VLAN使用。当然，默认VLAN ID可以进行更改，不一定非要是1。

### VLAN设备

VLAN设备类型分为 VLAN知晓设备 和 VLAN非知晓设备。

#### VLAN知晓设备 （VLAN aware）

能意识到VLAN的存在。

识别VLAN加标帧（802.1Q帧）并予以处理，同时也能处理非加标帧（基本MAC帧）。

VLAN知晓设备通常是：支持VLAN协议的交换机 或者 支持VLAN协议的服务器。

#### VLAN非知晓设备（VLAN unaware）

不能意识到VLAN的存在。

不能识别VLAN加标帧。

VLAN知晓设备可以是：不支持VLAN协议的交换机（其对VLAN加标帧按照基本MAC帧进行处理） 或者 普通PC机（其不能识别加标帧，将直接丢弃加标帧）。

### VLAN链路

VLAN不同链路对应不同端口。VLAN链路包括 **主干链路（Trunk Link）**、**接入链路（Access Link）**、**混合链路（Hybrid Link）**。其对应的端口分别为 **Trunk Port**、**Access Port**、**Hybrid Port**。

**Trunk**

Trunk Link 通常用于互连VLAN知晓设备。Trunk Port 是连接 Trunk Link 的两端口，Trunk Port收发VLAN加标帧。一个Trunk Port可以同时属于多个VLAN（这有什么好处？我们后面讲）。

**Access**

Access Link通常用于（VLAN知晓设备）将非知晓设备接入VLAN。Access Port 是VLAN**知晓**设备上连接Access Link的端口。（注意这里是VLAN知晓设备的端口，而不像之前Trunk是两端口都是Trunk Port）。

Access Port只能收发无标识帧。

![image_2022-04-01_22-41-55](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-01_22-41-55.5l49rq23j9s0.png)

**Hybrid**

Hybrid Link能同时连接VLAN知晓设备和VLAN非知晓设备。Hybrid Port 可以接收**不同**VLAN的帧为它们带上对应的 Tag，也能**发送**不同VLAN带上Tag的帧或者是不同VLAN不带Tag的帧。

这张[图](https://www.utepo.net/article/detail/251.html)可以方便理解Hybrid Link和Port

![image_2022-04-01_21-21-03](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-01_21-21-03.3pwinylpm1s0.png)

蓝色端口被配置为Hybrid Port，其下的不同的VLAN发送帧到达这个端口时，Hybrid Port根据不同帧给它们打上不同的Tag。而Hybrid Port向下发送帧时，可以根据配置给某些VLAN发送带Tag的帧，给某些VLAN发送不带Tag的帧。

Hybrid Port的存在主要是为VLAN知晓设备的一个转发端口接到不同VLAN里面而存在的。

下层的VLAN由于受限集线器（Hub）的存在并未实现隔离广播域的效果。

Hybrid和Trunk Port都能同时配置多个VLAN，Hybrid Link通常使用共享链路，而Trunk Link使用点到点全双工链路。

### VLAN帧格式

VLAN帧和以太网基本MAC帧的唯一不同在于VLAN帧有一个4字节的Tag部分。下图中未标注单位均为字节。

![image_2022-04-09_00-42-32](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_00-42-32.1l22noqh4n6o.png)

**TPID**：标签协议标识符（Tag Protocol Identifier），用以标识加标帧的类型，802.1Q Tag帧取值固定为0x8100，极VLAN协议

**P**：优先级（Priority）。0~7共8个等级，优先级高的帧先发出。不同设备支持优先级的等级数有差异。

**CFI**：标准格式指示位（Canonical Format Indicator），表示MAC地址是否是经典格式。CFI为0说明是标准格式，CFI为1表示为非标准格式。用于区分以太网帧、FDDI（Fiber Distributed Digital Interface）帧和令牌环网帧。在以太网中，CFI的值为0。

**VLAN ID**：VLAN Identifier，有效值0~4095。

0：VLAN ID = null，该帧为用户优先级帧（仅使用优先级P的功能），非VLAN加标帧。

1：默认VLAN ID，该默认VLAN ID可以改变成不是1的数字。

4095：保留。

所以VLAN ID的有效范围：1~4094。

#### 注意

VLAN加标帧允许的最长帧大小超过基本MAC帧长的1518字节，其最长能允许1522（1518+4）字节，而最小帧长字节与基本MAC帧最小帧长相同，均为64字节。

值得注意的是，以上帧格式是802.1Q的封装格式，而思科交换机使用的是其专有协议[**ISL**](https://www.cisco.com/c/zh_cn/support/docs/lan-switching/8021q/17056-741-4.html)，此种封装只能在思科设备间使用，若在组网时使用了思科还有其他品牌的设备一定要注意把思科交换机的VLAN封装格式更换成802.1Q封装！

我之前就在组网的时候遇到了这个问题，琢磨半天也想不出问题在哪里，最后发现是思科交换机的VLAN帧封装的问题。。

在思科交换机中开启802.1Q封装解决问题：

``` Switch(config-if)#switchport trunk encapsulation dot1q ```

![image_2022-04-09_01-25-22](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_01-25-22.5oixo0nq2h4.webp)

对于上面的802.1Q封装，VLAN支持三种帧类型。

无标帧，即基本MAC帧

优先级加标帧，无有效VID的802.1Q帧

VLAN加标帧，既有优先级又有有效VID的802.1Q帧

### 再谈Trunk Link

为什么一个Trunk Port要同时属于多个VLAN？

事实上，在不使用Trunk的情况下，VLAN也完全可以实现，甚至在这种情况下实现路由功能的设备（可以是三层交换机也可以是路由器）可以是VLAN非知晓设备。

具体拓扑如下：

<img src="https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_02-02-37.1ke1r07mwyjk.webp" alt="image_2022-04-09_02-02-37" style="zoom:50%;" />

这种情况下，有多少个VLAN就需要L2 SW和L3 SW之间有多少个连接线，显然没有必要这样做，Trunk Link使得L2 SW和L3 SW之间只需要一条线。

![image_2022-04-09_02-09-24](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_02-09-24.40bymf2gl540.webp)

### VLAN间通信案例

下面结合Packet Tracer来实现VLAN间通信的仿真。

拓扑如下

![image_2022-04-09_02-30-29](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_02-30-29.60kvzdtufb00.webp)

网关和ip地址在每台PC上静态配置。

此时VLAN10内的PC可以互ping，VLAN20内的设备可以互ping，又因为双方不在同一个子网规划下，且此时L3 SW未绑定它俩的网关ip地址，VLAN10和VLAN20不能互ping。（注意此时只是配置了四台PC的ip和网关，VLAN什么的还没有配置）。

![IMG_0964(20220409-025615)](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/IMG_0964(20220409-025615).mj9eh85qyls.webp)

接下来配置L2 SW（access port）

```
Switch>enable 
Switch#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#vlan 10 
Switch(config-vlan)#exit 
Switch(config)#vlan 20
Switch(config-vlan)#exit
Switch(config)#int f0/1 
Switch(config-if)#switchport mode access 
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
Switch(config)#int f0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
Switch(config)#int f0/2
Switch(config-if)#switchport mode access 
Switch(config-if)#switchport access vlan 20
Switch(config-if)#exit
Switch(config)#int f0/4
Switch(config-if)#switchport mode access 
Switch(config-if)#switchport access vlan 20
Switch(config-if)#exit
```

show vlan 检测配置

``` 
Switch#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
10   VLAN0010                         active    Fa0/1, Fa0/3
20   VLAN0020                         active    Fa0/2, Fa0/4
```

接下来配置trunk port，此时需要配置L2 SW的 f0/5 和 L3 SW的 f0/1

配置L2 SW f0/5

```
Switch(config)#int f0/5
Switch(config-if)#switchport mode trunk 
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to up
```

配置L3 SW f0/1

```
Switch(config)#int f0/1
Switch(config-if)#switchport mode trunk
Command rejected: An interface whose trunk encapsulation is "Auto" can not be configured to "trunk" mode.
# 没有指定端口的VLAN封装类型
Switch(config-if)#switchport trunk encapsulation dot1q #指定为802.1Q封装
Switch(config-if)#switchport mode trunk #设置端口为Trunk Port
Switch(config-if)#exit
Switch(config)#vlan 10
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#exit
Switch(config)#int vlan 10
Switch(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan10, changed state to up

Switch(config-if)#ip address 192.168.10.254 255.255.255.0
Switch(config-if)#int vlan 20
Switch(config-if)#
%LINK-5-CHANGED: Interface Vlan20, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan20, changed state to up

Switch(config-if)#ip address 192.168.20.254 255.255.255.0
Switch(config-if)#exit
Switch(config)#
Switch#
%SYS-5-CONFIG_I: Configured from console by console

Switch#show ip route
Default gateway is not set

Host               Gateway           Last Use    Total Uses  Interface
ICMP redirect cache is empty

Switch#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#ip routing #开启路由
Switch(config)#exit
Switch#show ip route
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, Vlan10
C    192.168.20.0/24 is directly connected, Vlan20
```

此时PC1 ping PC2

<img src="https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/image_2022-04-09_03-28-53.50wbrt7hwfg0.webp" alt="image_2022-04-09_03-28-53" style="zoom:50%;" />

可以看到已经通了。

#### PC1 ping PC2的数据包追踪流程图

![IMG_0965(20220409-035030)](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/VLAN技术学习总结/IMG_0965(20220409-035030).6amubcp68rs0.PNG)
