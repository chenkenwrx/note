### Redis

> 基本的理论先学习，然后将知识融会贯通

### 1、基本应用

可用于缓存，消息（支持 publish/subscribe 通知），按key设置过期时间，过期后将会自动删除，具体淘汰策略有

- volatile-lru：从已经设置过期时间的数据集中，挑选最近最少使用的数据淘汰

- volatile-ttl：从已经设置过期时间的数据集中，挑选即将要过期的数据淘汰

- volatile-random：从已经设置过期时间的数据集中，随机挑选数据淘汰

- allkeys-lru：从所有的数据集中，挑选最近最少使用的数据淘汰

- random：从所有的数据集中，随机挑选数据淘汰

- no-enviction：禁止淘汰数据

### 2、Redis入门

#### 1、概述

> Redis 是什么？

支持网络、可基于内存亦可持久化的日志型、Key-Value数据库

> Redis 能干嘛？

1. 内存存储、持久化、内存中是断电即失、持久化很重要（RDB、AOF）
2. 效率高、可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器（浏览量！）

> 特性

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务

#### 3、基础知识

> redis 默认 16 个数据库，默认使用第 0 个

> Redis 是单线程的

很快！是基于内存操作，CPU并不是 Redis 的性能瓶颈，瓶颈是根据 **机器内存和网络带宽**

**Redis 为什么单线程还这么快**？

1、误区1：高性能的服务器一定是多线程的

2、误区2：多线程（CPU上下文切换）一定比单线程效率高

**核心：redis 所有数据放到内存、所以说单线程操作效率最高、对于内存系统来说，没有上下文切换效率就是最高的！多次读写都是在一个CPU上边的**

#### 4、五大数据结构

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用==作数据库、缓存和消息中间件==。 它支持多种类型的数据结构，如 ==字符串（strings）==， ==散列（hashes）==， ==列表（lists）==， ==集合（sets）==， ==有序集合（sorted sets）== 与==范围查询==， ==bitmaps==， ==hyperloglogs== 和 ==地理空间（geospatial）== 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

**Redis-Key**

~~~bash
D:\software\redis>redis-cli.exe -h 127.0.0.1 -p 6379  //windows 连接redis
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> set name haosong  
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set age 1
OK
127.0.0.1:6379> keys *
1) "name"
2) "age"
127.0.0.1:6379> EXISTS NAME
(integer) 0
127.0.0.1:6379> EXISTS NAME1
(integer) 0
127.0.0.1:6379> EXISTS name
(integer) 1
127.0.0.1:6379>  keys *  // 所有的 key
1) "name"
2) "age"
127.0.0.1:6379> set name
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> get name  // 获取键值
"haosong"
127.0.0.1:6379> EXPIRE name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> move name 0
(error) ERR source and destination objects are the same
127.0.0.1:6379> move name 1 // 移除固定键值
(integer) 0
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> set name haosong 10 10
(error) ERR syntax error
127.0.0.1:6379> type name // 查看键值类型
none
127.0.0.1:6379> type age
string
127.0.0.1:6379>
~~~

##### 1、String（字符串）

String 类似的应用场景：value 除了我们的字符串还可以是我们的数字

- 计数器
- 多维度统计数量

##### 2、LIst

在 Redis 里边，可以吧它当成 栈、队列、阻塞队列

> 小结

- 实际上是一个链表，两端都可以操作
- 如果 key 不存在，创建新的链表
- 如果 key 存在，新增内容
- 如果移除了所有的值，空链表，也表示不存在
- 两边插入或者改动，效率最高！中间元素，效率较低

消息排队！消息队列 （Lpush Rpop）、栈 （Lpush Lpop）

##### 3、Set

微博、A用户将所有关注的人放在一个 Set集合中！将他的粉丝也放在一个集合中

共同关注、共同爱好、二度好友

##### 4、Hash

Map 集合，key-map 是一个map集合！本质 和 String 类型没有太大区别，是一个简单的 key-value!

hash 变更的数据 user name age 尤其是用户信息之类的，经常变动的信息！hash 更适合对象存储，String 更适合字符串

##### 5、Zset（有序集合）

在 set 基础上，增加了一个值

案例思路：set 排序、班级成绩表、工资表排序

普通消息：1、重要消息；2、带权重进行判断！

排行榜应用实现

#### 5、三种特殊数据类型

##### 1、geospatial 地理位置

