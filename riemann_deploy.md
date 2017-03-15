# 有关riemann的部署

## riemann 的极限
据作者博客所说，单实例riemann的极限值是 ["200k events per second"](https://aphyr.com/posts/269-reaching-200k-events-sec) 在四核主机上，已经很高的吞吐了。riemann的性能是与cpu，网络很相关的，而且与stream的处理过程也有一定关系。
riemann并没有提供分布式的解决方案，也没有高可用的方案，所有的状态(index state, stream state)都是在内存中的，所以当riemann死掉或者重启的话所有的状态都[丢掉了](https://groups.google.com/d/msg/riemann-users/O__GiHQ2PAA/CaefnZZ9h2oJ)，当然我们可以说作为一个实时流处理的系统可以接受丢失一小段时间数据，但是如果你的使用场景下来说不能接受的话，就要另寻它法来解决了。

## 一个方案？
那是不是就我们简单的把event打到不同的riemann实例就行了？当然不行，不同的实例里面的状态是独立的，如果你的某个服务的event的处理需要一些其他event来提供信息，那么这个处理就乱套了。
简单来说，就是一套服务要对应两个riemann实例。
假设你有两个不同的服务集群A,B，那么你应该让A集群的机器持续地往riemannA发event，B持续地往riemannB发event，而且为了保证高可用，你应该还有一个riemannA1和riemannB1, 即一个集群往两个riemann实例发数据，其中一个实例关闭告警等功能，另一个开启告警功能，如果开启告警功能的riemann实例挂掉了，马上把另一台的告警功能开启就好了。因为两个实例的状态是一样的，所以告警不会有什么问题。
