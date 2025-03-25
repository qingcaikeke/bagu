[上岸字节了！分享一些 idea (qq.com)](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247527249&idx=1&sn=25c54a80ae53d86528a259840fe6e77b&chksm=f98d2ffbcefaa6edccd86a629f92714b27f8b8525f31bba7dda567f07abaf627bbd4e76f783f&scene=21#wechat_redirect)项目指点

[又被百度捞起来了，能赢吗？ (qq.com)](https://mp.weixin.qq.com/s/1WkJBMMAW1lUZtmWcRQIxQ)合并区间 （排序加双指针），接雨水（动态规划，双指针，单调栈），

[[被字节拷打了！基础还是太重要了... (qq.com)](https://mp.weixin.qq.com/s/EQjV5mBP-0Al1HFFYzOcJA)验证对称二叉树（递归，bfs），岛屿数量（dfs，bfs）

[阿里问的相当基础！ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247527392&idx=1&sn=b1720341b9685328e7f4de0b8bc5c63a&chksm=f98d2f4acefaa65c04857a67686ba00dd7babe54e2364434328ab9eb1dc4219903f3d46a67fa&scene=21#wechat_redirect)数据库的多表联查

[腾讯面试体验倍儿好 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247527244&idx=1&sn=54432eaf69b68b2d8cea97c15b9aa578&chksm=f98d2fe6cefaa6f07a5f3532937e5a3c92de2ed516f08861f220e0b1b2ce5735aa90bc233ba5&scene=21#wechat_redirect)链表判断相交

[一面—京东_牛客网 (nowcoder.com)](https://www.nowcoder.com/feed/main/detail/199c48fb045f467ca899bd7712ae6e8d?sourceSSR=search)

3-8

[快手 效率中心 后端实习 一面_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/581515501239554048?sourceSSR=users)

[momenta 后端实习 一面_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/581516033962287104?sourceSSR=users)

3-13[腾讯云一面 3.11 20.30_牛客网 (nowcoder.com)](https://www.nowcoder.com/feed/main/detail/27a1543b25a047cc86fd48334eafa238)

3-16[腾讯后台开发方向-暑期实习一面_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/596743063414816768)

3-29[OPPO 后端二面，凉凉。。。 (qq.com)](https://mp.weixin.qq.com/s/QrGpIzX7OqLAR5fXp_B0YA)

4-5[字节 飞书办公套件 后端实习 一面_牛客网 (nowcoder.com)](https://www.nowcoder.com/discuss/581892301245390848?sourceSSR=users)较难

- 大概讲一讲项目
- 讲一下简历里的技术难点







[SpringBoot下如何实现Redis + Caffeine二级分布式高性能缓存_redis+caffeine-CSDN博客](https://blog.csdn.net/hxx688/article/details/120601972)

### 多级缓存

- 请求到达Nginx后，优先读取Nginx本地缓存
- 如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）
- 如果Redis查询未命中，则查询Tomcat
- 请求进入Tomcat后，优先查询JVM进程缓存
- 如果JVM进程缓存未命中，则查询数据库



问题：进程空间（jvm）大小有限，不支持大数据存储 --》只存储最热的数据（热门商品）

​			重启项目会丢失数据  --》项目重启时重新拉取热点数据

​			分布式场景下，系统间数据不一致--》根据id做负载均衡

**一致性问题：**

设置一个较短的过期时间，过期重新重远程缓存拉取，保证数据最终一致性，一直到热点数据非热点，到时候直接访问远程缓存

**如何判断热数据：**缓存热商品，如疫情中的口罩

技术服务（通过mq调用一个计数服务）/滑动窗口（开源热点探测框架jdhotkey京东）

伪代码

```
boolean exist = false
if(是热点数据){
​	//读本地缓存
​	if(本地缓存存在){
​		return 数据
​	}
	exist = true;
}
//不是热点数据
//查数据
redis.get()
//是热点数据但缓存不存在
if(exist){
    //缓存到本地,并设置过期时间
}
return 数据
```

![image-20240222142206076](C:\Users\16776\AppData\Roaming\Typora\typora-user-images\image-20240222142206076.png)


springCache适合缓存控制没有那么细致的场景，如门户首页，偏静态展示页面，榜单等等

分布式服务可以用 redis 的 pub/sub 实现分布式多级缓存

补：缓存预热，利用大数据统计用户访问的热点数据，在项目启动时将这些热点数据提前查询并保存到Redis中，利用InitializingBean接口来实现，在对象被Spring创建并且成员变量全部注入后执行。

补：为减轻Tomcat压力的多级缓存结构，先查浏览器缓存，然后nginx本地，然后redis，然后tomact，最后再数据库









### 其他

优化思路：多级缓存（caffeine），冷热数据隔离（数据归档）



针对全局流量压力：聚合10s内的点赞数，一次性写入。减少数据库写入次数。异步化写入数据库，保证了数据库能以合理的速率处理写入请求。

单点流量压力：当一个稿件成为超级热门的时候，大量的流量就会涌入到存储的单个分片上，造成读写热点问题。此时需要有热点识别机制来识别该热点，并将数据缓存至本地，并设置合理的TTL。



本地缓存一般就是热点探测+LRU，对于可以预知的热数据，可以使用白名单



**异步任务层（thumbup-job）**

- 异步任务主要作为点赞数据写入、刷新缓存、为下游其他服务发送点赞、点赞数消息等功能

首先是最重要的用户行为数据（点赞、点踩、取消等）的写入。搭配对数据库的限流组件以及消费速度监控，保证数据的写入不超过数据库的负荷的同时也不会出现数据堆积造成的C数据端查询延迟问题。

### rpc项目

**开发过程**

1.整体结构，接口定义（框架暴露的，注册中心的）

2.通信与序列化，netty异步传输（请求id+CompletableFuture构建一个map），背压机制（信号量）控制客户端请求速率

3.客户端：生成桩，代理模式，生成类，编译，加载，不用rpc生成桩的方法不一样

4.服务端：找到服务提供者，利用 Java 反射机制调用服务的对应方法

补：序列化：专用序列化：你要为每一种类型的数据专门编写序列化和反序列化方法。一般的 RPC 框架采用的都是通用的序列化实现，比如 gRPC 采用的是 Protobuf 序列化实现，Dubbo 支持 hession2 等好几种序列化实现

是一个优化点，影响响应速度，吞吐量

补：生成桩：参考Dubbo部分源码，实现了自定义的SPI机制，目前仅支持根据接口类型加载配置文件中的所有具体的扩展实现类，并且可以根据指定的key获取特定的实现类，具体实现类逻辑在

**RPC的流程**

调用远程服务器的方法像调用自己的一样

服务端启动服务，完成服务注册（包含两部分 1.接收到请求后要能找到对应的实现类 2. 要把ip地址和服务名注册到注册中心）

客户端调用注册中心接口的方法，查询服务地址，服务中心从服务列表中根据据负载均衡选择一个服务地址

然后调用rpc框架的方法，拿到桩，然后把参数传给桩，桩发送请求，请求包含方法参数、方法名等

io过程涉及序列化成字节流

服务端调用对应的服务实现类，得到结果，返回给rpc框架，再返回给客户端

在接口定义上，需要使用`@Service`注解标识服务提供者的服务接口，使用`@Reference`注解标识消费者的引用接口。这些注解用于标识Dubbo的服务提供者和消费者，以便Dubbo框架能够识别并管理这些服务。

**单例模式，责任链模式，自定义注解**，建造者模式

注册中心得是单例：减少资源消耗，保证一致性

调用远程服务通过代理模式，

处理请求通过责任链模式

注册中心redis/zk，储存ip地址等，本地再有一级缓存，本地无法命中时从远程拉

客户端服务发现（找对应的实现类）策略模式+命令注册机制，省略ifelse

**你的rpc框架接口用什么做约定，知道工业界是怎么做的吗**

在接口定义上，需要使用`@Service`注解标识服务提供者的服务接口，使用`@Reference`注解标识消费者的引用接口。这些注解用于标识Dubbo的服务提供者和消费者，以便Dubbo框架能够识别并管理这些服务。

自定义的通信协议及编解码器（rpcMessage）（魔术+版本号+序列化算法+消息类型+状态类型+序列号+消息长度）

编解码用于解析请求头中的信息，检验魔术，版本号等，检验消息类型**（心跳还是回复）**，拿到序列化算法，根据消息类型，完成消息体的转化（rpcRequest）

```
工厂创建代理，代理（生成字节码，类加载器加载，这个时候他的方法实际上就已经是远端过程调用了，创建对象，调用方法）invoke，RemoteMethodCall封装rpcmessage，服务发现负载均衡得到服务地址，封装metada，rpcClient使用netty异步发送请求，
```

启动 10000 个线程同时访问 sayHello 接口，总共进行 3 轮测试

**ZK 集群的读写吞吐量不高**所以需要缓存

**序列化和反序列化以及编解码有什么区别**

编解码用于netty发送，序列化反序列化用于接收方和发送方能够理解并还原信息

服务发现--生成桩--序列化--接收--处理

TPS（Transactions Per Second，每秒事务数）

**幂等性怎么做？**

客户端做接口幂等性，服务端setnx加锁

rpc幂等不太重要，像dubbo就没有提供幂等

**常用的RPC框架和他们的区别**

grpc：基于http2

dubbo：易于集成spring

加密算法，压缩算法

### 名词

1bite（字节）是8bit（二进制）1kb=1024bite 1mb=1024kb

本地缓存：Guava，Caffeine，Ehcache或者自己简单的使用HashMap实现

分布式缓存：Redis，Memcached等。

Canal：监听mysql的binLog，实现缓存和数据库的数据同步（缓存数据同步的常见方式：设置有效期，同步双写（改数据库同时直接该缓存），异步通知（改数据库后异步通知改缓存，Canal））

wireshark：抓包用的

mogodb：文档型数据库，类json储存

elasticsearch：分词，倒排索引，可以当数据库用

openrestry：通过lua扩展nginx实现可伸缩的web平台

spring-boot-starter：一组依赖的集成，快速启动某一项目

closable：实现该接口：意味着该类持有一些资源，比如网络连接或文件句柄，需要在不再需要时释放这些资源。

docker -v 主机路径：容器路径(volume) 挂载

netfilx：网飞，电影厂，程序开发（hystrix）

webSocket：双工，长连接，弹幕，体育赛事实时更新（没刷新但页面变化）

URI（identifier）包括URL（locator）和URN（name，如uuid）

为什么要用网关：1.转发，路由，前端对接网关，网关对接后端，2.负载均衡（一个服务有多个实例）3.认证、授权、限流

sha256，md5：加密算法

字节：生活服务不行
