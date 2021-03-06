---
title: Flink的实际项目应用初探
date: 2019-10-20 15:00:09
categories:
 - Tech
tags:
---
![正文分割](/assets/images/focus.gif) 
## **写在前面**
&emsp;&emsp;今天聊的话题是流式计算，如未特殊说明，这里的应用环境均指的是大数据环境，即对海量数据的分布式实时处理场景。
其中文中涉及到的流式计算框架均为开源框架，且为目前的主流框架。本文主要是对流式计算的定义和主流框架简要对比，包括各个框架的应用场景分析，引出在实际项目中，
社区最为活跃的Flink的简单应用场景，希望可以抛砖引玉，不正确的地方希望可以批评指正。

## **流式计算是什么**
&emsp;&emsp;流式计算也叫流式处理，即对源源不断的数据进行实时或近乎实时的处理，这个处理由一个或多个算子组成，算子就是一个运算单元。
流式处理通常意义上包括数据输入（数据源）、处理算子（一个或多个）和数据输出，当然数据源和数据输出也可以有一个或多个。
下面用图示简要说明流式处理的两种模式。

&emsp;&emsp;实现流式处理系统有两种完全不同的方式：一种是称作原生流处理，意味着所有输入的记录一旦到达即会一个接着一个进行处理，如下图所示。 这种模式最直观的优点就是延迟低。

![stream1](/assets/images/stream1.png) 

&emsp;&emsp;第二种称为微批处理。把输入的数据按照某种预先定义的时间间隔（典型的是几秒钟）分成短小的批量数据，流经流处理系统，如下图所示。这种模式最直观的优点就是吞吐量高。

![stream2](/assets/images/stream2.png) 

## **主流开源流式计算引擎**

&emsp;&emsp;分布式流式计算的发展目前分为三代，Hadoop时期出现的第一代Storm，然后是基于内存处理的第二代Spark，再到现在的批流统一的第三代Flink。

![stream3](/assets/images/stream3.png) 

&emsp;&emsp;其中，第一代Storm是一种原生的流处理计算框架，而Storm Trident是一种微批处理（一次处理几个Tuple），增加了exactly-once的语义、状态管理和聚合API 等；Spark Streaming是一种微批模式的流处理计算框架，非原生的流处理；
Structured Streaming将实时流抽象成一张无边界的表，输入的每一条数据当成输入表的一个新行，同时将流式计算的结果映射为另外一张表，完全以结构化的方式去操作流式数据。Structured Streaming相比于Spark Streaming，基于Event-Time，相比于Spark Streaming的Processing-Time更精确，更符合业务场景。解决了Spark Streaming存在的代码升级，DAG图变化引起的任务失败，无法断点续传的问题。
Flink是目前较火的大数据处理引擎，在流式计算领域，有很多区别于前面两代的流式计算的优势。
尽管批流本是两套系统，但是这两套系统统一起来确实很有必要，我们有时候确实需要将我们的流处理逻辑运行到批数据上面。

&emsp;&emsp;上述三代主流开源流处理计算框架的关键指标对比如下表所示。

![stream4](/assets/images/stream4.png) 

&emsp;&emsp;其中，Declarative API，中文一般叫做声明式编程 API,而Compositional API是组合式API。举个例子，我们要一个糕点，去糕点店直接去定做告诉店员我们要什么样式的糕点，然后店员去给我们做出来，这就是 Declarative。而 Compositional 对应的就是面粉、鸡蛋等，我们自己去做。

## **前两代流处理计算框架的应用场景分析**

* ### Storm 

1. 纯实时，毫秒级延迟的场景，比如实时计算系统，要求纯实时进行交易和分析时。
2. 在实时计算中要求可靠的事务机制和可靠性机制，即数据的处理完全精准，可以考虑使用Storm，但是Spark Streaming也适用。
3. 需要考虑针对高峰低峰时间段，动态调整实时计算程序的并行度，以较大限度利用集群资源。

* ### Spark Streaming 

1. 不满足上述特定要求，可以考虑使用Spark Streaming来进行实时计算。
2. 如果一个项目除了实时计算之外，还包括了离线批处理、交互式查询、图计算和MLIB机器学习等业务功能，而且实时计算中，可能还会牵扯到高延迟批处理、交互式查询等功能，那么就应该推荐Spark生态，用Spark Core开发离线批处理，用Spark SQL开发交互式查询，用Spark Streaming开发实时计算，三者可以无缝整合，给系统提供非常高的可扩展性。
3. 需要考虑针对高峰低峰时间段，动态调整实时计算程序的并行度，以较大限度利用集群资源。

## **第三代流处理计算框架Flink的优势和特性**

* ### 优势

1. 支持高吞吐、低延迟、高性能的流处理。
2. 支持带有事件时间的窗口（Window）操作。
3. 支持有状态计算的Exactly-once语义。
4. 支持高度灵活的窗口（Window）操作，支持基于time、count、session，以及data-driven的窗口操作。
5. 支持具有Backpressure功能的持续流模型。
6. 支持基于轻量级分布式快照（Snapshot）实现的容错。
7. 一个运行时同时支持Batch on Streaming处理和Streaming处理。
8. Flink在JVM内部实现了自己的内存管理。
9. 支持迭代计算。
10. 支持程序自动优化，避免特定情况下Shuffle、排序等昂贵操作，中间结果有必要进行缓存。

* ### 区别于Spark的流批处理一体化解决方案

