 ## 重要TIP

* 所有测试流量都为TLS加密数据,更符合实际上网情形,**也满足XTLS测试环境.**
* 数据流向为服务器流向客户端,更符合实际上网情形.
* XTLS的相关测试中,XTLS已经100%处理原始TLS加密流量.
* v2ray的XTLS测试基于v4.32.1,在v4.33.0中XTLS已经被移除。
* v2ray v4.33.0没有性能相关的改动，所以数据同样适用于v4.33.0

## 测试方式
* 与以往一致,4VPS模式,具体可参见[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)


## 测试内容

* 对比测试各种常用组合方式的性能


## 测试数据及总结

### 常用组合性能测试对比一
---

* 数据流向及说明
  * 4VPS模式(同[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md))
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * **除非特别注明**,实际测试过程中B为满负荷运行
* B的CPU使用cgroup精确限制.
* trojan-go的测试即把BC的客户端和服务端程序替换为trojan-go,其余不变.
* 每项单次测试50S,多次测试取均值.

协议配置组合方式|v2ray速率(Mbps)|Xray速率(Mbps)|备注
--- | ---| --- | ---
VLESS over TCP, no TLS	|  1936  |2399|仅用于对照,裸奔
VLESS over TCP, XTLS(direct)	|1873 | 2267
VLESS over TCP, with TLS	|334  |716
vmess over TCP, with TLS	| 286    |  634
vmess over TCP, (aes-128-gcm)	|829 | 1528
vmess over TCP, (chacha20-poly1305)	| 684 |  1114
vmess over ws, with TLS	| 239  |373|前置nginx http分流
vmess over ws, (aes-128-gcm)	| 255  |598|前置nginx http分流
vmess over ws, (chacha20-poly1305)	|  232  |493|前置nginx http分流
trojan-go	|  1409   |-|
trojan-gfw	|  505  |
go-shadowsocks2+outline(aes-128-gcm)	|  1256   |-|
shadowsocks-rust(aes-128-cfb)	|  394   |-|
shadowsocks-libev(aes-128-cfb)	|  405   |-|

![avatar](https://raw.githubusercontent.com/badO1a5A90/v2ray-doc/main/performance_test/Xray/img/xray20201119.jpg)

  ### 总结
  --- 
  1. 因为硬件基本一致,所以v2ray速率表现,和[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)基本是一致的(整体略低一点点)
      * 除vmess(加密)的性能经过反复测试,应以本次为准,需要对之前测试做一个勘误.
  2. 因为主要用于对比测试,所以模式选取较少,其他性能皆可参照[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)  
  3. Xray所有的性能都得到了大幅提升.
  4. direct模式始终稳定和裸奔基本一致.
---

### 常用组合性能测试对比二

* 来自TG群网友i553041的测试
* 服务器为树莓派4B,客户端为PC
* PC通过树莓派的v2ray服务端上网,并使用speedtest测试.

---

协议配置组合方式|v2ray速率(Mbps)|Xray速率(Mbps)
--- | ---| --- 
VLESS over TCP, XTLS(direct)	|第1次715.32 / 第2次815.42 / 第3次730.72 | (网速上限)第1次896.05 / 第2次885.21 / 第3次857.45
VLESS over TCP, with TLS	|第1次298.37 / 第2次289.26 /第3次297.22|第1次443.99 / 第2次444.10 /第3次441.08
vmess over TCP, with TLS	| 第1次180.34 / 第2次181.25 / 第3次173.99    |  第1次362.64 / 第2次372.58 / 第3次364.96
VLESS over ws, with TLS	|第1次295.35/ 第2次290.12 / 第3次266.58 | 第1次303.08 / 第2次275.59 / 第3次265.38
vmess over ws, with TLS	| 第1次182.85 / 第2次178.70 / 第3次178.19 |  第1次212.78 / 第2次219.61 / 第3次208.49

* Xray的VLESS over TCP, XTLS(direct)跑到了1Gbps的网速上限(与不用代理时测试速度一致),实际上限应该更高.
* 带ws的组合提升不明显,或许和前置分流有关
----

## 测试环境
* 测试一
  * 四台VPS：
      - AD:Intel Core Processor (Skylake, IBRS)*2,4G
      - BC:Intel Xeon Processor (Skylake, IBRS)*1,2G
      - 10Gbps上下行带宽
      - 公网IP
  * 系统：Debian 10
  * 所有机器均支持 AES-NI 硬件加速指令集
* 测试二
  * 树莓派4B
  * 带宽1G
  * 公网IP
  
## 测试工具
* iperf 3.6
* v2ray 4.32.1(with VLESS XTLS readV)
* Xray
* trojan-go 0.8.2
* trojan-gfw 1.16.0
* nginx openresty/1.17.8.2

## 测试时间
    2020.11
