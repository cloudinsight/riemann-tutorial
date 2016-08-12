# riemann 的介绍

## 这玩意儿是啥
首先，有个[德国数学家](https://en.wikipedia.org/wiki/Bernhard_Riemann)叫riemann，而且我有理由相信这个名字就是从这个数学家来的，因为python有个riemann client就是叫Bernhard（数学家叫 Bernhard Riemann）。

[riemann](http://riemann.io/)是一个监控工具，可以将监控的指标作为事件吐到事件流中去操作，还内置了报警，也支持将数据吐到开源的数据展示服务如Graphite中做展示，如果不嫌简陋也可以使用它内置的一个dashboard。总结起来就是，可以作为数据采集工具，或一个完整的带收集、事件处理、报警的完整的监控系统，或做为报警引擎，而我们的使用就是将它作为一个报警引擎。

## 聊点监控的事情
在这里想闲扯一点东西。

riemann的实现是以事件(event)为中心的push模型，stastd也是push模型(cloudinsight是使用的statsd)。而像一些比较成熟的监控工具如nagios或zabbix(pull方式）是需要server来轮询去目标主机拿数据，比如ping一下目标主机，这种监控方式主要是获悉服务可用性、保证服务在运行，在这种架构下监控是围绕着不让目标主机或服务挂掉，而不是让已有资源发挥最大的价值。当然目前这些工具也发展的很完善了，提供了比较多的预置的插件、模板，可以比较洞悉服务的运行情况。

push模型对比于pull模型，更多的重心是放在



