持久化 原子操作 异步复制

**介绍一下redis数据库？**（1.单线程，内存型，读写快2.数据结构3.用途：缓存消息队列分布式锁4.支持持久化，集群）

Redis 是一种基于内存的数据库，对数据的读写操作都是在内存中完成，因此**读写速度非常快**，常用于**缓存，消息队列、分布式锁等场景**。

Redis 提供了多种数据类型来支持不同的业务场景，比如 String(字符串)、Hash(哈希)、 List (列表)、Set(集合)、Zset(有序集合)、Bitmaps（位图）、HyperLogLog（基数统计）、GEO（地理信息）、Stream（流），并且对数据类型的操作都是**原子性**的，因为执行命令由单线程负责的，不存在并发竞争的问题。

除此之外，Redis 还支持**事务 、持久化、Lua 脚本、多种集群方案（主从复制模式、哨兵模式、切片机群模式）、发布/订阅模式，内存淘汰机制、过期删除机制**等等。

Redis常见的**应用场合**有：

- 缓存，消息队列，排行榜或计数，消息发布和订阅，商品列表

**Redis五种基础数据结构及其底层实现**

**string（字符串,整数,浮点数），List（列表），Set（集合），Hash（哈希），Zset（有序集合）**

String 类型的应用场景：二进制安全，能储存任何数据，缓存对象、常规计数、分布式锁、共享 session 信息等。
List 应用场景：消息队列(如何保证消息有序，不重复，可靠性)，生产者需要自行实现全局唯一 ID，不能以消费组形式消费数据
Hash 类型：缓存对象、购物车等。
Set 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动等。
Zset 类型：排序场景，比如排行榜、电话和姓名排序等。底层：跳表（改进的多层有序链表）

补：常用指令 (set k y)(get k)(exist k)(strlen k)(del k)(incr k)(incrby k 10)(decr k)(decrby k 10)

(EXPIRE k 60 / SET k v EX 60)(TTL k)

(set lock_key unique_value EX PX 10000分布式锁，不存在就插入，过期时间10s)

补：pv访问次数（page view），uv(unique visitor)访问人数

**SDS（简单动态字符串simple dynamic string）**

因为传统的c语言字符串是以'/0'结尾，这样导致很多问题，求长度需要遍历，拼接字符串是遍历到尾端然后继续写新的，这样可能导致缓冲区溢出，读到'/0'就认为是结尾了，长度可能计算错误，无法储存二进制数据。

sds结构设计：一个struct，包含字符串长度，缓冲区长度，sds类型，字节数组（保存实际数据）
因为有len记录长度，所以不需要通过'/0'判断结尾，因此是二进制安全的，可以储存图片，视频，音频等数据，同时求长度快
有缓冲区大小，防止 strcat等字符串追加函数导致缓冲区溢出，所需内存小于1MB，翻倍扩容，之后一次加1MB，同时原来的strcat函数需要遍历找到才能找到字符串结尾，然后进行追加，处理效率不高

sds类型的区别在于struct中的len和alloc结构类型不同，标识最大长度和分配空间，同时使用了专门的编译优化，**取消结构体在编译过程中的优化对齐，按照实际占用字节数进行对齐**。从而节省内存

**String和hash存对象：**

hash便于修改单个字段，能节省网络流量，如果字段经常变动或者经常查单个字段信息，用hash

string消耗的内存更少，缓存相同数量的对象数据，String 消耗的内存约是 Hash 的一半，储存**嵌套对象**也很方便，多数情况下用string

**list的底层实现：**压缩列表或双向链表（redis的链表是双向的，有一个节点数量字段）。后改为只使用**quicklist** (双向链表，节点是一个压缩列表，如果插入空间不够只需要新建一个节点)

**map的底层实现：**压缩列表（个数小于512，已改为listpack，先来一个节点储存key在紧跟一个节点储存val）或哈希表

**set的底层实现**：整数集合（元素个数小于512）或 哈希表

