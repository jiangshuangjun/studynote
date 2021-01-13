# Redis Hash(哈希)详解

Map 集合，key-map 时，这个值是一个 map 集合！本质和 String 类型没有太大区别，还是一个简单的 key-value

```shell
############################################
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> HSET myhash field1 value1  # set 一个具体的 key-value
(integer) 1
127.0.0.1:6379> HGET myhash field1  # 获取一个字段的值
"value1"
127.0.0.1:6379> HMSET myhash field1 hello field2 world  # set 多个 key-value
OK
127.0.0.1:6379> HMGET myhash field1 field2  # 获取多个字段的值
1) "hello"
2) "world"
127.0.0.1:6379> HGETALL myhash  # 获取全部的值
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> HDEL myhash field1  # 删除 hash 指定的 key，对应的 value 也一并删除
(integer) 1
127.0.0.1:6379> HGETALL myhash
1) "field2"
2) "world"
############################################
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> HMSET myhash k1 v1 k2 v2 k3 v3 k4 v4
OK
127.0.0.1:6379> HGETALL myhash
1) "k1"
2) "v1"
3) "k2"
4) "v2"
5) "k3"
6) "v3"
7) "k4"
8) "v4"
127.0.0.1:6379> HLEN myhash  # 获取 hash 表的字段数量
(integer) 4
127.0.0.1:6379> HEXISTS myhash k1  #  判断 hash 中指定字段是否存在，返回 1 表示存在，返回 0 表示不存在
(integer) 1
127.0.0.1:6379> HEXISTS myhash k5
(integer) 0
127.0.0.1:6379> HKEYS myhash  # 获取 hash 表中的 key
1) "k1"
2) "k2"
3) "k3"
4) "k4"
127.0.0.1:6379> HVALS myhash  # 获取 hash 表中的 value
1) "v1"
2) "v2"
3) "v3"
4) "v4"
############################################
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> HSET myhash k1 5
(integer) 1
127.0.0.1:6379> HGET myhash k1
"5"
127.0.0.1:6379> HINCRBY myhash k1 5  # 指定增量
(integer) 10
127.0.0.1:6379> HINCRBY myhash k1 -3
(integer) 7
127.0.0.1:6379> HSETNX myhash k2 hello  # 如果 key 不存在，就可以设置成功，返回 1 表示成功
(integer) 1
127.0.0.1:6379> HGETALL myhash
1) "k1"
2) "7"
3) "k2"
4) "hello"
127.0.0.1:6379> HSETNX myhash k1 world  # 如果 key 存在，就不可以设置，返回 0 表示设置失败
(integer) 0
127.0.0.1:6379> HGETALL myhash
1) "k1"
2) "7"
3) "k2"
4) "hello"
############################################
```

hash 可以用来存变更的数据，尤其是用户信息之类的，经常变动的信息，hash 更适合对象的存储，String 更适合字符串的存储，示例如下：

```shell
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> HSET user:1 name xiaolang age 23 sex man
(integer) 3
127.0.0.1:6379> HKEYS user:1
1) "name"
2) "age"
3) "sex"
127.0.0.1:6379> HVALS user:1
1) "xiaolang"
2) "23"
3) "man"
```



