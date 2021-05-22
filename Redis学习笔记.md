# Redis学习笔记

## 1.Redis基础知识

### 1.连接数据库

```bash
redis-cli -p 6379
```

### 2.配置命令

```bash
config get * #获取所有配置项
config get requirepass #获取密码
config set requirepass mantou #设置密码为mantou
```

### 3.默认redis有16个数据库

```bash
#切换数据库
select [index]
#查看该数据库中的数据条数
dbsize
#查看所有的key
keys *
#清除当前数据库
flushdb
#清空全部数据库内容
flushall
```

### 4.Redis是单线程的

Redis的瓶颈在于机器内存和网络带宽

Redis为什么单线程还这么快？

因为redis是把数据保存在内存中的，多线程会产生上下文切换，对于内存来说，没有上下文切换效率才是最高的。

## 2.Redis-Key的基本命令

### 1.查看所有的key

keys  *

### 2.设置key

set name mantou

### 3.判断当前key是否存在

exists name

### 4.获取key

get name

### 5.移除当前的key

move name 1

### 6.设置过期时间

expire name  10   #设置name这个key 10秒后过期

ttl  name    #查看name这个key的剩余时间

### 7.查看key的类型

type  name  

## 3.String字符串类型

### 1.基础命令

```bash
# 给key设置值
127.0.0.1:6379> set name mantou   
OK
# 给 name 这个key追加值
127.0.0.1:6379> append name 222    
(integer) 9
#获取name这个key的值
127.0.0.1:6379> get name    
"mantou222"
#获取name这个key对应的值的长度
127.0.0.1:6379> strlen name    
(integer) 15
```

### 2.自增自减

```bash
################# 自增自减 #####################
#设置初始浏览量为0
127.0.0.1:6379> set views 0
OK
#初始浏览量自增1
127.0.0.1:6379> incr views
(integer) 1
127.0.0.1:6379> get views
"1"
#初始浏览量自减1
127.0.0.1:6379> decr views
(integer) 0
127.0.0.1:6379> get views
"0"
#让初始浏览量自增10
127.0.0.1:6379> incrby views 10
(integer) 10
#让初始浏览量自减5
127.0.0.1:6379> decrby views 5
(integer) 5
```

### 3.字符串范围range

```bash
################# 字符串范围range #####################
#设置一个key
127.0.0.1:6379> set name hello,mantou
OK
#获取key的值
127.0.0.1:6379> get name
"hello,mantou"
#获取[0,4]范围内的key的值
127.0.0.1:6379> getrange name 0 4
"hello"
#获取key的全部值
127.0.0.1:6379> getrange name 0 -1
"hello,mantou
#替换指定位置开始的字符串
127.0.0.1:6379> setrange name 6 xxx
(integer) 12
127.0.0.1:6379> get name
"hello,xxxtou"
```

### 4.setex和setnx

```bash
################# setex和setnx #####################
#设置age的值为24并10秒后过期
127.0.0.1:6379> setex age 10 24
OK
127.0.0.1:6379> ttl age
(integer) 6
#判断是否存在name这个key如果存在则不进行创建
127.0.0.1:6379> setnx name mantou
(integer) 0
127.0.0.1:6379> get name
"hello,xxxtou"
127.0.0.1:6379> keys *
1) "name"
2) "views"
```

### 5.批量设置和批量获取

```bash
################# 批量设置和批量获取 #####################
#同时设置多个值
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
#同时获取多个值
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
#msetnx 是一个原子性的操作，要么全成功，要么全失败
127.0.0.1:6379> msetnx k1 v1 k4 v4
(integer) 0
127.0.0.1:6379> get k4
(nil)
```

### 6.对象

```bash
################# 对象 #####################
set user:1 {name:zhangsan,age:23}
#key的巧妙设计user:{id}:{filed}
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 23
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "23"
127.0.0.1:6379> keys *
1) "user:1:name"
2) "k1"
3) "user:1:age"
4) "k2"
5) "k3"
```

### 7.getset

```bash
################# getset #####################
#如果不存在则返回nil
127.0.0.1:6379> getset db redis
(nil)
127.0.0.1:6379> keys *
1) "user:1:name"
2) "k1"
3) "user:1:age"
4) "db"
5) "k2"
6) "k3"
127.0.0.1:6379> get db
"redis"
#如果存在值，获取原来的值，并设置新的值
127.0.0.1:6379> getset db mogodb
"redis"
127.0.0.1:6379> get db
"mogodb"
```

### 8.String数据的应用场景

a.计数器

b.统计多单位的数量

c.粉丝数

d.对象缓存数据

## 4.List类型

可以把list玩成栈、队列和阻塞队列！

list可以存重复的值

### 1.向list存值lpush和rpush

 ```bash
 #将一个值或者多个值，插入到列表头部(左)
 127.0.0.1:6379[1]> lpush list one
 (integer) 1
 127.0.0.1:6379[1]> lpush list two
 (integer) 2
 127.0.0.1:6379[1]> lpush list three
 (integer) 3
 #获取list中的值
 127.0.0.1:6379[1]> lrange list 0 -1
 1) "three"
 2) "two"
 3) "one"
 #通过区间获取具体的值
 127.0.0.1:6379[1]> lrange list 0 1
 1) "three"
 2) "two"
 #将一个值或者多个值，插入到列表尾部(右)
 127.0.0.1:6379[1]> rpush list zreo
 (integer) 4
 127.0.0.1:6379[1]> lrange list 0 -1
 1) "three"
 2) "two"
 3) "one"
 4) "zreo"
 ```

### 2.移除元素lpop和rpop

```bash
127.0.0.1:6379[1]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "zreo"
#移除list的第一个元素
127.0.0.1:6379[1]> lpop list 1
1) "three"
#移除list的最后一个元素
127.0.0.1:6379[1]> rpop list 1
1) "zreo"
127.0.0.1:6379[1]> lrange list 0 -1
1) "two"
2) "one"
```

### 3.通过下标获取值lindex

```bash
#通过下标获得list中的某一个值
127.0.0.1:6379[1]> lindex list 0
"two"
127.0.0.1:6379[1]> lindex list 1
"one"
```

### 4.返回列表的长度llen

```bash
127.0.0.1:6379[1]> llen list
(integer) 2
```

### 5.移除指定的值lrem

```bash
127.0.0.1:6379[1]> lpush list one two three three
(integer) 4
127.0.0.1:6379[1]> lrange list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
#移除list集合中指定个数的value
127.0.0.1:6379[1]> lrem list 1 three
(integer) 1
127.0.0.1:6379[1]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
```