**zset的底层实现**：有序集元素个数小于128且每个元素的值小于64字节时用压缩列表（后改为listPack）否则使用跳表

再结合哈希表（key是member，val是score）为什么能排序？基于分数进行跳表的插入删除操作

**压缩列表（已改为listpack）**：区分链表，不是通过指针实现，而是通过字节数

压缩列表：预分配一块**连续的内存空间**（压缩的含义），O(1)找头尾，根据表头字段实现（储存了尾距离头多少字节），每个节点存放当前节点的长度和前一个节点的长度（用于后续遍历）和实际数据，存在连锁更新问题，优点是节省内存，缺点是只会用于保存节点数量不多的场景

listPack：仍然是连续的内存空间，每个节点包含编码类型+实际数据+当前节点长度（取消了记录前一个节点的长度字段，也不需要记录尾）

**哈希表：压缩列表或哈希表**

结构设计：一个struct，包括哈希表数组指针，哈希表大小，已存放的大小

通过**链式哈希**解决哈希冲突，但随着链表长度的增加，在这一位置查询数据的耗时就会增加，通过rehash解决（不是红黑树了）

rehash定义两个哈希表，交替使用，表一储存数据，表二没有分配内存，随着表一空间增大，触发rehash，给表二分配空间，大小为表一的二倍，把表一的数据迁移到表二，释放一的内存，最后把表二置为表一，1置为2。

为了防止hash中数据过大导致迁移时服务阻塞的问题，又提出**渐进式rehash**

**map怎么扩容，扩容时会影响缓存吗，渐进式rehash**

为了防止阻塞，提出渐进式rehash，迁移工作分多次完成，每次增删改查的时候除了执行操作，还会考到表2上，保证1的数据只会减少，随着请求操作，最终把所有数据迁移到表二，增删改查会同时在两个表执行，如查的时候先去表1查，表1查不到去表2

补：是否触发通过计算负载因子：保存的节点数除以哈希表大小

补：为什么是扩容二倍：减少扩容时的迁移，（16扩容32）将原来的哈希值与新增加的容量（16）进行&运算，如果为0就不用扩容，否则原来的位置加16

**跳表结构了解吗，和二叉树有什么区别，（从时间复杂度和空间复杂度分析，不会？）**

跳表相当于给链表加了一个多级索引，使得查找更快，一般来说上一层节点数为下一层一半

查找复杂度**logn**，实际上也使用了个二分的思想，从最上层开始，要么向右要么向下，空间复杂度o（n）

链表查询复杂度on，查询效率低，所以产生了跳表，设计灵感来源平衡树

跳表是多层有序链表，底层包含所有元素，是双向链表

**跳表怎么维护平衡** ：生成0到1随机数，小于0.25就加一层，以此维持相邻两层的节点数量的比例为 2 : 1（n+a = 2(n/2+a/4)）

**补：为什么用跳表而不用平衡树？**

1.跳表实现更简单，不需要复杂的平衡调整操作，2.维护更简单，通过控制层级，更容易控制内存消耗，3.**范围查找效率更高**

**补：有序表在什么情况使用跳表**

跳表就是有序表加了个多级索引，所以查找比原来更快，同时增删也很快，所以redis的zset用了跳表，然后范围查找的话也很方便，因为最下层是双向链表。

**Redis里面的操作是原子操作吗，如何实现的**？

 单线程，支持事务，数据结构操作也是原子的（如：set key  value，给list push 元素）

**说说你对Redis操作原子性的理解**

redis是单线程模型的，命令的执行过程不会涉及线程切换，整个命令的执行不会被中断，要么全部执行成功要么全部失败

**redis事务**

redis的事务是将所有命令序列化然后按顺序执行，期间不会被其他命令打断，但是如果有一条命令执行失败不会回滚，而是会执行后面的命令。（redis没有回滚的概念），因此不保证原子性

本质上是最后执行一个批处理

K) **Redis 是单线程吗？**（多核机器里使用会不会浪费机器资源？）

主要工作单线程（解析请求，进行读写），6.0之后，io处理多线程，**持久化，异步删除，集群数据同步**使用后台线程

