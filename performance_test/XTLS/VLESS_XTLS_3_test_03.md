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
* v2ray 4.31.2(with VLESS preview 2.5, XTLS)
* ethr

## 测试时间
2020.10

## 重要TIP
  * 2020.11更新,关于下面结论,更多请参考[第四次测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_test_20201103.md)
  * 测试模式中数据未能100%被XTLS处理,并且不可知多少部分被XTLS处理,所以
    * 提升仅能代表部分数据被XTLS处理的提升效果
    * 并不能代表XTLS可以带来的最大提升效果.
    * 待有能进行100%XTLS处理的测试方式后重新测试.

## 测试数据
  * 此测试去掉了关闭特殊功能
  * 以下单位均为Mbps
  * 1K,2K,4K,16K指TLS record大小
  * 具体测试方式参考[第二次测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/performance_test/XTLS/VLESS_XTLS_3_test_02.md)
  
1. 限制B的性能时

-|	1K|	2K|	4K|	16K
---- | ---| ---| ---| ---
none|	618.1|	653.6|	714.9|	754.1
TLS	|308.8|	329.3	|342.6|	351.4
XTLS(origin)|	358.6|	399.7|	443.4|	473.4
XTLS(direct)|	413.6|	437|	461.3|	482.6

2. 限制C的性能时

-|	1K|	2K|	4K|	16K
---- | ---| ---| ---| ---
none|	620.2|	660.5|	719.2	|743.2
TLS|	254.4|	264.2|	270.1|	275.4
XTLS(origin)|	223.2|	234.6	|318.1|	380.2
XTLS(direct)|	298.2	|326.1|	334.9|	330.4

## 结论
  
  * 测试模式中数据未能100%被XTLS处理,并且不可知多少部分被XTLS处理,所以
    * 提升仅能代表部分数据被XTLS处理的提升效果
    * 并不能代表XTLS可以带来的最大提升效果.
    * XTLS两种模式的性能对比也并不一定非常准确.
  * XTLS(origin)和XTLS(direct)的性能对比,如果认为两种模式下被XTLS处理的数据量大致相当,那么:
    * origin的表现,和第二次测试一致,限制B时稳定提升,限制C时由低到高,16K时甚至反超了direct(可能是因为被XTLS处理的数据量不一致引起)
    * direct的表现,限制B和C时均是稳定提升,应该可证明direct模式的修改优化对于origin模式还是很有效果的.
    * direct 比 origin 更快提升更稳定
  * 待有能进行100%XTLS处理的测试方式后重新测试.

