---
title: "JAVA - Redis"
date: 2022-10-25T17:47:32+08:00
categories:
    - Java
tags:
    - 中间件
draft: false
---

## NoSQL与Redis

​		NoSQL的全称是Not Only SQL，它泛指非关系型的数据库，相比于传统的关系型数据库，它并不保证关系数据的ACID特性，也并不遵循SQL标准，消除了数据之间的关联性，但取而代之的是，它拥有：

* 易扩展
* 高性能 *（选择它主要就是因为它够快）
* 灵活的数据模型
* 高可用

​		Redis就是一种非关系型的数据库。在Web应用较小时，MySQL + MyBatis自带的缓存就可以胜任大部分的数据存储工作，但MySQL的数据是存储在硬盘上的，如果遇到快速更新或频繁使用的数据，这些数据需要服务器提供更高的相应速度，还需要面对短时间内大量的访问，这时磁盘的I/O性能就力不从心了，这时就需要使用I/O速度要高于硬盘的内存。Redis正是这样做的。

​		NoSQL分为下面几种

* 键值存储(key-value)数据库：所有的数据都是以键值对的方式存储
* 列存储数据库：通常用来应付分布式存储的海量数据。键仍然存在，但是它们的特点是指向了多个列。
* 文档型数据库：以特定的文档格式存储数据，比如JSON。文档型数据库可以看作是键值数据库的升级版，允许之间嵌套键值，在处理网页等复杂数据时，文档型数据库比传统键值数据库的查询效率更高。
* 图形数据库：利用类似于图的数据结构存储数据，结合图相关算法实现高速访问。

​		Redis就是开源的键值存储数据库，数据都放在内存中，但它同样可以实现数据的持久化，在实际开发中，MySQL和Redis往往会组合使用，并不是说使用Redis代替MySQL。

## 基本操作

​		Redis并不像MySQL那样拥有严格的表结构，它是键值存储数据库，所以它操作起来就像HashMap。

​		在Redis中默认有16个数据库（可以通过修改其配置文件来修改数据库的数量），数据库不以名称做区分，而是以整数索引标识，比如默认的就是0-15号数据库。默认情况下连接Redis会使用0号数据库，可以通过`select`语句进行切换：

```sql
select 数据库序号
```

​		（要与MySQL中select用以查询数据区别开来）



​		基本命令集合：

```sql
-- 使用set或mset往数据库中插入数据
-- 插入单个数据
set <key> <value> EX|PX s|ms
-- 插入多个输入
mset [<key> <value>] ...
-- 在插入数据时可以设定其过期时间，只需要在后面加上EX或PX和对应的时间，其中EX和PX分别代表以秒和毫秒作为单位
-- 当数据到达设定时间会被自动删除，如果想在创建数据后单独为其设立过期时间，可以使用expire
expire <key> <s>
-- 使用ttl和pttl查看数据剩余过期时间，分别对应秒和毫秒作为单位
ttl <key>
pttl <key>
-- 使用persist将有过期时间的数据转为永久
persist<key>

-- 使用get通过key来获取对应的value
get <key>

-- 使用del删除一个或多个数据
del <key> ...

-- 查看数据库中所有数据
keys *

-- 查询某个键是否存在
exists <key>

-- 使用move将一个数据从当前数据库转移到另一个数据库
move <key> 数据库序号

-- 使用rename修改键名
rename <key> <新键名>
```

​		更多Redis相关命令查询官方文档：

