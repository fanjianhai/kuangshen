# 1. 链接

- [Redis官网](https://redis.io/)
- [Redis中文网址](http://www.redis.cn/)
- [Github链接](https://github.com/redis/redis/)



# 2. Linux下安装

- 下载安装包
- 解压redis安装包，程序/opt
- 基本的环境安装

```bash
yum -y install gcc-c++
# 查看gcc版本
gcc -v

make
make install
```

- redis 默认安装路径 `/usr/local/bin`
- 将源码路径下redis配置文件，复制到`/usr/local/bin/myconfig/`

- 修改`redis.conf`配置文件
- 客户端进行验证
- 检测redis默认端口是否在使用中，如在使用则安装程。 netstat -ntlp grep 6379
- 查看redis进程是否开启`ps -ef |grep redis`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200907103256811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 如何关闭redis服务 `shutdown`  不建议使用，防止把服务器给关掉
- [阿里云安装Redis教程与相关问题](https://www.cnblogs.com/blackBlog/p/12924891.html

# 3. 性能测试 - redis-benchmark

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200907105245658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

测试：100个并发链接，100000请求！

```bash
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200907105910353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

# 4. Redis基础知识

- redis默认有16个数据库
- 选择数据库`select index`
- 查看当前数据库的大小`dbsize`
- 清空当前数据库`flushdb` \ 清空所有数据库`flushall`
- 查看指定key是否存在`EXISTS key`
- 移出指定的key`move key 1`
- 让指定数据在指定的时间失效`expire key 10`,单位是秒
- 查看当前key的剩余时间`ttl key`
- 查看指定key的类型`type key`

- [Redis更多命令](http://www.redis.cn/commands.html)

# 5. 五大数据类型

## 5.1. String类型

- 追加字符串`APPEND key value`
- 获取字符串的长度`STRLEN`
- 自增/自减运算 `INCR/DECR`
- 自增/自减指定的值 `INCRBY/DECRBY`

- 获取某个范围`GETRANGE`的字符串
- 替换某个范围的字符串`SETRANGE`
- 设置过期时间`SETEX`

- 不存在的时候才设置`SETNX`
- 批量设置/获取值`MSET,MSETNX/MGET`

- 获取当前的值的同时设置一个新值`GETSET`



## 5.2. List类型

- List当中插入一个元素`LPUSH/RPUSH`
- List当中获取指定元素`LRANGE`
- 移出List集合中的一个元素`LPOP/RPOP`
- 移出List集合中指定的元素`LREM`
- 根据索引（下标）获取集合中元素值`LINDEX`

- 获取List的长度`LLEN`
- 裁剪List为原来的一部分（直接修改原有List集合）

- 移出列表的最后一个元素，并移动到新的列表中`RPOPLPUSH`

- 将列表中指定的下标值替换为另一个值，更新操作，如果下标不存在就会报错
- 在list集合中的指定元素前面插入值`linsert lst before world xiaofan`

## 5.3.Set集合

- 集合中添加元素`sadd myset hello kuangshen lovekuangshen`
- 查看集合中的元素`SMEMBERS`
- 判断某个元素是否存在于集合中`SISMEMBER myset hello`

- 获取集合中元素个数`SCARD`
- 移出集合中指定的元素值`SREM`
- 随机获取Set集合中的值`SRANDMEMBER`
- 随机删除集合中的一个元素`SPOP`
- 移动指定的元素到另一个集合当中去`SMOVE`
- 差集/交集/并集`SDIFF/SINTER/SUNION`

## 5.4. Hash集合

- 设置一个具体的字段值`hset myhash field2 value2`
- 设置多个个具体的字段值`hmset myhash field2 value2 field3 value3`

- 获取多个指定的字段值`hmget myhash field1 field2`
- 获取全部的值`hgetall myhash`
- 删除hash指定的键`HDEL myhash field1`
- 获取键值对的个数`HLEN`判断hash中指定字段是否存在`HEXISTS`
- 获取所有的key、value`HKEYS/HVALS`

## 5.5. Zset有序集合

- 添加元素`zadd myset 1 one 2 two 3 three`
- 获取所有元素`zrange myset 0 -1`
- 排序`ZREVRANGEBYSCORE key max min WITHSCORES LIMIT offset count`
- 升序/降序`zrange myzset 0 -1/ZREVRANGE myzset 0 -1`

- 获取指定区间的元素个数` zcount myzset 1 4`

# 6. 三种特殊的饿数据类型

## 6.1. geospatial 地理位置

- [获取测试数据](http://www.jsons.cn/lngcode/)

- geoadd 添加地理位置
  - [官方文档](https://www.php.cn/manual/view/36388.html)

```shell
127.0.0.1:6379> geoadd china:city 116.405285 39.904989 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.472644 31.231706 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.504962 29.533155 chongqing
(integer) 1
127.0.0.1:6379> geoadd china:city 120.153576 30.287459 hangzhou
(integer) 1
127.0.0.1:6379> geoadd china:city 125.14904 42.927 xian
(integer) 1
127.0.0.1:6379> geoadd china:city 112.549248 37.857014 taiyuan
(integer) 1

```

- geopos 获取指定城市的经纬度

```shell
127.0.0.1:6379> geopos china:city taiyuan
1) 1) "112.54924803972244263"
   2) "37.85701483724372451"
127.0.0.1:6379> geopos china:city beijing
1) 1) "116.40528291463851929"
   2) "39.9049884229125027"