### 6.截断ltrim

```bash
127.0.0.1:6379[1]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
#通过下标截取指定的长度，这个list已经被改变了，只剩下截取的元素。
127.0.0.1:6379[1]> ltrim list 1 2
OK
127.0.0.1:6379[1]> lrange list 0 -1
1) "two"
2) "one"
```

### 7.rpoplpush命令

```bash
127.0.0.1:6379[1]> rpush mylist hello hello1 hello2
(integer) 3
127.0.0.1:6379[1]> lrange mylist 0 -1
1) "hello"
2) "hello1"
3) "hello2"
#移除列表的最后一个元素，将他移动到新的列表中
127.0.0.1:6379[1]> rpoplpush mylist myotherlist
"hello2"
127.0.0.1:6379[1]> lrange mylist 0 -1
1) "hello"
2) "hello1"
127.0.0.1:6379[1]> lrange myotherlist 0 -1
1) "hello2"
```

### 8.更新操作lset命令

```bash
lset  #将列表中指定的下标的值替换为另外一个值，更新操作
#判断这个列表是否存在
127.0.0.1:6379[1]> exists list
(integer) 0
#如果不存在列表，更新会报错
127.0.0.1:6379[1]> lset list 0 item
(error) ERR no such key
127.0.0.1:6379[1]> lpush list v1
(integer) 1
127.0.0.1:6379[1]> lrange list 0 -1
1) "v1"
#如果存在，更新当前下标的值
127.0.0.1:6379[1]> lset list 0 item
OK
127.0.0.1:6379[1]> lrange list 0 -1
1) "item"
#如果不存在，则报错
127.0.0.1:6379[1]> lset list 1 other
(error) ERR index out of range
```

### 9.插入值linsert命令

```bash
127.0.0.1:6379[1]> rpush mylist hello world
(integer) 2
127.0.0.1:6379[1]> lrange mylist 0 -1
1) "hello"
2) "world"
#将某个具体的value插入到某个值的前面或后面
127.0.0.1:6379[1]> linsert mylist before world redis
(integer) 3
127.0.0.1:6379[1]> lrange mylist 0 -1
1) "hello"
2) "redis"
3) "world"
127.0.0.1:6379[1]> linsert mylist after world ok
(integer) 4
127.0.0.1:6379[1]> lrange mylist 0 -1
1) "hello"
2) "redis"
3) "world"
4) "ok"
```

## 5.Set类型

set无序不重复

### 1.添加|查看|判断

```bash
#set集合中添加元素
127.0.0.1:6379[2]> sadd myset hello mantou
(integer) 2
#查看指定set的所有值
127.0.0.1:6379[2]> smembers myset
1) "hello"
2) "mantou"
#如果添加的元素重复则不会添加
127.0.0.1:6379[2]> sadd myset hello
(integer) 0
#判断某一个值是不是在set集合中
127.0.0.1:6379[2]> sismember myset hello
(integer) 1
127.0.0.1:6379[2]> sismember myset world
(integer) 0
```

### 2.获取元素个数

```bash
#获取set集合中元素的个数
127.0.0.1:6379[2]> scard myset
(integer) 2
```

### 3.移除元素

```bash
#移除set集合中的某一个元素
127.0.0.1:6379[2]> srem myset hello
(integer) 1
127.0.0.1:6379[2]> smembers myset
1) "mantou"
```

### 4.随机抽选出一个元素

```bash
#随机抽选出一个元素
127.0.0.1:6379[2]> srandmember myset
"mantou3"
127.0.0.1:6379[2]> srandmember myset
"mantou"
#随机抽选出指定个数的元素
127.0.0.1:6379[2]> srandmember myset 2
1) "mantou5"
2) "mantou2"
```

### 5.随即删除元素

```bash
#随机删除一些set集合中的元素
127.0.0.1:6379[2]> spop myset
"mantou"
127.0.0.1:6379[2]> spop myset
"mantou2"
127.0.0.1:6379[2]> smembers myset
1) "mantou5"
2) "mantou3"
3) "mantou4"
```

### 6.将指定值移动到另外一个key中

```bash
127.0.0.1:6379[2]> smembers myset
1) "mantou5"
2) "mantou3"
3) "mantou4"
#将指定值移动到另外一个set集合中
127.0.0.1:6379[2]> smove myset myset2 mantou5
(integer) 1
127.0.0.1:6379[2]> keys *
1) "myset"
2) "myset2"
127.0.0.1:6379[2]> smembers myset2
1) "mantou5"
127.0.0.1:6379[2]> smembers myset
1) "mantou3"
2) "mantou4"
```

### 7.差集交集并集

```bash
127.0.0.1:6379[2]> sadd key1 a b c
(integer) 3
127.0.0.1:6379[2]> sadd key2 c d e
(integer) 3
#差集
127.0.0.1:6379[2]> sdiff key1 key2
1) "a"
2) "b"
#交集，共同好友可以实现
127.0.0.1:6379[2]> sinter key1 key2
1) "c"
#并集
127.0.0.1:6379[2]> sunion key1 key2
1) "b"
2) "c"
3) "e"
4) "d"
5) "a"
```

应用场景:共同好友，共同关注

## 6.Hash类型

Hash类型是key-map类型，这个时候的值是一个map集合，本质和String没有太大区别，还是一个简单的key-value

### 1.添加获取元素

```bash
#添加一个具体的field-value
127.0.0.1:6379[3]> hset myhash field1 mantou
(integer) 1
#获取一个值
127.0.0.1:6379[3]> hget myhash field1
"mantou"
#同时添加多个值
127.0.0.1:6379[3]> hmset myhash field1 hello field2 mantou
OK
#同时获取多个值
127.0.0.1:6379[3]> hmget myhash field1 field2
1) "hello"
2) "mantou"
#获取hash中全部的数据
127.0.0.1:6379[3]> hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "mantou"
```

### 2.删除元素

```bash
#删除hash指定的field字段，对应的value也就没有了
127.0.0.1:6379[3]> hdel myhash field1
(integer) 1
127.0.0.1:6379[3]> hgetall myhash
1) "field2"
2) "mantou"
```

### 3.获取键值对的个数

```bash
127.0.0.1:6379[3]> hgetall myhash
1) "field2"
2) "mantou"
3) "field1"
4) "hello"
#获取hash表的键值对的个数
127.0.0.1:6379[3]> hlen myhash
(integer) 2
```

### 4.判断某个field是否存在