Redis 单线程指的是主要工作是单线程，即「接收客户端请求->解析请求 ->进行数据读写等操作->发送数据给客户端」这个过程是由一个线程（主线程）来完成的。支持高并发是因为结合io多路复用，一个线程监听多个socket（监听soc和已连接soc），通过事件循环函数
6.0之后采用多个io处理线程处理网络请求建立socket请求）

Redis 程序并不是单线程的，Redis 在启动的时候，是会启动后台线程的（持久化，异步删除，集群数据同步）

**Redis为啥单线程还这么快**（内存，单线程，多路复用）（处理io请求是多线程，命令执行是单线程）

内存存储、单线程模型、高效的数据结构、io多路复用，事件驱动的非阻塞

内存型数据库，读写快（性能瓶颈在内存和网络带宽，不在cpu，也就不需要多线程）。

单线程模型，简化并发控制，避免多线程竞争与同步开销（加锁解锁，上下文切换），没有死锁问题

非阻塞io结合io多路复用（epoll）处理大量客户端socket请求（一个线程处理多个io流）。

**优化：批量操作，管道技术**，减少了网络通信开销

补： Redis 6.0 版本之后，因为网络硬件能力提升，有时候redis的性能瓶颈为网络io处理，所以也采用了多个 I/O 线程来处理io请求，对命令的执行仍然是单线程，多路复用在调用epoll的过程是阻塞的，高并发会成为性能瓶颈

**那如何利用多核心？**docker部署多个redis

K) **Redis里的数据落地(持久化)机制**

（aof存命令，文件大，恢复慢，丢失少）（rbd存数据，文件小，恢复快，丢失多）

**AOF日志**：**先**执行写命令，**后**将执行的写命令追加到一个文件末尾    配置参数（内核写回到硬盘的时机，Always，Everysec，No）

优点：无需**额外检查**，不会阻塞当前命令执行    缺点：数据可能丢失，可能阻塞其他操作

具体流程：**日志写到aof_buf(用户区)->拷贝到page cache(内核中的磁盘高速缓冲区)->等待内核将数据写到硬盘**

**AOF重写**：避免AOF文件越写越大，在重写时，获取所有的k-v，用一条新命令记录到新的AOF文件。

具体过程：创建**子进程**（不是线程）避免阻塞，父子进程**共享内存**但只读，主进程修改时发生**写时复制**，创建独立的数据副本进行修改，

重写期间，redis执行一个写命令，同时将命令写到AOF缓冲区和AOF重写缓冲区，完成后像主进程发信号，将重写缓冲区的内容**追加**到新AOF，最后改名覆盖现有的AOF

**RBD快照**：把某个时间点的数据以二进制的方式保存到硬盘的一个文件中，而非操作命令，恢复效率更高，但操作频率太高会对redis性能产生影响，频率太低会丢失数据，执行快照的时候可以修改数据，通过写时复制实现（可以定时，可以手动，可以设置触发条件）

**混合持久化**：aof丢失数据少，恢复慢，rbd丢失数据多，恢复快，混合持久化工作在AOF日志重写这一过程，重写子进程先将数据以Rbd的形式写到AOF文件，这个过程中的写操作记录到AOF重写缓冲区，再将这部分增量命令以命令格式写到AOF

补：fork创建子进程会复制页表（虚拟内存到物理内存的映射）

补：写时复制，多任务访问相同的数据用同一个数据而非创建多个数据副本，一旦有一个任务决定修改数据，系统才会为该任务创建独立的数据副本

**Redis 如何实现服务高可用（redis宕机怎么办）**

**1.主从复制，读写分离**，数据修改只在主服务器进行，收到写命令，发给从服务器。但是，主服务器并不会等到从服务器实际执行完命令后，再把结果返回给客户端，而是主服务器自己在本地执行完命令后，就会向客户端返回结果了。如果从服务器还没有执行主服务器同步过来的命令，主从服务器间的数据就不一致了（异步的）（不是强一致性）

