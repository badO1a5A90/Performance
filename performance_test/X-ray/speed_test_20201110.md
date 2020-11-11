
## 前言

本次性能测试的组合除个别用于性能对比参考, 都能相对保证安全性，并且通过分流或回落具有较高伪装性，可以根据自己的需求不同使用不同组合模式.

* 无需关注
  
## 重要TIP

* 所有测试流量都为TLS加密数据,更符合实际上网情形,**也满足XTLS测试环境.**
* 数据流向为服务器流向客户端,更符合实际上网情形.
* XTLS的相关测试中,XTLS已经100%处理原始TLS加密流量.

## 测试方式
* 与以往一致,4VPS模式


## 测试内容

* 对比测试各种常用组合方式的性能(含回落)


## 测试数据及总结

### 常用组合性能测试对比一(20201110)
---

* 数据流向及说明
  * 4VPS模式
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * **除非特别注明**,实际测试过程中B为满负荷运行
* B的CPU限制,使用cgroup精确限制在50%.
* trojan-go的测试即把BC的客户端和服务端程序替换为trojan-go,其余不变.
* only once test.

协议配置组合方式|v2ray速率(TLS record=2K)|Xray速率1(TLS record=2K)|Xray速率2(TLS record=2K)|Xray速率(TLS record=16K)|备注
--- | --- | ---| ---| ---|---
A-D直连(跳过BC)|	2958 Mbits/sec|2851 Mbits/sec|	3037 Mbits/sec|6279 Mbits/sec|仅用于对照
无协议(dokodemo-door+freedom)|	 1524   Mbits/sec|1951 Mbits/sec|	2036 Mbits/sec |2106 Mbits/sec|仅用于对照
VLESS over TCP, no TLS	|  1552  Mbits/sec|1947 Mbits/sec|	2044 Mbits/sec|2155 Mbits/sec|仅用于对照,裸奔
VLESS over TCP, with TLS	|307 Mbits/sec|607 Mbits/sec |	867 Mbits/sec|848 Mbits/sec|
VLESS over TCP, XTLS(origin)	| 284 Mbits/sec|417 Mbits/sec|	471 Mbits/sec|1396 Mbits/sec|
VLESS over TCP, XTLS(direct)	|1511 Mbits/sec|1874 Mbits/sec|	2013 Mbits/sec|2067 Mbits/sec|
**v2ray**'trojan over TCP, with TLS	|  300 Mbits/sec|600 Mbits/sec|	850 Mbits/sec |815 Mbits/sec|
**v2ray**'trojan over TCP, with XTLS(origin)	| 287 Mbits/sec|420 Mbits/sec|	445 Mbits/sec |1312 Mbits/sec|
**v2ray**'trojan over TCP, with XTLS(direct)	|  1480 Mbits/sec|1875 Mbits/sec|	2002 Mbits/sec |2040 Mbits/sec|
**direct,no readV**	|  390 Mbits/sec|-|	1672 Mbits/sec |-|-
vmess over TCP, with TLS	| 271   Mbits/sec|539 Mbits/sec |failed|failed|
vmess over TCP, (aes-128-gcm)	| 637 Mbits/sec|801 Mbits/sec|	928 Mbits/sec|1011 Mbits/sec|
vmess over TCP, (chacha20-poly1305)	| 617 Mbits/sec|841 Mbits/sec|	929 Mbits/sec|987 Mbits/sec|
vmess over ws, with TLS	| 215 Mbits/sec|313 Mbits/sec |failed|failed |	前置nginx http分流
vmess over ws, (aes-128-gcm)	| 256 Mbits/sec|496 Mbits/sec|	failed |failed|前置nginx http分流
vmess over ws, (chacha20-poly1305)	|  262 Mbits/sec|488 Mbits/sec|	failed |failed|前置nginx http分流
fallback	|  1680 Mbits/sec|2321 Mbits/sec|	2799 Mbits/sec |2459 Mbits/sec|非上限,C都满载了
trojan-go	|  894 Mbits/sec|918 Mbits/sec|	878 Mbits/sec |918 Mbits/sec|
trojan-gfw	|  -|	- |

  ### 总结
  ---
  1. v2ray速率表现,和[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)是一致的
  2. Xray所有的性能都得到了大幅甚至近3倍的提升.
  3. Xray有4项失败,原因待查.
  4. 之前测试fallback和dokodemo-door性能几乎一致的结论在Xray中被推翻了.测试触发了C满负载,似乎可以接近AD直连?
  5. direct模式始终稳定和裸奔一致,包括提升.
  6. origin依然比较奇怪,表现和record size相关.(似乎仅origin受record size影响)
  7. direct,no readV 对照组比较有意思...
       - 去掉TLS解密
          - 去掉解密是有意义的,有性能提升.(v2ray30%,Xray100%.)
          - Xray去掉解密的性能提升被明显放大了
          - 去掉解密+readV达到了裸奔上限
     - readV
       - v2ray中,对比有无readV的direct,readV的性能提升能力似乎达到了3倍,在[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)的测试一中也有一组数据可以参考(测试一中,VLESS direct 带readV,trojan direct不带readV,两项也接近3倍,且VLESS direct已到上限)
       - Xray中,因为去掉解密的性能提升明显放大,带readV后达到了裸奔上限,readV的实际提升无法对比.
  8. 似乎trojan-go并没有显得那么强了...
---

## 测试环境
* 测试一
  * 四台VPS：
      - AD:Intel Core Processor (Skylake, IBRS)*2,4G
      - BC:Intel Xeon Processor (Skylake, IBRS)*1,2G
      - 10Gbps上下行带宽
      - 公网IP
  * 系统：Debian 10
  * 所有机器均支持 AES-NI 硬件加速指令集

## 测试工具
* iperf 3.6
* v2ray 4.32.1(with VLESS XTLS readV)
* ~~Xray~~
* trojan-go 0.8.2
* trojan-gfw 1.16.0
* nginx openresty/1.17.8.2

## 测试时间
    2020.11