朋友定位、附近的人、打车距离计算

六个命令

- geoadd
- getpos                           获取当前定位：一定是一个坐标值
- geodist +（长度单位）   两个位置之间的距离
- georadius                       以定位的经纬度为中心，找出某一半径内的元素
- georadiusbymember     找出指定元素周围的其他元素
- geohash                         返回一个或者多个元素的 geohash 表示

​      注意：geo 底层实现原理就是 Zset！可以使用Zset命令来操作geo！

##### 2、Hyperloglog

> 基数：不重复的元素

Redis Hyperloglog 基数统计的算法！

优点：内存固定、2^64 不同元素的基数，只需要12kb

##### 3、Bitmaps

> 位存储

统计疫情感染人数：01010

统计用户信息：活跃、不活跃；登录、未登录；打卡、未打卡

Bitmaps 位图，数据结构！操作二进制来进行记录，只有 0 和 1

### 6、事务

Redis 事务本质：一组命令的集合！一个事务中所有命令都会被序列化，事务执行过程中，按照顺序执行

一次性、顺序性、排他性！执行一系列的命令

**Redis 事务没有隔离级别的概念**

所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行

**Redis 单挑命令保证原子性、事务不保证原子性！**

redis 的事务：

- 开启事务（multi）
- 命令入队（........）
- 执行事务（exec）

**正常执行事务**！

**编译时异常**（代码有问题！命令有错），所有命令都不会被执行

**运行时异常**（1/0），如果存在语法性，那么执行命令的时候，其他可以正常执行，错误命令抛出异常

> 监控 Watch

**悲观锁：**

- **认为什么时候都不会出现问题、无论做什么操作都加锁**

**乐观锁：**

- **认为什么时候都会出现问题、不会上锁！更新数据时候去判断是否有人修改过数据，version！Watch**
- **获取 version**
- **更新的时候比较version**

**Redis** 

**测试多线程修改值，改用 watch 可以当做 redis 的乐观锁操作**

**事务执行失败，重新获取锁，再修改即可**

### 7、Jedis

> Redis 官方推荐的 java 连接开发工具！

添加依赖包

~~~java
<dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.3.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.73</version>
        </dependency>
~~~

测试连接

~~~java
public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        System.out.println(jedis.ping());
    }
~~~

事务操作

~~~java
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("Hello", "World");
        jsonObject.put("name", "haosong");
        Transaction transaction = jedis.multi();
        String result = jsonObject.toJSONString();

        try {
            transaction.set("user1", result);
            transaction.set("user2", result);
            int i = 1 / 0;
            transaction.exec();
        } catch (Exception e) {
            transaction.discard();
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
        }

    }
~~~

#### 8、SpringBoot 整合

SpringBoot 操作数据：spring-data jpa jdbc mongodb redis

SpringData 也是和 SpringBoot 齐名的项目

说明：SpringBoot2.x之后，jedis被替换成了lettuce

jedis：采用的直连、多个线程操作不安全；要避免不安全，使用jedis pool 连接池！BIO 模式

lettuce：采用netty，实例在多个线程中进行共享，不存在线程不安全情况！NIO 模式

**源码分析**：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
        // 默认 RedisTemplate 没有过多设置，redis 对象都需要系列化
        // 两个泛型都是 Object, Object 的类型，后期使用需要强制转换
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean  
       // String 类型最常使用，单独提出来一个
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

**自定义 RedisConfig**

~~~java
@Configuration
public class RedisConfig {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        Jackson2JsonRedisSerializer j2jRS = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        j2jRS.setObjectMapper(om);

        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setValueSerializer(stringSerializer);
        template.setHashValueSerializer(stringSerializer);
        template.afterPropertiesSet();
        
        return template;
    }

}
~~~

### 8、Redis.conf 详解

- **配置文件对大小写不敏感**

  ~~~bash
  # units are case insensitive so 1GB 1Gb 1gB are all the same.
  ~~~

- **可以使用 include包含多个文件**

  ~~~bash、
  # include /path/to/local.conf
  # include /path/to/other.conf
  ~~~

- **网络配置**

  ~~~bash
  # bind 127.0.0.1 ::1      #绑定的ip
  protected-mode yes
  port 6379
  ~~~