2.**哨兵模式**：监控主从服务器，提供主从节点故障迁移的功能（ping），哨兵节点主要负责三件事情：**监控（ping）、选主、通知**。

3.**切片集群**：数据量大到一台服务器无法缓存，将数据分不到不同服务器上，形成集群，一个集群16384个哈希槽，通过哈希槽处理数据和节点之间的映射关系

**脑裂问题**

主从复制存在集群脑裂问题，可能导致数据丢失

由于网络问题，集群节点之间失去联系。主节点仍能接收请求，但无法像从节点复制，主从数据不同步；哨兵重新组织选举，产生两个主服务。等网络恢复，旧主节点会降级为从节点，再与新主节点进行同步复制的时候，会清空自己的缓冲区，导致之前客户端写入的数据丢失了。

解决方法：主库连接的从库中至少有 N 个从库，和主库进行数据复制时的 ACK 消息延迟不能超过 T 秒，否则，主库就不会再接收客户端的写请求了。

**Redis主从复制**（为什么要用？如何实现？缺点是什么）

**redis新节点加入集群会发生什么**+

主从复制分**全量复制**和**增量复制**

全量复制： bgsave 命令来生成 RDB 文件（生成子进程，不会阻塞主线程工作），生成rbd，发送，从服务器接收并加载这个过程中的新的写命令，会存到 **replication buffer 缓冲区**里，从服务器加载完rbd会回复一个确认，然后主服务器把缓冲区里的命令发送给从服务器，此时第一次同步完成

之后双方维护一个tcp长连接，进行增量复制（写传播），主服务器有一个环形缓冲区（repl_backlog_buffer），主从各自维护一个offset，主记录自己写到的位置，从记录自己读到的位置，然后从把自己的offset发送过去，主节点将需要发送的数据从环形缓冲区存到 **replication buffer 缓冲区**，然后发送。

补：主会给从发送心跳，确定从在线，从会给主发offset确定主在线，过期key会模拟一个del命令，存在两个缓冲区，replication每个从节点都有一个，环形缓冲区只有主节点有

补：应对主从不一致，offset大于阈值，不让从这个从节点读取数据，减少丢失：同步所需时间超过阈值，认为故障丢失数据会很多，禁止向主节点写数据，可以通过降级操作，把命令存到kafka或是直接写到磁盘

补：可能丢失写操作，主写完，返回成功，从还没来的及同步，主宕机

 **主从复制中两个 Buffer(replication buffer 、repl backlog buffer)有什么区别？**

replication buffer 、repl backlog buffer 区别如下：

- 出现的阶段不一样：
  - repl backlog buffer 是在增量复制阶段出现，**一个主节点只分配一个 repl backlog buffer**；
  - replication buffer 是在全量复制阶段和增量复制阶段都会出现，**主节点会给每个新连接的从节点，分配一个 replication buffer**；
- 这两个 Buffer 都有大小限制的，当缓冲区满了之后，发生的事情不一样：
  - 当 repl backlog buffer 满了，因为是环形结构，会直接**覆盖起始位置数据**;
  - 当 replication buffer 满了，会导致连接断开，删除缓存，从节点重新连接，**重新开始全量复制**

K) **Redis过期删除策略**（有什么？redis用的是什么？）

可以给key设置过期时间，将key和过期时间储存到过期字典，查key时先看过期字典是否存在，存在的话再比较当前时间与过期时间

**惰性删除：**不主动删除过期key，每次访问时检查是否过期，若过期则删除，返回null

优点：系统资源占用少 缺点：如果过期的key一直没被访问，就会一直留在库中，对内存不友好

**定期删除：**每隔一段时间取出一定数量的key检查，删除过期的，如果发现过期比例过高会再次执行，如果大量过期，循环时间会很长，可能导致线程卡死，因此除了设置删除执行的频率还要设置一个执行时间上限

优点：减少过期键对空间的占用  缺点：难以确定执行时常和频率，太频繁对cpu不好，太少又和惰性删除一样

