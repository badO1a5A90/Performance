 ## 重要TIP

* 所有测试流量都为TLS加密数据,更符合实际上网情形,**也满足XTLS测试环境.**
* 数据流向为服务器流向客户端,更符合实际上网情形.
* XTLS的相关测试中,XTLS已经100%处理原始TLS加密流量.

## 测试方式
* 与以往一致,4VPS模式,具体可参见[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)中的详细描述.


## 测试内容

* 对比测试各种常用组合方式的性能
* 本次测试主要加入了splice模式


## 测试数据及总结

### 常用组合性能测试对比一
---

* 数据流向及说明
  * 4VPS模式(可参见[v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)中的详细描述)
* 所有测试的变量仅在于BC之间的协议配置组合方式
* 仅需关注BC协议配置组合方式. 
  * AD负责产生和接收TLS数据及iperf运行,所有测试过程中,AD均不会满载,不会对测试有任何影响.
  * 除非特别注明,实际测试过程中B为满负荷运行
* B的CPU使用cgroup精确限制.
* 非v2ray的测试即把BC的客户端和服务端程序替换为其他工具,其余不变.
* 每项单次测试50S,多次测试取均值.XTLS

协议配置组合方式|v2ray v4.32.1速率(Mbps)|v2ray v4.33.0速率(Mbps)|Xray v1.1.0速率(Mbps)|备注
--- | ---| --- |  --- |---
VLESS over TCP, XTLS(splice)	|  -  | -  | 4435 |only Xray v1.1.0
VLESS over TCP, no TLS	|  1936  | 1891  |2336|仅用于对照,裸奔
VLESS over TCP, XTLS(direct)	|1873 | - | 2132|v4.33.0 removed XTLS
VLESS over TCP, with TLS	|334  | 332  |722
vmess over TCP, with TLS	| 286    | 308  |  645
vmess over TCP, (aes-128-gcm)	|829 |  882 |1391
vmess over TCP, (chacha20-poly1305)	| 684 | 720  |  1042
vmess over ws, with TLS	| 239  | 239  |371|前置nginx http分流
vmess over ws, (aes-128-gcm)	| 255  | 254  |597 |前置nginx http分流
vmess over ws, (chacha20-poly1305)	|  232  | 227 |498|前置nginx http分流
trojan-go	|  1409   |-|
trojan-gfw	|  505  | - |
go-shadowsocks2+outline(aes-128-gcm)	|  1256   |-|
shadowsocks-rust(aes-128-cfb)	|  394   |-|
shadowsocks-libev(aes-128-cfb)	|  405   |-|

![avatar](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/img/xray20201202.png?raw=true)

  ### 总结
  --- 
  1. Xray v1.1.0和v1.0.0 除增加splice模式之外,没有额外性能的相关改动,所以除splice模式外,其他测试数据沿用了[Xray v1.0.0版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201124.md)
  2. 本次测试主要用于对比测试,所以模式选取较少,更多组合的性能皆可参考[v2ray v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md)  
  3. Xray所有的性能大幅提升.
     - splice模式一骑绝尘.
     - direct模式始终稳定和裸奔基本一致
     - 其余组合相对v2ray也有大幅提升
  4. Xray中使用trojan协议使用XTLS也同样具有readV,性能参考VLESS的组合即可.
       - v2ray v4.32.1 中的trojan不具有readV
       - Xray中使用trojan协议并使用XTLS开挂,可能是目前最快的trojan实现.
  5. v2ray v4.32.1和v4.33.0没有性能的相关改动,基本一致,少许浮动为正常表现.
---
  ### 关于splice
  --- 
  - Splice 工作方式：
    - 读取数据时，Linux kernel 直接转发 TCP，不再经过 Xray 的内存
    - 可以减少至少两次数据拷贝，节省 CPU 切换，提升路由器、移动设备上的性能
  - Splice 使用场景限制：
    - linux平台,如安卓和路由器,以及用 linux 当桌面等使用场景
    - inbound:目前仅支持任意门/socks/http入站
    - outbound:支持XTLS的协议出站(目前仅VLESS)

  
---

### 常用组合性能测试对比二

(待补充更多数据)
可参考 [Xray v1.0.0版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201124.md) 中TG群友提供数据

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
* 测试二
  * 树莓派4B,3B+
  * 带宽1G
  * 公网IP
  
## 测试工具
* iperf 3.6
* v2ray 4.32.1(with VLESS XTLS readV)
* v2ray 4.33.0
* Xray v1.1.0
* trojan-go 0.8.2
* trojan-gfw 1.16.0
* nginx openresty/1.17.8.2

## 测试时间
    2020.12
