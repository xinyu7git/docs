---
title: 与其他同类产品的比较
sort_rank: 4
---

# 与其他同类产品的比较

## Prometheus vs. Graphite

### 工作范围

[Graphite](http://graphite.readthedocs.org/en/latest/) 专注于成为一个被动时间序列数据库，具有查询语言和绘图功能，其他的任何问题都是由外部组件解决。

Prometheus是一个具有监控和趋势分析功能的完整的监控生态系统，它基于时序数据进行采集、存储、查询、绘图和报警等一些列功能。它知道整个世界该有的样子（它知道该监控哪些数据端点，知道哪些时序数据意味着出现了故障），从而积极的尝试着找到哪里出现了问题。
、
### 数据模型

Graphite存储指定时间序列的数字样本，跟Prometheus很像，但Prometheus的元数据模型更为丰富：Graphite的指标名称是使用点号(.)将各个明确的含义连接起来，而Prometheus是通过对指标名称添加叫做标签的键值对来表示这些含义。这样就可以很容易通过查询语言对这些标签进行过滤、分组和匹配，

此外，特别是将Graphite和StatsD结合使用的时候，通常情况下只能存储所有被监控实例的聚合后的数据，而不是将实例作为一个维度保留下来而且能够深入到个别问题实例中。

举个例子，在Grahpite/StatsD 中存储某个API服务器的HTTP请求中 `/tracks` 路径的 `POST`方法中返回码为 `500` 的指标名称会被编码成下面的格式：

```
stats.api-server.tracks.post.500 -> 93
```

而在Prometheus中，相同的数据则会被编码成这样（假设有3个api-server实例）：

```
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample1>"} -> 34
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample2>"} -> 28
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample3>"} -> 31
```

### 存储

Graphite将时序数据使用[Whisper](http://graphite.readthedocs.org/en/latest/whisper.html)格式存储在本地磁盘上， 它是一种RRD风格的数据库，期望能够在固定的时间间隔内获取采样数据。所有的时序数据会被存储在一个单独的文件中，并且经过一段时间后，新的采样数据会覆盖之前旧的采样数据。

Prometheus也是将时序数据存在一个本地文件中，但是允许以任意的时间间隔存储采样数据，无论是进行数据采集还是规则验证。因为最新的采样数据是通过简单的追加方式，因此旧的数据也可以保存任意长的时间。Prometheus也适用于许多短暂的、频繁变化的时序数据集合。

### 总结

除了更容易运行和集成到您现有的环境中， Prometheus还提供丰富的数据模型和查询语言。如果你想要一个集群式的可以保存长期历史数据的解决方案，Graphite可能是一个更好的选择。

## Prometheus vs. InfluxDB

[InfluxDB](https://influxdata.com/) 是一个开源的时序数据库，商业版本具有可扩展和集群功能。InfluxDB项目是在Prometheus开始开发的一年后发布的，所以我们没有考虑把它当作当时的替代品。尽管如此，Prometheus与InfluxDB之间仍有很大差异，并且这两种系统都适用于略有不同的使用场景。

### 工作范围

为了公平比较，我们必须将 [Kapacitor](https://github.com/influxdata/kapacitor) 与InfluxDB作为整体一起考虑，它们组合在一起解决了Prometheus和Alertmanager组合所解决的相同的问题。

上文中讲到的Prometheus与[Graphite](#prometheus-vs-graphite)的工作范围差异也同样适用于InfluxDB。另外InfluxDB提供了连续查询功能，这等同于Prometheus的记录规则。

Kapacitor的工作范围包含了Prometheus的记录规则，报警规则和Alertmanager的通知功能。Prometheus为绘图和报警提供了[一个更强大的查询语言](https://www.robustperception.io/translating-between-monitoring-languages/). Prometheus的Alertmanager还提供报警分组、报警收敛去重和报警屏蔽的功能。


### 数据模型 / 存储

跟Prometheus类似，InfluxDB的数据模型也是使用key-value键值对作为标签（Prometheus中的labels)，在InfluxDB中被叫做tags。此外，InfluxDB中还有一个被称为字段（field）的二级标签，它在使用场景上更受限。InfluxDB支持最高达纳秒级分辨率的时间戳，以及float64，int64，bool和string数据类型。相比之下，Prometheus的float64的数据类型只能支持有限的字符串类型和毫秒分辨率的时间戳。

InfluxDB使用[日志结构合并树](https://docs.influxdata.com/influxdb/v1.2/concepts/storage_engine/)的变体进行存储，并提前写入日志。 这比Prometheus将时序数据追加到文件的方式。更适合于事件日志（event logging）。

[Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/) 这篇文章描述了事件日志和指标记录的区别。

### 架构

Prometheus Server彼此独立运行，只依靠其本地存储实现其核心功能：数据采集，规则处理和报警，和InfluxDB的开源版本类似。

商业版本的InfluxDB产品在设计上是一个分布式存储集群，一次的存储和查询是由多个节点处理。

这意味着商业版本的InfluxDB将更容易水平扩展，这也意味着您必须从一开始就要管理一个复杂的分布式存储系统。Prometheus更简单的运行，但是要求使用者在某些时候，需要明确地沿着可扩展性边界（如产品，服务，数据中心或类似方面）分割Server。独立的Server（可以冗余部署）也可以为使用者提供更好的可靠性和故障隔离。

Kapacitor的开源版没有为规则，报警或通知内置分布式/冗余的选择能力。Kapacitor的开源版本可以通过用户的手动分片进行扩展，类似Prometheus。Influx 提供了[企业版本的Kapacitor](https://docs.influxdata.com/enterprise_kapacitor), 是一个具有高可用冗余能力的报警系统。

相比之下，Prometheus和Alertmanager通过运行多分片的Prometheus Server以及Alertmanager的[高可用模式](https://github.com/prometheus/alertmanager#high-availability) ，提供了一个完整的开源冗余方案。

### 总结

两个系统之间有很多的相似之处：两者都使用标签（Prometheus中叫做labels，influxDB中叫做tags）来有效的支持多维度的指标。两者都使用
基本上是相同的数据压缩算法。两者都有广泛的整合，包括它们相互之间。两者都有hooks能力，可以让使用者进一步扩展它们，如使用统计工具分析数据或执行一些自动化操作。

InfluxDB的优势：

  * 更适合事件日志（event logging）
  * 商业版本为InfluxDB提供了集群能力，对于长期的数据存储来说更合适
  * 多个复制分片之间的数据会保证最终一致性

Prometheus的优势：

  * 更适合指标记录（metric recording)
  * 更强大的查询语言、报警和通知能力
  * 绘图和报警的可用性和正常运行时间更高。

InfluxDB由一家遵循开放核心（open-core，和open-source有所不同）模式的商业公司维护，它们还提供商业化（闭源）的集群版本，服务托管和技术支持。而Prometheus是一个[完全开源和独立的项目](/community/)，它由许多公司和个人维护，其中还有一些（人或公司）提供商业服务和技术支持。

## Prometheus vs. OpenTSDB

[OpenTSDB](http://opentsdb.net/) 是一个基于[Hadoop](http://hadoop.apache.org/) 和 [HBase](http://hbase.apache.org/)的分布式时序数据库。

### 工作范围

The same scope differences as in the case of
[Graphite](/docs/introduction/comparison/#prometheus-vs-graphite) apply here.
它和Prometheus的工作范围区别同[Graphite](/docs/introduction/comparison/#prometheus-vs-graphite)与Prometheus相类似。

### Data model

OpenTSDB's data model is almost identical to Prometheus's: time series are
identified by a set of arbitrary key-value pairs (OpenTSDB tags are
Prometheus labels). All data for a metric is 
[stored together](http://opentsdb.net/docs/build/html/user_guide/writing/index.html#time-series-cardinality),
limiting the cardinality of metrics. There are minor differences though: Prometheus
allows arbitrary characters in label values, while OpenTSDB is more restrictive. 
OpenTSDB also lacks a full query language, only allowing simple aggregation and math via its API.

### Storage

[OpenTSDB](http://opentsdb.net/)'s storage is implemented on top of
[Hadoop](http://hadoop.apache.org/) and [HBase](http://hbase.apache.org/). This
means that it is easy to scale OpenTSDB horizontally, but you have to accept
the overall complexity of running a Hadoop/HBase cluster from the beginning.

Prometheus will be simpler to run initially, but will require explicit sharding
once the capacity of a single node is exceeded.

### Summary

Prometheus offers a much richer query language, can handle higher cardinality
metrics, and forms part of a complete monitoring system. If you're already
running Hadoop and value long term storage over these benefits, OpenTSDB is a
good choice.

## Prometheus vs. Nagios

[Nagios](https://www.nagios.org/) is a monitoring system that originated in the
1990s as NetSaint.

### Scope

Nagios is primarily about alerting based on the exit codes of scripts. These are 
called “checks”. There is silencing of individual alerts, however no grouping, 
routing or deduplication.

There are a variety of plugins. For example, piping the few kilobytes of
perfData plugins are allowed to return [to a time series database such as Graphite](https://github.com/shawn-sterling/graphios) or using NRPE to [run checks on remote machines](https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details).

### Data model

Nagios is host-based. Each host can have one or more services and each service
can perform one check.

There is no notion of labels or a query language.

### Storage

Nagios has no storage per-se, beyond the current check state.
There are plugins which can store data such as [for visualisation](https://docs.pnp4nagios.org/).

### Architecture

Nagios servers are standalone. All configuration of checks is via file.

### Summary

Nagios is suitable for basic monitoring of small and/or static systems where
blackbox probing is sufficient.

If you want to do whitebox monitoring, or have a dynamic or cloud based
environment, then Prometheus is a good choice.

## Prometheus vs. Sensu

[Sensu](https://sensuapp.org/) is broadly speaking a more modern Nagios.

### Scope

The same general scope differences as in the case of
[Nagios](/docs/introduction/comparison/#prometheus-vs-nagios) apply here.

The primary difference is that Sensu clients [register themselves](https://sensuapp.org/docs/0.27/reference/clients.html#what-is-a-sensu-client),
and can determine the checks to run either from central or local configuration.
Sensu does not have a limit on the amount of perfData.

There is also a [client socket](https://sensuapp.org/docs/0.27/reference/clients.html#what-is-the-sensu-client-socket) permitting arbitrary check results to be pushed into Sensu.

### Data model

Sensu has the same rough data model as [Nagios](/docs/introduction/comparison/#prometheus-vs-nagios).

### Storage

Sensu has storage in Redis called stashes. These are used primarily for storing
silences. It also stores all the clients that have registered with it.

### Architecture

Sensu has a [number of components](https://sensuapp.org/docs/0.27/overview/architecture.html). It uses
RabbitMQ as a transport, Redis for current state, and a separate server for
processing.

Both RabbitMQ and Redis can be clustered. Multiple copies of the server can be
run for scaling and redundancy.

### Summary

If you have an existing Nagios setup that you wish to scale as-is, or want to 
take advantage of the registration feature of Sensu, then Sensu is a good choice.

If you want to do whitebox monitoring, or have a very dynamic or cloud based
environment, then Prometheus is a good choice.
