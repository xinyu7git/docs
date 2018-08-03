---
title: FAQ
sort_rank: 5
toc: full-width
---

# FAQ（经常被问道的问题）

## 通用

### 什么是Prometheus ?

Prometheus是一个具有活跃生态系统的开源的系统监控和报警工具包，可以查看[概述](/docs_cn/introduction/overview/) 页面了解更多信息。

### Prometheus其他监测系统进行的对比情况如何？

查看[与其他同类产品的比较](/docs_cn/introduction/comparison/)页面了解更多信息。

### Prometheus依赖哪些条件？

Prometheus的主服务Server是可以独立运行的，并且不需要任何的外部依赖。

### Prometheus可以保证高可用性吗？

是的，可以在两台或多台独立机器上运行相同的Prometheus Server，相同的报警会经过 [Alertmanager](https://github.com/prometheus/alertmanager) 进行去重。

至于 [Alertmanager的高可用](https://github.com/prometheus/alertmanager#high-availability)，你可以在一个 [Mesh集群](https://github.com/weaveworks/mesh) 中运行多个Alertmanager实例，然后配置Prometheus将报警通知发给每一个实例。

### I was told Prometheus “doesn't scale”.
### 我听说Prometheus是不能够扩展的。

实际上有很多种方式可以实现Prometheus的扩展和联合(federate)。 可以阅读Robust Perception的博客 [Prometheus的扩展和联合](https://www.robustperception.io/scaling-and-federating-prometheus/) 来进一步了解。

### Prometheus是用什么语言开发的？

大部分的Prometheus组件是使用Go语言开发的，当然也有一些是使用了Java，Python和Ruby。

### Prometheus的各项功能、存储格式、API的稳定性如何？

Prometheus Github中所有已经达到1.0.0版本的代码库都是广泛遵循[语义版本控制](http://semver.org/)。重大的修改会表现在主版本的变化上。 实验组件可能有例外情况，这些情况在公告中会有明确的标记。

即使是尚未达到1.0.0版本的版本库，一般来说也相当稳定。我们旨在为每个存储库提供适当的发布流程和最终1.0.0版本。在任何情况下较大的代码变化都将会在发布说明文档中加以注明（使用 `[CHANGE]` 标识），也包括清楚地传达尚未正式发布的组件。

### 为什么Prometheus采用Pull的机制而不是Push？

基于HTTP的拉取模式（Pull) 有很多优势：

* 在进行开发调试的时候你可以在自己的笔记本上运行监控。
* 你可以更容易的判断目标是否停止服务。
* 你可以通过浏览器检查目标的健康情况

总体而言，我们认为拉取略好于推送，但在考虑监测系统时不应将其视为重点。

如果你有必须要通过Push方式来监控的场景，我们提供了 [Pushgateway](/docs_cn/instrumenting/pushing/) 来解决你的问题。

### 如何将日志输入Prometheus？

简短的回答：不要使用Prometheus！ 去使用类似 [ELK技术栈](https://www.elastic.co/products) 的方式来解决日志问题。

较长的回答：Prometheus是一个收集和处理指标的系统，而不是一个日志时间系统。Raintank的博客 [Logs and Metrics and Graphs, Oh My!](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/) 中提供了更详细的关于日志和指标的区别。

如果你想从应用日志中解析出Prometheus需要的指标，Google的[mtail](https://github.com/google/mtail) 或许可以帮助你。

### Prometheus是由谁开发的？

Prometheus项目最初是由 [Matt T. Proud](http://www.matttproud.com) 和 [Julius Volz](http://juliusv.com) 共同发起。其最初的开发大部分是由 [SoundCloud](https://soundcloud.com) 赞助。

现在Prometheus由多个公司和个人进行维护和扩展。

### Prometheus在那个协议下发布的？

Prometheus是在遵循 [Apache 2.0](https://github.com/prometheus/prometheus/blob/master/LICENSE) 协议下发布的。

### What is the plural of Prometheus?
### Prometheus的复数写法是什么？

在进行了[广泛研究](https://youtu.be/B_CDeYrqxjQ) 之后，已经确定 'Prometheus' 正确的复数写法是 'Prometheis'。

### Prometheus 可以进行配置重载吗？

是的，给Prometheus Server进程发送 `SIGHUP` 信号，或者通过HTTP POST的方式调用Prometheus的 `/-/reload` 接口都可以让Prometheus重载当前配置文件中的配置，各种组件都会尝试优雅的处理可能失败的情况。

### Prometheus可以发送报警吗？

是的，可以使用[Alertmanager](https://github.com/prometheus/alertmanager)来发送报警。

目前 Alertmanager 支持以下的外部系统：

* Email
* Generic Webhooks
* [HipChat](https://www.hipchat.com/)
* [OpsGenie](https://www.opsgenie.com/)
* [PagerDuty](http://www.pagerduty.com/)
* [Pushover](https://pushover.net/)
* [Slack](https://slack.com/)

### 可以创建dashboard吗？

是的，我们推荐在生产环境中使用 [Grafana](/docs_cn/visualization/grafana/) 来绘制 dashboard, 不过也可以通过 [Console templates](/docs_cn/visualization/consoles/) 的方式来实现。

### Can I change the timezone? Why is everything in UTC?
### 可以更改时区吗？ 为什么都是用UTC时区？

为了避免任何时区混淆，特别是当涉及所谓的夏令时间时，我们决定在内部专门使用Unix时间，并且在Prometheus的所有组件中使用UTC。你可以在UI中设计一个精致的时区选择组件，我们也欢迎你为解决时区的问题贡献你的代码，你可以查看[issue #500](https://github.com/prometheus/prometheus/issues/500)了解这个问题当前的状态。

## 性能测量

### 哪些语言具备性能测量(instrumentation)的库（library)？

有很多的客户端库可以让你的服务产生Prometheus的性能监控指标。可以点击[客户端库](/docs_cn/instrumenting/clientlibs/)这一章查看更多详细内容。

如果你有兴趣为新的语言贡献一个客户端库，可以参阅  [exposition formats](/docs_cn/instrumenting/exposition_formats/).

### Prometheus可以用来监控服务器吗？

是的，  [Node Exporter](https://github.com/prometheus/node_exporter) 可以在Linux或其他Unix操作系统上部署，通过接口暴露出一系列的服务器级别的监控指标数据，例如：CPU使用率，内存，磁盘利用率，文件系统状态和网络带宽等。

### Prometheus可以监控网络设备吗？

是的，[SNMP Exporter](https://github.com/prometheus/snmp_exporter) 可以用来监控任何支持SNMP协议的设备。

### Can I monitor batch jobs?
### Prometheus可以监控批量任务吗？

是的，可以使用 [Pushgateway](/docs_cn/instrumenting/pushing/). 你也可以查看 [最佳实践](/docs_cn/practices/instrumentation/#batch-jobs) 一文中关于监控批量任务的内容。

### Prometheus开箱即用的监控应用程序有哪些？

可以查看 [exporters and integrations列表](/docs_cn/instrumenting/exporters/) 这一章了解详细内容.

### Prometheus可以通过JMX监控JVM应用程序吗？

是的，对于无法直接使用Java客户端进行检测的应用程序，你可以单独使用 [JMX Exporter](https://github.com/prometheus/jmx_exporter) ，也可以将其用作Java代理。 

### 添加性能监控对服务本身的性能有多大的影响？

不同的客户端库和语言对性能的影响可能会有所不同。例如，对Java来说，[基准测试](https://github.com/prometheus/client_java/blob/master/benchmark/README.md) 表明，使用Java客户端递增计数器(counter)/计量器(gauge)将需要花费12-17ns，跟关键步骤的延时相比，这几乎可以忽略不计。

## Troubleshooting

### My Prometheus 1.x server takes a long time to start up and spams the log with copious information about crash recovery.

You are suffering from an unclean shutdown. Prometheus has to shut down cleanly
after a `SIGTERM`, which might take a while for heavily used servers. If the
server crashes or is killed hard (e.g. OOM kill by the kernel or your runlevel
system got impatient while waiting for Prometheus to shutdown), a crash
recovery has to be performed, which should take less than a minute under normal
circumstances, but can take quite long under certain circumstances. See
[crash recovery](/docs_cn/prometheus/1.8/storage/#crash-recovery) for details.

### My Prometheus 1.x server runs out of memory.

See [the section about memory usage](/docs_cn/prometheus/1.8/storage/#memory-usage)
to configure Prometheus for the amount of memory you have available.

### My Prometheus 1.x server reports to be in “rushed mode” or that “storage needs throttling”.

Your storage is under heavy load. Read
[the section about configuring the local storage](/docs_cn/prometheus/1.8/storage/)
to find out how you can tweak settings for better performance.

## Implementation

### Why are all sample values 64-bit floats? I want integers.

We restrained ourselves to 64-bit floats to simplify the design. The
[IEEE 754 double-precision binary floating-point
format](http://en.wikipedia.org/wiki/Double-precision_floating-point_format)
supports integer precision for values up to 2<sup>53</sup>. Supporting
native 64 bit integers would (only) help if you need integer precision
above 2<sup>53</sup> but below 2<sup>63</sup>. In principle, support
for different sample value types (including some kind of big integer,
supporting even more than 64 bit) could be implemented, but it is not
a priority right now. A counter, even if incremented one million times per
second, will only run into precision issues after over 285 years.

### Why don't the Prometheus server components support TLS or authentication? Can I add those?

While TLS and authentication are frequently requested features, we have
intentionally not implemented them in any of Prometheus's server-side
components. There are so many different options and parameters for both (10+
options for TLS alone) that we have decided to focus on building the best
monitoring system possible rather than supporting fully generic TLS and
authentication solutions in every server component.

If you need TLS or authentication, we recommend putting a reverse proxy in
front of Prometheus. See, for example [Adding Basic Auth to Prometheus with
Nginx](https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/).

This applies only to inbound connections. Prometheus does support
[scraping TLS- and auth-enabled targets](/docs_cn/operating/configuration/#%3Cscrape_config%3E), and other
Prometheus components that create outbound connections have similar support.