​		[Redis官方命令文档](https://redis.io/commands/)

## Redis的数据类型

​		Redis至今有九种数据类型，**这里的数据类型是指键值对中的类型**，分别是：

* String（最基本的，上面默认的就是这种）
* Hash（本质是嵌套哈希表）
* List（列表）
* Set（集合）
* Sorted Set（有序集合）
* BitMap（位图）
* HyperLogLog（一般做基数统计算法）
* Geospatial（地理位置）
* Stream（消息队列，5.0后才推出的，不熟）

​		实际最常用的就是前面五种（后面的我也不熟），它们每种都有不同的指令集合，但都是基于上面的String。

### Hash

​		Redis本身就是键值对，在Redis使用Hash实际就是嵌套一下，比如说一般Redis存String类型在Java中像这样：
```java
Map<String, String> hash = new HashMap<>();
```

​		那么Redis存Hash类型就是：

```java
Map<String, Map<String, String>> hash = new HashMap<>();
```

​		但Hash中只允许存String类型，不允许套娃。



​		Hash的指令操作大多数是在前面加个h：

```sql
-- 插入数据
hset <key> [<key> <value>]...

-- 获取数据
hget <key> <key>
-- 一次性获取所有字段和值
hgetall <key>

-- 删除字段
hdel <key>

-- 查询某个字段是否存在
hexists <key> <key>

-- 查询一个有多少键值对
hlen <key>

-- 获得所有字段值
hvals <key>
```

​		同样，包括下面所有数据类型的更多相关指令查询官方文档。

### List

​		它比较接近Java中的LinkedList，是一个双端列表，可以对列表两端进行操作。

​		可以直接向一个已存在或不存在的List中添加数据，如果List不存在会自动存在：

```sql
-- 向头部添加
lpush <key> <element>...
-- 向尾部添加
rpush <key> <element>...
-- 指定在某个元素前/后添加
linsert <key> before/after <指定元素> <element>

-- 通过下标获取元素
lindex <key> <下标>
-- 获取并移除头部
lpop <key>
-- 获取并移除尾部
rpop <key>
-- 获取指定范围内的元素
lrange <key> start end
```

​		List有一个阻塞的操作，比如说如果列表中没有元素，想等列表中有元素了以后再将其pop出去，那么就可以使用`blpop`：

```sql
-- timeout代表这个指令在多少秒后过期，比如说设定为10，10秒后若列表中还没有元素，那么这个操作作废
blpop <key> ... timeout
```

### Set

​		本质就是一个集合，不能出现重复的元素，不支持随机访问，但查找效率较高。

​		同样，如果往不存在的set添加数据会自动创建：

```sql
-- 往set中添加一个或多个数据
sadd <key> <value>...

-- 移除数据
srem <key> <value>...
-- 如果用pop会随机移除一个元素
spop <key>

-- 移动指定值到另一个集合中
smove <key> 目标集合 value

-- 查看集合中有多少值
scard <key>

-- 数学中集合的运算同样可以通过指令实现，比如最简单的交并差集
-- 交集
sinter <key1> <key2>
-- 并集
sunion <key1> <key2>
-- 差集
sdiff <key1> <key2>
```

### SortedSet

​		SortedSet也叫zset，就是有序的集合。它的基本操作就是改成z开头，添加时可以为每个数据添加一个权值，权值大小决定值的位置：
```sql
-- 添加带权值的数据
zadd <key> [<value> <score>]...

-- 移除
zrem <key> <value>...

-- 查看有多少值
zcard <key>

-- 获取区间内的所有值
zrange <key> start end
-- 获取权值段内的所有值
zrangebyscore <key> start end [withscores] [limit]
-- 根据权值获取指定值排名
zrank <key> <value>
-- 统计权值段内数量
zcount <key> start end
```

## 数据持久化

​		Redis的数据都存放在内存上，但遭遇特殊情况时，比如碰撞或者断电，那内存上的数据就直接丢失了，因此需要将Redis中的数据持久化，就是把数据备份到硬盘上。

​		数据的持久化有两种方案，，一种叫做RDB，它就是直接保存当前已有的所有数据，相当于把内存中的数据复制到硬盘上，恢复数据时再复制回来。一种叫做AOF，它将所有对数据保存的过程都存储起来，恢复数据时将这些操作再执行一遍（有点像之前数据库原理中的数据库恢复技术）。

### RDB

​		RDB的用法十分简单，就是使用`save`命令：

```sql
save
-- 这个是直接保存，也可以开一个子进程进行保存
bgsave
```

​		执行后会在根目录下生成名为`dump.rdb`的文件，它就是内存中存放的数据。

​		这种方式较为简单，但如果数据量比较大，复制一次就会花费大量时间，所以可以更改配置文件，设置每隔一段时间或多少写入自动保存。

​		在配置文件中修改：

```txt
save 300 100 # 300秒内有10次写入就自动保存
save 60 1000 # 60秒内有1000次写入就自动保存
```

​		**在配置文件中的save实际上都是以bgsave的形式保存。**

### AOF

​		RDB虽然可以解决持久化的问题，但缺点也很明显，它并不是实时的，如果在自动保存执行之前遭遇了断电，那仍然会造成少量数据丢失。

​		AOF会以日志的形式将每次执行的命令保存，恢复数据时会重复执行所有命令，这样就能很好解决实时性存储问题。

​		AOF有三种保存策略：

* always: 每条命令执行都会保存一次
* everysec: 每秒保存一次，这样就算丢失数据也只会丢失一秒内的数据
* no: 不做控制，什么时候保存看操作系统

​		Redis的默认配置是everysec。

​		在配置文件中修改：

```txt
appendonly yes	开启AOF
appendfsync everysec/always/no
```

​		日志文件默认在根目录下，名为`appendonly.aof`。

​		AOF每次都会进行过程的重演，所以相比RDB它更加耗费时间，并且随着操作不断增多，`appendonly.aof`文件会越来越大，因此我们需要一个方案来优化这个问题。Redis有一个rewrite重写机制，可以对语句进行优化。

​		比如连续执行了三次往列表a中添加数据的操作：

```sql
lpush a 1
lpush a 2
lpush a 3
```

​		实际上这可以优化为一句指令：

```sql
lpush a 1 2 3
```

​		重写操作可以手动执行：

```sql
bgrewriteaof
```

​		也可以在配置文件中配置自动重写：

```txt
auto-aof-rewrite-percentage 百分比重写，和上次重写时的大小大于多少百分比就自动重写
auto-aof-rewrite-min-size 64mb 达到一定大小自动重写
```

## Redis中的事务与锁

### 事务

​		和MySQL一样，Redis也有事务机制，可以直接用命令：

```sql
-- 开启事务
multi
-- 提交事务
exec
-- 中途取消事务
discard
```

​		Redis中的事务是创建了一个命令队列，会在multi后将命令都装在队列中，不会立即执行，而是等提交后统一执行，因此discard只是会取消这个队列，并不是rollback回滚的概念，比如如果事务中有命令出现问题，MySQL会将已完成的操作全部撤销，回滚到事务开始前的状态，而Redis中的事务，出问题的命令后面的问题仍会执行，这是因为Redis的事务并不保证ACID原则（这里体现的是原子性）。

### 锁

​		Redis中同样有锁机制，但MySQL中使用的是悲观锁，Redis使用的是乐观锁。

* 悲观锁：总是假设最坏的情况，时刻认为别人会抢占资源，所以共享资源时每次只给一个线程使用，其他线程会进入阻塞状态，直到释放锁
* 乐观锁：总是假设最好的情况，并不认为别人会抢占资源，会直接对数据进行操作，在操作时再验证数据是否被更新

#### CAS算法

​		CAS(Compare And Swap)算法是乐观锁的一种实现，它共有三个操作数：当前内存值V，预期内存值A，修改后的新值B。

```java
// 如果当前当前内存值V与预期值A相等，就将V修改为B
if (V == A) {
    V = B;
}
```

​		其实就是判断取值时值是否发生了改变。

#### ABA问题

​		ABA问题是CAS算法衍生出来的问题：

​		假设此时有两条指令：
```sql
原本的数据X = 100
A指令：X -> 200
B指令：X -> 300 X -> 100
```

​		如果B指令先执行，A指令后执行，那么A指令判断X时仍为100，那么就正确执行A指令了，但是这中间实际对X的值已经有过了改动。

​		ABA问题看似对结果没有影响，但中间多出的过程可能会对业务结果产生影响。

​		通常解决ABA问题的做法是添加版本号，就是对数据X添加版本号version，每一次改动都会使其版本号自增，而CAS比较的不再是数据值，而是数据的版本号，这样就避免了ABA问题。

## 在Java中使用Redis

### 使用Jedis

​		在Java中连接Redis数据库只需要使用Jedis框架，导入其依赖：

```xml
<dependencies>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>4.3.1</version>
    </dependency>
</dependencies>
```

​		创建Jedis对象：

```java
public static void main(String[] args) {
    //创建Jedis对象
    Jedis jedis = new Jedis("localhost", 6379);		// Host和端口号
  	
  	//使用之后关闭连接
  	jedis.close();
}
```

​		Jedis已经封装好了所以操作，所以直接调用同名方法就可以了：

```java
public static void main(String[] args) {
    // try-with-resource语法糖
    try(Jedis jedis = new Jedis("localhost", 6379)){
        jedis.set("test", "1234");
        System.out.println(jedis.get("test"));
    }
}
```

### SpringBoot整合Redis

​		SpringBoot导入框架：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

​		官方依赖的底层并没有使用Jedis，而是Lettuce。

​		默认配置会自动去连接本地的Redis服务器，，默认使用0号数据库，当然也可以在配置文件中手动配置：

```yaml
spring:
  redis:
  	#Redis服务器地址
    host: localhost
    #端口
    port: 6379
    #使用几号数据库
    database: 0
```

​		SpringBoot中是使用Redis模板类的，直接注入后使用即可，不过要多封装一层`ValueOperations`：

```java
@SpringBootTest
class SpringBootTestApplicationTests {

    @Autowired
    StringRedisTemplate template;

    @Test
    void contextLoads() {
        ValueOperations<String, String> operations = template.opsForValue();
        operations.set("c", "xxxxx");
        System.out.println(operations.get("c"));
        template.delete("c");
        System.out.println(template.hasKey("c"));
    }

}
```

## 应用：使用Redis做MyBatis的二级缓存

​		MyBatis的二级缓存是Mapper级别的缓存，能够参与所有会话，但它是单机的，如果一个数据库对应多台服务器，二级缓存只会在对应的服务器上生效，Redis可以作为MyBatis的二级缓存实现多台服务器共用同一个二级缓存，因为它们只需要连接同一个Redis服务器即可。

![图片](https://img-blog.csdnimg.cn/img_convert/5afd7713f9a97615dc3a0b1d3bc7db27.png)

​		（二级缓存可以跨对话，但只能在对应服务器上生效）



​				首先需要实现MyBatis的Cache接口：

```java
public class RedisMybatisCache implements Cache {
    private final String id;
    private static RedisTemplate<Object, Object> template;

    //注意构造方法必须带一个String类型的参数接收id
    public RedisMybatisCache(String id){
        this.id = id;
    }

    //初始化时通过配置类将RedisTemplate给过来
    public static void setTemplate(RedisTemplate<Object, Object> template) {
        RedisMybatisCache.template = template;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public void putObject(Object o, Object o1) {
        //这里直接向Redis数据库中丢数据即可，o就是Key，o1就是Value，60秒为过期时间
        template.opsForValue().set(o, o1, 60, TimeUnit.SECONDS);
    }

    @Override
    public Object getObject(Object o) {
        //这里根据Key直接从Redis数据库中获取值即可
        return template.opsForValue().get(o);
    }

    @Override
    public Object removeObject(Object o) {
        //根据Key删除
        return template.delete(o);
    }

    @Override
    public void clear() {
        //由于template中没封装清除操作，只能通过connection来执行
        template.execute((RedisCallback<Void>) connection -> {
            //通过connection对象执行清空操作
            connection.flushDb();
            return null;
        });
    }

    @Override
    public int getSize() {
        //这里也是使用connection对象来获取当前的Key数量
        return template.execute(RedisServerCommands::dbSize).intValue();
    }
}
```

​		编写配置类：

```java
@Configuration
public class MainConfiguration {
    @Resource
    RedisTemplate<Object, Object> template;

    @PostConstruct
    public void init(){
      	//把RedisTemplate给到RedisMybatisCache
        RedisMybatisCache.setTemplate(template);
    }
}
```

​		最后在Mapper上启用缓存：
```java
//只需要修改缓存实现类implementation为创建的RedisMybatisCache即可
@CacheNamespace(implementation = RedisMybatisCache.class)
@Mapper
public interface MainMapper {
	
}
```

