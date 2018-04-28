---
title: 概述
sort_rank: 1
---

# 概述

## 什么是 Prometheus ?

[Prometheus](https://github.com/prometheus) 是一个开源的系统监控和报警工具包，最初使用在[SoundCloud](https://github.com/prometheus) (http://soundcloud.com) 平台之上。
从2012年开始，许多公司和组织开始使用Prometheus，该项目的开发人员和[用户社区](/community)非常活跃。目前Prometheus是一个独立的开源项目，且不依赖任何公司进行维护。为了强调这一点，并澄清项目的治理结构，Prometheus于2016年加入[CNCF基金会(Cloud Native Computing Foundation)](https://cncf.io/) ，作为Kubernetes之后的第二个托管项目。

有关Prometheus更详尽的概述，请参阅[media](/docs/introduction/media/)部分的链接。

### 功能特性

Prometheus的主要的功能特性包括如下几项：

* 一个多维[数据模型](/docs_cn/concepts/data_model/)，由指标名称(metric)和键/值(key/value)对组成的时间序列数据
* 一种[灵活的查询语言](/docs_cn/prometheus/latest/querying/basics/), 可以充分利用多维的数据模型
* 不依赖分布式存储，单个服务节点是自治的
* 通过HTTP的主动拉取模式(Pull)进行时间序列收集工作
* 时序数据[推送(Push)](/docs_cn/instrumenting/pushing/)的方式可以通过中间层gateway的方式支持（但有局限性）
* 通过服务发现或静态配置来得到监控目标
* 多种模式的图表和Dashboard支持

### 组件

Prometheus的生态系统多个组件组成，其中许多组件是可选的：

* [Prometheus Server](https://github.com/prometheus/prometheus). Prometheus主程序，负责收集和存储时序数据
* [client libraries](/docs_cn/instrumenting/clientlibs/). Prometheus 客户端代码库，用于入侵应用程序代码汇总监控数据并暴露相关HTTP接口。？
* [push gateway](https://github.com/prometheus/pushgateway). 推送网关， 用于支持短时任务的数据推送。
* [exporters](/docs_cn/instrumenting/exporters/). 特殊功能的数据收集及展示工具，专门用于如HAProxy，StatsD，Graphite等服务的数据收集。
* [alertmanager](https://github.com/prometheus/alertmanager). 报警管理工具，用于进行报警的收敛及通知。
* 其他各种支撑工具。

Prometheus的大多数组件都是用 [Go](https://golang.org/) 编写的，因此可以很容易构建为静态的二进制文件进行部署。
### 架构

该图说明了Prometheus及其生态系统组件的整体架构：

![Prometheus architecture](/assets/architecture.svg)

Prometheus可以直接拉取目标任务数据，或者间接地通过中间网关(适用短时任务的数据推送的Push Gateway)拉取数据。Prometheus在本地存储所有抓取的数据样本，并对这些数据运行相应的处理规则，以便聚合和记录新的时间序列或者产生报警。可以使用[Grafana](https://grafana.com/) 或其他API消费者来可视化Prometheus收集的数据。

## Prometheus 适用场景

Prometheus 可以很好地记录任何纯数字时间序列。它既适用于以机器为中心的监控，也适用于高度动态的面向服务的体系(SOA)监控。在微服务领域，Prometheus的多维数据收集和强大灵活的查询功能更是彰显出它的优势。

Prometheus 专为可靠性而设计，当服务出现故障时，它可以使你快速定位和诊断问题。每个Prometheus Server都是独立的，不依赖于网络存储或其他远程服务。你可以依靠Prometheus来监控你的基础设施，而不需要花费太多的部署成本。

## Prometheus 不适用场景

Prometheus 重视可靠性，即时在出现故障的情况下，你都可以随时访问它来查看相关系统指标的统计信息。如果你需要100％的准确性（例如按请求计费），Prometheus并不是一个好选择，因为收集的数据可能不够详细和完整。 在这种情况下，你最好使用其他系统来收集和分析计费数据，Prometheus则可以进行其他功能的监控工作。