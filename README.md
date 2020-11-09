# v2ray性能测试
v2ray推荐使用模式和性能测试,以及一些针对性性能测试.

# update 20201109
增加v4.32.1一组新测试数据(不同硬件及性能限制),同时此组数据增加了vmess over TCP的各种组合测试对照.   
增加trojan-go和trojan-gfw的测试数据.

**点击查看 [v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md).**

  新的测试数据可以验证 XTLS Direct Mode 性能与无加密裸奔持平.
- 硬件性能越低,VLESS XTLS Direct Mode性能提升越明显.
- 实测开启v2ray内置的流量统计会使所有的readV失效(性能损失一半),并且受到额外性能打击


# update 20201108

最新的 v4.32.1 版本中，VLESS XTLS Direct Mode 性能已与 VLESS 无加密裸奔持平（接近于纯流量转发).

**点击查看 [v4.32.1版本测试](https://github.com/badO1a5A90/v2ray-doc/blob/master/v2ray_speed_test_v4.32.1.md).**
