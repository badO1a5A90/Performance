
## 推荐使用的几种组合的传输速率对比

首先列出推荐组合，因为有些组合虽然快但并不安全
以下组合能相对保证安全，推荐使用（更多数据在后文）：

* VLESS over tcp, with TLS 
  - 典型使用场景为，服务器用途单一，如主要用于翻墙，回落网站只是伪装。
    ```
    703 Mbits/sec  sender
    688 Mbits/sec  receiver
    ```

* nginx---TCP本地端口自交--->VLESS over TCP， with TLS
  - 典型使用场景为，服务器用途多样化，需要前置服务器进行多种用途分流。
    ```
    572 Mbits/sec  sender
    551 Mbits/sec  receiver
    ```

* nginx---TCP本地端口自交--->vmess(aes-128-gcm) over ws， no TLS
  - 典型使用场景为，没有TLS证书/就是不想用TLS。
    ```
    436 Mbits/sec  sender
    414 Mbits/sec  receiver
    ```
* nginx(stream)---TCP本地端口自交--->vmess(aes-128-gcm) over TCP， no TLS 
  - 典型使用场景为，没有TLS证书/就是不想用TLS。
    ```
    625 Mbits/sec sender
    618 Mbits/sec  receiver
    ```
* 不推荐任何基于socks协议的组合，因为使用此协议，即便 with TLS，udp数据依旧是明文的。

## 测试环境
* 两台VPS，搬瓦工DC6和DC9机房vps最低配置/1Gbps带宽
* 系统，Debian 10

## 测试工具
* iperf3.6
* v2ray 4.26(with VLESS preview1.1)

## 测试方式
* 为实际的网络传输场景的速率测试，一台vps做iperf的服务端和v2ray服务端，另一台vps作为iperf客户端和v2ray客户端。客户端通过dokodemo-door传入v2ray，以各种不同测试方式中的协议及传输方式传输到v2ray服务端，服务端通过freedom传出到iperf的服务端。
* 单线程单连接无并发连接（实际上网肯定是多并发的）

## 测试时间
    2020.08

## 测试内容

* 测试了vmess和VLESS两种协议，分别在TCP和ws两种传输方式下，以及有无TLS的传输速率。
* 测试了vmess在TCP传输方式下，两种加密"aes-128-gcm"，"chacha20-poly1305"的传输速率。
* 测试了有前置分流情况下，部分组合的传输速率。

## 数据摘要

1.  vmess over tcp, no TLS （不建议实际应用） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.48 GBytes   710 Mbits/sec   46             sender
    [  5]   0.00-30.01  sec  2.43 GBytes   695 Mbits/sec                  receiver
    CPU Utilization: local/sender 6.7% (0.5%u/6.2%s), remote/receiver 4.9% (0.4%u/4.6%s)
    ```
2.  VLESS over tcp, no TLS （不建议实际应用） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.67 GBytes   764 Mbits/sec   57             sender
    [  5]   0.00-30.01  sec  2.63 GBytes   752 Mbits/sec                  receiver
    CPU Utilization: local/sender 7.4% (0.7%u/6.7%s), remote/receiver 1.1% (0.1%u/1.0%s)
    ```
3. vmess over tcp, with TLS （需其他软件前置分流，见下方带前置的测试）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.29 GBytes   655 Mbits/sec   28             sender
    [  5]   0.00-30.02  sec  2.25 GBytes   645 Mbits/sec                  receiver
    CPU Utilization: local/sender 6.6% (1.1%u/5.5%s), remote/receiver 19.7% (1.7%u/18.0%s)
    ```

4.  VLESS over tcp, with TLS （可直接前置，回落，推荐模式之一）  
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.45 GBytes   703 Mbits/sec   37             sender
    [  5]   0.00-30.05  sec  2.40 GBytes   688 Mbits/sec                  receiver
    CPU Utilization: local/sender 7.9% (0.6%u/7.2%s), remote/receiver 20.5% (1.8%u/18.7%s)
    ```

5.  vmess over ws ,no TLS （不建议实际应用）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.36 GBytes   676 Mbits/sec   45             sender
    [  5]   0.00-30.02  sec  2.29 GBytes   656 Mbits/sec                  receiver
    CPU Utilization: local/sender 6.8% (0.5%u/6.4%s), remote/receiver 20.6% (1.8%u/18.9%s)
    ```

6.  VLESS over ws, no TLS （不建议实际应用） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.42 GBytes   693 Mbits/sec   42             sender
    [  5]   0.00-30.03  sec  2.36 GBytes   676 Mbits/sec                  receiver
    CPU Utilization: local/sender 7.0% (0.7%u/6.3%s), remote/receiver 20.0% (1.7%u/18.3%s)
    ```

