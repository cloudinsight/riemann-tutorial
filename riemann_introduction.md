# riemann 的介绍

## 这玩意儿是啥
首先，有个[德国数学家](https://en.wikipedia.org/wiki/Bernhard_Riemann)叫riemann，而且我有理由相信这个名字就是从这个数学家来的，因为python有个riemann client就是叫Bernhard（数学家叫 Bernhard Riemann）。

[riemann](http://riemann.io/)是一个监控工具，可以将监控的指标作为事件吐到[事件流](https://en.wikipedia.org/wiki/Event_stream_processing)中去操作，还内置了报警，也支持将数据吐到开源的数据展示服务如Graphite中做展示，如果不嫌简陋也可以使用它内置的一个dashboard。总结起来就是，可以作为数据采集工具，或一个完整的带收集、事件处理、报警的完整的监控系统，或做为报警引擎，而我们的使用就是将它作为一个报警引擎。

关于事件流处理，有一个中文的解释我觉得比较清晰：事件流处理(Event Stream Processing,ESP)技术是以近实时的方式对大量数据流进行复杂分析的技术。使用该技术可以高速吸收大量事件数据流,对事件进行历史和实时的分析,当特定事件发生时触发某些行动。事件流处理引擎类似于数据库,只不过是把数据库反过来,语句是固定的,而数据进进出出。[出处](http://cdmd.cnki.com.cn/Article/CDMD-10335-2008066040.htm)

## 聊点监控的事情
在这里想*闲扯*一点东西。

riemann是以事件(event)为中心的push模型，近些年比较流行的stastd就是push模型(cloudinsight是使用的statsd)。而像一些比较成熟的监控工具如nagios或zabbix(pull方式）这种模型一般是需要server来轮询去目标主机拿数据，比如ping一下目标主机，这种监控方式主要是获悉服务可用性、保证服务在运行，在这种架构下监控是围绕着不让目标主机或服务挂掉，而不是让已有资源发挥最大的价值。当然目前这些工具也发展的很完善了，提供了比较多的预置的插件、模板，可以比较洞悉服务的运行情况。

push模型对比于pull模型，弱化了监控中心的概念，围绕着对应用、用户体验等去监控（当然这个也跟监控的需求的发展有关），对系统的架构引入了一些变动，当一个新的主机或服务增加了，不再是去监控中心注册一个新的纪录，增加新的配置，而只需主机配置好了客户端就行了，且收集端可以不止一个，这样很方便做到横向扩展。这里有篇文章可以[参考](http://blog.sflow.com/2012/08/push-vs-pull.html)。

至于riemann的事件，在riemann中体现为一个数据结构，携带了一些一般监控指标之外的数据，包括host,service,state,time,description,tags,metric,ttl，总之就是很多描述数据的元信息，而我觉得跟‘事件’这个词最相关的字段就是state，一般是'ok''critical', 也可以是任何对你有意义的名字。就像是在应用程序中打的log，比如打印程序捕捉到了异常，只有在程序异常了才会出现，它是有意义的一个数据（个人觉得发送给riemann的数据应该是经过简单处理过的，不然跟一般的监控接收的数据就没有太大差别了，其事件流的强大的功能也就不能体现出来了）。然后在riemann里面，所有的过滤、聚合、查询等都是基于收到的事件。

## 几个riemann的核心概念

### event
即上面说到的事件，它是riemann的基本数据元素。
默认的字段有:
host 主机名
service 服务名，如 "api port 8000 reqs/sec"，可以理解为指标名
state 长度小于255字符的字符串，一般用来标识一个状态，如"ok"
time 事件的unix时间戳
description  描述，随便写吧
tags 一个字符串数组， 如 ["api" "dev"]
metric 这个事件的值
ttl 单位为秒的一个浮点数，代表的是时间的存活时间

### index
就是一个当前所有监控的服务的最新的状态的表(state table)。
一个event是用host和service来唯一索引的，即index只记录了某个(host, service)的最新的一个event。
event中的ttl是用来确定其在index中的有效时间的，超过了这个时间的event将会从index中删掉，然后将state=expired重新放到事件流中。这就很方便处理过期事件了。

### streams
也就是事件流中的"流"，它简单理解起来就是一个个函数，对每个接收到的event做处理。这些"函数"就是写在riemann的配置文件中的clojure代码，可以对event过滤、聚合，还可以嵌套（也就是一个"流"可以有"子流"），怎么组合就看自己需要了。

### server
riemann 就是一个运行在jvm上的处理事件的进程，而接收或查询是通过tcp/udp的server，内置了还有websocket server和repl server，这些都是在配置文件中指定的。


之后会介绍riemann的基本使用，几个核心stream的分析，以及对应的使用riemann来实现。
