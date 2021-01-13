# Redis Set 详解

Set 中的值是不能重复的

```shell
############################################
# 添加、移除元素，判断某元素是否在 set 中
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD myset one  # set 集合中添加元素
(integer) 1
127.0.0.1:6379> SADD myset two
(integer) 1
127.0.0.1:6379> SADD myset three
(integer) 1
127.0.0.1:6379> SMEMBERS myset  # 查看指定 set 的所有值
1) "one"
2) "three"
3) "two"
127.0.0.1:6379> SISMEMBER myset one  # 判断某一个值是不是在 set 集合中
(integer) 1
127.0.0.1:6379> SISMEMBER myset four
(integer) 0
127.0.0.1:6379> SCARD myset  # 获取 set 集合中的元素个数
(integer) 3
127.0.0.1:6379> SREM myset one  # 移除掉 set 集合中的指定值的元素
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "three"
2) "two"
############################################
# SRANDMEMBER 随机抽选 set 中的元素
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD myset one  # set 集合中添加元素
(integer) 1
127.0.0.1:6379> SADD myset two
(integer) 1
127.0.0.1:6379> SADD myset three
(integer) 1
127.0.0.1:6379> SADD myset four
(integer) 1
127.0.0.1:6379> SRANDMEMBER myset  # 随机抽选出一个元素
"four"
127.0.0.1:6379> SRANDMEMBER myset
"two"
127.0.0.1:6379> SRANDMEMBER myset 2  # 随机抽选出指定个数的元素
1) "one"
2) "three"
127.0.0.1:6379> SRANDMEMBER myset 2
1) "two"
2) "three"
############################################
# 删除指定的 key，随机删除 key
127.0.0.1:6379> SMEMBERS myset
1) "one"
2) "four"
3) "three"
4) "two"
127.0.0.1:6379> SPOP myset  # 随机删除 set 中的某个元素
"three"
127.0.0.1:6379> SMEMBERS myset
1) "one"
2) "four"
3) "two"
############################################
# 将一个指定的值，移动到另一个 set 集合中
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD myset one
(integer) 1
127.0.0.1:6379> SADD myset two
(integer) 1
127.0.0.1:6379> SADD myset three
(integer) 1
127.0.0.1:6379> SMOVE myset myset2 one
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "three"
2) "two"
127.0.0.1:6379> SMEMBERS myset2
1) "one"
############################################
# 微博，B 站，共同关注（并集）
# 微博，A 用户将所有关注的人放在一个 set 集合中，将它的粉丝也放在一个集合中，可以做共同关注、共同爱好、二度好友、推荐好友等功能
# 数字集合类
# - 差集 SDIFF
# - 交集 SINTER
# - 并集 SUNION
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> SADD set1 a
(integer) 1
127.0.0.1:6379> SADD set1 b
(integer) 1
127.0.0.1:6379> SADD set1 c
(integer) 1
127.0.0.1:6379> SADD set2 c
(integer) 1
127.0.0.1:6379> SADD set2 d
(integer) 1
127.0.0.1:6379> SADD set2 e
(integer) 1
127.0.0.1:6379> SDIFF set1 set2  # 差集
1) "a"
2) "b"
127.0.0.1:6379> SINTER set1 set2  # 交集，共同好友就可以这样实现
1) "c"
127.0.0.1:6379> SUNION set1 set2  # 并集
1) "a"
2) "b"
3) "c"
4) "e"
5) "d"
############################################
```