7.  vmess over ws, with TLS  （需其他软件前置分流，见下方带前置的测试） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  1.94 GBytes   556 Mbits/sec   37             sender
    [  5]   0.00-30.06  sec  1.91 GBytes   545 Mbits/sec                  receiver
    CPU Utilization: local/sender 6.0% (0.3%u/5.6%s), remote/receiver 19.7% (1.7%u/18.0%s)
    ```
8.  VLESS over ws, with TLS  （需其他软件前置分流，见下方带前置的测试）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.06 GBytes   590 Mbits/sec   29             sender
    [  5]   0.00-30.01  sec  2.03 GBytes   580 Mbits/sec                  receiver
    CPU Utilization: local/sender 6.7% (0.6%u/6.1%s), remote/receiver 19.4% (2.2%u/17.2%s)
    ```
9.  vmess(aes-128-gcm) over TCP， with TLS （no TLS下加密对速度几乎无影响，两次加密无太大实际意义）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  1.62 GBytes   463 Mbits/sec   46             sender
    [  5]   0.00-30.02  sec  1.58 GBytes   452 Mbits/sec                  receiver
    CPU Utilization: local/sender 5.4% (0.6%u/4.8%s), remote/receiver 2.9% (0.3%u/2.6%s)
    ```
10. vmess(chacha20-poly1305) over TCP， with TLS （no TLS下加密对速度几乎无影响，两次加密无太大实际意义）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  1.44 GBytes   411 Mbits/sec   41             sender
    [  5]   0.00-30.04  sec  1.39 GBytes   397 Mbits/sec                  receiver
    CPU Utilization: local/sender 5.2% (0.8%u/4.4%s), remote/receiver 0.6% (0.1%u/0.6%s)
    ```
11. nginx---TCP本地端口自交--->VLESS over TCP， with TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [  5]   0.00-30.00  sec  2.00 GBytes   572 Mbits/sec   27             sender
    [  5]   0.00-30.02  sec  1.93 GBytes   551 Mbits/sec                  receiver
    ```

12. nginx---domain socket--->VLESS over TCP， with TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [  5]   0.00-30.00  sec  1.71 GBytes   488 Mbits/sec   32             sender
    [  5]   0.00-30.01  sec  1.66 GBytes   474 Mbits/sec                  receiver
    ```
13. nginx---TCP本地端口自交--->vmess(aes-128-gcm) over ws， no TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  1.52 GBytes   436 Mbits/sec   33             sender
    [  5]   0.00-30.03  sec  1.45 GBytes   414 Mbits/sec                  receiver
    ```
14. nginx(stream)---TCP本地端口自交--->vmess(aes-128-gcm) over TCP， no TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  2.18 GBytes   625 Mbits/sec   36             sender
    [  5]   0.00-30.03  sec  2.16 GBytes   618 Mbits/sec                  receiver
    ```

## 总结

1.  在有足够基础带宽的条件下，VLESS比vmess的性能有提升，能多转化出5%-8%的带宽。
    - 如果带宽足够而硬件性能有限，选择VLESS可以提高速度。

2.  在测试环境的硬件条件下，VLESS和vmess在1Gbps带宽下，均跑出500Mbps-700Mbps以上的数据。
    - 所以如果硬件性能尚可，基础带宽才是速度瓶颈情况下（比如常见的家用100M宽带），选择VLESS和vmess都可以轻易跑满带宽；但是VLESS可以节约流量，节约cpu，节约电量（理论推测）

3.  TCP和ws的对比，其余条件相同，ws要比TCP慢10%-15%。

4.  TLS
    - 在TCP下约损失8%左右速度，ws下损失的速度更为明显。
    - 但是出于安全考虑，使用VLESS和未加密的vmess协议，无论TCP还是ws，都建议加上TLS。 

5.  vmess的加密
    - TCP，TLS下，两种方式加密损失约35%-40%
    - TCP，no TLS下，两种方式加密速度均无明显变化（所以数据不在列表中）
    - 这个对比疑似有bug存在？

6.  前置nginx或其他分流器
    - VLESS over TCP， with TLS的基础模式下，前置nginx分流，走端口自交模式降低15-20%，走ds模式降低30%
    - 其他分流器待测试