```

- geodist 获取两个位置之间的距离

```shell
m 为米。
km 为千米。
mi 为英里。
ft 为英尺。

127.0.0.1:6379> geodist china:city beijing taiyuan km
"404.1120"
127.0.0.1:6379> geodist china:city beijing shanghai
"1067597.9668"
127.0.0.1:6379> geodist china:city beijing shanghai km
"1067.5980"

```

- GEORADIUS具体指定位置多少范围内的所有城市

```shell
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km
1) "chongqing"
2) "hangzhou"
3) "taiyuan"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km
1) "chongqing"

127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km withcoord
1) 1) "chongqing"
   2) 1) "106.50495976209640503"
      2) "29.53315530684997015"
2) 1) "hangzhou"
   2) 1) "120.15357345342636108"
      2) "30.28745790721532671"
3) 1) "taiyuan"
   2) 1) "112.54924803972244263"
      2) "37.85701483724372451"
      
# 获取指定的个数
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km withcoord count 2
1) 1) "chongqing"
   2) 1) "106.50495976209640503"
      2) "29.53315530684997015"
2) 1) "taiyuan"
   2) 1) "112.54924803972244263"
      2) "37.85701483724372451"

```

- GEORADIUSBYMEMBER 找出位于指定元素周围的其他元素

```shell
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1000 km 
1) "taiyuan"
2) "beijing"
3) "xian"
```

- geohash 命令，返回一个或者多个位置元素的geohash表示

该命令将返回11个字符的Geohash字符串，如果两个字符串越接近，表示距离越近！

```shell
127.0.0.1:6379> geohash china:city beijing chongqing
1) "wx4g0b7xrt0"
2) "wm78p86e170"
```

>GEO 底层的实现原理其实就是Zset！我们可以使用Zset命令来操作geo！

```shell
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "chongqing"
2) "hangzhou"
3) "shanghai"
4) "taiyuan"
5) "beijing"
6) "xian"
127.0.0.1:6379> ZREM china:city xian
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "chongqing"
2) "hangzhou"
3) "shanghai"
4) "taiyuan"
5) "beijing"
```

## 6.2. Hyperloglog 基数统计

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090814225199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```bash
127.0.0.1:6379> PFADD mykey a b c d e f g h i j		# 创建第一组元素 mykey
(integer) 1
127.0.0.1:6379> PFCOUNT mykey 	# 统计第一组元素的个数
(integer) 10
127.0.0.1:6379> PFADD mykey2 i j z x c v b n m		# 创建第二组元素 mykey2
(integer) 1
127.0.0.1:6379> PFCOUNT mykey2	# 统计第二组元素的个数
(integer) 9
127.0.0.1:6379> PFMERGE mykey3 mykey mykey2		# 合并第一组元素和第二组元素
OK
127.0.0.1:6379> PFCOUNT mykey3
(integer) 15

```

## 6.7. Bitmaps

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090814393551.png#pic_center)

```bash
127.0.0.1:6379> setbit sign 0 0		# 设置数据
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 1
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0

127.0.0.1:6379> getbit sign 6	# 获取指定数据
(integer) 1
127.0.0.1:6379> getbit sign 5
(integer) 0

