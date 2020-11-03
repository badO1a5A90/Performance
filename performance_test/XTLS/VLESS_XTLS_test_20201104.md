## 测试环境
* VPS*4：
    - CPU Model name:Intel Core Processor (Haswell, no TSX, IBRS)*1 
    - CPU MHz: 2399.996
    - 支持AES指令
    - 1G内存
    - 10Gbps上下行带宽
    - 公网IP
* 系统：Debian 10

## 测试工具
* v2ray 4.32.0(with VLESS XTLS ReadV)
* ethr

## 测试时间
2020.11

## 测试方式
* 参见[VLESS_XTLS_20201103测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)
* 因[VLESS_XTLS_20201103测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)的版本有导致panic的bug,随后进行了修正,因此再次测试了一次readV模式.
* 增加了一个TLS record 为8K的测试
* 与4.32.0的direct的对比,以及限制B(发送)性能的测试,见[VLESS_XTLS_20201103测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)即可

## 测试说明
  * 此测试去掉了关闭特殊功能
  * 以下单位均为Mbps
  * 1K,2K,4K,8K,16K指TLS record大小
  * none指没有TLS或者加密,即纯TCP模式裸奔(VLESS over TCP),可以认为是测试模式的上限参考值.
  * 数据流向：
   - A,ethr client  
  -->B,inbound(dokodemo-door),outbound(VLESS over TCP/with TLS/with XTLS)  
  -->C,inbound(VLESS over TCP/with TLS/with XTLS),outbound(freedom)  
  -->D,ethr server
  * 本次测试硬件同样和之前测试再次不同,数据没有横向对比的意义
  * 单次测试时长30S,多次测试取平均值

## 测试数据
1. 限制C的性能为15%时

ReadV|	1K|	2K|	4K| 8K|	16K
---- | ---| ---| ---|--- |---
none|	**526**|**552**|	**590**|**611** |	**622**
TLS	|268|	284	|290|301 |	296 
XTLS(origin)|	241|	263 |325 |359 |	382 
XTLS(direct+readV)|	**434**|	**475**|	**508**|**512**	|**509**

## 结论
  
  * 测试模式中数据未能100%被XTLS处理,并且不可知多少部分被XTLS处理,但去掉关闭特殊功能后,可以观察得到=10的输出仅在测试开始,并且后续10个连接均很快进入流控模式.所以应该可以大致认为未被XTLS处理的数据并不多.
  * direct+readV与4.32.0的direct比较见[VLESS_XTLS_20201103测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)
  * 一些与[VLESS_XTLS_20201103测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)一样的结论
    * readV模式:
      * **当限制C时(即接收为瓶颈),ReadV模式强悍,提升到与纯TCP模式82%-87%的速率,考虑到未能100%被XTLS处理的因素,大致可以认为是几乎和none(裸奔)持平的水准.**
    * XTLS相对于TLS能提升多少与硬件相关并没有标准(性能越低提升越高,无硬解更能多一次翻倍提升),**若XTLS表现能接近纯TCP,应该可认为已经达到v2ray框架下目前所有协议/配置组合所能达到的极限值(接近裸奔)**
    * 补充:关闭特殊功能在实际使用中,正常的TLS数据流几乎不会触发