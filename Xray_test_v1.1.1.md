
## 前言

本次性能测试的组合除个别组合用于性能对比参考, 其余组合都能相对保证安全性，并且通过分流或回落具有较高伪装性，可以根据自己的需求不同使用不同组合模式.

*个人自建上网,首先考虑安全性和伪装性,然后选择能满足自己需求的最高性能组合*

* 不推荐在公网环境下使用Xray中任何基于socks协议的组合.

* 各配置的一键脚本/模板/客户端可以在[Project X](https://github.com/XTLS/Xray-core)找到.
  
## 重要TIP

* 可以先下跳看测试结果数据列表,但是测试方式非常重要.
* 所有测试流量都为TLS加密数据,更符合实际上网情形,**也满足XTLS测试环境.**
* 数据流向为服务器流向客户端,更符合实际上网情形.
* XTLS的相关测试中,XTLS已经100%处理原始TLS加密流量.
* 暂时没有DS相关测试(DS可能存在未知缺陷,待性修复)


## 测试方式
<details>
<summary>点击展开查看测试方式</summary>

* 要测试协议之间性能差距，那么必然是硬件CPU负荷满的情况下，其他变量不变，仅有协议组合差别下测试的情况才是有意义的。
* 使用4台VPS功能如下，以下简单命名4台VPS为，ABCD
  - A 负责以TLS加密测试工具客户端的数据，发送给B
  - B 负责使用Xray以各种方式对A产生的数据进行处理并转发给C，也即类似通常所说的上网设备(Xray客户端)。
  - C 负责使用Xray接收和以各种方式处理B的数据发送给D,也即类似通常所说的服务器(Xray服务端).
  - D 负责接收和处理TLS加密数据并发送至测试工具服务端 
  - 实际测试数据流向为服务器流向客户端,更符合实际上网情形.
* 之所以拆分,是因为测试较复杂,本地回环测试甚至2台独立机器测试会导致各种因素互相影响较大，不易控制流程中各个环节和因素，也无法进行细致比较分析
* 测试将数据流向的各个环节完全拆开
  * 令原始数据的TLS加密解密对测试无影响(AD独立负责),**变量仅为各协议配置组合方式**
  * 完全去除各种因素相互影响
  * 可以模拟使用中间硬件的实用场景并控制性能瓶颈进行不同测试对照
  * 可以分别分析各数据处理环节对性能的影响
* 各组合单次测试时长50s,多次取均值
</details>

## 测试内容

* 对比测试Xray各种常用组合方式的性能
* 与v2ray/trojan/ss更多的对比测试参见 [Xray v1.1.0的性能对比测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201202.md)
* Xray路由/VLESS回落/nginx分流性能的对比(待测)

## 测试数据及总结

### 常用组合性能测试对比

* 数据流向及说明

  * 从服务器流向客户端

    A,iperf client+Xray inbound(dokodemo-door),outbound(freedom with TLS)  
    <--B,inbound(dokodemo-door),outbound(各种协议组合)  
    <--C,inbound(各种协议组合),outbound(freedom)  
    <--D,Xray inbound(dokodemo-door with TLS),outbound(freedom)+iperf server
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * 实际测试过程中B为满负荷运行
* B的CPU限制,使用cgroup精确限制
* 非Xray的测试即把BC的客户端和服务端程序替换为其他工具,其余不变.

协议配置组合方式|速率(TLS record=8K)|备注
--- | --- | ---
A-D直连(跳过BC)|	5246 Mbits/sec |仅用于对照
无协议(dokodemo-door+freedom)|	 2127   Mbits/sec |仅用于对照
VLESS over TCP, no TLS	|  2116  Mbits/sec |仅用于对照,裸奔
VLESS over TCP, with TLS	|728 Mbits/sec 
VLESS over TCP, XTLS(origin)	| 673 Mbits/sec 
VLESS over TCP, XTLS(direct)	| 2012 Mbits/sec 
**VLESS over TCP, XTLS(splice)**	|**4372 Mbits/sec**
vmess over TCP, with TLS	| 621   Mbits/sec 
vmess over TCP, (aes-128-gcm)	| 1346 Mbits/sec 
vmess over TCP, (chacha20-poly1305)	| 1059 Mbits/sec 
vmess over ws, with TLS	| 350 Mbits/sec  |前置nginx http分流
vmess over ws, (aes-128-gcm)	| 570 Mbits/sec  |前置nginx http分流
vmess over ws, (chacha20-poly1305)	|  487 Mbits/sec  |前置nginx http分流
**Xray**'trojan over TCP, with TLS	|  730 Mbits/sec |对照VLESS over TCP, with TLS
**Xray**'trojan over TCP, with XTLS(origin)	| 681 Mbits/sec  |对照VLESS over TCP, XTLS(origin)
**Xray**'trojan over TCP, with XTLS(direct)	|  2010 Mbits/sec|对照VLESS over TCP, XTLS(direct)
trojan-go	|  1409 Mbits/sec |
go-shadowsocks2+outline(aes-128-gcm)	|  1256 Mbits/sec  |

![avatar](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/img/20201204.png?raw=true)

  ### 总结
  
  * **关于测试数据的意义**
    * **各种组合配置的速率测试是性能对比测试, 不是吞吐量上限测试.**
    * **测试可以体现在不同使用模式下，或不同程序版本下性能有差异.**
    * **测试代表测试环境下的差异，并不表示绝对差异，也不是每个人使用后的实际性能提升.**
    * **通常情况下硬件性能越差(尤其有无AES指令集),XTLS相对提升越多.**
  1. A-D直连(跳过BC)可视为A-D使用TLS加密数据测试的上限值
  2. VLESS over TCP, no TLS 即常说的裸奔速度
  3. splice模式一骑绝尘.
  4. 协议用VLESS或者trojan可认为没有性能区别,**但目前trojan尚未加入splice支持.**
  5. XTLS(origin)的速度与TLS record大小相关(本次测试为8K).
  6. vmess over TCP, (aes-128-gcm)/(chacha20-poly1305)较其他vmess组合速率较高是因为他们也具有readV优化.
  7. trojan-go/ss沿用的是[Xray v1.1.0的性能对比测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201202.md)中的数据.


  ### 关于splice
  <details>
<summary>点击展开查看关于splice</summary>

* Splice 工作方式：
    - 读取数据时，Linux kernel 直接转发 TCP，不再经过 Xray 的内存
    - 可以减少至少两次数据拷贝，节省 CPU 切换，提升路由器、移动设备上的性能
* Splice 使用场景限制：
    - linux平台,如安卓和路由器,以及用 linux 当桌面等使用场景
    - inbound:目前仅支持任意门/socks/http入站
    - outbound:支持XTLS的协议出站(目前仅VLESS)
    - 当你的性能瓶颈是客户端时才能发挥作用
</details>

### Xray路由/VLESS回落/nginx分流性能测试(待测)

## 测试环境
* 测试一
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
* [v2ray v4.32.1(with VLESS XTLS readV)](https://github.com/v2fly/v2ray-core/releases/tag/v4.32.1)
* [v2ray v4.33.0](https://github.com/v2fly/v2ray-core)
* [Xray v1.1.1](https://github.com/XTLS/Xray-core)
* [trojan-go v0.8.2](https://github.com/p4gefau1t/trojan-go)
* [trojan-gfw v1.16.0](https://github.com/maskedeken/trojan-gfw)
* [go-shadowsocks2](https://github.com/shadowsocks/go-shadowsocks2)
* [outline v0.50.0](https://github.com/outline/outline)
* [shadowsocks-libev v3.3.5](https://github.com/shadowsocks/shadowsocks-libev)
* [shadowsocks-rust v1.8.23](https://github.com/shadowsocks/shadowsocks-rust)
* [nginx openresty/1.17.8.2](https://openresty.org/en/)

## 测试时间
    2020.12