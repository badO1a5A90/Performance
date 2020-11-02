## 测试环境
* VPS*4：
    - CPU Model name:Intel Core Processor (Haswell, no TSX, IBRS)*1 
    - CPU MHz: 2399.996
    - 支持AES指令
    - 1G内存
    - 2.5Gbps上下行带宽
    - 公网IP
* 系统：Debian 10

## 测试工具
* v2ray 4.29.0(with VLESS preview 2, XTLS)
* iperf3.6
* ethr

## 测试时间
2020.09

## 2020.11 更新
  * 关于下面结论,更多请参考[第四次测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)

## 2020.10 更新
  * 在第三次测试时发现,测试模式中数据未能100%被XTLS处理,并且不可知多少部分被XTLS处理,所以
    * 提升仅能代表部分数据被XTLS处理的提升效果
    * 并不能代表XTLS可以带来的最大提升效果.
    * 待有能进行100%XTLS处理的测试方式后重新测试.


## 结论
* 经过多次测试后，可以确认
  * XTLS相比较TLS，会有较明显的性能提升，尤其大流量场景，如测速、视频、下载时. 即使在有AES硬解的设备上也可以提升40%-50%.
  * 硬件性能越低，可带来的提升就越大，（尤其没有AES硬解的设备上，有用户有翻倍的提升）
  * 无论你使用什么，XTLS都是一个非常值得尝试的功能，能带来性能提升，减少系统开销，甚至节约电量等好处。
  * 对比没有二次加密的性能，XTLS仍有较大优化空间，尤其是接收和解密部分,可能还存在一些缺陷。
  * TLS对性能的影响在各种v2ray配置组合中影响都很大（更多数据可参见之前的4.27版本的配置组合和性能测试）。

## 重要TIP！
* XTLS对性能的提升受各自使用环境和习惯影响较大，并不是替换协议和配置组合方式的较固定的提升比例。
* 基于上一点，强烈建议亲自使用，体验，测试，贡献数据。
* 本文数据旨在为XTLS的更优化提供更多的数据参考信息，并不代表其他硬件环境实际使用效果。
* 文中所有数据和结论可以随意使用甚至不必提及原作者

## 测试方式
* 由于第一次测试只使用了2台硬件，并且TLS的加密和再次加密均在同一进程，各种因素互相影响较大，不易控制流程中各个环节和因素，也无法进行细致比较分析。  
  因此这次测试将数据流向的各个环节完全拆开，并且使用CPUlimit来控制各环节的CPU使用上限
  * 令第一次TLS加密解密对测试无影响,变量仅为二次加密,来准确对比XTLS和TLS的性能
  * 可以模拟使用中间硬件的实用场景
  * 可以去除各种因素相互影响
  * 可以控制性能瓶颈进行不同测试对照
  * 可以分别分析各数据处理环节对性能的影响
* 使用4台VPS功能如下，以下简单命名4台VPS为，ABCD
  - A 负责使用测试工具客户端产生TLS加密数据，发送给B
  - B 负责使用v2ray以各种方式对A产生的数据进行处理并转发给C（无二次加密/TLS/XTLS等方式），也即类似一些用户场景的中间设备。
  - C 负责使用v2ray接收和以各种方式处理B的数据（无二次加密/TLS/XTLS等方式），发送给D
  - D 负责处理和接收数据并发送至测试工具服务端
* 使用CPUlimit控制各个数据处理节点的CPU使用上限
  - 测试的主要目的是，对比BC处理过程中无二次加密/TLS/XTLS的性能
  - 因此，AD的CPU上限不进行限制，只进行BC的CPU限制，尽量保证对非上限评估的测试中，AD的CPU负荷不会满载（限制BC为瓶颈，也同时使AD不会成为瓶颈）
* 多线程使用iperf/ethr

## 测试数据1（iperf)
*实际测试的各种组合形式（比如实际还包括了对AD无TLS的测试+各种CPU限制+各种线程数量等各种组合+各种次数）数据过多，冗长无趣，也没有完整罗列数据细节的意义，所以此部分列举的均为数据总结和有意义的对比。*

- 再次理解一下ABCD
  - A 初始加密数据制造
  - B 对数据二次处理发送至C（无二次加密/TLS二次加密/XTLS）
  - C 接收B的数据进行（无二次解密/TLS二次解密/XTLS）
  - D 数据处理和接收并发送至测试工具服务端
- 对照的数据组中同一CPU限制下，唯一区别仅为B-->C的方式（无二次加密/TLS/XTLS）
- iperf线程均为5
- TLS record 2K
- 数据为3次30S的平均数据
- 都有硬解（还不知道如何关闭

