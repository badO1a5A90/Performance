# Xray/v2ray/trojan/SS性能测试
Xray/v2ray推荐使用模式和性能测试,及一些针对性性能测试.  
trojan-go,trojan-gfw,SS各种实现的性能测试.
# update 20201216
- Xray v1.1.3版本相对Xray v1.1.1并没有性能上的改动,因此仍可参照Xray v1.1.1的相关测试及对比测试.
- 
# update 20201204
- 增加[Xray v1.1.1版本各组合方式性能测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/Xray_test_v1.1.1.md)
- [软路由上splice性能下降的研究及解决方案](https://github.com/XTLS/Xray-core/discussions/59)
# update 20201202
- 增加[Xray v1.1.1的性能对比测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201202.md)
- **splice模式一骑绝尘**

<!-- <details>
<summary>点击展开查看更多测试</summary> -->

# update 20201124
- 增加[Xray v1.0.0的性能对比测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_20201124.md)
  
# update 20201119
- 增加Xray和v2ray的[对比测试](https://github.com/badO1a5A90/v2ray-doc/blob/main/performance_test/Xray/speed_test_2020119.md).
- 增加ss各种实现的测试数据.

# update 20201116
- 经过更多实测和代码分析后,确定v2ray 的流量统计功能会使裸协议的 ReadV 和 WriteV 同时失效(除特殊机制外)
- 具体参见[issue#416](https://github.com/v2fly/v2ray-core/issues/416)

# update 20201109
增加v4.32.1一组新测试数据(不同硬件及性能限制),同时此组数据增加了vmess over TCP的各种组合测试对照.   
增加trojan-go和trojan-gfw的测试数据.

**点击查看 [v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md).**

  新的测试数据可以验证 XTLS Direct Mode 性能与无加密裸奔持平.
- 硬件性能越低,VLESS XTLS Direct Mode性能提升越明显.
- ~~实测开启v2ray内置的流量统计会使所有的readV失效,并且受到额外性能打击~~
- 经过更多实测和代码分析后,确定v2ray 的流量统计功能会使裸协议的 ReadV 和 WriteV 同时失效(除特殊机制外)

# update 20201108

最新的 v4.32.1 版本中，VLESS XTLS Direct Mode 性能已与 VLESS 无加密裸奔持平（接近于纯流量转发).

**点击查看 [v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md).**

<!-- </details> -->