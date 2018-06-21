#### windows redis 配置哨兵

参考文章

~~~http
https://www.cnblogs.com/ruiati/p/6374152.html
~~~

书籍：Java EE互联网轻量级框架整合开发 SSM框架（Spring MVC+Spring+MyBatis）和Redis实现

##### 1 说明

* Red is 可以存在多台服务器，并且实现了主从复制的功能。哨兵模式是一种特殊的模式，

首先Red is 提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其
原理是哨兵通过发送命令， 等待Red is 服务器响应，从而监控运行的多个Redi s 实例



* 这里的哨兵有两个作用：

  * 通过发送命令，让Red is 服务器返回监测其运行状态，包括主服务器和从服务器。

  * 当哨兵监测到master 看机， 会自动将slave 切换成master ，然后通过发布订阅模式

    通知到其他的从服务器，修改配置文件，让它们切换主机。

只是现实中**一个哨兵**进程对Redis 服务器进行监控，也可能出现问题，为了处理这个
问题，还可以使用**多个哨兵**的监控，而各个哨兵之间还会相互监控，这样就变为了多个哨
兵模式。多个哨兵不仅监控各个Re dis 服务器，而且哨兵之间互相监控， 看看哨兵们是否
还“活” 着。



* 论述一下故障切换（ failover ）的过程。假设主服务器岩机，哨兵1 先监测到这个结果，

当时系统并不会马上进行failover 操作，而仅仅是哨兵l 主观地认为主机己经不可用，这个
现象被称为主观下线。当后面的哨兵监测也监测到了主服务器不可用， 并且有了一定数量
的哨兵认为主服务器不可用，那么哨兵之间就会形成一次投票，投票的结果由一个哨兵发
起，进行fail over 操作，在fa ilover 操作的过程中切换成功后，就会通过发布订阅方式，让
各个哨兵把自己监控的服务器实现切换主机， 这个过程被称为客观下线。这样对于客户端
而言， 一切都是透明的。



##### 2 配置

###### 2.1 配置主从

​	参考上一篇文章

###### 2.2 配置哨兵

创建并修改sentinel.conf

该模式使用了3个sentinel文件，所以我们需要复制3份sentinel.conf配置文件，并分别命名为sentinel26479.conf和sentinel26579.conf,其中修改sentinel.conf配置文件中的如下几个参数：

~~~shell
port 26379 // 当前Sentinel服务运行的端口  
sentinel monitor mymaster 127.0.0.1 6379 2   
sentinel down-after-milliseconds mymaster 5000  
sentinel parallel-syncs mymaster 1  
sentinel failover-timeout mymaster 15000  
~~~

sentinel26479.conf

~~~she
port 26479 // 当前Sentinel服务运行的端口  
sentinel monitor mymaster 127.0.0.1 6379 2   
sentinel down-after-milliseconds mymaster 5000  
sentinel parallel-syncs mymaster 1  
sentinel failover-timeout mymaster 15000  
~~~

sentinel26579.conf

~~~she
port 26579 // 当前Sentinel服务运行的端口  
sentinel monitor mymaster 127.0.0.1 6379 2   
sentinel down-after-milliseconds mymaster 5000  
sentinel parallel-syncs mymaster 1  
sentinel failover-timeout mymaster 15000  
~~~



###### 2.3 配置文件说明

* port :当前Sentinel服务运行的端口  
* sentinel monitor mymaster 127.0.0.1 6379 2:Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为本机地址127.0.0.1，端口号为6379，而将这个主实例判断为失效至少需要2个 Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行  
* sentinel down-after-milliseconds mymaster 5000:指定了Sentinel认为Redis实例已经失效所需的毫秒数。当 实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行  
* sentinel parallel-syncs mymaster 1：指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长  
* sentinel failover-timeout mymaster 15000：如果在该时间（ms）内未能完成failover操作，则认为该failover失败  

###### 2.4 启动redis服务

~~~she
启动三个cmd窗口

redis-server.exe redis.conf  
redis-server.exe redis6380.conf  
redis-server.exe redis6381.conf  

也可以配置成Windows服务启动
~~~



 ###### 2.5 启动哨兵

~~~shel
启动三个cmd窗口

redis-server.exe sentinel.conf --sentinel  
redis-server.exe sentinel26479.conf --sentinel  
redis-server.exe sentinel26579.conf --sentinel   
~~~



可以看到哨兵启动的日志，在监控redis服务和其他哨兵的状态



###### 2.6 关闭redis主服务

会发现三个哨兵的命令窗口监控到主redis down了，并最终投票决定那个从redis为主redis



最终会修改对应 从服务的 redis.windowsxxx.conf 重新自动配置主从关系