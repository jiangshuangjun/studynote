# Redis String 字符串详解

90% 的 Java 程序员使用 Redis 只会使用一个 String 类型

```shell
############################################
127.0.0.1:6379> SET key1 v1  # 设置值
OK
127.0.0.1:6379> GET key1  # 获得值
"v1"
127.0.0.1:6379> KEYS *  # 获得所有的 key
1) "key1"
127.0.0.1:6379> EXISTS key1  # 判断某一个 key 是否存在
(integer) 1
127.0.0.1:6379> APPEND key1 "hello"  # 追加字符串，如果当前 key 不存在，就相当于 set key
(integer) 7
127.0.0.1:6379> GET key1
"v1hello"
127.0.0.1:6379> STRLEN key1  # 获取字符串的长度
(integer) 7
127.0.0.1:6379> APPEND key1 ",world"
(integer) 13
127.0.0.1:6379> STRLEN key1
(integer) 13
127.0.0.1:6379> GET key1
"v1hello,world"
127.0.0.1:6379>
############################################
# i++
# 步长 i+=
127.0.0.1:6379> SET views 0n  # 初始浏览量为 0
OK
127.0.0.1:6379> GET views
"0"
127.0.0.1:6379> INCR views  # 自增 1，浏览量 + 1
(integer) 1
127.0.0.1:6379> INCR views
(integer) 2
127.0.0.1:6379> GET views
"2"
127.0.0.1:6379> DECR views  # 自减 1，浏览量 - 1
(integer) 1
127.0.0.1:6379> DECR views
(integer) 0
127.0.0.1:6379> GET views
"0"
127.0.0.1:6379> INCRBY views 10  # 设置自增步长为 10，浏览量 + 10
(integer) 10
127.0.0.1:6379> INCRBY views 10
(integer) 20
127.0.0.1:6379> GET views
"20"
127.0.0.1:6379> DECRBY views 5  # 设置自减步长为 5，浏览量 - 5
(integer) 15
127.0.0.1:6379> DECRBY views 5
(integer) 10
127.0.0.1:6379> GET views
"10"
############################################
# 字符串范围 range
127.0.0.1:6379> SET key1 "Hello World"  # 设置 key1 的值
OK
127.0.0.1:6379> get key1
"Hello World"
127.0.0.1:6379> GETRANGE key1 0 4  # 截取字符串 [0, 4]
"Hello"
127.0.0.1:6379> GETRANGE key1 0 -1  # 截取全部的字符串，和 GET key 是一样的
"Hello World"

# 替换
127.0.0.1:6379> SET key2 "abcdefg"
OK
127.0.0.1:6379> GET key2
"abcdefg"
127.0.0.1:6379> SETRANGE key2 1 "xx"  # 替换指定位置开始的字符串
(integer) 7
127.0.0.1:6379> GET key2
"axxdefg"
############################################
# setex (set with expire)  # 设置过期时间
# setnx (set if not exist)  # 不存在再设置(在分布式锁中常常使用)

127.0.0.1:6379> SETEX key3 30 "hello"  # 设置 key3 的值为 hello, 30 秒后过期
OK
127.0.0.1:6379> TTL key3
(integer) 25
127.0.0.1:6379> TTL key3
(integer) 21
127.0.0.1:6379> SETNX mykey "redis"  # 如果 mykey 不存在，创建 mykey，值设置为 redis
(integer) 1
127.0.0.1:6379> KEYS *
1) "key3"
2) "mykey"
127.0.0.1:6379> TTL key3
(integer) 2
127.0.0.1:6379> SETNX mykey "MongoDB"  # 如果 mykey 存在，创建失败
(integer) 0
127.0.0.1:6379> GET mykey
"redis"
127.0.0.1:6379> TTL key3
(integer) -2
127.0.0.1:6379>
############################################
# MSET, MGET, MSETNX
127.0.0.1:6379> MSET k1 v1 k2 v2 k3 v3  # 同时设置多个值
OK
127.0.0.1:6379> KEYS *
1) "k3"
2) "k1"
3) "k2"
127.0.0.1:6379> MGET k1 k2 k3  # 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> MSETNX k1 v1 k4 v4  # MSETNX 是一个原子性操作，要么一起成功，要么一起失败
(integer) 0
127.0.0.1:6379> GET k4
(nil)

# 对象
SET user:1 {name:zhangsan, age:3}  # 设置一个 user:1 对象，值为 json 字符串来保存一个对象
# 这里的 key 是一个巧妙的设计：user:{id}:{field}
127.0.0.1:6379> MSET user:1:name zhangsan user:1:age 2
OK
127.0.0.1:6379> MGET user:1:name user:1:age
1) "zhangsan"
2) "2"
############################################
# GETSET 先 GET 然后再 SET
127.0.0.1:6379> GETSET db redis  # 如果不存在值，则返回 nil
(nil)
127.0.0.1:6379> GET db
"redis"
127.0.0.1:6379> GETSET db mongodb  # 如果存在值，就获取原来的值，并设置新的值
"redis"
127.0.0.1:6379> get db
"mongodb"
```

String 类似的使用场景：value 除了是我们的字符串，还可以是我们的数字

- 计数器

- 统计多单位的数量

  > uid:9525678:follow 0

- 粉丝数

- 对象缓存存储