127.0.0.1:6379> bitcount sign	# 统计1的个数
(integer) 4

```

# 7. 事务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200908150411369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```bash
127.0.0.1:6379> multi			# 开启事务
OK
# 入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec			# 执行事务
1) OK
2) OK
3) "v2"
4) OK

```

> 取消事务

```bash
127.0.0.1:6379> multi			# 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> get k3
QUEUED
127.0.0.1:6379> DISCARD			# 取消事务
OK
127.0.0.1:6379> keys *			# 事务队列中命令都不会被执行
(empty array)

```

> 编译性异常（代码有问题！ 命令有错！），事务中所有的命令都不会被执行！

```bash
127.0.0.1:6379> multi			# 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> setget k3		# 命令报错
(error) ERR unknown command `setget`, with args beginning with: `k3`, 
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> set k5 v5
QUEUED
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5			# 队列中的其他命令都没有被执行
(nil)

```

> 运行时异常（I/O），如果事务队列中存在语法行，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常！

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> INCR k1		# 会执行的时候失败
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> get k3
QUEUED
127.0.0.1:6379> EXEC
1) (error) ERR value is not an integer or out of range	# 虽然其中一条命令执行失败了，但是不影响其他命令执行
2) OK
3) OK
4) "v3"
127.0.0.1:6379> get k3
"v3"
127.0.0.1:6379> get k2
"v2"

```

# 8. Redis实现乐观锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200908154725687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```bash
# 正常执行
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money			# 监视money对象
OK
127.0.0.1:6379> multi				# 事务正常结束，数据期间没有发生变化，这个时候就正常执行成功！
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20

```

测试多线程修改值，使用watch可以当做redis的乐观锁操作！

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY out 20
QUEUED
127.0.0.1:6379> exec	# 执行之前，另一个线程修改了我们的值， 这个时候，就会导致事务执行失败！
(nil)

```

```bash
127.0.0.1:6379> set salary 100
OK
127.0.0.1:6379> watch salary
OK
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> DECRBY salary 10
QUEUED
127.0.0.1:6379> INCRBY out 10
QUEUED
127.0.0.1:6379> exec			# 执行事务的时候报错
(nil)
127.0.0.1:6379> unwatch			# 取消监听
OK
127.0.0.1:6379> watch salary	# 重新监听
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DECRBY salary 10
QUEUED
127.0.0.1:6379> INCRBY out 10
QUEUED
127.0.0.1:6379> exec
1) (integer) 990
2) (integer) 10

```

# 9. Jedis

- 导入依赖

```bash
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
```

- 测试

```bash
Jedis jedis = new Jedis("47.94.235.187", 6379);
System.out.println(jedis.ping());
```

# 10. springboot

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200908180215690.png#pic_center)

源码分析：

```java
@Bean
@ConditionalOnMissingBean(name = {"redisTemplate"})
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<Object, Object> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```



> 测试一下

1. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



2. 配置连接

```properties
spring.redis.host=47.94.235.187
spring.redis.port=6379
```

3. 测试

```java
package com.xiaofan;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class Redis02SpringbootApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        redisTemplate.opsForValue().set("mykey", "范建海");
        System.out.println(redisTemplate.opsForValue().get("mykey"));
    }

}
```

4. 自定义RedisTemplate

```java
package com.xiaofan.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);

        // 序列化配置
        Jackson2JsonRedisSerializer<Object> objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping();
        objectJackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        template.setKeySerializer(stringRedisSerializer);
        template.setHashKeySerializer(stringRedisSerializer);
        template.setValueSerializer(objectJackson2JsonRedisSerializer);
        template.setHashValueSerializer(objectJackson2JsonRedisSerializer);

        template.afterPropertiesSet();

        return template;
    }
}

```

# 11. redis.conf详解