redis采用惰性删除和定期删除配合使用，但还是可能会漏掉很多过期key，有引入了内存淘汰策略

K**Redis内存淘汰策略（xx如何保证redis中都是热点数据）**（有点像页面置换）

**内存淘汰：**允许在资源紧张时，根据一些策略，主动删除一些键值对，以释放空间保证系统稳定性

常见策略：LRU（淘汰最近最久未使用）,LFU（淘汰频率最低）,RANDOM（随机）,TTL（淘汰剩余生存时间最短）,

不淘汰策略，空间不足以容纳新写入的数据时，返回错误，写操作执行失败

补：lru和LFU和RANDOM分全局的和只在设置过期时间的里面选

#### **缓存雪崩、缓存击穿、缓存穿透？**

**缓存雪崩**：**大量缓存数据在同一时间过期或者 Redis 故障宕机**，若此时有大量用户请求，都无法在redis处理，全部冲向数据库，数据库压力骤增，严重的会造成数据库宕机，从而形成一系列连锁反应，造成整个系统崩溃。
解决方案：**1.将缓存失效时间随机打散**（在原有时间基础上加一个随机值）
**宕机：2.**服务熔断（暂停访问，直接返回错误）请求限流（只接收少量请求发送到数据库）**3.**构建集群，主节点故障切换到从节点

**缓存击穿**：**热点数据**过期，并且重建比较复杂（缓存可能涉及多张表及运算最后缓存一个结果）

解决方案：**1.互斥锁（分布式锁）**，线程1查缓存未命中，查数据库，构建缓存，构建过程中，线程2，3，4也未命中，都去数据库并构建缓存，导致数据库访问压力大，所以用互斥锁和双重检测给构建缓存这个过程加锁，线程1加锁构建缓存，线程234拿不到锁，休眠一会重试，等拿到锁进行**二次检测**，检测当前缓存是否存在，若存在直接返回（锁也要加过期时间，防止发生意外一直不释放）
 **2.设置逻辑过期时间，**过期时间不作用于redis，而是从value中判断当前数据过期了（把原来打算存的类和一个过期时间封装成新的类，当作新的val），线程1发现过期，加锁，创建一个新线程**异步更新**数据，更新完了再返回，线程234也发现过期了，但获取不到锁，直接返回旧数据  --------- 黑马点评中是先加分布式锁，然后给线程池提交任务，得不到锁返回null

**缓存穿透**：要访问的数据**不在缓存也不在数据库**，没法构建缓存来服务后续请求（可能是误删除或黑客恶意请求不存在的信息）

解决方案：**1.**限制非法请求（api入口处判断请求参数是否合理，是否有非法值，请求字段是否存在）
**2.**缓存空值，这样就不会继续查数据库（问题：占用空间，缓存大量非法数据，可能造成数据不一致（因为后续可能插入了这个数据），所以要设置ttl）
**3.**使用布隆过滤器快速判读数据是否存在，判断不存在就不用访问数据库（redis本身支持布隆过滤器，存在误判）需要将所有可能存在的key预先加载到过滤器中

其他：多级缓存（本地缓存+分布式缓存）高可用架构（主从复制加集群，避免单点故障）

补：双key策略，主key设置过期时间，备key永久，主过期，返回备key得内容

**缓存预热：**

业务刚上线或redis宕机恢复的时候，我们最好提前把数据缓起来，而不是等待用户访问才来触发缓存构建，这就是所谓的缓存预热

**Redis如何实现分布式锁**

分布式锁：**分布式系统或集群模式下**，多线程可见并互斥的锁，实现的核心思想是大家都使用一把锁

举例：一人一单问题，需要先根据用户id加锁，拿到锁对象，去数据库找有没有用户id相同且订单id相同，下单，释放锁

问题：分布式环境下，用户多次点击，请求到不同的服务器，每个服务器都能加锁（锁对象不同），就会导致多次下单

为什么用redis：redis本身**被多个客户端共享**，多进程可见，可以用来保存分布式锁，读写性能高，可以应对高并发