```bash
#判断key是否存在
127.0.0.1:6379[3]> hexists myhash field1
(integer) 1
127.0.0.1:6379[3]> hexists myhash field3
(integer) 0
```

### 5.只获得所有的field

```bash
127.0.0.1:6379[3]> hkeys myhash
1) "field2"
2) "field1"
```

### 6.只获得所有的value

```bash
127.0.0.1:6379[3]> hvals myhash
1) "mantou"
2) "hello"
```

### 7.自增自减

```bash
127.0.0.1:6379[3]> hset myhash field3 5
(integer) 1
#自增1
127.0.0.1:6379[3]> hincrby myhash field3 1
(integer) 6
#自减1
127.0.0.1:6379[3]> hincrby myhash field3 -1
(integer) 5
```

### 8.判断field是否存在，不存在则创建，存在不创建

```bash
127.0.0.1:6379[3]> hset myhash field4 hello
(integer) 1
127.0.0.1:6379[3]> hset myhash field4 world
(integer) 0
```

### 9.应用场景

hash更适合于对象的存储，string更适合字符串存储。

示例:

```bash
127.0.0.1:6379[3]> hset user:1 name mantou age 24 sex man
(integer) 3
127.0.0.1:6379[3]> hmget user:1 name age
1) "mantou"
2) "24"
```

## 7.Zset有序集合

在set的基础上增加了一个值，变得有序

### 1.添加遍历集合

```bash
#添加多个值
127.0.0.1:6379[4]> zadd myset 1 one 2 two 3 three
(integer) 3
#遍历zset集合
127.0.0.1:6379[4]> zrange myset 0 -1
1) "one"
2) "two"
3) "three"
```

### 2.排序

```bash
#添加三个薪资数据
127.0.0.1:6379[4]> zadd salary 3000 zhangsan 2500 lisi 10000 mantou
(integer) 3
#薪资从负无穷到正无穷排序
127.0.0.1:6379[4]> zrangebyscore salary -inf +inf
1) "lisi"
2) "zhangsan"
3) "mantou"
#薪资从负无穷到正无穷排序,并把薪资显示出来
127.0.0.1:6379[4]> zrangebyscore salary -inf +inf withscores
1) "lisi"
2) "2500"
3) "zhangsan"
4) "3000"
5) "mantou"
6) "10000"
#降序排序
127.0.0.1:6379[4]> zrevrange salary 0 -1
1) "mantou"
2) "zhangsan"
```

### 3.移除元素

```bash
127.0.0.1:6379[4]> zrange salary 0 -1
1) "lisi"
2) "zhangsan"
3) "mantou"
#移除有序集合中的指定元素
127.0.0.1:6379[4]> zrem salary lisi
(integer) 1
127.0.0.1:6379[4]> zrange salary 0 -1
1) "zhangsan"
2) "mantou"
#获取有序集合中的元素个数
127.0.0.1:6379[4]> zcard salary
(integer) 2
```

### 4.获取区间内的元素个数

```bash
127.0.0.1:6379[4]> zrevrange salary 0 -1 withscores
 1) "mantou"
 2) "10000"
 3) "zhangsan"
 4) "3000"
 5) "lisi"
 6) "2000"
 7) "wangwu"
 8) "1000"
 9) "zhaoliu"
10) "200"
#获取区间内的元素个数
127.0.0.1:6379[4]> zcount salary 1000 5000
(integer) 3
127.0.0.1:6379[4]> zcount salary 1000 10000
(integer) 4
```

### 5.应用场景

a.排行榜

b.成绩排名

c.带权重排名消息

## 8.三种特殊的数据类型

### 1.Geospatial地理位置

#### 1.geoadd添加地理位置

- 有效的经度从-180度到180度。
- 有效的纬度从-85.05112878度到85.05112878度。

```bash
127.0.0.1:6379[5]> geoadd china:city 116.35 40.40 beijing 114.46 38.03 hebei 104.07 30.65 sichuan 123.42 41.83 liaoning
(integer) 4
```

#### 2.geopos获取地理位置

```bash
127.0.0.1:6379[5]> geopos china:city hebei
1) 1) "114.46000009775161743"
   2) "38.02999941748397106"
127.0.0.1:6379[5]> geopos china:city hebei beijing
1) 1) "114.46000009775161743"
   2) "38.02999941748397106"
2) 1) "116.35000258684158325"
   2) "40.39999918756212338"
```

#### 3.geodist两地距离

单位(默认是米):

- **m** for meters.
- **km** for kilometers.
- **mi** for miles.
- **ft** for feet.

```bash
127.0.0.1:6379[5]> geodist china:city beijing hebei km
"309.8442"
```

#### 4.georadius通过半径查询

georadius key  经度 纬度 范围 单位

```bash
127.0.0.1:6379[5]> georadius china:city 110 30 1000 km
1) "sichuan"
2) "hebei"
#获得指定数量的人
127.0.0.1:6379[5]> georadius china:city 110 30 10000 km withcoord withdist count 2
1) 1) "sichuan"
   2) "573.8276"
   3) 1) "104.07000213861465454"
      2) "30.6499990746355806"
2) 1) "hebei"
   2) "982.9062"
   3) 1) "114.46000009775161743"
      2) "38.02999941748397106"
```

#### 5.georadiusbymember通过元素进行范围查询

```bash
127.0.0.1:6379[5]> georadiusbymember china:city beijing 1000 km withcoord withdist
1) 1) "hebei"
   2) "309.8442"
   3) 1) "114.46000009775161743"
      2) "38.02999941748397106"
2) 1) "beijing"
   2) "0.0000"
   3) 1) "116.35000258684158325"
      2) "40.39999918756212338"
3) 1) "liaoning"
   2) "613.2173"
   3) 1) "123.41999977827072144"
      2) "41.83000015042058095"
```

#### 6.geohash返回一个或多个位置的hash值

把经纬度转换为字符串

```bash
127.0.0.1:6379[5]> geohash china:city beijing hebei
1) "wx4tzde7p90"
2) "wwc2ke5hz30"
```

#### 7.geo的实现原理

geo底层使用的是Zset，所以我们可以直接使用Zset命令来操作geo

```bash
127.0.0.1:6379[5]> zrange china:city 0 -1
1) "sichuan"
2) "hebei"
3) "beijing"
4) "liaoning"
#移除指定元素位置信息
127.0.0.1:6379[5]> zrem china:city hebei
(integer) 1
127.0.0.1:6379[5]> zrange china:city 0 -1
1) "sichuan"
2) "beijing"
3) "liaoning"
```

### 2.hyperloglog基数统计