```conf
基本配置

内存单位的表示

# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 10241024 bytes
# 1g => 1000000000 bytes
# 1gb => 10241024*1024 bytes

单位中不区分大小写1GB 1Gb 1gB是一样的

后台运行，yes是后台运行，no前台运行，将输出，输出到终端（默认）

daemonize yes

如果daemonize参数为yes的话就会产生pid文件，一下是pid文件的定义

pidfile /usr/local/redis-master/run/redis.pid

监听的端口

port 6379

绑定监听的IP地址

bind 127.0.0.1

如果在本地调用redis可以直接用sock文件

unixsocket /tmp/redis.sock //sock文件的位置

unixsocketperm 755      //sock文件的权限

如果一个链接在N秒内是空闲的，就将其关闭

timeout 0

如果对方down了或者中间网络断了发送ACK到客户端在指定的时间内没有收到对方的回应就断开TCP链接（时间单位秒记），此参数会受到内核参数的影响，推荐配置60。

tcp-keepalive 0

指定输出消息的级别

# debug (调试级别，详细信息，信息量大)
# verbose (详细信息，信息量较大)
# notice (通知，生产环境推荐)
# warning (错误信息警告信息)

loglevel notice

日志输出文件，默认在前端运行的时候此key的默认值是stdout输出到终端，如果用守护进程运行此key的stdout的时候将日志输入到/dev/null，如果想记录日志，就必须为其指定logfile位置

logfile /var/log/redis.log

将日志记录的哦syslog

syslog-enabled no

指定syslog的身份

syslog-ident redis

指定syslog的级别，必须是LOCAL0-LOCAL7之间

syslog-facility local0

设置数据库的数量

databases 16

设置数据库的数量。默认数据库DB 0，你可以选择一个不同的per-connection的使用SELECT<dbid>这儿的DBID是一个介于0和'databases'-1

databases 16

2.快照配置

将DB保存到磁盘的规则定义（快照）

格式：save <seconds> <changes>

例子：save 900 1 //在900秒（15分钟）内如果至少有1个键值发生变化 就保存

      save 300 10 //在300秒（6分钟）内如果至少有10个键值发生变化 就保存 
save 900 1           //每一条表示一个存盘点
save 300 10
save 60 10000

如果启用如上的快照（RDB），在一个存盘点之后，可能磁盘会坏掉或者权限问题，redis将依然能正常工作

stop-writes-on-bgsave-error yes

是否将字符串用LZF压缩到.rdb 数据库中，如果想节省CPU资源可以将其设置成no，但是字符串存储在磁盘上占用空间会很大，默认是yes

rdbcompression yes

rdb文件的校验，如果校验将避免文件格式坏掉，如果不校验将在每次操作文件时要付出校验过程的资源新能，将此参数设置为no，将跳过校验

rdbchecksum yes

转储数据的文件名

dbfilename dump.rdb

redis的工作目录，它会将转储文件存储到这个目录下，并生成一个附加文件

dir /usr/local/redis-master/db

3.主从参数
如果本地是salve服务器那么配置该项

# slaveof <masterip> <masterport>

slaveof 127.0.0.1 65532

master的验证密码

masterauth <master-password>

当从主机脱离主的链接时，如果此值为yes当客户端查询从时，回响应客户端，如果是第一次同步回返回一个日期数据或这空值，如果设置为no，则返回“SYNC with master in progress”到INFO and SLAVEOF

slave-serve-stale-data yes

从服务器只读（默认）

slave-read-only yes

从发送ping到主的时间间隔(单位：秒)

repl-ping-slave-period 10

批量传输I / O超时和主数据或ping响应超时 默认60s 必须大于repl-ping-slave-period值

repl-timeout 60

此选项如果是“yes”那么Redis的使用数量较少的TCP数据包和更少的带宽将数据发送到，在从主机上延迟40毫秒（linux kernel中的40毫秒）出现。如果是no将在slave中减少延迟，但是流量使用回相对多一些，如果用多个从主机，此处建议设置成yes

repl-disable-tcp-nodelay no

从主机的优先级，如果当主主机挂了的时候，将从从主机中选取一个作为其他从机的主，首先优先级的数字最低的将成为主，0是一个特殊的级别，0将永远不会成为主。默认值是100.

slave-priority 100

```

# 12. 持久化操作

## 12.1. RDB(Redis DataBase)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909094246510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909100730533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 触发机制

  - save的规则满足的情况下，会自动触发rdb规则！
  - 执行flushall命令，也会触发我们的rdb规则！
  - 退出redis，也会产生rdb文件！

  备份就自动生成一个dump.rdb

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909101609279.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## 12.2. AOF(Append Only File )

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909102650337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909102904492.png#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909104121334.png#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909104450845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090910484767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

