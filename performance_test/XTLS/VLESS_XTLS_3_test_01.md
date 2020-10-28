请移步至[第二次测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_3_test_02.md)

## 测试环境
* 两台VPS：
    - CPU Model name:Intel Core Processor (Haswell, no TSX, IBRS)*1 
    - CPU MHz: 2399.996
    - 1G内存
    - 2.5Gbps上下行带宽
    - 公网IP
* 系统：Debian 10

## 测试工具
* iperf3.6
* v2ray 4.28.2(with VLESS preview1.5, XTLS)
* ethr

## 测试时间
    2020.09

## 测试内容
* 在多线程使用iperf，满载CPU的状态下，以三种方式对比速率，每种方式进行了3次测试
* 使用ethr的HTTPS测速模式，满载CPU的状态下，进行TLS和XTLS的对比测试。

## 重要TIP！
* XTLS在硬件性能较低，比如路由器和树莓派等CPU较弱的硬件上，表现的更好。
* 硬件性能越低，可带来的提升就越大（有用户测试可提升一倍性能），尤其是没有AES硬解的硬件系统上。
* TLS对性能的影响在各种v2ray配置组合中影响都很大（可参见之前的4.27版本的性能测试）。本文数据目前仅仅是一个初步测试，也不代表实用效果，尚有更多的对照组待测试，旨在为XTLS的更优化提供更多的数据参考信息。
* XTLS对性能的影响受各自使用环境和习惯远比替换协议大的多，所以更建议亲自使用，体验，测试。

## 测试1数据（iperf
------------------------

1. 基准的 VLESS over TCP with TLS

    数据流向
    * iperf client-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP with TLS)
    ---->v2server,inbound(VLESS over TCP with TLS),outbound(freedom)-->iperf server
    ```
    [SUM]   1.49 Gbits/sec               sender
    [SUM]   1.11 Gbits/sec                  receiver

    [SUM]   1.46 Gbits/sec               sender
    [SUM]   1.09 Gbits/sec                  receiver

    [SUM]   1.46 Gbits/sec               sender
    [SUM]   1.01 Gbits/sec                  receiver
    ```
------------

2. 将数据进行TLS加密后，通过 VLESS over TCP with TLS 测试

    数据流向
    * iperf client-->v2client,inbound(dokodemo-door),outbound(freedom with TLS)-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP with TLS)
    -->v2server,inbound(VLESS over TCP with TLS),outbound(freedom)-->v2server,inbound(dokodemo-door with TLS),outbound(freedom)-->iperf server

    ```
    [SUM]   892 Mbits/sec                sender
    [SUM]   495 Mbits/sec                  receiver

    [SUM]   889 Mbits/sec               sender
    [SUM]   481 Mbits/sec                  receiver

    [SUM]   869 Mbits/sec                sender
    [SUM]   488 Mbits/sec                  receiver
    ```

----------------------

3. 将数据进行TLS加密后，通过 VLESS over TCP with XTLS 测试

    数据流向
    * iperf client-->v2client,inbound(dokodemo-door),outbound(freedom over TCP with TLS)-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP with XTLS)
    -->v2server,inbound(VLESS over TCP with XTLS),outbound(freedom)-->v2server,inbound(dokodemo-door with TLS),outbound(freedom)-->iperf server

    ```
    [SUM]   735 Mbits/sec                sender
    [SUM]   393 Mbits/sec                  receiver

    [SUM]   769 Mbits/sec                sender
    [SUM]   405 Mbits/sec                  receiver

    [SUM]   737 Mbits/sec                sender
    [SUM]   391 Mbits/sec                  receiver
    ```

## 总结

* 很不可思议的XTLS居然低于TLS？

* 设置环境变量V2RAY_VLESS_XTLS_SHOW=true后，观察发现XTLS的数据包大小均为2070，是否因为第一次TLS加密时候使用的也是v2ray的TLS的缘故导致XTLS低于TLS？

* 换用microsoft的ethr进行测试，此工具可以直接进行HTTPS的速率测试。[https://github.com/microsoft/ethr]

## 测试2数据（ethr）
- - - - - - - - - - - - - - - - - - - - - - -

1. ethr HTTPS模式直连
    ```
    Parallel Sessions=10，Length of buffer to use=1MB
    均值：
    737MBits/s

    Parallel Sessions=10，Length of buffer to use=16KB
    均值：
    260MBits/s
    ```
    

- - - - - - - - - - - - - - - - - - - - - - -
2. ethr HTTPS模式 + VLESS over TCP
    数据流向
    * ethr client-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP)
    ---->v2server,inbound(VLESS over TCP),outbound(freedom)-->ethr server
    ```
    Parallel Sessions=10，Length of buffer to use=1MB
    均值：
    500.8 MBits/s
    497.5 MBits/s
    471.2 MBits/s

    Parallel Sessions=10，Length of buffer to use=16KB
    均值：
    153.4 MBits/s
    147.6 MBits/s
    147.8 MBits/s

    ```
- - - - - - - - - - - - - - - - - - - - - - -

3. ethr HTTPS模式 + VLESS over TCP with TLS

    数据流向
    * ethr client-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP with TLS)
    ---->v2server,inbound(VLESS over TCP with TLS),outbound(freedom)-->ethr server
    ```
    Parallel Sessions=10，Length of buffer to use=1MB
    三次均值分别为： 
    360.8 MBits/s
    363.2 MBits/s
    335.6 MBits/s

    Parallel Sessions=10，Length of buffer to use=16KB
    三次均值分别为： 
    107.2 MBits/s
    110.1 MBits/s
    107.5 MBits/s
    ```

  - - - - - - - - - - - - - - - - - - - - - - -
4. ethr HTTPS模式 + VLESS over TCP with XTLS

    数据流向
    * ethr client-->v2client,inbound(dokodemo-door),outbound(VLESS over TCP with XTLS)
    ---->v2server,inbound(VLESS over TCP with XTLS),outbound(freedom)-->ethr server
    ```
    Parallel Sessions=10，Length of buffer to use=1MB
    三次均值分别为：
    394.4 MBits/s
    395.2 MBits/s
    404.8 MBits/s

    Parallel Sessions=10，Length of buffer to use=16KB      
    三次均值分别为：
    115.5 MBits/s
    114.8 MBits/s
    118.6 MBits/s
    ```

## 总结

* CPU都满负载
* XTLS的数据包大小均以4118为主
* ethr自身的Length of buffer to use=1MB时速率与iperf测试较为接近，可能比较接近上限，16KB时速率非常低
* XTLS对比TLS都略高，但并不明显
* 是否还有更合适的工具或者方式构建测试？
