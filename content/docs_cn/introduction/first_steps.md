---
title: 入门
sort_rank: 3
---

# Prometheus 入门

欢迎使用Prometheus！Prometheus是一个监控平台，它通过从监控目标对外暴露的HTTP接口来抓取相关的数据指标。这篇入门文章会告诉你如何安装和配置Prometheus来监控你的第一个集群资源。你将会知道如何下载、安装和运行Prometheus，以及下载和安装一个 exporter， exporter是一个用于暴露主机或服务时序数据的工具。这里使用的第一个exporter是Node Exporter，它是用于提供主机级别的相关指标，比如CPU、内存以及磁盘等信息。

## 下载 Prometheus

你可以点击这里下载相关对应平台[最新版本](/download)的Prometheus，并进行解压缩：

```language-bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```
Prometheus Server是一个命名为 `prometheus` 的二进制文件 (在Microsoft Windows上命名为`prometheus.exe`). 你可以使用 `--help` 参数运行这个二进制文件来查看相关的使用帮助。

```language-bash
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server

. . .
```

在启动Prometheus之前，需要先进行配置.

## 配置 Prometheus

Prometheus使用[YAML](http://www.yaml.org/start.html)语法进行配置。下载Prometheus压缩包并解压之后会发现一个叫做 `prometheus.yml` 的样例配置文件，我们可以从它开始进行配置。

我们已经删除了示例文件中的大部分注释（以`#`为前缀的行为注释），以使其更加简洁。

```language-yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

在上面的样例配置文件中有三个配置区域： `global`, `rule_files`, and `scrape_configs`.

`global` 配置区域控制Prometheus server的全局配置，当前支持两个选项。第一个是 `scrape_interval`, 该选项控制Prometheus抓取所有目标数据的频率，当然你也可以为具体的目标单独配置。在上面的样例中，我们设置的全局抓取数据的时间间隔为15秒。`evaluation_interval` 选项是用来控制Prometheus计算规则的时间间隔，Prometheus使用相关的规则`rules`来创建新的时序数据以及生成报警。

`rule_files`配置区域中配置了所有我们希望Prometheus Server所要加载的规则地址。不过到目前为止，我们还没有配置任何规则。

最后的`scrape_configs`配置区域控制Prometheus所要监控的资源信息。因为Prometheus也将自己的数据通过HTTP接口暴露出来，因此它也可以抓取并监控自己的健康状况。在默认的配置中，只有一个job_name为 `prometheus` 的任务，它负责抓取Prometheus Server自身的时序数据，这个任务只包含一个单一的、静态配置的目标，那就是本地`localhost`的`9090`端口。Prometheus期望可以从服务的 `/metrics` 路径上获取可用的监控指标，因此默认的数据收集的URL地址为： http://localhost:9090/metrics 。

返回的时间序列数据将详细说明Prometheus Server的状态和性能。

有关配置选项的完整说明，见[配置文档](/docs_cn/operating/configuration).

## 启动 Prometheus

进入 `prometheus` 可执行文件的目录，使用我们刚刚创建的配置文件来启动Prometheus，运行如下命令：

```language-bash
./prometheus --config.file=prometheus.yml
```

Prometheus应该已经启动完成了。你可以通过浏览 http://localhost:9090 查看关于它自己健康情况的状态页面，30s之后就可以看到它从自己的HTTP监控指标接口中收集到的数据了。

你也可以访问Prometheus自身的监控指标路径：http://localhost:9090/metrics，从而来验证Prometheus是在收集它自身的监控指标。

## 使用表达式浏览器

我们试着查看一些Prometheus服务自身产生的数据。要使用Prometheus内置的表达式浏览器，需要在浏览器输入 http://localhost:9090/graph 地址, 并选择"Graph"标签页中的"Console"视图。

如果你可以从 http://localhost:9090/metrics 路径查看到收集的指标数据，Prometheus自身的监控指标中有一个叫做 `http_requests_total` (Prometheus server收到的HTTP请求的总数)，继续并将其输入到表达式Console中：

```
http_requests_total
```

这应该会返回许多不同的时序数据（以及每个时序数据的最新数值），所有的数据有相同的metric名称 `http_requests_total` ，但是标签各不相同，而这些标签则代表了不同类型的请求。

如果我们仅仅对HTTP状态码是 `200` 的请求结果感兴趣，我们可以使用下面的查询语句来获取相关的数据信息：

```
http_requests_total{code="200"}
```

如果想知道时序数据的个数，你可以这样写：

```
count(http_requests_total)
```

有关表达式语言的更详细说明，见[表达式语言文档](/docs_cn/querying/basics/).

## 使用图形界面

要使用图形界面，需要跳转到 http://localhost:9090/graph 页面并选择 "Graph" 标签页。

例如，输入下面的表达式来绘制Prometheus自身的每秒HTTP请求率：

```
rate(http_requests_total[1m])
```

你可以尝试修改graph图形界面中的range参数或者其他设置。

## 安装Node Exporter

Collecting metrics from Prometheus alone is not a good representation of Prometheus' capabilities. So let's use the Node Exporter to monitor our first resource. We're going to monitor the local Linux host that the Prometheus server is running on but you could monitor any Linux or OS X host. There's also [a WMI exporter](https://github.com/martinlindhe/wmi_exporter) for Microsoft Windows hosts too.

[Download the latest release of the Node Exporter](/download/#node_exporter) of Prometheus for your platform, then extract it:

```language-bash
tar xvfz node_exporter-*.tar.gz
cd node_exporter-*
```

The Node Exporter is a single binary, `node_exporter`, and has a configurable set of collectors for gathering various types of host-based metrics. By default, collectors gather [CPU, memory, disk, and other metrics](https://github.com/prometheus/node_exporter#enabled-by-default) and expose them for scraping.

Let's start the Node Exporter now on our Linux host.

```language-bash
./node_exporter
```

The Node Exporter's metrics are available on port `9100` on the host at the `/metrics` path. In our case this is: http://localhost:9100/metrics.

You can browse to this URL to see the metrics being exposed.

We now need to tell Prometheus about our new exporter.

## Configuring Prometheus to monitor the host

We will configure Prometheus to scrape this new target. To achieve this, add a new job definition to the `scrape_configs` section in our `prometheus.yml`:

```
- job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

Our new job is called `node`. It scrapes a static target, `localhost` on port `9100`. You would replace this name with the name or IP address of the host you're monitoring. 

Now we restart our Prometheus server to activate our new job.

Go to the expression browser and verify that Prometheus now has information
about the time series that this endpoint exposes. Navigate to
http://localhost:9090/graph and use the dropdown next to the "Execute" button to see a list of metrics this server is collecting. In the list you'll see a number of metrics prefixed with `node_`, that have been collected by the Node Exporter by our `node` job. For example, you can see the node's CPU usage via the `node_cpu` metric. 

One useful metric to look for is the `up` metric. The `up` metric can be used to track the status of the target. If the metric has a value of `1` then the scrape of the target was successful, if `0` it failed. This can help give you an indication of the status of the target. You'll see two `up` metrics, one for each target we're scraping: the Prometheus server and the Node Exporter.

## Summary

Now you've been introduced to Prometheus, installed it, and configured it to monitor your first resources. We've also installed our first exporter and seen the basics of how to work with time series data scraped using the expression browser. You can find more [documentation](/docs/introduction/overview/) and [guides](/docs/guides/) to help you continue to learn more about Prometheus. 