原理：SET lock_key unique_value NX PX 10000 

例如:  key是lock:用户id（一人一单），val是uuid+线程id（不同服务器线程id可能相同）

三个要求1.读取锁变量、检查锁变量，设置变量值要以原子操作完成

2.设置过期时间，防止客户端发生异常，无法释放锁

3.解锁就是把锁删除，要保证删锁的客户端就是加锁的客户端，防止误释放，包括两个操作（1.检验，2.删除）需要lua脚本保证原子性

误删：持有锁的线程1阻塞，没有完成任务，没有触发解锁，但到了时间自动释放锁，线程2得到锁；线程1恢复，继续执行，最终会把线程2的锁释放

**分布式锁怎么解决缓存击穿的**

击穿：热数据过期，多个分布式服务去访问数据库想要重新构建缓存

一个请求发现缓存不存在，首先尝试获得分布式锁，如果拿到锁，进行二次检查，缓存仍不存在则访问数据库构建缓存

**分布式id或者说全局唯一id**

需要有一个分布式环境中的唯一标识符，标识各种实体。

如果使用主键自增会有一些问题1.规律性太明显，可能被猜到信息（一天卖了多少单）2.数据量大需要分表，分表后逻辑上仍然是一张表，需要保证id唯一性和递增性。

**UUID**：值是随机的，导致随机插入，页分裂，影响查询性能

**Snowflake算法**（雪花）：时间戳、数据中心ID、机器ID和序列号

**redis/数据库自增ID:**

**数据库与缓存双写一致性**

改缓存和改数据库一定要同时成功或失败，所以必须**事务操作**

为什么是删除缓存而非更新缓存（1.防止一直没有使用，但不断发生更新数据库，从而导致不断更新缓存2.可能有并发问题，两个线程同时更新数据库，更新缓存，执行顺序不同，结果不同3.缓存可能通过复杂计算得到，如果更新频率高会浪费性能）

一定是先更新数据库再删缓存（删缓存的动作很快，被别的线程打断这一动作的概率小，就不会从缓存读到与数据库不一致的数据）（缓存恰好过期（或缓存不存在），读旧数据但没来的及写缓存，新线程改了数据库，改缓存，旧线程又切换回来，把旧数据写入缓存）

先删缓存，再更新数据库（可能刚删完缓存还没有更新数据库，此时发生线程切换，新线程访问发现没有数据，去数据库中找，然后用旧数据构建了缓存，导致不一致，发生的概率较大，因为删缓存是一个较快的动作，改数据库是一个较慢的动作）（删缓存，写数据库），（查缓存，读数据库）后者执行快，前者慢

**补：延迟双删**：先删缓存再删数据库，延迟N秒再删一次缓存

**如何实现按照积分降序排序，按照时间升序排序**

把时间信息和积分想办法编码到一起，用于分数，按socre降序，所以要想办法让时间小的反而分数高，可以指定一个时间，可以让时间变成负数

**布隆过滤器**

一种数据结构，解决海量数据的存在性问题且容忍轻微误差这一场景，对缓存穿透、海量数据去重这一场景很适合

比list，map，set占用空间更少，效率更高

具体实现：用一个较大的bit数组（位图）保存所有数据，每个位的值只能是1，**用多个哈希函数**，对指定元素做哈希计算，得到哈希值对位图数组长度取模，得到位置，将对应位数组置为1，判断是否存在时，就重新进行相同的哈希计算，看是否所有位都为1（说存在可能为误判，说不在则一定不在）

补：哈希函数：随机映射函数，常用取余的方式

**如何解决热key问题**

热 Key 问题是指在缓存系统中，某些特定的缓存key受到高频访问，导致对这些热门数据的读取/写入操作集中在少数几个缓存节点上，使得这些节点的负载过高

解决方法：1.缓存预热，业务低峰期提前加载热门数据 2.一致性哈希，尽可能把热数据分到不同接待你 3.数据分片，让热key尽可能均分

