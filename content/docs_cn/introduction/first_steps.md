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

最后的`scrape_configs`配置区域控制Prometheus所要监控的资源信息。因为Prometheus也将自己的数据通过HTTP接口暴露出来，因此它也可以抓取并监控自己的健康状况。在默认的配置中，只有一个job name为 `prometheus` 的任务，它负责抓取Prometheus Server自身的时序数据，这个任务只包含一个单一的、静态配置的目标，那就是本地`localhost`的`9090`端口。Prometheus期望可以从服务的 `/metrics` 路径上获取可用的监控指标，因此默认的数据收集的URL地址为： http://localhost:9090/metrics 。

返回的时间序列数据将详细说明Prometheus Server的状态和性能。

有关配置选项的完整说明，见[配置文档](/docs_cn/operating/configuration).

## 启动 Prometheus

进入 `prometheus` 可执行文件的目录，使用我们刚刚创建的配置文件来启动Prometheus，运行如下命令：

```language-bash
./prometheus --config.file=prometheus.yml
```

Prometheus应该已经启动完成了。你可以通过浏览 http://localhost:9090 查看关于它自己健康情况的状态页面，30s之后就可以看到它从自己的HTTP监控指标接口中收集到的数据了。

你也可以访问Prometheus的监控路径 http://localhost:9090/metrics 来验证Prometheus是在收集它自身的监控指标。

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

仅仅搜集Prometheus Server本身的监控指标并不能很好的代表Prometheus的能力，因此，我们使用Node Exporter来监控我们的第一个资源。我们将会监控运行Prometheus Server的本地Linux服务器，不过你可以监控任何Linux或者OS X服务器。如果你想监控微软Windows服务器，可以使用[WMI exporter](https://github.com/martinlindhe/wmi_exporter)。

请针对不同的操作系统[下载最新版本的Node Exporter](/download/#node_exporter), 然后进行解压缩：

```language-bash
tar xvfz node_exporter-*.tar.gz
cd node_exporter-*
```

Node Exporter是一个叫做`node_exporter`的单独的二进制文件，它包含一组可配置的搜集器，可用于搜集各种类型的基于主机的监控指标。默认情况下，搜集器会搜集 [CPU, 内存， 磁盘和其他指标](https://github.com/prometheus/node_exporter#enabled-by-default) 并将它采集到的数据暴露出来。

现在在我们的Linux服务器上启动Node Exporter。

```language-bash
./node_exporter
```

Node Exporter服务的指标可以通过 `/metrics` 路径在服务器上的 `9100` 端口上获得。在我们的示例中，它对应的服务地址为：http://localhost:9100/metrics.

你可以在浏览器中打开上面的URL来查看Node Exporter暴露的监控指标。

现在，我们需要告诉Prometheus这个新的exporter在哪里。

## 配置Prometheus来监控主机

我们将会配置Prometheus来抓取新的目标。为了达到这个目的，需要在 `prometheus.yml` 配置文件的 `scrape_configs` 配置段中添加一个新的任务描述。

```
- job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

新的任务名称为 `node` ，它抓取一个静态的目标， `localhost` 的 `9100` 端口，你可以将此名称替换为您正在监控的主机的名称或IP地址。

现在，我们重启Prometheus Server来激活新的任务。

现在，去表达式浏览器来验证Prometheus现在已经采集到这个节点暴露的时间序列数据。在浏览器中打开 http://localhost:9090/graph 地址，使用 "Execute" 按钮旁边的下拉菜单查看该服务器正在收集的指标列表。在列表中，你会看到以 `node_` 为前缀的多个指标，这些指标是通过我们定义的 `node` 任务，从Node Exporter中采集的。例如，你可以通过 `node_cpu` 指标查看节点的CPU使用情况。

一个有用的指标是 `up` 指标， `up` 指标可以用来跟踪目标的状态。 如果这个指标的值为 `1` 则表示该目标的指标采集是成功的，如果是 `0`，则表示采集失败，这种方式可以帮助我们了解目标的状态情况。现在，你可以看到两个 `up` 指标，一个代表的是Prometheus Server，另一个代表的是Node Exporter。

## 总结

现在你已经入门了Prometheus，了解了如何安装以及配置它来监控你的第一个资源。我们还安装了我们的第一个exporter，并通过使用表达式浏览器了解了时间序列数据被采集的基本原理。你可以查看更多的 [文档][documentation](/docs_cn/introduction/overview/) 和 [向导](/docs_cn/guides/) 来帮助你进一步了解Prometheus。