优点:占用的内存是固定的

应用场景:网页的访问量统计，一个人访问多次算作一个人

缺点:有0.81%的错误率

```bash
#创建第一组元素
127.0.0.1:6379[6]> pfadd mykey a b c d e f g h i j
(integer) 1
#统计mykey元素的基数数量
127.0.0.1:6379[6]> pfcount mykey
(integer) 10
#创建第二组元素
127.0.0.1:6379[6]> pfadd mykey2 i j z x c v b n m
(integer) 1
127.0.0.1:6379[6]> pfcount mykey2
(integer) 9
#合并两组元素并命名为mykey3
127.0.0.1:6379[6]> pfmerge mykey3 mykey mykey2
OK
127.0.0.1:6379[6]> pfcount mykey3
(integer) 15
```

### 3.Bitmap位图

应用场景:统计用户活跃不活跃，登录未登录，打卡未打卡，一般只有两个状态的都可以使用bitmap

示例:使用bitmap记录一周的打卡天数

```bash
#添加一周内的打卡情况
127.0.0.1:6379[7]> setbit sign 0 1
(integer) 0
127.0.0.1:6379[7]> setbit sign 1 1
(integer) 0
127.0.0.1:6379[7]> setbit sign 2 0
(integer) 0
127.0.0.1:6379[7]> setbit sign 3 0
(integer) 0
127.0.0.1:6379[7]> setbit sign 4 1
(integer) 0
127.0.0.1:6379[7]> setbit sign 5 0
(integer) 0
127.0.0.1:6379[7]> setbit sign 6 1
(integer) 0
#查看某一天的打卡情况
127.0.0.1:6379[7]> getbit sign 4
(integer) 1
127.0.0.1:6379[7]> getbit sign 3
(integer) 0
#统计这一周打卡了多少天
127.0.0.1:6379[7]> bitcount sign
(integer) 4
```

## 9.事务

==Redis单条命令是保证原子性的，但是事务是打不保证原子性的==

==Redis没有隔离级别的概念==

==所有的命令在事务中并没有直接被执行！只有发起执行命令的时候才会执行!Exec==

redis事务的本质:

​	一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，会按顺序执行!

redis事务：

- 开启事务multi
- 命令入队
- 执行事务exec

### 1.正常执行事务

```bash
#开启事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1 
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
#执行事务
127.0.0.1:6379(TX)> exec
1) OK
2) OK
3) "v2"
4) OK
```

### 2.放弃事务

```bash
#开启事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> set k5 v5
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
#放弃事务,放弃后不会执行事务中的命令
127.0.0.1:6379(TX)> discard
OK
127.0.0.1:6379> get k4
(nil)
```

### 3.编译时错误

代码有问题，命令有错误，事务中的所有命令都不会被执行。

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
#命令出错
127.0.0.1:6379(TX)> getset k3
(error) ERR wrong number of arguments for 'getset' command
#执行事务
127.0.0.1:6379(TX)> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
(nil)
```

### 4.运行时异常

错误的命令抛出异常，其他命令可以正常执行。

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> incr k1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> get k1
QUEUED
127.0.0.1:6379(TX)> get k3
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
 #虽然这条命令报错，但是其他命令依然成功执行
2) (error) ERR value is not an integer or out of range  
3) OK
4) OK
5) "v1"
6) "v3"
```

## 10.Redis实现乐观锁

### 1.监控Watch

==悲观锁：==

​	很悲观，认为什么时候都会出问题，无论做什么都会加锁!

==乐观锁：==

​	很乐观，认为什么时候不会出问题，所以一般不上锁，更新数据的时候去判断一下再此期间是否有人修改过这个数据。

Redis监视测试:

a.正常执行

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
#监视money对象
127.0.0.1:6379> watch money   
OK
#事务正常结束，数据期间没有发生变动，这个时候就正常执行成功！
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> decrby money 20
QUEUED
127.0.0.1:6379(TX)> incrby out 20
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 80
2) (integer) 20
```

b.多线程修改

```bash
#第一个线程
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
#监视money
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> decrby money 10
QUEUED
127.0.0.1:6379(TX)> incrby out 10
QUEUED
#在执行之前第二个线程修改了money的值，就会导致事务执行失败！
127.0.0.1:6379(TX)> exec
(nil)
#如果发现事务执行失败，先进行解锁，然后再重新获取锁
127.0.0.1:6379> unwatch
OK

#第二个线程
mantou_redis:0>get money
"100"
mantou_redis:0>incrby money 200
"300"
```

## 11.Jedis

### 1.导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.manotu</groupId>
    <artifactId>jedis-study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>
    </dependencies>
</project>
```

### 2.测试连接

```java
package com.mantou.ping;

import redis.clients.jedis.Jedis;

public class Ping {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```

### 3.TestString

```java
package com.mantou.base;

import redis.clients.jedis.Jedis;

import java.util.concurrent.TimeUnit;

public class TestString {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        jedis.flushDB();
        System.out.println("===========增加数据===========");
        System.out.println(jedis.set("key1","value1"));
        System.out.println(jedis.set("key2","value2"));
        System.out.println(jedis.set("key3", "value3"));
        System.out.println("删除键key2:"+jedis.del("key2"));
        System.out.println("获取键key2:"+jedis.get("key2"));
        System.out.println("修改key1:"+jedis.set("key1", "value1Changed"));
        System.out.println("获取key1的值："+jedis.get("key1"));
        System.out.println("在key3后面加入值："+jedis.append("key3", "End"));
        System.out.println("key3的值："+jedis.get("key3"));
        System.out.println("增加多个键值对："+jedis.mset("key01","value01","key02","value02","key03","value03"));
        System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));
        System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03","key04"));
        System.out.println("删除多个键值对："+jedis.del("key01","key02"));
        System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));

        jedis.flushDB();
        System.out.println("===========新增键值对防止覆盖原先值==============");
        System.out.println(jedis.setnx("key1", "value1"));
        System.out.println(jedis.setnx("key2", "value2"));
        System.out.println(jedis.setnx("key2", "value2-new"));
        System.out.println(jedis.get("key1"));
        System.out.println(jedis.get("key2"));

        System.out.println("===========新增键值对并设置有效时间=============");
        System.out.println(jedis.setex("key3", 2, "value3"));
        System.out.println(jedis.get("key3"));
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(jedis.get("key3"));

        System.out.println("===========获取原值，更新为新值==========");
        System.out.println(jedis.getSet("key2", "key2GetSet"));
        System.out.println(jedis.get("key2"));

        System.out.println("获得key2的值的字串："+jedis.getrange("key2", 2, 4));
    }
}
```

