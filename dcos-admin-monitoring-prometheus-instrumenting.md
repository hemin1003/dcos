## 监测与采集

### 指标监测

通过Prometheus提供的客户端库可以向代码添加额外的监测代码，这些客户端库实现了Prometheus的[指标类型](/dcos-admin-monitoring-prometheus-concepts.md)，可以让你定义自己的内部应用度量指标并通过HTTP将这些指标数据发布给Prometheus服务器。

Prometheus为大多数开发语言提供了[客户端库](https://prometheus.io/docs/instrumenting/clientlibs/)。如果现有的库不适合你的应用或者想避免依赖，可以参照[指标格式](https://prometheus.io/docs/instrumenting/exposition_formats/)提供自己的实现。


### 监测代码开发

#### 客户端库

客户端库的关键类是Collector，它有一个方法（通常称为“`collect`”），返回零个或多个指标及其样本。采集器（Collectors）注册到CollectorRegistry，通过将CollectorRegistry传递到一个“桥接”类/方法/函数，这个“桥接”类/方法/函数以Prometheus支持的格式返回指标数据。每次抓取时，CollectorRegistry必须回调每一个注册的采集器的`collect`方法。

大多数用户使用的接口是Counter，Gauge，Histogram和Summary采集器，这些接口代表单一指标，可以涵盖绝大多数用户的需求用例。
更高级的用例（例如从另一个监视/度量的系统进行代理）需要编写自定义收集器。

CollectorRegistry提供了`register()`/`unregister()`函数，并且允许收集器注册到多个CollectorRegistry。

客户端库是线程安全的。

#### 度量指标

客户端库实现一个默认的CollectorRegistry，标准的指标默认会注册到这里而不需要用户参与。客户端库也提供将指标注册到其它CollectorRegistry的方法。

```java
class YourClass {
  static final Counter requests = Counter.build()
      .name("requests_total")
      .help("Requests.").register();
}
```

上述示例代码将requests注册到默认的CollectorRegistry。

Counter提供如下方法：

- inc(): Counter值增1

- inc(double v)：将计数器增加给定的数量,v>=0。


Gauge表示一个可上下浮动的值，Gauge提供如下方法：

- inc(): gauge的值增1
- inc(double v): gauge的值增指定的数量
- dec(): gauge的值减1
- dec(double v): gauge的值减指定的数量
- set(double v): 设定gauge为给定的值


### 指标发布

#### 发布器

Prometheus提供了一些[官方的](https://github.com/prometheus)指标发布器，例如JMX exporter可以从各种基于JVM的应用程序发布指标，例如Kafka和Cassandra。同时，[第三方](https://prometheus.io/docs/instrumenting/exporters/)也提供了众多的指标发布器。

#### 发布器开发