* 数据流向：
  - iperf client-->A,inbound(dokodemo-door),outbound(freedom with TLS)  
    -->B,inbound(dokodemo-door),outbound(VLESS over TCP/with TLS/with XTLS)  
    -->C,inbound(VLESS over TCP/with TLS/with XTLS),outbound(freedom)  
    -->D,inbound(dokodemo-door with TLS),outbound(freedom)-->iperf server

---
1. 限制B的CPU使用上限为5%（类似于使用了一个低性能中间设备比如树莓派上网的场景）
```
VLESS over TCP
487 Mbits/sec                sender
405 Mbits/sec                  receiver           

VLESS over TCP with TLS
192 Mbits/sec                sender
142 Mbits/sec                  receiver

VLESS over TCP with XTLS
261 Mbits/sec                sender
217 Mbits/sec                  receiver  
```

2. 限制B的CPU使用上限为10%（类似于使用了一个低性能中间设备比如树莓派上网的场景）
```
VLESS over TCP
831 Mbits/sec                sender
741 Mbits/sec                  receiver           

VLESS over TCP with TLS
338 Mbits/sec                sender
278 Mbits/sec                  receiver

VLESS over TCP with XTLS
474 Mbits/sec                sender
421 Mbits/sec                  receiver  
```

* CPU
  * AC均维持较低的CPU使用状态
  * B的CPU使用上限为10%的 无二次加密 模式下，D的CPU使用率100%（iperf占30%），可认为此时因为D达到速度瓶颈，即此测试环境下的理想速度。
  * with TLS/with XTLS模式下， 此时的瓶颈明显在B，D的CPU使用率最高约可以达到60%（AC较低），因此未拆分硬件时的测试中最后的TLS解码可能是很大的影响因素
* 无加密比XTLS高80%，XTLS性能比TLS高35-40%，性能顺序符合理论理想
* 可以认为，当使用低性能中间设备时，XTLS比TLS已经有较大提升，但XTLS如果想达到无二次加密的理想速度，还有优化空间。
---  
  
3. 限制C的CPU使用上限为5%（类似于使用了一个低性能VPS的场景）
```
VLESS over TCP
636 Mbits/sec                sender
477 Mbits/sec                  receiver           

VLESS over TCP with TLS
186 Mbits/sec                sender
95 Mbits/sec                  receiver

VLESS over TCP with XTLS
165 Mbits/sec                sender
75 Mbits/sec                  receiver  
```

4. 限制C的CPU使用上限为10%（类似于使用了一个低性能VPS的场景）
```
VLESS over TCP
947 Mbits/sec                sender
765 Mbits/sec                  receiver           

VLESS over TCP with TLS
253 Mbits/sec                sender
169 Mbits/sec                  receiver

VLESS over TCP with XTLS
244 Mbits/sec                sender
143 Mbits/sec                  receiver  
```
* CPU
  * AB均维持较低的CPU使用状态
  * C的CPU使用上限为10%的 无二次加密 模式下，D的CPU使用率100%，可认为此时因为D达到速度瓶颈，即此测试环境下的理想速度。
  * with TLS/with XTLS模式下， 此时的瓶颈明显在C，D的CPU使用率也并不过高
* 限制同样程度的CPU，C对性能的影响比B更为明显
* 与第一次未拆分硬件测试类似的迷惑又出现了，XTLS略低于TLS？
---
5. 限制BC的CPU使用上限均为10%
```
VLESS over TCP
785 Mbits/sec                sender
655 Mbits/sec                  receiver           

VLESS over TCP with TLS
251 Mbits/sec                sender
162 Mbits/sec                  receiver

VLESS over TCP with XTLS
225 Mbits/sec                sender
136 Mbits/sec                  receiver  
```


* 无二次加密的性能略低了一些，其余和4基本一致，因为瓶颈在C。
---
6. 不限制CPU上限
```
VLESS over TCP
870 Mbits/sec                sender
732 Mbits/sec                  receiver           

VLESS over TCP with TLS
831 Mbits/sec                sender
726 Mbits/sec                  receiver

VLESS over TCP with XTLS
856 Mbits/sec                sender
756 Mbits/sec                  receiver  
```

* 不限制CPU模式下，3组数据基本相当，也与其他各组数据的理想速率基本一致，因为此时数据瓶颈在D（BC限制在约30%+开始，D已成为瓶颈）
---
7. AD不进行加密解密，纯转发，BC之间VLESS over TCP
```
VLESS over TCP
4146 Mbits/sec                sender
3983 Mbits/sec                  receiver           
```
* 这组数据可以对照6-VLESS over TCP，即去掉第一层的TLS速度提升近5倍
* 由于一次加密后速度上限的限制，无法判断二次加密对性能的真实影响（是否也是5倍）
---
## 总结
* 未拆分硬件测试时的迷惑（XTLS某个情况下低于TLS）仍在，能更确定这个现象因为负责处理入站解密的硬件性能较差引起的
* v2ray处理入站和解密似乎更消耗CPU（D满负荷时，A大约只在40%+）？
* 类似上一条，硬件性能对入站和解密的影响，似乎要大于出站和加密的影响（数据34对比数据12）。
* ~~似乎二次加密对性能的影响小于一次加密对性能的影响？~~  
  
