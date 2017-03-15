# streams 下的函数

##  moving-time-window n
这个是以当前最新的时间戳向前n秒为一个范围截取一组event，这组event会缓存下来, 不在范围内的event就会从这个窗口丢掉。
注意这个最新的时间戳，意思就是如果你传入的event在时间上是乱序的，那么因为这个范围是以riemann接收到的最“新”的event为截止时间划取的，所以
如果你以2为参数，传入的是以下的event：
```[{:time 5} {:time 1} {:time 2} {:time 6} {:time 3} {:time 8} {:time 4} {:time 8} {:time 5} {:time 9}]```
那么你将得到的event是这样的：
```[{:time 5} {:time 6}] ```
```[{:time 8}] ```
```[{:time 8} {:time 8}] ```
```[{:time 8} {:time 8} {:time 9}]```
在:time 1 时候并没有得到event，因为最新的时间是5，1不是[3,5]范围内的，所以就抛弃了。后面的2、3、4、5都是这种情况。
