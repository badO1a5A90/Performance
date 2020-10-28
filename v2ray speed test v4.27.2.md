
## 推荐使用的几种组合的传输速率对比

首先列出推荐组合，因为有些组合虽然快但并不安全或缺乏伪装。

以下组合能相对保证安全，并且通过分流或回落具有较高伪装性，推荐使用（更多数据在后文）：

* VLESS over tcp, with TLS--->回落
    - 典型使用场景为，各种场景。
    - 在4.27.2中VLESS的回落得到了增强，可以根据path进行分流，已可以适应各种场景
    - 性能最强，同时VLESS回落分流的几乎不损失性能，高于nginx的ws分流（约11%）
    ```
    832 Mbits/sec  sender
    828 Mbits/sec  receiver
    ```
* nginx(ws分流)--->vmess over ws, with TLS
    - 典型使用场景为，各种场景。
    - 目前vmess比较流行的标准配置方式之一（白话文的WebSocket + TLS + Web）
    ```
    535 Mbits/sec  
    530 Mbits/sec  
    ```
*  nginx(ws分流)--->vmess(aes-128-gcm) over ws, no TLS
    - 典型使用场景为，没有TLS证书/就是不想用TLS。
    ```
    693 Mbits/sec sender
    689 Mbits/sec  receiver
    ```
*  nginx(sni分流)--->VLESS over TCP, with TLS
    - 典型使用场景为，服务器用途复杂，习惯用前置服务器进行多种用途分流。
    - 也可使用vmess协议（性能低一些）
    - 此方式类似白话文的TCP + TLS + Web
    ```
    776 Mbits/sec sender
    772 Mbits/sec  receiver
    ```
* 不推荐任何基于socks协议的组合，因为使用此协议，即便 with TLS，udp数据依旧是明文的。

* 因4.26时测试v2ray的ds反而落后于端口自交（与理论相悖），并疑似可能存在问题，所以本次测试前置服务器分流至v2ray均为端口自交模式。

* 以上配置的简单模板可以在v2ray模板仓库找，有空可能会写一下白话文教程。

## 关于VLESS和vmess性能对比

一个重要的TIP：

目前常见对比VLESS和vmess的测试，是在WebSocket+TLS+Web方案中，把vmess简单的替换为VLESS来对比。这种性能测试对比VLESS的性能提升可能只在10%左右（4.26版本测试8%，本次测试提升在14%左右，两次硬件环境不同）。

但是目前VLESS回落机制已经相当完善，可以前置适用很多场景，并且回落分流的性能甚至超过nginx，所以两者的使用方式是完全不同的，实际性能的对比，应该使用
    - VLESS over tcp, with TLS--->回落 对比 WebSocket+TLS+Web（也即 nginx(ws分流)--->vmess over ws, with TLS）

这样的使用方式下，VLESS的性能相较传统模式可以提高 50%+，是的，你没有看错，是50%+。


## 测试环境
* 两台VPS：
    - Intel Xeon Processor2.6GHz*2
    - 4G内存
    - 2.5Gbps上下行带宽
    - 公网IP
* 系统：Debian 10

## 测试工具
* iperf3.6
* v2ray 4.27.2(with VLESS preview1.5)
* nginx为openresty/1.17.8.2

## 测试方式
* 为实际的网络传输场景的速率测试，一台vps做iperf的服务端和v2ray服务端，另一台vps作为iperf客户端和v2ray客户端。客户端通过dokodemo-door传入v2ray，以各种不同测试方式中的协议及传输方式传输到v2ray服务端/前置服务器分流，服务端通过freedom传出到iperf的服务端。
* 单线程单连接无并发连接（当然实际上网肯定是多并发的）

## 测试时间
    2020.08

## 测试内容

* 测试了vmess和VLESS两种协议，分别在TCP和ws两种传输方式下，以及有无TLS的传输速率。
* 测试了vmess在ws传输方式下，两种加密（"aes-128-gcm"，"chacha20-poly1305"）的传输速率。
* 测试了有前置分流情况下，部分组合的传输速率。
* 特别对比了VLESS回落分流和nginx分流的效率。

## 性能测试数据摘要


1.  VLESS over tcp, no TLS （不建议实际应用） (此组数据受到2.5Gbps网卡口上限限制)
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  17.3 GBytes  2.47 Gbits/sec  130             sender
    [  5]   0.00-60.01  sec  17.2 GBytes  2.47 Gbits/sec                  receiver
    CPU Utilization: local/sender 32.3% (2.0%u/30.2%s), remote/receiver 13.7% (1.0%u/12.7%s)
    ```

2.  VLESS over tcp, with TLS （可直接前置，回落分流，推荐模式之一）  
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  5.81 GBytes   832 Mbits/sec  381             sender
    [  5]   0.00-60.00  sec  5.79 GBytes   828 Mbits/sec                  receiver
    CPU Utilization: local/sender 12.3% (0.8%u/11.5%s), remote/receiver 17.5% (2.8%u/14.7%s)
    ```

