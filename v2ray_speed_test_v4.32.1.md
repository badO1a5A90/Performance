
## 前言

本次性能测试的组合除个别用于性能对比参考, 都能相对保证安全性，并且通过分流或回落具有较高伪装性，可以根据自己的需求不同使用不同组合模式.

*个人自建上网,首先考虑安全性和伪装性,然后选择能满足自己需求的最高性能组合*

* 不推荐在v2ray中使用任何基于socks协议的组合.

* 各配置的模板可以在[v2ray模板仓库](https://github.com/v2fly/v2ray-examples)找.
  
## 重要TIP

* 可以先下跳看测试结果数据列表,但是测试方式非常重要.
* **各种组合配置的速率测试是性能对比测试,不是吞吐量上限测试.**
* 所有测试流量都为TLS加密数据,更符合实际上网情形,**也满足XTLS测试环境.**
* 数据流向为服务器流向客户端,更符合实际上网情形.
* XTLS的相关测试中,XTLS已经100%处理原始TLS加密流量.
* 因v2ray的DS已有多次测试证明可能存在问题导致性能不佳,且没有被修复过,所以没有DS相关测试.


## 测试方式
* 使用4台VPS功能如下，以下简单命名4台VPS为，ABCD
  - A 负责以TLS加密测试工具客户端的数据，发送给B
  - B 负责使用v2ray以各种方式对A产生的数据进行处理并转发给C，也即类似通常所说的上网设备(v2ray客户端)。
  - C 负责使用v2ray接收和以各种方式处理B的数据发送给D,也即类似通常所说的服务器(v2ray服务端).
  - D 负责接收和处理TLS加密数据并发送至测试工具服务端 
  - 实际测试数据流向为服务器流向客户端,更符合实际上网情形.
* 之所以拆分,是因为测试较复杂,本地回环测试甚至2台独立机器测试会导致各种因素互相影响较大，不易控制流程中各个环节和因素，也无法进行细致比较分析
* 测试将数据流向的各个环节完全拆开
  * 令原始数据的TLS加密解密对测试无影响(AD独立负责),**变量仅为各协议配置组合方式**
  * 完全去除各种因素相互影响
  * 可以模拟使用中间硬件的实用场景并控制性能瓶颈进行不同测试对照
  * 可以分别分析各数据处理环节对性能的影响
* 各组合单次测试时长50s,多次取均值


## 测试内容

* 对比测试各种常用组合方式的性能
* v2ray路由/VLESS回落/nginx分流性能的对比(待测)


## 测试数据及总结

### 常用组合性能测试对比二(20201109)
---
* **这组数据与测试一的硬件以及CPU限制不同,没有对比参照的意义**
* **各种组合配置的速率测试是性能对比测试,不是吞吐量上限测试.**
* 数据流向及说明
  * 与测试一相同
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * 实际测试过程中B为满负荷运行
* B的CPU限制,使用cgroup精确限制在50%.
* trojan-go/trojan-gfw的测试即把BC的客户端和服务端程序替换为trojan-go/trojan-gfw,其余不变.

协议配置组合方式|速率(TLS record=2K)|备注
--- | --- | ---
A-D直连(跳过BC)|	2534 Mbits/sec|仅用于对照
无协议(dokodemo-door+freedom)|	 2030   Mbits/sec |仅用于对照
VLESS over TCP, no TLS	|  1999  Mbits/sec|仅用于对照,裸奔
VLESS over TCP, with TLS	|379 Mbits/sec 
VLESS over TCP, XTLS(origin)	| 319 Mbits/sec
**VLESS over TCP, XTLS(direct)**	|**1962 Mbits/sec**
vmess over TCP, with TLS	| 328   Mbits/sec 
vmess over TCP, (aes-128-gcm)	| 795 Mbits/sec
vmess over TCP, (chacha20-poly1305)	| 797 Mbits/sec
vmess over ws, with TLS	| 254 Mbits/sec |前置nginx http分流
vmess over ws, (aes-128-gcm)	| 300 Mbits/sec |前置nginx http分流
vmess over ws, (chacha20-poly1305)	|  301 Mbits/sec |前置nginx http分流
**v2ray**'trojan over TCP, with TLS	|  379 Mbits/sec |对照VLESS over TCP, with TLS
**v2ray**'trojan over TCP, with XTLS(origin)	| 317 Mbits/sec |对照VLESS over TCP, XTLS(origin)
**v2ray**'trojan over TCP, with XTLS(direct)	|  1974 Mbits/sec |对照VLESS over TCP, XTLS(direct)
trojan-go	|  1513 Mbits/sec |
trojan-gfw	|  505 Mbits/sec |

  ### 总结
  ---
  1. 虽然硬件及限制不同,但测试一中的结论仍然都成立.(参见下方测试一数据和结论)
  2. **VLESS over TCP, XTLS(direct) 仍然和裸奔完全一致(必须是4.32.1版本以上的VLESS over TCP, XTLS(direct)才有readV,才可以达到这个性能)**
  3. 该测试硬件性能整体低于测试一,且进行了硬件性能限制,可以见得,
     - 硬件性能越低VLESS over TCP, XTLS(direct)和其他协议的性能差距就越大.
  4. 与测试一不同, 此组数据的**v2ray**'trojan over TCP, with XTLS(direct)与VLESS一致,因为我加了~~洋葱~~readV.(**v4.32.1中并无此特性,见测试一数据**)
  5. vmess over TCP, (aes-128-gcm)/(chacha20-poly1305)较其他vmess组合速率较高是因为他们也具有readV优化.
  6. ~~实测开启v2ray内置的流量统计会使所有的readV失效(性能损失一半),并且额外受到约15%的性能打击(该测试硬件基础上)~~
  7. 经过更多实测和代码分析后,确定v2ray的流量统计功能会使裸协议的 ReadV 和 WriteV 同时失效(性能损失一半以上),具体参见[issue#416](https://github.com/v2fly/v2ray-core/issues/416)
---

### 常用组合性能测试对比一(20201107)
---

* **各种组合配置的速率测试是性能对比测试,不是吞吐量上限测试.**
* 数据流向及说明

  * 从服务器流向客户端

    A,iperf client+v2ray inbound(dokodemo-door),outbound(freedom with TLS)  
    <--B,inbound(dokodemo-door),outbound(各种协议组合)  
    <--C,inbound(各种协议组合),outbound(freedom)  
    <--D,v2ray inbound(dokodemo-door with TLS),outbound(freedom)+iperf server
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * 实际测试过程中B为满负荷运行

协议配置组合方式|速率(TLS record=2K)|备注
--- | --- | ---
A-D直连(跳过BC)|	3113 Mbits/sec|仅用于对照
无协议(dokodemo-door+freedom)|	2898 Mbits/sec |仅用于对照
VLESS over TCP, no TLS	|2673 Mbits/sec|仅用于对照,裸奔
VLESS over TCP, with TLS	|816 Mbits/sec 
VLESS over TCP, XTLS(origin)	|598 Mbits/sec
**VLESS over TCP, XTLS(direct)**	|**2651 Mbits/sec**
vmess over ws, with TLS	|540 Mbits/sec |前置nginx http分流
vmess over ws, (aes-128-gcm)	|603 Mbits/sec
vmess over ws, (chacha20-poly1305)	|590 Mbits/sec
**v2ray**'trojan over TCP, with TLS	|835 Mbits/sec |对照VLESS over TCP, with TLS
**v2ray**'trojan over TCP, with XTLS(origin)	|597 Mbits/sec |对照VLESS over TCP, XTLS(origin)
**v2ray**'trojan over TCP, with XTLS(direct)	|893 Mbits/sec |对照VLESS over TCP, XTLS(direct)

  ### 总结
  ---
  1. A-D直连(跳过BC)可视为A-D使用TLS加密数据测试的上限值
  2. VLESS over TCP, no TLS 即常说的裸奔速度
  3. **VLESS over TCP, XTLS(direct) 几乎和裸奔完全一致(必须是4.32.1版本以上的VLESS over TCP, XTLS(direct)才有readV,才可以达到这个性能)**
  4. XTLS(origin)的速度仍然比较奇怪,不论是VLESS还是trojan,存疑.
  5. 如果使用v2ray,协议用VLESS或者trojan可认为没有性能区别.
  6. **v2ray**'trojan over TCP, with XTLS(direct),没有readV因此提升性能不明显(且可能因本次测试未过多限制CPU性能,因此提升比例也不如以往XTLS测试明显-性能越低提升越明显)
---


### v2ray路由/VLESS回落/nginx分流性能测试(待测)

## 测试环境
* 测试一
  * 四台VPS：
      - AD:Intel(R) Xeon(R) CPU E3-1270 v6 3.80GHz*8 ,32G
      - C:Intel Core Processor (Skylake, IBRS)*2,4G
      - B:Intel Xeon Processor (Skylake, IBRS)*1,2G
      - 10Gbps上下行带宽
      - 公网IP
  * 系统：Debian 10
  * 所有机器均支持 AES-NI 硬件加速指令集
* 测试二
  * 四台VPS：
      - AD:Intel Xeon Processor (Skylake, IBRS)*2,4G
      - C:Intel Xeon Processor (Skylake, IBRS)*1,2G
      - B:Intel Xeon Processor (Skylake, IBRS)*1,2G
      - 10Gbps上下行带宽
      - 公网IP
  * 系统：Debian 10
  * 所有机器均支持 AES-NI 硬件加速指令集

## 测试工具
* iperf 3.6
* v2ray 4.32.1(with VLESS XTLS readV)
* trojan-go 0.8.2
* trojan-gfw 1.16.0
* nginx openresty/1.17.8.2

## 测试时间
    2020.11