### 4.TestList

```java
package com.mantou.base;

import redis.clients.jedis.Jedis;

public class TestList {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("===========添加一个list===========");
        jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("collections", "HashSet");
        jedis.lpush("collections", "TreeSet");
        jedis.lpush("collections", "TreeMap");
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));//-1代表倒数第一个元素，-2代表倒数第二个元素,end为-1表示查询全部
        System.out.println("collections区间0-3的元素："+jedis.lrange("collections",0,3));
        System.out.println("===============================");
        // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
        System.out.println("删除指定元素个数："+jedis.lrem("collections", 2, "HashMap"));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("删除下表0-3区间之外的元素："+jedis.ltrim("collections", 0, 3));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（左端）："+jedis.lpop("collections"));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("collections添加元素，从列表右端，与lpush相对应："+jedis.rpush("collections", "EnumMap"));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（右端）："+jedis.rpop("collections"));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("修改collections指定下标1的内容："+jedis.lset("collections", 1, "LinkedArrayList"));
        System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
        System.out.println("===============================");
        System.out.println("collections的长度："+jedis.llen("collections"));
        System.out.println("获取collections下标为2的元素："+jedis.lindex("collections", 2));
        System.out.println("===============================");
        jedis.lpush("sortedList", "3","6","2","0","7","4");
        System.out.println("sortedList排序前："+jedis.lrange("sortedList", 0, -1));
        System.out.println(jedis.sort("sortedList"));
        System.out.println("sortedList排序后："+jedis.lrange("sortedList", 0, -1));
    }
}
```

### 5.TestSet

```java
package com.mantou.base;

import redis.clients.jedis.Jedis;

public class TestSet {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("============向集合中添加元素（不重复）============");
        System.out.println(jedis.sadd("eleSet", "e1","e2","e4","e3","e0","e8","e7","e5"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));
        System.out.println("删除一个元素e0："+jedis.srem("eleSet", "e0"));
        System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));
        System.out.println("删除两个元素e7和e6："+jedis.srem("eleSet", "e7","e6"));
        System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));
        System.out.println("随机的移除集合中的一个元素："+jedis.spop("eleSet"));
        System.out.println("随机的移除集合中的一个元素："+jedis.spop("eleSet"));
        System.out.println("eleSet的所有元素为："+jedis.smembers("eleSet"));
        System.out.println("eleSet中包含元素的个数："+jedis.scard("eleSet"));
        System.out.println("e3是否在eleSet中："+jedis.sismember("eleSet", "e3"));
        System.out.println("e1是否在eleSet中："+jedis.sismember("eleSet", "e1"));
        System.out.println("e1是否在eleSet中："+jedis.sismember("eleSet", "e5"));
        System.out.println("=================================");
        System.out.println(jedis.sadd("eleSet1", "e1","e2","e4","e3","e0","e8","e7","e5"));
        System.out.println(jedis.sadd("eleSet2", "e1","e2","e4","e3","e0","e8"));
        System.out.println("将eleSet1中删除e1并存入eleSet3中："+jedis.smove("eleSet1", "eleSet3", "e1"));//移到集合元素
        System.out.println("将eleSet1中删除e2并存入eleSet3中："+jedis.smove("eleSet1", "eleSet3", "e2"));
        System.out.println("eleSet1中的元素："+jedis.smembers("eleSet1"));
        System.out.println("eleSet3中的元素："+jedis.smembers("eleSet3"));
        System.out.println("============集合运算=================");
        System.out.println("eleSet1中的元素："+jedis.smembers("eleSet1"));
        System.out.println("eleSet2中的元素："+jedis.smembers("eleSet2"));
        System.out.println("eleSet1和eleSet2的交集:"+jedis.sinter("eleSet1","eleSet2"));
        System.out.println("eleSet1和eleSet2的并集:"+jedis.sunion("eleSet1","eleSet2"));
        System.out.println("eleSet1和eleSet2的差集:"+jedis.sdiff("eleSet1","eleSet2"));//eleSet1中有，eleSet2中没有
        jedis.sinterstore("eleSet4","eleSet1","eleSet2");//求交集并将交集保存到dstkey的集合
        System.out.println("eleSet4中的元素："+jedis.smembers("eleSet4"));
    }
}
```

### 6.TestHash

```java
package com.mantou.base;

import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.Map;

public class TestHash {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        Map<String,String> map = new HashMap<String,String>();
        map.put("key1","value1");
        map.put("key2","value2");
        map.put("key3","value3");
        map.put("key4","value4");
        //添加名称为hash（key）的hash元素
        jedis.hmset("hash",map);
        //向名称为hash的hash中添加key为key5，value为value5元素
        jedis.hset("hash", "key5", "value5");
        System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));//return Map<String,String>
        System.out.println("散列hash的所有键为："+jedis.hkeys("hash"));//return Set<String>
        System.out.println("散列hash的所有值为："+jedis.hvals("hash"));//return List<String>
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 6));
        System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 3));
        System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
        System.out.println("删除一个或者多个键值对："+jedis.hdel("hash", "key2"));
        System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
        System.out.println("散列hash中键值对的个数："+jedis.hlen("hash"));
        System.out.println("判断hash中是否存在key2："+jedis.hexists("hash","key2"));
        System.out.println("判断hash中是否存在key3："+jedis.hexists("hash","key3"));
        System.out.println("获取hash中的值："+jedis.hmget("hash","key3"));
        System.out.println("获取hash中的值："+jedis.hmget("hash","key3","key4"));
    }
}
```

### 7.TestKey

```java
package com.kuang.base;

import redis.clients.jedis.Jedis;

import java.util.Set;

public class TestKey {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        System.out.println("清空数据："+jedis.flushDB());
        System.out.println("判断某个键是否存在："+jedis.exists("username"));
        System.out.println("新增<'username','kuangshen'>的键值对："+jedis.set("username", "kuangshen"));
        System.out.println("新增<'password','password'>的键值对："+jedis.set("password", "password"));
        System.out.print("系统中所有的键如下：");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys);
        System.out.println("删除键password:"+jedis.del("password"));
        System.out.println("判断键password是否存在："+jedis.exists("password"));
        System.out.println("查看键username所存储的值的类型："+jedis.type("username"));
        System.out.println("随机返回key空间的一个："+jedis.randomKey());
        System.out.println("重命名key："+jedis.rename("username","name"));
        System.out.println("取出改后的name："+jedis.get("name"));
        System.out.println("按索引查询："+jedis.select(0));
        System.out.println("删除当前选择数据库中的所有key："+jedis.flushDB());
        System.out.println("返回当前数据库中key的数目："+jedis.dbSize());
        System.out.println("删除所有数据库中的所有key："+jedis.flushAll());
    }
}
```