3.  VLESS over ws, no TLS （不建议实际应用） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [  5]   0.00-60.00  sec  6.62 GBytes   947 Mbits/sec  171             sender
    [  5]   0.00-60.03  sec  6.59 GBytes   943 Mbits/sec                  receiver
    CPU Utilization: local/sender 13.5% (1.5%u/12.0%s), remote/receiver 24.7% (3.1%u/21.6%s)
    ```

4.  VLESS over ws, with TLS  （需其他软件前置分流）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  4.59 GBytes   657 Mbits/sec  530             sender
    [  5]   0.00-60.00  sec  4.57 GBytes   654 Mbits/sec                  receiver
    CPU Utilization: local/sender 10.9% (0.8%u/10.1%s), remote/receiver 16.6% (2.6%u/14.0%s)
    ```

5.  vmess over tcp, no TLS （不建议实际应用） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  15.1 GBytes  2.17 Gbits/sec  118             sender
    [  5]   0.00-60.01  sec  15.1 GBytes  2.16 Gbits/sec                  receiver
    CPU Utilization: local/sender 28.8% (2.2%u/26.6%s), remote/receiver 5.5% (0.8%u/4.7%s)
    ```

6. vmess over tcp, with TLS （需其他软件前置分流，见下方带前置的测试）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  4.94 GBytes   707 Mbits/sec  463             sender
    [  5]   0.00-60.00  sec  4.92 GBytes   704 Mbits/sec                  receiver
    CPU Utilization: local/sender 11.1% (1.0%u/10.1%s), remote/receiver 16.9% (2.5%u/14.4%s)
    ```

7.  vmess over ws ,no TLS （不建议实际应用）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  5.53 GBytes   792 Mbits/sec  346             sender
    [  5]   0.00-60.01  sec  5.51 GBytes   789 Mbits/sec                  receiver
    CPU Utilization: local/sender 14.5% (1.1%u/13.4%s), remote/receiver 22.1% (3.0%u/19.1%s)
    ```

8.  vmess over ws, with TLS  （需其他软件前置分流，见下方带前置的测试） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  3.92 GBytes   561 Mbits/sec  483             sender
    [  5]   0.00-60.01  sec  3.91 GBytes   559 Mbits/sec                  receiver
    CPU Utilization: local/sender 9.8% (1.0%u/8.9%s), remote/receiver 15.6% (2.3%u/13.3%s)
    ```

9.  vmess(aes-128-gcm) over ws, no TLS （需其他软件前置分流，见下方带前置的测试） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  4.95 GBytes   709 Mbits/sec  344             sender
    [  5]   0.00-60.00  sec  4.91 GBytes   703 Mbits/sec                  receiver
    CPU Utilization: local/sender 11.3% (1.0%u/10.3%s), remote/receiver 17.4% (2.7%u/14.7%s)
    ```
10. vmess(chacha20-poly1305) over ws, no TLS （需其他软件前置分流，见下方带前置的测试） 
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [  5]   0.00-60.00  sec  4.41 GBytes   631 Mbits/sec  463             sender
    [  5]   0.00-60.00  sec  4.38 GBytes   628 Mbits/sec                  receiver
    CPU Utilization: local/sender 10.4% (1.9%u/8.5%s), remote/receiver 16.1% (2.5%u/13.6%s)
    ```
11. vmess(aes-128-gcm) over ws, with TLS （不建议实际应用，两次加密无太大实际意义）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  3.53 GBytes   505 Mbits/sec  384             sender
    [  5]   0.00-60.01  sec  3.51 GBytes   502 Mbits/sec                  receiver
    CPU Utilization: local/sender 9.3% (0.8%u/8.5%s), remote/receiver 14.9% (2.1%u/12.8%s)
    ```
12. vmess(chacha20-poly1305) over ws, with TLS （不建议实际应用，两次加密无太大实际意义）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    Test Complete. Summary Results:
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  3.17 GBytes   454 Mbits/sec  263             sender
    [  5]   0.00-60.00  sec  3.14 GBytes   449 Mbits/sec                  receiver
    CPU Utilization: local/sender 14.5% (1.4%u/13.0%s), remote/receiver 13.1% (2.0%u/11.1%s)
    ```

13. nginx(ws分流)--->vmess over ws, with TLS （推荐模式之一，也是目前比较流行模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-55.00  sec  3.42 GBytes   535 Mbits/sec  369             sender
    [  5]   0.00-55.00  sec  3.39 GBytes   530 Mbits/sec                  receiver
    CPU Utilization: local/sender 14.6% (1.3%u/13.3%s), remote/receiver 26.2% (3.9%u/22.3%s)
    ```
14. nginx(ws分流)--->vmess(aes-128-gcm) over ws, no TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-55.00  sec  4.44 GBytes   693 Mbits/sec  430             sender
    [  5]   0.00-55.00  sec  4.41 GBytes   689 Mbits/sec                  receiver
    CPU Utilization: local/sender 15.2% (0.9%u/14.3%s), remote/receiver 19.1% (2.7%u/16.3%s)
    ```