## 测试数据2（ethr) 4KB
- ethr以HTTPS协议进行测试
- TLS record 大小 4KB
- ABCD功能
  - A ethr client
  - B 无二次加密/TLS二次加密/XTLS
  - C 无二次解密/TLS二次解密/XTLS
  - D ethr server
- 对照的数据组中同一CPU限制下，唯一区别仅为B-->C的方式（无二次加密/TLS/XTLS）
- 都有硬解（还不知道如何关闭

* 数据流向：
  - A，ethr client  
  -->B,inbound(dokodemo-door),outbound(VLESS over TCP/with TLS/with XTLS)  
  -->C,inbound(VLESS over TCP/with TLS/with XTLS),outbound(freedom)  
  -->D,ethr server
---
1. 限制b的CPU使用上限为10%
```
VLESS over TCP
486 Mbits/sec   

VLESS over TCP with TLS
225 Mbits/sec

VLESS over TCP with XTLS
313 Mbits/sec               
```

2. 限制b的CPU使用上限为20%
```
VLESS over TCP
831 Mbits/sec   

VLESS over TCP with TLS
433 Mbits/sec

VLESS over TCP with XTLS
566 Mbits/sec              
```

3. 限制b的CPU使用上限为30%
```
VLESS over TCP
1100 Mbits/sec   

VLESS over TCP with TLS
585 Mbits/sec

VLESS over TCP with XTLS
749 Mbits/sec              
```


* 因为TLS加密数据由ethr制造，效能高了很多，当B/C限制CPU在30%+时，D才会到达瓶颈（ethr占满CPU，速率上限1.1G/bps左右，注：非网卡上限，网卡2.5G口），当然，此上限比iperf组合上限高
* 无加密比XTLS高50-60%（比iperf组接近）
* XTLS性能比TLS高30-40%（与iperf组一致）
* 可以确认，当使用低性能中间设备时，XTLS比TLS已经有较大提升（如无硬解将更加明显），但还有优化空间。
---

4. 限制C的CPU使用上限为10%
```
VLESS over TCP
404 Mbits/sec   

VLESS over TCP with TLS
157 Mbits/sec

VLESS over TCP with XTLS
174 Mbits/sec               
```

5. 限制C的CPU使用上限为20%
```
VLESS over TCP
802 Mbits/sec   

VLESS over TCP with TLS
315 Mbits/sec

VLESS over TCP with XTLS
340 Mbits/sec    
```

6. 限制C的CPU使用上限为30%
```
VLESS over TCP
1030 Mbits/sec   

VLESS over TCP with TLS
432 Mbits/sec

VLESS over TCP with XTLS
506 Mbits/sec    
```



* 与123组数据类似，C限制CPU在30%+时，D才会到达瓶颈（ethr占满CPU，速率上限1G/bps左右
* 与iperf组最大的不同在于，性能顺序符合理论推测
  * 无加密比XTLS高100%+
  * XTLS性能比TLS高10-15%
* 限制同样程度的CPU，C对性能的影响比B更为明显（与iperf组一致）
---
1. 限制BC的CPU使用上限为20%
```
VLESS over TCP
788 Mbits/sec   

VLESS over TCP with TLS
305 Mbits/sec

VLESS over TCP with XTLS
349 Mbits/sec    
```

* 当BC同时限制CPU为20%，数据和5一致，因为性能瓶颈在C（与iperf组一致
---
8. 无限制
```
VLESS over TCP
1460 Mbits/sec   

VLESS over TCP with TLS
1080 Mbits/sec

VLESS over TCP with XTLS
1050 Mbits/sec               
```
* 没什么好说的
---
## 总结
* 与iperf组得到的结论几乎都一致
  * 这组测试环境下XTLS对比TLS约提高40%性能。
  * v2ray处理入站和解密更消耗CPU（补充观察，B的CPU限制在10%进行测试时，C的CPU消耗在15%）
  * 硬件性能对入站和解密的影响，要大于出站和加密的影响
  * 仍有优化空间，尤其入站和解密
* 与iperf组不一致，XTLS在C性能受限时，性能也高于TLS（符合理论理想）