# 13. Redis发布订阅

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090911132830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

> 测试

- 发布者

```bash
127.0.0.1:6379> PUBLISH xiaofanshuo hello,xiaofan	# 发布消息
(integer) 1
127.0.0.1:6379> PUBLISH xiaofanshuo hello,redis
(integer) 1
```



- 订阅者

```bash
127.0.0.1:6379> SUBSCRIBE xiaofanshuo		# 订阅频道
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "xiaofanshuo"
3) (integer) 1
# 等待消息
1) "message"		# 消息
2) "xiaofanshuo"	# 消息频道
3) "hello,xiaofan"	# 消息内容
1) "message"
2) "xiaofanshuo"
3) "hello,redis"

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909111422268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 使用场景
  - 实时消息系统！
  - 实时聊天（频道当做聊天室，将信息回显给所有人即可！）
  - 订阅，关注系统都是可以的！
  - 稍微复杂的场景我们就是用消息中间件MQ Kafka...

# 14. Redis主从复制

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909113105105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 环境配置

```bash
[root@iZ2zeg4ytp0whqtmxbsqiiZ bin]# redis-server myconfig/redis.conf 
4229:C 09 Sep 2020 11:33:21.661 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
4229:C 09 Sep 2020 11:33:21.661 # Redis version=6.0.7, bits=64, commit=00000000, modified=0, pid=4229, just started
4229:C 09 Sep 2020 11:33:21.661 # Configuration loaded
[root@iZ2zeg4ytp0whqtmxbsqiiZ bin]# redis-cli 
127.0.0.1:6379> info replication		# 查看当前的库信息
# Replication
role:master								# 角色
connected_slaves:0						# 没有从机
master_replid:724a0bd2cfd29650f2aa0df97418cab7d68b9b41
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```

复制3个配置文件，然后修改对应的信息

1. 端口
2. pid名字
3. log文件名字
4. dump.rdb 名字



- 一主二从

  - 默认情况下，每台redis服务器都是主节点 

  - 命令模式`SLAVEOF 127.0.0.1 6379` 找自己的老大
  - 配置文件配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909121311672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

# 15. 哨兵模式

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909130444226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909130725477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

- 哨兵配置文件sentinel.conf

```bash
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面这个数字1，代表主机挂了，slave投票让谁接替成为主机，票数最多的就成为主机

启动哨兵`redis-sentinel myconfig/sentinel.conf`

```bash
4484:X 09 Sep 2020 14:25:14.436 # Sentinel ID is 96d49dd51a76685defccae4a712ce14a3c5e7e7f
4484:X 09 Sep 2020 14:25:14.436 # +monitor master myredis 127.0.0.1 6379 quorum 1
4484:X 09 Sep 2020 14:25:14.436 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:25:14.438 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.524 # +sdown master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.524 # +odown master myredis 127.0.0.1 6379 #quorum 1/1
4484:X 09 Sep 2020 14:26:20.524 # +new-epoch 1
4484:X 09 Sep 2020 14:26:20.524 # +try-failover master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.526 # +vote-for-leader 96d49dd51a76685defccae4a712ce14a3c5e7e7f 1
4484:X 09 Sep 2020 14:26:20.526 # +elected-leader master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.526 # +failover-state-select-slave master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.592 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.592 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.654 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.709 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.709 # +failover-state-reconf-slaves master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:20.782 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:21.751 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:21.751 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:21.816 # +failover-end master myredis 127.0.0.1 6379
4484:X 09 Sep 2020 14:26:21.816 # +switch-master myredis 127.0.0.1 6379 127.0.0.1 6380
4484:X 09 Sep 2020 14:26:21.816 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6380
4484:X 09 Sep 2020 14:26:21.816 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6380
4484:X 09 Sep 2020 14:26:51.863 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6380
4484:X 09 Sep 2020 14:29:32.130 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6380
4484:X 09 Sep 2020 14:29:42.080 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6380
```

# 16. Redis缓存穿透和雪崩

**缓存穿透** 查不到

**缓存击穿** 量太大，缓存过期



**布隆过滤器**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909144655324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

**缓存空对象**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909145400931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

**缓存雪崩**



- [缓存穿透、缓存击穿、缓存雪崩区别和解决方案](https://blog.csdn.net/kongtiao5/article/details/82771694)