### 8.事务

```java
package com.mantou.multi;

import com.alibaba.fastjson.JSONObject;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

public class TestMulti {
    public static void main(String[] args) {
        //创建客户端连接服务端，redis服务端需要被开启
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello", "world");
        jsonObject.put("name", "java");
        //开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        try{
            //向redis存入一条数据
            multi.set("json", result);
            //再存入一条数据
            multi.set("json2", result);
            //这里引发了异常，用0作为被除数
            int i = 100/0;
            //如果没有引发异常，执行进入队列的命令
            multi.exec();
        }catch(Exception e){
            e.printStackTrace();
            //如果出现异常，回滚
            multi.discard();
        }finally{
            System.out.println(jedis.get("json"));
            System.out.println(jedis.get("json2"));
            //最终关闭客户端
            jedis.close();
        }
    }
}
```

## 12.SpringBoot集成Redis

### 1.自定义redisTemplate

```java
package com.mantou.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

  @Bean
  @SuppressWarnings("all")
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
      RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
      template.setConnectionFactory(factory);
      Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      ObjectMapper om = new ObjectMapper();
      om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      jackson2JsonRedisSerializer.setObjectMapper(om);
      StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

      // key采用String的序列化方式
      template.setKeySerializer(stringRedisSerializer);
      // hash的key也采用String的序列化方式
      template.setHashKeySerializer(stringRedisSerializer);
      // value序列化方式采用jackson
      template.setValueSerializer(jackson2JsonRedisSerializer);
      // hash的value序列化方式采用jackson
      template.setHashValueSerializer(jackson2JsonRedisSerializer);
      template.afterPropertiesSet();

      return template;
  }
}
```

### 2.封装redisUtil

```java
package com.kuang.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
public final class RedisUtil {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================
    /**
     * 指定缓存失效时间
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }


    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }


    // ============================String=============================

    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 普通缓存放入并设置时间
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */

    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 递增
     * @param key   键
     * @param delta 要增加几(大于0)
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }


    /**
     * 递减
     * @param key   键
     * @param delta 要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }


    // ================================Map=================================

    /**
     * HashGet
     * @param key  键 不能为null
     * @param item 项 不能为null
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * HashSet 并设置时间
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }


    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }


    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }


    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }


    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     * @param key 键
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 获取set缓存的长度
     *
     * @param key 键
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */

    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取list缓存的长度
     *
     * @param key 键
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将list放入缓存
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */

    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */

    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }

}
```

## 13.Redis.conf详解

### 1.units单位

```
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
# 
# units are case insensitive so 1GB 1Gb 1gB are all the same.


配置大小单位,开头定义了一些基本的度量单位,不支持bit
对大小写不敏感
```

### 2.includes包含

```properties
# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```

### 3.network

```properties
################################## NETWORK #####################################

# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511

设置tcp的backlog,backlog其实是一个连接队列,backlog队列总和=未完成三次握手队列+已完成三次握手队列.
在高并发环境下你需要一个高的backlog值来避免慢客户端连接问题.
注意linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值,所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的效果



# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0

超时时间设置,0为关闭




# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300

单位为秒,如果设置为0,则不会进行keepalive检测,建议设置成60
```

### 4.general通用

```properties
# 是否以守护进行
daemonize no
# 进程管道id文件,如果没有指定,则在这个路径下
pidfile /var/run/redis_6379.pid
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
# 日志级别
loglevel notice
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
# 日志名字
loglevel notice
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# 是否把日志输出到syslog中
# syslog-enabled no
# Specify the syslog identity.
# 指定syslog里的日志表示
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# 指定syslog设备,值可以是user或者local0-local7
# syslog-facility local0
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
# 默认有16个数据库
databases 16
```

### 5.snapshotting快照

```properties
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
# flushall和shutdown会立即出发save命令,进行备份
# 禁用RDB持久化策略,只要不设置任何save指令(在redis的命令窗口中使用save命令)或者使用下面的save ""也可以(save传入一个空字符串参数也可以)
#   save ""
# 下面三个条件符合其一就触发备份
# 900秒内有一个key改变过就备份
save 900 1
# 300秒内有10个key改变过就备份
save 300 10
# 60秒内有10000个key改变就触发备份
save 60 10000


# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
# 如何配置成no,表示不在乎数据不一致或者其他的手段发现和控制
stop-writes-on-bgsave-error yes


# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
# 对于存储到磁盘中的快照,可以设置是否进行压缩存储.
# 如果是的话,redis会采用LZF算法进行压缩.
# 如果不想消耗CPU进行要锁,可以设置为关闭此功能
rdbcompression yes


# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
# 在存储快照后,还可以让redis使用CRC64算法来进行数据检验
# 这样做会增加10%的性能消耗,
# 如果想获得最大的性能提升,则可以关闭此功能
rdbchecksum yes


# The filename where to dump the DB
# 保存时的文件名称,断电重启时读取的文件名称
dbfilename dump.rdb


# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
# 工作目录
dir ./
```

### 6.replication复制

### 7.security安全

```properties
# 获取登录密码
config get requirepass
127.0.0.1:8686> config get requirepass
1) "requirepass"
2) "51310400"
# 查询启动时所在的目录
config get dir
127.0.0.1:8686> config get dir
1) "dir"
2) "/alidata/redis-5.0.3/db"

# 设置redis密码
config set requirepass 123456

# 登录redis
[root@izm5e2q95pbpe1hh0kkwoiz /]# redis-cli -p 8686
127.0.0.1:8686> ping
(error) NOAUTH Authentication required.
127.0.0.1:8686> auth 51310400
OK
127.0.0.1:8686> ping
PONG
```

### 8.CLIENTS

```properties
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
# 最大连接数10000
# maxclients 10000
```

### 9.limits限制

```properties
# 最大内存
# maxmemory <bytes>

## 达到最大内存时清除策略
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
## 使用LUR算法移除key,只对设置了过期时间的键.LRU最近最少使用算法
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
## 使用LRU算法移除key
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
## 在过期集合中移除随机的key,只对设置了过期时间的键
# volatile-random -> Remove a random key among the ones with an expire set.
## 移除随机的key
# allkeys-random -> Remove a random key, any key.
## 移除那些ttl值最小的key,即那些最近要过期的key
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
## 不进行移除.针对写操作,只是返回错误信息   
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
# 默认配置是不清除,但是配置没有开启
# maxmemory-policy noeviction
# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
# 设置样本你数量,LRU算法和最小TTL算法都并非是精确的算法,而是估算值,醉意可以设置样本的大小.redis默认会检查这么多个key并选择其中LRU的那个
# maxmemory-samples 5
```

