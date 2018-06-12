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

The same scope differences as in the case of
[Graphite](#prometheus-vs-graphite) apply here for InfluxDB itself. In addition
InfluxDB offers continuous queries, which are equivalent to Prometheus
recording rules.

上文中讲到的Prometheus与[Graphite](#prometheus-vs-graphite)的工作范围差异也同样适用于InfluxDB。另外InfluxDB提供了连续查询功能，这等同于Prometheus的recording rules。？

Kapacitor’s scope is a combination of Prometheus recording rules, alerting
rules, and the Alertmanager's notification functionality. Prometheus offers [a
more powerful query language for graphing and
alerting](https://www.robustperception.io/translating-between-monitoring-languages/).
The Prometheus Alertmanager additionally offers grouping, deduplication and
silencing functionality.

### Data model / storage

Like Prometheus, the InfluxDB data model has key-value pairs as labels, which
are called tags. In addition, InfluxDB has a second level of labels called
fields, which are more limited in use. InfluxDB supports timestamps with up to
nanosecond resolution, and float64, int64, bool, and string data types.
Prometheus, by contrast, supports the float64 data type with limited support for
strings, and millisecond resolution timestamps.

InfluxDB uses a variant of a [log-structured merge tree for storage with a write ahead log](https://docs.influxdata.com/influxdb/v1.2/concepts/storage_engine/),
sharded by time. This is much more suitable to event logging than Prometheus's
append-only file per time series approach.

[Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)
describes the differences between event logging and metrics recording.

### Architecture

Prometheus servers run independently of each other and only rely on their local
storage for their core functionality: scraping, rule processing, and alerting.
The open source version of InfluxDB is similar.

The commercial InfluxDB offering is, by design, a distributed storage cluster
with storage and queries being handled by many nodes at once.

This means that the commercial InfluxDB will be easier to scale horizontally,
but it also means that you have to manage the complexity of a distributed
storage system from the beginning. Prometheus will be simpler to run, but at
some point you will need to shard servers explicitly along scalability
boundaries like products, services, datacenters, or similar aspects.
Independent servers (which can be run redundantly in parallel) may also give
you better reliability and failure isolation.

Kapacitor's open-source release has no built-in distributed/redundant options for 
rules,  alerting, or notifications.  The open-source release of Kapacitor can 
be scaled via manual sharding by the user, similar to Prometheus itself.
Influx offers [Enterprise Kapacitor](https://docs.influxdata.com/enterprise_kapacitor), which supports an 
HA/redundant alerting system.

Prometheus and the Alertmanager by contrast offer a fully open-source redundant 
option via running redundant replicas of Prometheus and using the Alertmanager's 
[High Availability](https://github.com/prometheus/alertmanager#high-availability)
mode. 

### Summary

There are many similarities between the systems. Both have labels (called tags
in InfluxDB) to efficiently support multi-dimensional metrics. Both use
basically the same data compression algorithms. Both have extensive
integrations, including with each other. Both have hooks allowing you to extend
them further, such as analyzing data in statistical tools or performing
automated actions.

Where InfluxDB is better:

  * If you're doing event logging.
  * Commercial option offers clustering for InfluxDB, which is also better for long term data storage.
  * Eventually consistent view of data between replicas.

Where Prometheus is better:

  * If you're primarily doing metrics.
  * More powerful query language, alerting, and notification functionality.
  * Higher availability and uptime for graphing and alerting.

InfluxDB is maintained by a single commercial company following the open-core
model, offering premium features like closed-source clustering, hosting and
support. Prometheus is a [fully open source and independent project](/community/), maintained
by a number of companies and individuals, some of whom also offer commercial services and support.

## Prometheus vs. OpenTSDB

[OpenTSDB](http://opentsdb.net/) is a distributed time series database based on
[Hadoop](http://hadoop.apache.org/) and [HBase](http://hbase.apache.org/).

### Scope

The same scope differences as in the case of
[Graphite](/docs/introduction/comparison/#prometheus-vs-graphite) apply here.

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
