# Redis ZSet(有序集合) 详解

在 Set 的基础上，增加了一个值，set k1 v1 —> zset k1 score1 v1

```shell
############################################
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> ZADD myzset 1 one  # 添加一个值
(integer) 1
127.0.0.1:6379> ZADD myzset 2 two 3 three 4 four  # 添加多个值
(integer) 3
127.0.0.1:6379> ZRANGE myzset 0 -1
1) "one"
2) "two"
3) "three"
4) "four"
############################################ 
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> ZADD salary 9000 zhangsan
(integer) 1
127.0.0.1:6379> ZADD salary 5500 lisi
(integer) 1
127.0.0.1:6379> ZADD salary 6700 wangwu
(integer) 1
127.0.0.1:6379> ZADD salary 8200 zhaoliu
(integer) 1
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf
1) "lisi"
2) "wangwu"
3) "zhaoliu"
4) "zhangsan"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores
1) "lisi"
2) "5500"
3) "wangwu"
4) "6700"
5) "zhaoliu"
6) "8200"
7) "zhangsan"
8) "9000"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf 8000 withscores
1) "lisi"
2) "5500"
3) "wangwu"
4) "6700"
############################################
```