### 10.append only mode追加

```properties
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.
# 默认是关闭状态
appendonly no
# The name of the append only file (default: "appendonly.aof")
# 备份文件的名字
appendfilename "appendonly.aof"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
# 备份时机
# always:同步持久化,每次发生数据变更会被立即就到磁盘,性能较差但数据记录完整性比较好
# ererysec:出厂默认配置,异步操作,每秒记录,如果一秒内宕机,有数据丢失
# no:不追加
appendfsync everysec

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.
# 重写时是否可以运用appendfsync,用默认no即可,保证数据安全性
no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.
# 设置重写的基准值,此时是上次重写体积的100%,也就是体积翻一倍
auto-aof-rewrite-percentage 100
# 设置重写的基准值,此时是重写时日志要大于64MB
auto-aof-rewrite-min-size 64mb
```

## 14.Redis持久化

Redis 提供了多种不同级别的持久化方式：

- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
- AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
- Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。
- 你甚至可以关闭持久化功能，让数据只在服务器运行时存在。

了解 RDB 持久化和 AOF 持久化之间的异同是非常重要的， 以下几个小节将详细地介绍这这两种持久化功能， 并对它们的相同和不同之处进行说明。

### 1.RDB 的优点

- RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。
- RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。
- RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 `fork` 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。
- RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

### 2.RDB 的缺点

- 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。
- 每次保存 RDB 的时候，Redis 都要 `fork()` 出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时， `fork()` 可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 `fork()` ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

### 3.AOF 的优点

