---
title: dubbo(概述)
date: 2017-10-09 18:50:11
tags: dubbo
categories: dubbo
---

### Dubbo的出现

由于国内电商发展规模越来越大,网站应用的规模不断扩大,常规的垂直应用架构已无法应对,所以分布式服务架构势在必行,我们需要一个治理系统确保架构有条不紊的演进.

分布式架构:

[![Cw1DhR.md.png](https://s1.ax1x.com/2018/05/09/Cw1DhR.md.png)](https://imgchr.com/i/Cw1DhR)

<!--more -->

Dubbo架构:

[![Cw1T3t.md.png](https://s1.ax1x.com/2018/05/09/Cw1T3t.md.png)](https://imgchr.com/i/Cw1T3t)

1. Web容器,服务提供者依赖Web容器(例如Tomcat)暴露服务(init)
2. 服务提供者向注册中心,注册服务.(init)
3. 服务消费者向注册中订阅自己所需要的服务.(init)
4. 注册中心返回暴露服务的url(异步的)
5. 服务消费者通过url以及RPC远程调用提供者的服务.(同步)
6. 监控中心监控服务的消费者与提供者.(异步)好处:能根据哪一个服务调用的较多,对该服务可进行优化.

| 节点          | 角色说明                |
| ----------- | ------------------- |
| `Provider`  | 暴露服务的服务提供方          |
| `Consumer`  | 调用远程服务的服务消费方        |
| `Registry`  | 服务注册与发现的注册中心        |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器              |

这么看来Dubbo与之前的WebService有点相像,它们之间有什么区别呢?

首先Dubbo比WebService多了两个概念,==注册中心和监控中心==.服务的提供者和消费者都需要通过注册中心进行数据的交互,并且通过监控中心,可以观察到那一个服务调用次数比较多,进而对该服务进行优化(集群);而WebService只有客户端以及服务端,WebService是基于SOAP协议进行数据的传输的,传输数据的格式是(XML和JSON),而Dubbo是通过RPC,而Rpc的通过二进制的方式进行数据传输的.

Dubbo 与Dubbox?

Dubbox的出现是由于之前阿里巴巴停止了对Dubbo的维护,而当当网就对Dubbo进行了维护并且把名字改成了Dubbox.也对Dubbo进行了一些扩展:

- 支持REST风格远程调用（HTTP + JSON/XML)；
- 支持基于Kryo和FST的Java高效序列化实现；
- 支持基于Jackson的JSON序列化；
- 支持基于嵌入式Tomcat的HTTP remoting体系；
- 升级Spring至3.x；
- 升级ZooKeeper客户端；
- 支持完全基于Java代码的Dubbo配置；