&emsp;&emsp;Spark和Flink都具有流和批处理能力，但是他们的做法是相反的。Spark Streaming是把流转化成一个个小的批来处理，这种方案的一个问题是我们需要的延迟越低，额外开销占的比例就会越大，这导致了Spark Streaming很难做到秒级甚至亚秒级的延迟。Flink是把批当作一种有限的流，这种做法的一个特点是在流和批共享大部分代码的同时还能够保留批处理特有的一系列的优化。

&emsp;&emsp;Flink作为原生流处理框架，在流处理方面的功能，延迟，一致性和性能上综合来看是目前社区最优秀的，所以可以采用Flink来实现流和批的一体化解决方案。

* ### 时间机制的丰富性

&emsp;&emsp;Spark Streaming 只支持处理时间，Structured streaming 支持处理时间和事件时间，同时支持 watermark 机制处理滞后数据。

&emsp;&emsp;Flink 支持三种时间机制：事件时间，注入时间，处理时间，同时支持 watermark 机制处理滞后数据。

* ### 阿里Blink与Flink的结合

![stream5](/assets/images/stream5.png) 

&emsp;&emsp;Flink在工程的实现上有一些不足之处。比如说不同的job的任务可能运行在同一个进程里，这样一个job的问题可能影响其他job的稳定性。Flink的工程实现也不能把集群的资源最合理地利用起来。Blink重新实行了Yarn的结合，完全解决了这些问题。另外Flink是通过checkpoint的机制来保证一致性的，但原有的机制效率比较低，导致在状态较大的时候不可用，Blink大大优化了checkpoint，能够高效地处理很大的状态。稳定性和scalability在生产上都是至关重要的，通过在大集群上的锤炼，Blink解决了一系列这方面的问题和瓶颈，已经成为一个能够支撑核心业务的计算引擎。同时我们扩展了Flink的Streaming SQL层，使得它能够比较完备地支持较复杂的业务。

&emsp;&emsp;blink开源，目前在[https://github.com/apache/flink/tree/blink](https://github.com/apache/flink/tree/blink)，有兴趣的可以去查阅。

* ### 生态较完善，广泛集成常用Connectors

![stream6](/assets/images/stream6.png) 

## **Flink在实际项目中的简单应用**

* ### 业务场景

![stream7](/assets/images/stream7.png) 

&emsp;&emsp;我们的业务场景是从kafka获取上游的日志数据，然后进行拼接计算，最后对拼接后的数据进行落盘（Hive或HDFS）、实时分析（监控）、离线评估（计算auc、gauc等）。这里的上游数据主要有请求、展现和点击日志。
拿计算auc为例，三种日志拼接计算时由于与点击拼接前需要拿请求日志去访问proxy，如果直接拿请求日志去访问proxy，会导致数据量太大，对proxy造成巨大的压力，这里会把请求和展现日志先进行拼接，把拼接后的数据作为新的"请求日志"去访问proxy，然后再与点击进行拼接。
其中访问proxy是为了从其他服务器获取pctr等指标数据。
&emsp;&emsp;此外，由于展现日志和点击日志会比请求日志晚到，所以需要在拼接时需要对展现和点击进行缓存，而且对点击的缓存时间较长，通常为1～2小时。

&emsp;&emsp;说明：请求和展现的拼接是"内连接"，即有展现的sku才保留，无展现的sku要丢弃。新请求（请求和展现日志拼接后的数据）与点击的拼接则为"外连接"，因为要计auc,这里对有点击的请求为正例，无点击的请求为负例。

* ### 原始的拼接框架

&emsp;&emsp;最初的拼接框架是基于Storm进行开发的，主要用了两个拓扑，一个用来做请求和展现的拼接，另一个用来做新请求和点击的拼接。但框架代码逻辑较复杂，维护成本较高，且需要自己维护队列进行日志缓存，一旦任务重启或失败，缓存的数据就会丢失（点击日志由于缓存时间长，缓存数据量较大），造成后续应用出现不正确的结果。

* ### 新的拼接框架

&emsp;&emsp;根据上述的各个框架主要指标的对比以及优缺点的分析，最后我们选择了Flink作为新的流处理计算框架，具体流程如下如所示。

![stream8](/assets/images/stream8.png) 

&emsp;&emsp;其中，两个红框即为对应的拼接逻辑。框架中的拼接任务一共涉及到三个任务，即上图中的三个pipeline。请求和展现的拼接充分利用了Flink的interval join operator和延迟1小时触发写Kafka（由于最终的拼接落盘是以小时为目录，这里延迟1小时是为了与点击进行拼接），而对新请求和点击拼接前对点击的缓存，我们采用了redis集群进行处理，并设置redis key的过期时间为2小时（保证点击的到达，增加拼接率）。
不仅满足业务的需要，而且具有以下优点：

1. 基于Flink开发，吞吐量大、低延迟。
2. 采用区间差分日志拼接，节省资源，增加拼接成功率。
3. 利用Flink自身缓存和redis缓存数据,不易丢失。
4. 支持CEP在线业务。
5. 代码简洁，维护成本降低。
6. 任务拆分，耦合性降低，可扩展性增强。

* ### 整个系统的数据流导向图

![stream9](/assets/images/stream9.png) 

## **总结**

&emsp;&emsp;由于历史遗留原因和业务场景不通，大多数公司可能Storm、Spark和Flink共存，但Flink是一个批流统一的框架，而且社区目前最活跃。个人感觉整个流处理计算框架生态中，新技术永远是趋势，即使还不够成熟，但社区的推动总会向着成熟迈进。

## **Contact**

&emsp;&emsp;**Email**: ke.luckystone@gmail.com

&emsp;&emsp;**微信公众号**：      
  ![微信扫码关注](/assets/images/wx_platform4.gif)  


