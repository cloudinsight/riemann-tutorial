# riemann 插件

其实riemann作为一个"事件处理系统"来说，它内置可连接的已有的监控软件、告警方式已经很丰富了，但是总有你想要额外支持的，比如kafka，在0.2.11版本是没有内置支持的，所以有[riemann-kafka](https://github.com/pyr/riemann-kafka)还有[riemann-kfk-plugin](https://github.com/yaiba/riemann-kfk-plugin), 这时候如果也找不到已有项目支持的话就得自己动手写一个了。

写一个插件其实很简单，这里介绍一个简单的，你只要做到三步：

* 你的功能代码
* riemann_plugin/\*/\*.edn 文件
* 将插件的jar包指定到riemann的 EXTRA_CLASSPATH

第一步不用说了，就是实现你的功能。

第二步需要建一个目录riemann_plugin，还有一个[edn](https://github.com/edn-format/edn)文件，里面必须提供两个字段：plugin(插件名)、require(指定namespace)。

第三步就是需要让riemann找的到你的jar包，riemann在jar包里面寻找riemann_plugin下面的edn文件，然后会读取里面的内容去找代码。

最后，哦，其实是四步，在riemann.config 开头加上一句 (load-plugins)。