## 测试数据3（ethr) 16KB
- ethr以HTTPS协议进行测试
- TLS record 大小 16KB
- 本组测试主要对比与4KB TLS record时的性能提升差异
---
1. 限制b的CPU使用上限为10%
```
VLESS over TCP
503 Mbits/sec   

VLESS over TCP with TLS
245 Mbits/sec

VLESS over TCP with XTLS
313 Mbits/sec               
```

2. 限制b的CPU使用上限为20%
```
VLESS over TCP
837 Mbits/sec   

VLESS over TCP with TLS
452 Mbits/sec

VLESS over TCP with XTLS
606 Mbits/sec              
```

* 各方式性能差距与4K时候一致
---
3. 限制C的CPU使用上限为10%
```
VLESS over TCP
469 Mbits/sec   

VLESS over TCP with TLS
163 Mbits/sec

VLESS over TCP with XTLS
233 Mbits/sec               
```

4. 限制C的CPU使用上限为20%
```
VLESS over TCP
792 Mbits/sec   

VLESS over TCP with TLS
311 Mbits/sec

VLESS over TCP with XTLS
471 Mbits/sec    
```

* 各方式性能差距与4K时候变化明显，达到40%-50%的性能差距
* 在16KB时（大流量时），XTLS对比TLS在接收和解密数据时也会有明显提升。
* 对比4K的10%提升，流量越大XTLS可能提升越明显
---
## 测试数据4（ethr) 2KB
- ethr以HTTPS协议进行测试
- TLS record 大小 2KB
- 本组测试主要用于印证在2KB TLS record时各方式的性能差异
  
1. 限制C的CPU使用上限为10%
```
VLESS over TCP
412 Mbits/sec   

VLESS over TCP with TLS
165 Mbits/sec

VLESS over TCP with XTLS
138 Mbits/sec               
```

2. 限制C的CPU使用上限为20%
```
VLESS over TCP
739 Mbits/sec   

VLESS over TCP with TLS
315 Mbits/sec

VLESS over TCP with XTLS
263 Mbits/sec    
```


* XTLS慢于TLS（与iperf组测试一致）
* 可以确认TLS record 2K时，存在一些需要优化的问题
---
## 测试数据5（ethr) 1KB
- ethr以HTTPS协议进行测试
- TLS record 大小 1KB
- 本组测试主要用于印证在1KB TLS record时各方式的性能差异
---
1. 限制B的CPU使用上限为20%
```
VLESS over TCP
702 Mbits/sec   

VLESS over TCP with TLS
385 Mbits/sec

VLESS over TCP with XTLS
484 Mbits/sec    
```

2. 限制C的CPU使用上限为20%
```
VLESS over TCP
670 Mbits/sec   

VLESS over TCP with TLS
298 Mbits/sec

VLESS over TCP with XTLS
245 Mbits/sec    
```


* 在1K时，C的性能受限时，和2K有类似问题，XTLS性能低于TLS
* 在1K时，B的性能受限时，XTLS和其他测试一样具有稳定提升
---

## 测试数据6（ethr) 3KB
- ethr以HTTPS协议进行测试
- TLS record 大小 3KB
- 本组测试主要用于印证在3KB TLS record时各方式的性能差异
---
1. 限制B的CPU使用上限为20%
```
VLESS over TCP
797 Mbits/sec   

VLESS over TCP with TLS
431 Mbits/sec

VLESS over TCP with XTLS
574 Mbits/sec    
```

2. 限制C的CPU使用上限为20%
```
VLESS over TCP
744 Mbits/sec   

VLESS over TCP with TLS
334 Mbits/sec

VLESS over TCP with XTLS
357 Mbits/sec    
```

* 在3K时，C的性能受限时，XTLS开始略高于TLS
* 在3K时，B的性能受限时，XTLS和其他测试一样具有稳定提升
---
## 总结
* 本组测试环境下，XTLS相对于TLS，约有40%-50%的性能提升,但在某些情况下仍有缺陷
* 综合以上几组不同大小TLS record数据来看  
  * 当C的性能受限（即数据接收和解密的性能受限）时，XTLS性能曲线似与TLS record大小有关。
  * TLS record越大性能提升越明显，起点略低，然后在3K左右相交TLS，并迅速拉升。
  * TLS record=16K时，性能的提升，在C受限时比B受限时能提升更多（意味着C受限时，能提升的上限更多
* 当B的性能受限时，XTLS对比TLS性能提升明显，且平稳，且不受TLS record的影响
* v2ray处理入站和解密更消耗CPU
* 硬件性能对数据接收和解密的影响，要大于加密和数据发送的影响
* 个人认为数据接收和解密部分很值得优化（且目前有低于TLS的情况