- **通用**

  ~~~bash
  # Use 'yes' if you need it.
  # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
  # NOT SUPPORTED ON WINDOWS daemonize no
  
  # 日志
  # Specify the server verbosity level.  
  # This can be one of:
  # debug (a lot of information, useful for development/testing)
  # verbose (many rarely useful info, but not a mess like the debug level)
  # notice (moderately verbose, what you want in production probably)  生产环境适用
  # warning (only very important / critical messages are logged)
  loglevel notice
  logfile "" # 日志的文件位置名
  database 16  # 数据库数量
  ~~~

- **快照**

  ~~~bash
  # 900 s内 ，至少有一个 key 进行了修改，进行持久化
  save 900 1
  # 300 s内 ，至少有10个 key 进行了修改，进行持久化
  save 300 10
  # 60 s内 ，至少有10000个 key 进行了修改，进行持久化
  # 之后会自己定义
  
  stop-writes-on-bgsave-error yes   # 持久化出错是否继续工作
  rdbcompression yes                       # 是否压缩 RDB 文件
  rdbchecksum yes                           # 保存 RDB 文件 进行错误校验
  dir ./                                               # RDB 保存路径
  ~~~

- **主从复制**

- **安全**

  ~~~bash
  127.0.0.1:6379> ping
  PONG
  127.0.0.1:6379> config get requirepass
  1) "requirepass"
  2) ""
  127.0.0.1:6379> config set requirepass "123456"
  OK
  127.0.0.1:6379> config get requirepass
  (error) NOAUTH Authentication required.
  127.0.0.1:6379> auth 123456
  OK
  127.0.0.1:6379> config get requirepass
  1) "requirepass"
  2) "123456"
  127.0.0.1:6379>
  ~~~

- **限制 CLIENTS**

  ~~~bash
  # maxclients 10000                                     # 最大连接客户端数
  # maxmemory <bytes>                              # 最大内存容量
  # maxmemory-policy noeviction                # 内存达到上限后的处理策略
  1、volatile-lru:从设置了过期时间的数据集中，选择最近最久未使用的数据释放；
  2、allkeys-lru:从数据集中(包括设置过期时间以及未设置过期时间的数据集中)，选择最近最久未使用的数据释放；
  3、volatile-random:从设置了过期时间的数据集中，随机选择一个数据进行释放；
  4、allkeys-random:从数据集中(包括了设置过期时间以及未设置过期时间)随机选择一个数据进行入释放；
  5、volatile-ttl：从设置了过期时间的数据集中，选择马上就要过期的数据进行释放操作；
  6、noeviction：不删除任意数据(但redis还会根据引用计数器进行释放),这时如果内存不够时，会直接返回错误。
  ~~~

- **AOF 配置**

  ~~~bash
  appendonly no # 默认使用 RDB，不开启；大部分情况，RDB 完全够用
  appendfilename "appendonly.aof" # 持久化文件名字
  
  # appendfsync always
  appendfsync everysec # 每秒 sync，可能会丢失1s数据
  # appendfsync no
  ~~~

### 9、Redis 持久化

> **什么是 RDB**

![image-20200812155419507](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812155419507.png)

**指定时间间隔将内存中数据集快照写入磁盘、snapshot**

**创建一个单独子线程（fork）来进行持久化、将数据写入临时文件，持久化完成后用临时文件替换持久化好的文件，缺点是最后一次持久化的数据可能丢失**

- 设置持久化的 RDB 文件路径

  ~~~bash
  dbfilename dump.rdb
  ~~~

- 触发机制

  1、save 满足，会触发

  2、执行 flushall 命令，会触发

  3、退出 redis，也会出发

- 如何恢复

  1、RDB 文件放到 redis 启动目录，启动的时候会自动检查 dump.rdb 回复其中的数据

  2、查看需要存放的位置

- 优点：
  1. 适合大规模数据恢复
  2. 对数据完整性要求不高

- 需要一定时间间隔进行操作，意外宕机，最后一次修改的数据就没有了
- fork 进程的时候，占用一定的内存空间

### 10、AOF

**所有命令记录下来，history，恢复的时候都执行一遍**

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812160750925.png" alt="image-20200812160750925" style="zoom:80%;" />

**默认不开启、需要手动配置！注意配置的时候文件名不要错位**

~~~bash
appendonly no

# The name of the append only file (default: "appendonly.aof")
appendfilename "appendonly.aof"
# appendfsync always
# appendfsync everysec
# appendfsync no

