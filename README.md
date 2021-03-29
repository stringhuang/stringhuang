企业: 广州黑格智造科技
地点：中国广州市黄埔区加速器
联系方式：tianyu1102@163.com
业务场景：物联网设备的数据存储，用于统计和分析设备的温度，湿度，压力差值等曲线或者状态。

目前遇到问题：【Tdengine 客户端连接 jdbc-jni 高可用】

我现在使用开源版Tdengine 2.0.16.0，总共3个物理节点，运行3个dnoe，3个monde，1个arbitrator。

我的数据库副本设置为2，当其中一个dnode offline的时候，集群依然是高可用。

我现在遇到的问题是，如何做客户端连接的高可用，我没有选择jdbc-restful方案，因为他和mybatis-plus结合得不好，也考虑到写入性能。所以，我选择使用jdbc-jni方式。

我在3个物理节点外面包装一层haproxy监听端口16030，然后再把流量转发到物理节点的6030，但是，通过16030进行连接，报错” aos connect failed, reason: Unable to establish connection.”。

我的haproxy主要配置如下：
frontend  taosd
    mode tcp
    bind *:16030
    default_backend taosd
    maxconn 50000
    timeout client 600s

backend taosd
    mode        tcp  # 模式tcp
    balance     roundrobin  # 采用轮询的负载算法
    timeout server 40s
    timeout check 4000
    server node-1 10.99.11.220:6030 check inter 30s fall 2 rise 5 weight 1 maxconn 2000
    server node-2 10.99.11.221:6030 check inter 30s fall 2 rise 5 weight 1 maxconn 2000
    server node-3 10.99.11.222:6030 check inter 30s fall 2 rise 5 weight 1 maxconn 2000

正常连接如下：
taos -h 10.99.11.222 -u root -ptaosdata -P 6030

Welcome to the TDengine shell from Linux, Client Version:2.0.16.0
Copyright (c) 2020 by TAOS Data, Inc. All rights reserved.

taos> exit

报错连接如下：
taos -h 10.99.11.222 -u root -ptaosdata -P 16030

Welcome to the TDengine shell from Linux, Client Version:2.0.16.0
Copyright (c) 2020 by TAOS Data, Inc. All rights reserved.

taos connect failed, reason: Unable to establish connection.

我想请教，是否tdengine client不支持外层包裹负载均衡进行连接？