**说说你对PIPELINE的理解**

用于批处理传送命令，减少io次数，适合大量请求，对延迟不敏感的数据

**哨兵机制**：自动发现故障并完成主节点的切换并通知应用方

作用：监控（ping）、故障转移、通知

1.leader选举 2.主观下线--客观下线 3.ping过滤调所有的下线节点，从剩下的节点选优先级最高，复制偏移量最大的

4.leader完成新主接待你的切换 5.通知客户端重定向

**高并发场景下我们如何保证幂等性**？

分布式id当主键而非自增id 2.乐观锁实现幂等性 3.分布式锁 4.token当key

补:**Redis健壮性**,健壮性指处理异常输入或发生故障情况的能力

从可持久化，主从复制，哨兵，内存管理，原子操作来讲

**geo怎么存地理位置的**

底层数据是一个zset，完成了从**经纬度到score**的映射，储存是key 经纬度 member（店铺，也是val）这样的结构，可以计算两点间距离，可以根据member查坐标，可以给定中心，按指定范围（方形或圆形）搜索member，并按距离排序返回

**HyperLogLog**

提供不精确的去重计数，并且不需要储存所有值，多用于UV统计

**redis写操作日志记录什么**

记录操作类型，例如 SET、DEL、HSET 等，指示执行的具体操作是写操作

记录被操作的键（Key）和相应的值（Value），包括写入操作的目标键和相应的值内容。

「`*3`」表示当前命令有三个部分，每部分都是以「`$+数字`」开头，后面紧跟着具体的命令、键或值。然后，这里的「`数字`」表示这部分中的命令、键或值一共有多少字节。例如，「`$3 set`」表示这部分有 3 个字节，也就是「`set`」命令这个字符串的长度。

记录时间戳，写操作发生的时间，以便进行操作追溯和日志分析。？

记录执行写操作的客户端信息，包括客户端IP地址、连接信息等，以便进行操作追踪和监控？

**和关系型数据库的区别**

**CAP原理**（consistent,available,partation tolerance）

分区发生时，一致性和可用性难以两全

#### redission/分布式锁

分布式锁，可重入锁，可续期

利用set nx ex，可重入通过val设为线程id+重入次数实现

续期通过看门狗实现，起一个后门线程，每隔有效期的1/3检查锁是否仍被持有，是的化延长有效期

**补：**zk怎么实现分布式锁？锁节点下创建临时节点，判断自己创建的节点是否是需要最小的，是的化解锁成功，不是的话监听比自己小的前一个节点的删除事件，该节点删除后，再次判断自己是否是最小的

**加完锁服务器挂了咋整？**防止其他线程加不了锁，设个过期时间，配合看门狗续期；防止锁误释放，val放线程id；

**加完锁redis挂了咋整？**redis集群+红锁，需要在超过一半的节点上加锁成功才加锁

#### 数据库缓存一致性

Cache-Aside旁路缓存：写请求先更新数据库再删缓存，适用于一致性较高缓存更新比较复杂的业务（旁路：绕过缓存，直接到数据库）

​	防止：删完缓存 数据库没来的及更新，旧的又被写进去了，数据库更完认为缓存删过了（先删缓存），之后到下次更新都是不一致

readThrough：应用只和缓存交互，缓存没有，缓存主动去数据库读

Write-Through：写请求先查cache，若不存在，只更新数据库，若存在，则先更新缓存再更新数据库。

write-Back：应用程序写入数据时，先写入缓存，缓存将数据标记为脏数据，在合适的时机（如缓存空间不足等情况）再将脏数据批量写入数据库。

### todo



命令：setnx key val

​			expire key time

​			ttl key

save 3600 1 -- 3600 秒内有1个key被修改，触发RDB
save 300 100 -- 300 秒内有100个key被修改，触发RDB
save 60 10000 -- 60 秒内有10000个key被修改，触发RDB

分布式锁 set kv nx ex 10(因为setnx和过期时间需要同时成功或失败，需要原子操作，只能写在一条语句里)