auto-aof-rewrite-percentage 100    #fork 新的进程来将文件进行重写
auto-aof-rewrite-min-size 64mb
~~~

**优点：**

1. **每一次修改都同步，文件完整性好**
2. **每秒同步一次，丢失1s的数据**
3. **从不同步，效率最高**

**缺点：**

1. **数据文件较大，修复速度慢**
2. **运行效率慢**

**扩展：**

![image-20200812161441984](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812161441984.png)

**性能建议**：

![image-20200812161536611](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812161536611.png)

### 11、Redis 发布订阅

 Redis 发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送信息，订阅者（pub）接收信息

订阅/发布消息示意图：

第一个：消息发送者

第二个：频道

第三个：消息接收者

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812204032925.png" alt="image-20200812204032925" style="zoom:80%;" />

下图展示了频道 channel1，以及订阅这个频道的三个客户端--client2、client5和client1 的关系

![image-20200812204301274](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812204301274.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时，这个消息就会被发送给订阅它的三个客户端

![image-20200812204427233](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812204427233.png)

> 命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html) 订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html) 查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html) 将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html) 退订所有给定模式的频道。 |
| 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html) 订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html) 指退订给定的频道。 |

原理：

![image-20200812205149918](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200812205149918.png)

这些命令被广泛用于构建即时通信应用，比如网络聊天室、是实时广播和实时提醒等

使用场景：

1、实时消息系统

2、实时聊天

3、订阅、关注系统

### 13、Redis 主从复制

概念：

**主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；**

**数据的复制是单向的，只能由主节点到从节点。**

**默认情况下，每台Redis服务器都是主节点，且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。**

- **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
- **高可用基石**：主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

**默认情况下，每台 Redis 服务器都是主节点；一般情况下只配置从机就好了**

**真正的主从配置应该在配置文件中；配置，这样就是永久的**

> 细节

- 主机可以写，从机不能写只能读取
- 主机断开连接、从机依旧连接到主机的，但是没有写操作；主机回来了仍然能获取到主机写的信息

> 复制原理

**Slave 启动成功连接到 master 后会发送一个 sync 同步命令**

**Master 接到命令，启动后台的存盘进程，同时手机所有接受到的修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。**

**全量复制：slave 服务接受到数据库文件数据后，将其存盘并加载到内存中**

**增量复制：Master 继续将新的所有收集到的修改命令传送给slave。完成同步**

**只要是重连 master，一次完全同步（全量复制）将被自动执行！数据一定会在从机中看到**

> 层层链路

 上一个 M 连接下一个 S！

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200813005757510.png" alt="image-20200813005757510" style="zoom:80%;" />

如果主机断开连接，可以使用 SLAVEOF no one 让自己变成主机！其他的节点就可以手动连接到最新的这个主节点！如果这个时候老大修复了，那就重新连接！

### 14、Redis 缓存穿透和雪崩

> 概述

**主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。**这不是一种推荐的方式，更多时候，我们优先考虑**哨兵模式**



哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**



这里的**哨兵有两个作用**

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机。