- 使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 `fsync` 策略，比如无 `fsync` ，每秒钟一次 `fsync` ，或者每次执行写入命令时 `fsync` 。 AOF 的默认策略为每秒钟 `fsync` 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ `fsync` 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
- AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行 `seek` ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， `redis-check-aof` 工具也可以轻易地修复这种问题。
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 [FLUSHALL](http://redisdoc.com/database/flushall.html#flushall) 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 [FLUSHALL](http://redisdoc.com/database/flushall.html#flushall) 命令， 并重启 Redis ， 就可以将数据集恢复到 [FLUSHALL](http://redisdoc.com/database/flushall.html#flushall) 执行之前的状态。

### 4.AOF 的缺点

- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
- 根据所使用的 `fsync` 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 `fsync` 的性能依然非常高， 而关闭 `fsync` 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
- AOF 在过去曾经发生过这样的 bug ： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。 （举个例子，阻塞命令 [BRPOPLPUSH source destination timeout](http://redisdoc.com/list/brpoplpush.html#brpoplpush) 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试： 它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。

### 5.RDB 和 AOF ，我应该用哪一个？

一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。

如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化。

有很多用户都只使用 AOF 持久化， 但我们并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快， 除此之外， 使用 RDB 还可以避免之前提到的 AOF 程序的 bug 。

Note

因为以上提到的种种原因， 未来我们可能会将 AOF 和 RDB 整合成单个持久化模型。 （这是一个长期计划。）

接下来的几个小节将介绍 RDB 和 AOF 的更多细节。

### 6.RDB 快照

在默认情况下， Redis 将数据库快照保存在名字为 `dump.rdb` 的二进制文件中。

你可以对 Redis 进行设置， 让它在“ `N` 秒内数据集至少有 `M` 个改动”这一条件被满足时， 自动保存一次数据集。

你也可以通过调用 [SAVE](http://redisdoc.com/persistence/save.html#save) 或者 [BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) ， 手动让 Redis 进行数据集保存操作。

比如说， 以下设置会让 Redis 在满足“ `60` 秒内有至少有 `1000` 个键被改动”这一条件时， 自动保存一次数据集：

```
save 60 1000
```

这种持久化方式被称为快照（snapshot）。

### 7.快照的运作方式

当 Redis 需要保存 `dump.rdb` 文件时， 服务器执行以下操作：

1. Redis 调用 `fork()` ，同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

### 8.只进行追加操作的文件（append-only file，AOF）

快照功能并不是非常耐久（durable）： 如果 Redis 因为某些原因而造成故障停机， 那么服务器将丢失最近写入、且仍未保存到快照中的那些数据。

尽管对于某些程序来说， 数据的耐久性并不是最重要的考虑因素， 但是对于那些追求完全耐久能力（full durability）的程序来说， 快照功能就不太适用了。

从 1.1 版本开始， Redis 增加了一种完全耐久的持久化方式： AOF 持久化。

你可以通过修改配置文件来打开 AOF 功能：

```
appendonly yes
```

从现在开始， 每当 Redis 执行一个改变数据集的命令时（比如 [SET key value [EX seconds\] [PX milliseconds] [NX|XX]](http://redisdoc.com/string/set.html#set)）， 这个命令就会被追加到 AOF 文件的末尾。

这样的话， 当 Redis 重新启时， 程序就可以通过重新执行 AOF 文件中的命令来达到重建数据集的目的。

### 9.AOF 重写

因为 AOF 的运作方式是不断地将命令追加到文件的末尾， 所以随着写入命令的不断增加， AOF 文件的体积也会变得越来越大。

举个例子， 如果你对一个计数器调用了 100 次 [INCR key](http://redisdoc.com/string/incr.html#incr) ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100 条记录（entry）。

然而在实际上， 只使用一条 [SET key value [EX seconds\] [PX milliseconds] [NX|XX]](http://redisdoc.com/string/set.html#set) 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。

为了处理这种情况， Redis 支持一种有趣的特性： 可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。

执行 [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 命令， Redis 将生成一个新的 AOF 文件， 这个文件包含重建当前数据集所需的最少命令。

Redis 2.2 需要自己手动执行 [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 命令； Redis 2.4 则可以自动触发 AOF 重写， 具体信息请查看 2.4 的示例配置文件。

### 10.AOF 的耐久性如何？

你可以配置 Redis 多久才将数据 `fsync` 到磁盘一次。

有三个选项：

- 每次有新命令追加到 AOF 文件时就执行一次 `fsync` ：非常慢，也非常安全。
- 每秒 `fsync` 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
- 从不 `fsync` ：将数据交给操作系统来处理。更快，也更不安全的选择。

推荐（并且也是默认）的措施为每秒 `fsync` 一次， 这种 `fsync` 策略可以兼顾速度和安全性。

总是 `fsync` 的策略在实际使用中非常慢， 即使在 Redis 2.0 对相关的程序进行了改进之后仍是如此 —— 频繁调用 `fsync` 注定了这种策略不可能快得起来。

### 11.如果 AOF 文件出错了，怎么办？

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。

当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

1. 为现有的 AOF 文件创建一个备份。
2. 使用 Redis 附带的 `redis-check-aof` 程序，对原来的 AOF 文件进行修复。

> ```
> $ redis-check-aof --fix
> ```

1. （可选）使用 `diff -u` 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。
2. 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。

### 12.AOF 的运作方式

AOF 重写和 RDB 创建快照一样，都巧妙地利用了写时复制机制。

以下是 AOF 重写的执行步骤：

1. Redis 执行 `fork()` ，现在同时拥有父进程和子进程。
2. 子进程开始将新 AOF 文件的内容写入到临时文件。
3. 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾： 这样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。
4. 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
5. 搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

### 13.怎么从 RDB 持久化切换到 AOF 持久化

在 Redis 2.2 或以上版本，可以在不重启的情况下，从 RDB 切换到 AOF ：

1. 为最新的 `dump.rdb` 文件创建一个备份。
2. 将备份放到一个安全的地方。
3. 执行以下两条命令：

> ```
> redis-cli> CONFIG SET appendonly yes
> 
> redis-cli> CONFIG SET save ""
> ```

1. 确保命令执行之后，数据库的键的数量没有改变。
2. 确保写命令会被正确地追加到 AOF 文件的末尾。

步骤 3 执行的第一条命令开启了 AOF 功能： Redis 会阻塞直到初始 AOF 文件创建完成为止， 之后 Redis 会继续处理命令请求， 并开始将写入命令追加到 AOF 文件末尾。

步骤 3 执行的第二条命令用于关闭 RDB 功能。 这一步是可选的， 如果你愿意的话， 也可以同时使用 RDB 和 AOF 这两种持久化功能。

Note

别忘了在 `redis.conf` 中打开 AOF 功能！ 否则的话， 服务器重启之后， 之前通过 `CONFIG SET` 设置的配置就会被遗忘， 程序会按原来的配置来启动服务器。

Note

译注： 原文这里还有介绍 2.0 版本的切换方式， 考虑到 2.0 已经很老旧了， 这里省略了对那部分文档的翻译， 有需要的请参考原文。

### 14.RDB 和 AOF 之间的相互作用

在版本号大于等于 2.4 的 Redis 中， [BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) 执行的过程中， 不可以执行 [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 。 反过来说， 在 [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 执行的过程中， 也不可以执行 [BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) 。

这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。

如果 [BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) 正在执行， 并且用户显示地调用 [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 命令， 那么服务器将向用户回复一个 `OK` 状态， 并告知用户， [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 已经被预定执行： 一旦 [BGSAVE](http://redisdoc.com/persistence/bgsave.html#bgsave) 执行完毕， [BGREWRITEAOF](http://redisdoc.com/persistence/bgrewriteaof.html#bgrewriteaof) 就会正式开始。

当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

### 15.备份 Redis 数据

在阅读这个小节前， 先将下面这句话铭记于心： 一定要备份你的数据库！

磁盘故障， 节点失效， 诸如此类的问题都可能让你的数据消失不见， 不进行备份是非常危险的。

Redis 对于数据备份是非常友好的， 因为你可以在服务器运行的时候对 RDB 文件进行复制： RDB 文件一旦被创建， 就不会进行任何修改。 当服务器要创建一个新的 RDB 文件时， 它先将文件的内容保存在一个临时文件里面， 当临时文件写入完毕时， 程序才使用 `rename(2)` 原子地用临时文件替换原来的 RDB 文件。

这也就是说， 无论何时， 复制 RDB 文件都是绝对安全的。

以下是我们的建议：

- 创建一个定期任务（cron job）， 每小时将一个 RDB 文件备份到一个文件夹， 并且每天将一个 RDB 文件备份到另一个文件夹。
- 确保快照的备份都带有相应的日期和时间信息， 每次执行定期任务脚本时， 使用 `find` 命令来删除过期的快照： 比如说， 你可以保留最近 48 小时内的每小时快照， 还可以保留最近一两个月的每日快照。
- 至少每天一次， 将 RDB 备份到你的数据中心之外， 或者至少是备份到你运行 Redis 服务器的物理机器之外。

### 16.容灾备份

Redis 的容灾备份基本上就是对数据进行备份， 并将这些备份传送到多个不同的外部数据中心。

容灾备份可以在 Redis 运行并产生快照的主数据中心发生严重的问题时， 仍然让数据处于安全状态。

因为很多 Redis 用户都是创业者， 他们没有大把大把的钱可以浪费， 所以下面介绍的都是一些实用又便宜的容灾备份方法：

- Amazon S3 ，以及其他类似 S3 的服务，是一个构建灾难备份系统的好地方。 最简单的方法就是将你的每小时或者每日 RDB 备份加密并传送到 S3 。 对数据的加密可以通过 `gpg -c` 命令来完成（对称加密模式）。 记得把你的密码放到几个不同的、安全的地方去（比如你可以把密码复制给你组织里最重要的人物）。 同时使用多个储存服务来保存数据文件，可以提升数据的安全性。
- 传送快照可以使用 SCP 来完成（SSH 的组件）。 以下是简单并且安全的传送方法： 买一个离你的数据中心非常远的 VPS ， 装上 SSH ， 创建一个无口令的 SSH 客户端 key ， 并将这个 key 添加到 VPS 的 authorized_keys 文件中， 这样就可以向这个 VPS 传送快照备份文件了。 为了达到最好的数据安全性，至少要从两个不同的提供商那里各购买一个 VPS 来进行数据容灾备份。

需要注意的是， 这类容灾系统如果没有小心地进行处理的话， 是很容易失效的。

最低限度下， 你应该在文件传送完毕之后， 检查所传送备份文件的体积和原始快照文件的体积是否相同。 如果你使用的是 VPS ， 那么还可以通过比对文件的 SHA1 校验和来确认文件是否传送完整。

另外， 你还需要一个独立的警报系统， 让它在负责传送备份文件的传送器（transfer）失灵时通知你。

## 15.Redis发布订阅

## 16.Redis集群环境搭建

## 17.主从复制

## 18.哨兵

## 19.缓存穿透和雪崩
