## Redis

### **认识Redis**

[Redis](https://redis.io/)是一个基于 C 语言开发的开源 NoSQL 数据库（BSD 许可）。与传统数据库不同的是，Redis 的数据是保存在内存中的（内存数据库，支持持久化），因此读写速度非常快，被广泛应用于分布式缓存方向。并且，Redis 存储的是 KV 键值对数据。

### **Redis数据结构**

| 基本数据结构                                                 | 高级数据结构                                                 |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| String 字符串类型： name: "kun"                              | bloomfilter（布隆过滤器，主要从大量的数据中快速过滤值，比如邮件黑名单拦截） |
| List 列表：names: ["kun","xiaoheizi", "kun"]                 | geo（计算地理位置）                                          |
| Set 集合：names: ["kun", "xiaoheizi"]（值不能重复）          | hyperloglog（pv / uv）                                       |
| Hash 哈希：nameAge: {  name1:"kun", age1：26 ，name2:"xiaoheizi" ，age2 ：18 } | pub / sub（发布订阅，类似消息队列）                          |
| Zset 集合：names: {  kun- 9,   xiaoheizi- 12  }（适合做排行榜） | BitMap （1001010101010101010101010101）                      |

> [!TIP]
>
> #### **Redis与mysql的区别**

|                |                     MySQL                      |                            Redis                             |
| :------------- | :--------------------------------------------: | :----------------------------------------------------------: |
| 数据库类型     |            SQL数据库(关系型数据库)             |                 NoSQL数据库(非关系型数据库)                  |
| 数据存储模型   |        结构化存储，具有固定行和列的表格        |               键值：(Key-Value)**键值对存储**                |
| ACID 属性      | 提供原子性、一致性、隔离性和持久性 (ACID) 属性 | 支持**部分**的原子性：如果事务中使用的命令和操作的数据类型不匹配的时候不保证原子性、支持一致性、支持隔离性、**无法**保证持久性 |
| 性能(硬件因素) |              性能通常取决于磁盘。              |             CPU、内存大小和速度、网络带宽和延迟              |



### **Redis连接与基本使用**

![image-20241116093048696](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116093048696.png)

![image-20241116101951561](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116101951561.png)

![image-20241116103324409](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116103324409.png)

![image-20241116103405773](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116103405773.png)

![image-20241116104039270](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116104039270.png)



### **SpringBoot操作Redis**

> 在pom.xml引入 spring-boot-starter-data-redis，能够操作 redis
>
> ```xml
>   <!-- redis -->
>         <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
>         <dependency>
>             <groupId>org.springframework.boot</groupId>
>             <artifactId>spring-boot-starter-data-redis</artifactId>
>             <version>2.6.4</version>
>         </dependency>
> ```
>
> 在application.yaml配置Redis
>
> ```yaml
>   # Redis Config
>   redis:
>     host: #域名
>     port: 6379
>     password: #密码
>     database: #数据库表(默认是0)
>     lettuce:
>       pool:
>         max-active: 8       #max-active: 最大活跃连接数为8
>         max-idle: 8         #max-idle: 最大空闲连接数为8
>         min-idle: 0         #min-idle: 最小空闲连接数为0
>         max-wait: 100ms     #max-wait: 获取连接的最大等待时间为100毫秒
> ```



>实现思路：
>
>- 用spring-boot-starter-data-redis整合好的**RedisTemplate<String, Object>**方法
>
>- 设置Redis的键redisKey，这个redisKey作为写数据和读数据的标识。
>- 如果有Redis缓存，直接通过redisKey来获取数据，否则通过redisKey写入数据。
>- 写入数据要设置过期时间，保障服务稳定运行。

```java
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @GetMapping("/recommend")
    public BaseResponse<Page<User>> RecommendUsers(long PageCurrent ,long PageSize, HttpServletRequest request) {
        User loginUser = userService.getLoginUser(request);
        String redisKey =String.format("cusie:%s",loginUser.getId());
        ValueOperations<String, Object> valueOperations =redisTemplate.opsForValue();
        //有Redis缓存直接读取
        Page<User> RedisUsersPage = (Page<User>) valueOperations.get(redisKey);
        if (RedisUsersPage!=null)
        {
            return ResultUtils.success(RedisUsersPage);
        }
        //无缓存,查MySQL数据库
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        RedisUsersPage = userService.page(new Page<>(PageCurrent,PageSize),queryWrapper);
        //读完后写缓存
        try{
            valueOperations.set(redisKey , RedisUsersPage ,1000, TimeUnit.MINUTES);
        }catch (Exception e)
        {
            log.error("Redis键有问题",e);
        }
           return ResultUtils.success(RedisUsersPage);
    }
```



![image-20241116111139875](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116111139875.png)

![image-20241116111244929](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116111244929.png)

![image-20241116110955172](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116110955172.png)

#### 对比结果：在我的这个项目中Redis的速度比Mysql快5倍+



### [Redis 为什么这么快？](#redis-为什么这么快)

1. Redis 基于内存，内存的访问速度比磁盘快很多；
2. Redis 基于 Reactor 模式设计开发了一套高效的事件处理模型，主要是单线程事件循环和 IO 多路复用；
3. Redis 内置了多种优化过后的数据类型/结构实现，性能非常高。
4. Redis 通信协议实现简单且解析高效。

> 一张图片总结，出自 [Why is Redis so fast?](https://twitter.com/alexxubyte/status/1498703822528544770)

![why-redis-so-fast](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/why-redis-so-fast-TbWX24ja.png)

> 那既然都这么快了，为什么不直接用 Redis 当主数据库呢？
>
> 主要是因为**内存成本**太高且 Redis 提供的数据持久化仍然有数据丢失的风险,就像我开头对比的，Redis**无法**保证持久性。



## Redisson

### 引子

#### Java对于Redis的实现方式

- Spring-Data-Redis
- Spring Data：

- - 通用的数据访问框架，定义了一组 增删改查 的接口mysql、redis、jpa

- Jedis：	

- - (独立于 Spring 操作 Redis 的 Java 客户端，要配合 Jedis Pool 使用)

- Lettuce

- - **高阶** 的操作 Redis 的 Java 客户端
  - 异步、连接池

- Redisson

- - 分布式操作 Redis 的 Java 客户端，让你像在使用本地的集合一样操作 Redis（分布式 Redis 数据网格）

### 对比

1. 如果你用的是 Spring，并且没有过多的定制化要求，可以用 Spring Data Redis，最方便
2. 如果你用的不是 SPring，并且追求简单，并且没有过高的性能要求，可以用 Jedis + Jedis Pool
3. 如果你的项目不是 Spring，并且追求高性能、高定制化，可以用 Lettuce，支持异步、连接池



- 如果你的项目是分布式的，需要用到一些分布式的特性（比如分布式锁、分布式集合），推荐用 redisson

## **Redisson 基础使用**

> Tips：SpringBoot项目整合Redisson，可以使用starter方式，也可以单独引入依赖,这里我使用单独引入。

> 在pom.xml引入 redisson
>
> ```xml
>         <dependency>
>             <groupId>org.redisson</groupId>
>             <artifactId>redisson</artifactId>
>             <version>3.17.5</version>
>         </dependency>
> ```
>
> 在application.yaml配置redisson
>
> ```yaml
>   # session 失效时间(分钟)
>   session:
>     timeout: 86400
>     store-type: redis
> ```
>
> 

#### 定义Redisson的配置类和基本使用

```java
@Configuration
@ConfigurationProperties(prefix = "spring.redis")
@Data
public class RedissonConfig {
    private String host;

    private String port;

    @Bean
    public RedissonClient redissonClient() {

        // 1.创建配置
        Config config = new Config();
        String redisAddress = String.format("redis://%s:%s", host, port);
        config.useSingleServer()
                .setAddress(redisAddress)
                .setDatabase(0);
        // 2.根据Config创建出RedissonClient实例
        return Redisson.create(config);
    }
}
```

> Redisson的使用方式和原生Java的集合用法不能说毫不相关，只能说是一模一样。

```java
@SpringBootTest
public class RedissonTest {

    @Resource
    private RedissonClient redissonClient;

    @Test
    void test() {
        // list，数据存在本地 JVM 内存中
        List<String> list = new ArrayList<>();
        list.add("kunkun");
        System.out.println("list:" + list.get(0));//输出   list:kunkun

        list.remove(0);

        // 数据存在 redis 的内存中
        RList<String> rList = redissonClient.getList("test-list");
        rList.add("kunkun");
        System.out.println("rlist:" + rList.get(0));//输出  rlist:kunkun
        //rList.remove(0);

        // map
        Map<String, Integer> map = new HashMap<>();
        map.put("kunkun", 10);
        System.out.println("map:" + map.get("kunkun"));//输出  map:10

        RMap<Object, Object> rmap = redissonClient.getMap("test-map");
        rmap.put("kunkun", 10);
        System.out.println("rmap:" + rmap.get("kunkun"));//输出 rmap:10

    }
}
```
**Redis数据：**

![image-20241116121849604](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116121849604.png)

![image-20241116121930444](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116121930444.png)



### **Redisson业务实战：（定时任务）缓存预热**

#### 实现需求：

**用定时任务，每天刷新所有用户的推荐列表**

> 注意点:
>
> - 缓存预热的意义(新增少、总用户多)
> - 缓存的空间不能太大，要预留给其他缓存空间
> - 缓存数据的周期(此处每天一次)

#### 分析定时任务

**使用SpringBoot默认整合的Spring Scheduler**

- 主类开启[@EnableScheduling](@EnableScheduling)
- 给要定时执行的方法添加[@Scheduling](@Scheduling)注解，指定[cron表达式](https://www.matools.com/crontab/)或者[执行频率](https://cron.qqe2.com/)

**要控制定时任务的执行**

要控制定时任务在同一时间只有1个服务器能执行。

> - 浪费资源，如果10000台服务器同时'打鸣(坤叫)'
> - 脏数据，比如重复插入

**解决方案**

> 1.分离定时任务程序和主程序，只在1个服务器运行定时任务。成本太大
> 2.写死配置，每个服务器都执行定时任务,但是只有ip符合配置的服务器才真实执行业务逻辑，其他的直接返回。成
> 本最低;但是我们的IP可能是不固定的,把IP写的太死了
> 3.动态配置，配置是可以轻松的、很方便地更新的(代码无需重启)，但是只有ip符合配置的服务器才真实执行业务
> 逻辑。
>
> ​		(1).数据库
>
> ​		(2).Redis
>
> ​		(3)配置中心(Nacos、 Apollo、 Spring Cloud Config)
>
> 问题:服务器多了、IP 可控还是很麻烦，还是要人工修改
>
> 4.分布式锁，只有抢到锁的服务器才能执行业务逻辑。坏处:增加成本;好处:不用手动配置，多少个服务器都一样。
>
> - 注意,只要是单机，就会存在单点故障。

#### 锁

> 有限资源的情况下，控制同一时间(段)只有某些线程(户/服务器)能访问到资源。
>
> Java实现锁: synchronized 关键字、并发包的类。
>
> 但存在一个问题:Java锁只对**单个**JVM有效(每一台虚拟机创建的对象都由这台虚拟机自己管理)。

#### 分布式锁

> **为啥需要分布式锁?**
> 1.有限资源的情况下，控制同- -时间(段)只有某些线程(用户/服务器)能访问到资源。
> 2.单个锁只对单个JVM有效
>
> **分布式锁实现的关键**
> ==抢锁机制:==
> 怎么保证同一时间只有1个服务器能抢到锁?
>
> 核心思想就是:先来的人先把数据改成自己的标识(服务器ip)，后来的人发现标识已存在，就抢锁失败,继续等待。
> 等先来的人执行方法结束，把标识清空,其他的人继续抢锁。
>
> MySQL数据库: select for update行级锁(最简单)，或者用乐观锁。
>
> Redis实现:内存数据库，读写速度快。支持setnx、lua 脚本，比较方便我们实现分布式锁。
> setnx: set if not exists如果不存在，则设置;只有设置成功才会返回true,否则返回false。
>
> 注意事项
> (1)用完锁要释放==(==如过任务提前完成了，可以马上释放，避免造成站着茅坑不拉屎的资源浪费情况==)==
> (2)锁-定要加过期时间==(==防止死锁和资源浪费==)==
> (3)如果方法执行时间过长，锁提前过期了?
>
> ​	出现的问题:
>
> ​	1.连锁效应:释放掉别人的锁
>
> ​	2.这样还是会存在多个方法同时执行的情况
>
> ​	解决方案: ==锁续期==	
>
> ```java
> boolean end = false;
> 
> new Thread(() -> {
>     if (!end)}{
>     续期
> })
> 
> end = true;
> 
> ```
>
> **Redisson看门狗机制**:
>
> > redisson中提供的续期机制
> > 开一个监听线程，如果方法还没执行完，就帮你重置redis锁的过期时间。
> > 原理:
> > 1.监听当前线程，默认过期时间是30秒,每10秒续期一次(补到30秒)。
> > 2.如果线程挂掉(注意debug模式也会被它当成服务器宕机)，则不会续期。
> >
> > [Redisson 分布式锁的watch dog自动续期机制](https://blog.csdn.net/qq_26222859/article/details/79645203)
>
> (4)释放锁的时候，有可能先判断出是自己的锁，但这时锁过期了，后还是释放了别人的锁
> 解决方案: Redis + lua脚本保证操作原子性
>
> ```java
> // 原子操作
> if(get lock == A) {
>     // set lock B
>     del lock
> }
> 
> ```
>
> (5)Redis如果是集群(而不是只有一个Redis)，如果分布式锁的数据不同步怎么办?
>
> [解决方案: Redis红锁](https://blog.csdn.net/feiying0canglang/article/details/113258494)



#### ==实现定时任务==

```java
@Component
@Slf4j
public class PreCacheJob {

    @Resource
    private UserService userService;
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    @Resource
    private RedissonClient redissonClient ;
    // 重点用户
    private List<Long> mainUserList = Arrays.asList(1L);

    // 每天执行，预热推荐用户
    @Scheduled(cron = "0 47 16 16 11 *")   //自己设置时间测试
    public void doCacheRecommendUser() {
        RLock lock = redissonClient.getLock("preCacheJobLock" + Thread.currentThread().getId());
        try {
            if (lock.tryLock(0, 30000L, TimeUnit.MILLISECONDS)){
                System.out.println("getLock");
                for(Long userId : mainUserList)
                {
                    //查数据库
                    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
                    Page<User> userPage = userService.page(new Page<>(1,20),queryWrapper);
                    String redisKey = String.format("cuise:%s",userId);
                    ValueOperations valueOperations = redisTemplate.opsForValue();
                    //写缓存,300s过期
                    try {
                        valueOperations.set(redisKey,userPage,300000, TimeUnit.MILLISECONDS);
                    } catch (Exception e){
                        log.error("redis set key error",e);
                    }
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }finally {
            // 只能释放自己的锁
            if (lock.isHeldByCurrentThread())
            {
                lock.unlock();
                System.out.println("unLock" + Thread.currentThread().getId());
            }
        }
    }
}
```

>#### **我在本地开了三个端口启动服务模拟三台服务器,可以看到第二个端口抢到了锁并执行了定时任务,其他端口不做任何处理**
>
>![image-20241116164829680](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116164829680.png)![image-20241116164802041](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116164802041.png)
>
>![image-20241116165325189](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116165325189.png)



## Redis的其他用处(八股篇)

### [为什么要用 Redis？](#为什么要用-redis)

**1、访问速度更快**

传统数据库数据保存在磁盘，而 Redis 基于内存，内存的访问速度比磁盘快很多。引入 Redis 之后，我们可以把一些高频访问的数据放到 Redis 中，这样下次就可以直接从内存中读取，速度可以提升几十倍甚至上百倍。

**2、高并发**

一般像 MySQL 这类的数据库的 QPS 大概都在 4k 左右（4H8G 服务器） ，但是使用 Redis 缓存之后很容易达到 5w+，甚至能达到 10w+（就单机 Redis 的情况，Redis 集群的话会更高）。

> QPS（Query Per Second）：服务器每秒可以执行的查询次数；

由此可见，直接操作缓存能够承受的数据库请求数量是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而，我们也就提高了系统整体的并发。

**3、功能全面**

Redis 除了可以用作缓存之外，还可以用于分布式锁、限流、消息队列、延时队列等场景，功能强大！



### [常见的缓存读写策略有哪些？](https://javaguide.cn/database/redis/redis-questions-01.html#常见的缓存读写策略有哪些)

**旁路缓存模式**、**读写穿透**、**异步缓存写入**，三种模式各有优劣，不存在最佳模式，根据具体的业务场景选择适合自己的缓存读写模式。



#### Cache Aside Pattern（旁路缓存模式）

**Cache Aside Pattern 是我们平时使用比较多的一个缓存读写模式，比较适合读请求比较多的场景。**

Cache Aside Pattern 中服务端需要同时维系 db 和 cache，并且是以 db 的结果为准。

**缓存写步骤**：

1.先更新 db

2.然后直接删除 cache 。

![image-20241116172143942](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116172143942.png)



**缓存读步骤**：

1.从Redis缓存中读取数据，读取到就直接返回

2.Redis中读取不到的话，就从数据库中读取数据返回

3.再把数据放到Redis中。

![image-20241116190603600](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116190603600.png)



#### 写数据的过程中，可以先删除Redis缓存 ，后更新数据库么？

  不行！因为这样可能会造成**数据库和Redis缓存数据不一致**的问题。

> 举例：管理员先写数据 A，用户随后读数据 A 的话，就很有可能产生数据不一致性的问题。
>
> > 过程：
> >
> > 管理员先把Redis中的 A 数据删除->用户从 数据库 中读取数据->管理员再把 数据库 中的 A 数据更新。
> >
> > **试着想一下管理员更新100w条数据，这个时候用户进来访问了这100w条数据**，100w条数据写进数据库还是需要一些时间的。



#### 那在写数据的过程中，先更新 数据库，后删除 Redis缓存 就没有问题了么？

理论上来说还是可能会出现数据不一致性的问题，不过概率非常小，因为**Redis缓存的写入速度是比数据库的写入速度快很多**。

> 举例：用户先读数据 A，管理员随后写数据 A，并且数据 A 在请求 1 请求之前不在缓存中的话，也有可能产生数据不一致性的问题。
>
> 这个过程可以简单描述为：
>
> > 用户从 数据库 读数据 A-> 管理员 更新 数据库 中的数据 A（此时缓存中无数据 A ，故不用执行删除缓存操作 ） -> 用户请求的 数据 A 自动写入 Redis
> >
> > **用户首次加载页面的过程中,管理员写入数据A。**



#### Cache Aside Pattern 的缺陷有哪些,有时间办法可以解决？

**缺陷 1：首次请求数据一定不在 Redis缓存 的问题**

解决办法：可以将热点数据可以提前放入 Redis缓存 中。



**缺陷 2：写操作比较频繁的话导致 Redis 中的数据会被频繁被删除，这样会影响缓存命中率 。**

解决办法：

- 数据库和缓存数据强一致场景：更新 db 的时候同样更新 cache，不过我们需要加一个锁/分布式锁来保证更新 cache 的时候不存在线程安全问题。
- 可以短暂地允许数据库和缓存数据不一致的场景：更新 db 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小。



#### Read/Write Through Pattern（读写穿透）

读写穿透中服务端把Redis视为主要数据存储，从中读取数据并将数据写入其中。Redis服务负责将此数据读取和写入 数据库，从而减轻了应用程序的职责。

这种缓存读写策略在开发过程中非常少见。抛去性能方面的影响，大概率是因为我们经常使用的分布式缓存 Redis 并没有提供 cache 将数据写入 db 的功能。

**写（Write Through）：**

1.先查 Redis缓存，Redis缓存 中不存在，直接更新 数据库。

2.Redis缓存 中存在，则先更新 Redis缓存，然后 Redis缓存 服务自己更新 数据库（**同步更新 cache 和 db**）。

![image-20241116202359770](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116202359770.png)



**读(Read Through)：**

1.从 Redis缓存 中读取数据，读取到就直接返回 。

2.读取不到的话，先从 数据库 加载，写入到 Redis 后返回响应。

![image-20241116202733611](https://jsd.onmicrosoft.cn/gh/cusie/resources/NoteImages/image-20241116202733611.png)



Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。在 Cache-Aside Pattern 下，发生读请求的时候，如果 cache 中不存在对应的数据，是由客户端自己负责把数据写入 cache，而 Read Through Pattern 则是 cache 服务自己来写入缓存的，这对客户端是透明的。

和 Cache Aside Pattern 一样， Read-Through Pattern 也有首次请求数据一定不再 cache 的问题，对于热点数据可以提前放入缓存中。



### Write Behind Pattern（异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 cache 服务来负责 cache 和 db 的读写。

但是，两个又有很大的不同：**Read/Write Through 是同步更新 cache 和 db，而 Write Behind 则是只更新缓存，不直接更新 db，而是改为异步批量的方式来更新 db。**

很明显，这种方式对数据一致性带来了更大的挑战，比如 cache 数据可能还没异步更新 db 的话，cache 服务可能就就挂掉了。

这种策略在我们平时开发过程中也非常非常少见，但是不代表它的应用场景少，比如消息队列中消息的异步写入磁盘、MySQL 的 Innodb Buffer Pool 机制都用到了这种策略。

Write Behind Pattern 下 db 的写性能非常高，非常适合一些数据经常变化又对数据一致性要求没那么高的场景，比如浏览量、点赞量。



### Redis 除了做缓存，还能做什么？

- **分布式锁**：通过 Redis 来做分布式锁是一种比较常见的方式。通常情况下，我们都是基于 Redisson 来实现分布式锁。
- **限流**：一般是通过 Redis + Lua 脚本的方式来实现限流。如果不想自己写 Lua 脚本的话，也可以直接利用 Redisson 中的 `RRateLimiter` 来实现分布式限流，其底层实现就是基于 Lua 代码+令牌桶算法。
- **消息队列**：Redis 自带的 List 数据结构可以作为一个简单的队列使用。Redis 5.0 中增加的 Stream 类型的数据结构更加适合用来做消息队列。它比较类似于 Kafka，有主题和消费组的概念，支持消息持久化以及 ACK 机制。
- **延时队列**：Redisson 内置了延时队列（基于 Sorted Set 实现的）。
- **分布式 Session** ：利用 String 或者 Hash 数据类型保存 Session 数据，所有的服务器都可以访问。
- **复杂业务场景**：通过 Redis 以及 Redis 扩展（比如 Redisson）提供的数据结构，我们可以很方便地完成很多复杂的业务场景比如通过 Bitmap 统计活跃用户、通过 Sorted Set 维护排行榜。