![img](https://upload-images.jianshu.io/upload_images/11320039-3f40b17c0412116c.png?imageMogr2/auto-orient/strip|imageView2/2/w/747/format/webp)

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

用文字描述一下**故障切换（failover）**的过程。假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。这样对于客户端而言，一切都是透明的。

> 测试！

1、配置哨兵配置文件

~~~bash
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.01 6379 1
~~~

后边的这个数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的，就会成为主机！

如果 Master 节点断开了，这个时候就会从从机中随机选择一个服务器（投票算法）

> 哨兵模式

如果主机此时后来了，只能归并到新的主机下，当做从机，这就是哨兵的规则！

优点：

1. 哨兵集群，基于中主从复制模式，所有的主从配置优点，全有
2. 主从可以切换，故障可以转移，系统可用性就会更好
3. 哨兵模式就是主从模式的升级，手动到自动，更加强壮

缺点：

1. Redis 不好在线扩容，集群容量到达上限，扩容很麻烦
2. 配置很麻烦，有很多选择

2、启动哨兵！

### 15、Redis 缓存穿透和雪崩

#### 1、缓存穿透

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是**缓存没有命中**，于是向**持久层数据库**查询。发现**也没有**，于是本次查询失败。当**用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透**（==缓存穿透是指**缓存和数据库中都没有的数据**==）

> 解决方案

- 布隆过滤器

  一种数据结构，对多有可能查询的参数以hash形式存储，在控制层先进行校验，不符合就丢弃，避免了对底层存储系统的压力

  <img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200813141233329.png" alt="image-20200813141233329" style="zoom:80%;" />

- 缓存空对象

  当存储层不命中后，及时返回空对象也将其缓存起来，设置一个会过期时间，之后就会从缓存中获取，保护了后端数据源

  <img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20200813141401070.png" alt="image-20200813141401070" style="zoom:80%;" />

  存在问题：

  1. 空值被缓存，意味着缓存更多空值得键
  2. 即使对空值设置了过期时间，还是会存在 缓存层 和 存储层的数据会有一端时间窗口的不一致，对于需要保持一致性的业务有影响

#### 2、缓存击穿

==***\*缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期）\****==，这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

> 解决方案

- **设置热点数据永远不过期**

- **加互斥锁**

  **分布式锁：保证每个key同时只有一个线程区查询后台服务，其他线程没有获得分布式锁的权限，因此只需要等待。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁考验很大。**

#### 3、缓存雪崩

**在某一个时间段内，缓存集中过期失效。Redis 宕机！缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库**

> 解决方案

- **redis 高可用（本质上及时搭建集群）**

- **限流降级**

  - 计数器并发数限流：使用共享变量实现
  - 信号量：使用java中的Semaphore

- QPS限流

  - 计数器法：实现简单，精度不高，重置节点时无法处理突发请求
  - 滑动窗口：滑动窗口的窗口越小，则精度越高，相应的资源消耗也更高。
  - 漏桶算法 : 限制的是流出速率，突发请求要排队，对服务保护较好；流入随机，流出固定
  - 令牌桶算法 ：限制的是平均流入速率，允许一定程度突发请求（无需排队）

  **缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量，比如对于某个key只允许一盒现成查询数据和写缓存，其他线程等待。**

- 数据预热

​        正式部署之前，把可能的数据先预访问一遍，这样部分可能大量访问的数据就会加载到缓存中，在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的点尽量均匀。

### 16、Redis 实现分布式锁

分布式锁特点：

- 互斥性： 同一时刻只能有一个线程持有锁
- 可重入性： 同一节点上的同一个线程如果获取了锁之后能够再次获取锁
- 锁超时：和J.U.C中的锁一样支持锁超时，防止死锁
- 高性能和高可用： 加锁和解锁需要高效，同时也需要保证高可用，防止分布式锁失效
- 具备阻塞和非阻塞性：能够及时从阻塞状态中被唤醒

#### 分布式锁实现方式（自己实现）

```java
public String defaultStock() throws InterruptedException {
        String lockKey = "product_001";

        String clientId =  UUID.randomUUID().toString();

        try {
            Boolean result = stringRedisTemplate.opsForValue().setIfAbsent(lockKey, clientId, 1, TimeUnit.SECONDS);
            if (!result) {
                System.out.println("当前抢购人数过多");
                return "error";
            }

            int stock = Integer.valueOf(stringRedisTemplate.opsForValue().get("stock"));
            if (stock > 0) {
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock + "");
                System.out.println("扣减成功，剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败，库存不足");
            }
        } finally {
            if (clientId.equals(stringRedisTemplate.opsForValue().get(lockKey))){
                stringRedisTemplate.delete(lockKey);
            }
        }
        return "end";
    }
```

#### 分布式锁实现方式（Redssion）

```java
public String defaultRedissionStock() throws InterruptedException {
        String lockKey = "product_001";

        RLock redissonLock = redisson.getLock(lockKey);

        try {
            redissonLock.lock(30, TimeUnit.SECONDS);
            int stock = Integer.valueOf(stringRedisTemplate.opsForValue().get("stock"));
            if (stock > 0) {
                int realStock = stock - 1;
                stringRedisTemplate.opsForValue().set("stock", realStock + "");
                System.out.println("扣减成功，剩余库存：" + realStock);
            } else {
                System.out.println("扣减失败，库存不足");
            }
        } finally {
            redissonLock.unlock();
        }
        return "end";
    }
```

使用 Redssion 主从架构 （redis QPS 接近10w、会导致锁 失效）

#### 高并发分布式锁实现

- ZooKeeper
- 超过半数加锁成功才证明加锁成功（RedLock）、也就是一个客户端同时向多个redis执行加锁操作

#### 分布式锁实现方式

##### 1、基于 Redis

###### 1、加锁实现

1. 利用setnx+expire命令（两步操作、不能保证原子性）
2. 使用Lua脚本（包含setnx和expire两条指令）
3. 2.6.12 版本开始、 SET 命令增加一系列选项（et命令的nx选项，就等同于setnx命令，**value必须要具有唯一性**，我们可以用UUID来做）
   - 客户端1获取锁成功
   - 客户端1在某个操作上阻塞了太长时间
   - 设置的key过期了，锁自动释放了
   - 客户端2获取到了对应同一个资源的锁
   - 客户端1从阻塞中恢复过来，因为value值一样，所以执行释放锁操作时就会释放掉客户端2持有的锁，这样就会造成问题
4. Redlock算法 与 Redisson 实现

###### 2、释放锁实现

​      释放锁时需要验证value值，也就是说我们在获取锁的时候需要设置一个value，不能直接用del key这种粗暴的方式，    因为直接del key任何客户端都可以进行解锁了，所以解锁时，我们需要判断锁是否是自己的，基于value值来判断

##### 2、Redis实现的分布式锁轮子

- 自定义注解（被注解的方法会执行获取分布式锁的逻辑）

  ~~~java
  @Target({ElementType.METHOD})  
  @Retention(RetentionPolicy.RUNTIME)  
  @Inherited  
  public @interface RedisLock {
  
       //锁定的资源，redis的键
      String value() default "default";
  
      //锁定保持时间（以毫秒为单位） 
      long keepMills() default 30000;
  
      //失败时执行的操作
      LockFailAction action() default LockFailAction.CONTINUE;
  
      //失败时执行的操作--枚举
      public enum LockFailAction{  
          GIVEUP,  
          CONTINUE;  
      }
      //重试的间隔
      long sleepMills() default 200;
      //重试次数
      int retryTimes() default 5;  
  }
  ~~~

- 具有分布式锁的Bean

  ~~~java
  @Configuration 
  @AutoConfigureAfter(RedisAutoConfiguration.class)
  public class DistributedLockAutoConfiguration {    
      @Bean    
      @ConditionalOnBean(RedisTemplate.class)    
      public DistributedLock redisDistributedLock(RedisTemplate redisTemplate){       
          return new RedisDistributedLock(redisTemplate);   
      }
  }
  ~~~

- 面向切面编程-定义切面

  ~~~java
  @Aspect  
  @Configuration  
  @ConditionalOnClass(DistributedLock.class)  
  @AutoConfigureAfter(DistributedLockAutoConfiguration.class)  
  public class DistributedLockAspectConfiguration {
  
      private final Logger logger = LoggerFactory.getLogger(DistributedLockAspectConfiguration.class);
  
      @Autowired  
      private DistributedLock distributedLock;
  
      @Pointcut("@annotation(com.itopener.lock.redis.spring.boot.autoconfigure.annotations.RedisLock)")  
      private void lockPoint(){
  
      }
  
      @Around("lockPoint()")  
      public Object around(ProceedingJoinPoint pjp) throws Throwable{  
          Method method = ((MethodSignature) pjp.getSignature()).getMethod();  
          RedisLock redisLock = method.getAnnotation(RedisLock.class);  
          String key = redisLock.value();  
          if(StringUtils.isEmpty(key)){  
              Object\[\] args = pjp.getArgs();  
              key = Arrays.toString(args);  
          }  
          int retryTimes = redisLock.action().equals(LockFailAction.CONTINUE) ? redisLock.retryTimes() : 0;  
           //获取分布式锁 
          boolean lock = distributedLock.lock(key, redisLock.keepMills(), retryTimes, redisLock.sleepMills());  
          if(!lock) {  
              logger.debug("get lock failed : " + key);  
              return null;  
          }
  
         //执行方法之后，释放分布式锁
          logger.debug("get lock success : " + key);  
          try {  
              return pjp.proceed();   //执行方法
          } catch (Exception e) {  
              logger.error("execute locked method occured an exception", e);  
          } finally {  
              boolean releaseResult = distributedLock.releaseLock(key);  //释放分布式锁
              logger.debug("release lock :" + key + (releaseResult ?" success" : "failed"));  
          }  
          return null;  
      }  
  }
  ~~~



#### API 网关设计与实践