15. nginx(sni分流)--->vmess over tcp, with TLS
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-55.00  sec  4.46 GBytes   697 Mbits/sec  402             sender
    [  5]   0.00-55.00  sec  4.44 GBytes   693 Mbits/sec                  receiver
    CPU Utilization: local/sender 11.2% (1.1%u/10.1%s), remote/receiver 19.2% (2.7%u/16.5%s)
    ```
16. nginx(sni分流)--->VLESS over TCP, with TLS （推荐模式之一）
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-60.00  sec  5.42 GBytes   776 Mbits/sec  395             sender
    [  5]   0.00-60.00  sec  5.39 GBytes   772 Mbits/sec                  receiver
    CPU Utilization: local/sender 14.4% (1.0%u/13.4%s), remote/receiver 2.9% (0.4%u/2.5%s)
    ```

## 总结

1.  在有足够基础带宽的条件下，VLESS比vmess的性能有提升，能多转化出10%+的带宽(比4.26版本测试提升更多)。
    - VLESS over ws, with TLS 对比 vmess over ws, with TLS, 提升18%
    - VLESS over tcp, with TLS 对比 vmess over tcp, with TLS, 提升14%
    - 如果带宽足够而硬件性能有限，选择VLESS可以提高速度。
    - 但VLESS在这个版本后，使用方式与vmess不同，实际使用对比在50%+（请回看本文开头部分）。

2.  在测试环境的硬件条件下，VLESS和vmess在1Gbps带宽下，均跑出500Mbps-1000Mbps的数据。
    - 所以如果硬件性能尚可，基础带宽才是速度瓶颈情况下（比如常见的家用100M宽带），选择VLESS和vmess都可以轻易跑满带宽；但是VLESS可以节约流量，节约cpu，节约电量（理论推测）

3.  TCP和ws的对比，ws要比TCP慢非常多。（请投入TCP的怀抱吧！）
    - 4.26版本的测试和本次测试第一个版本受到了速率上限的限制，因此认为TCP和ws差距在20%左右结论并不准确
    - 新一轮测试中，VLESS over tcp, no TLS 依旧跑到了网卡口速率2.5Gbps上限。
    - 因此给出一组在另一组高性能/10Gbps口vps上的测试对比更为直观：
        - 直连： 8.96 Gbits/sec
        - VLESS over TCP： 6.56 Gbits/sec
        - vmess over TCP： 2.72 Gbits/sec
        - VLESS over ws： 1.41 Gbits/sec
        - vmess over ws： 1.07 Gbits/sec

4.  TLS，各种组合对比都降低了约30%速率
    - 这次测试TLS对速率的影响明显比4.26版本测试大，可能是两次测试CPU的差距。
    - 出于安全考虑，使用VLESS和未加密的vmess协议，无论TCP还是ws，都建议加上TLS。 

5.  vmess的加密
    - aes-128-gcm， 损失10%左右性能
    - chacha20-poly1305， 损失20%左右性能
    - 加密对速率的影响明显比4.26版本测试大，可能是两次测试CPU的差距。

6.  前置nginx或其他分流器
    - 前置nginx(openresty)的四种方案，对速率的影响程度在3%-7%之间。（似乎也印证了VLESS回落分流几乎无性能损失所以比nginx分流也高出6-7%）
    - 其他分流器待测试

## VLESS分流和nginx分流性能的对比

1. VLESS
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [  5]   0.00-30.00  sec  2.50 GBytes   715 Mbits/sec  221             sender
    [  5]   0.00-30.00  sec  2.48 GBytes   709 Mbits/sec                  receiver
    ```
2. openresty
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [  5]   0.00-30.00  sec  2.34 GBytes   670 Mbits/sec  199             sender
    [  5]   0.00-30.01  sec  2.31 GBytes   661 Mbits/sec                  receiver
    ```
3. nginx
    - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
    [  5]   0.00-30.00  sec  2.24 GBytes   642 Mbits/sec  202             sender
    [  5]   0.00-30.01  sec  2.22 GBytes   635 Mbits/sec                  receiver
   ```
- 结论
    - VLESS 比 openresty高约6-7%，比标准nginx版本高约11%。
- 数据流
    - VLESS：iperf客户端—>dokodemo-door(in),freedom over ws with tls(out)—->服务端VLESS over tcp with TLS(fallback by path)—->dokodemo-door over ws(in),freedom(out)—->iperf服务端
    - nginx：iperf客户端—>dokodemo-door(in),freedom over ws with tls(out)—->nginx(openrestry)(proxy_pass by path)—->dokodemo-door over ws(in),freedom(out)—->iperf服务端
    - 数据流向除分流方式不同，其余均一致，用于对比分流效率
    - TLS的处理为v2ray或nginx分别各自处理，可能对效率稍有影响。    

