# Redis List 列表详解

基本的数据类型，列表

在 Redis 里面，我们可以把 List 玩成：栈、队列、阻塞队列

所有的 List 命令都是 L 开头的，Redis 命令不区分大小写

```shell
############################################
127.0.0.1:6379> LPUSH list one  # 将一个或多个值，插入到列表头部（左）
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1  # 获取 list 中的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> LRANGE list 0 1  # 通过区间获取具体的值，栈的获取方式（先进后出）
1) "three"
2) "two"
127.0.0.1:6379> RPUSH list right  # 将一个或多个值，插入到列表尾部（右）
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
############################################
# LPOP, RPOP
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
127.0.0.1:6379> LPOP list  # 移除 list 第一个元素
"three"
127.0.0.1:6379> RPOP list  # 移除 list 最后一个元素
"right"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
############################################
# LINDEX 通过下表获取列表中的值
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LINDEX list 0  # 通过下标获得 list 中的某一个值
"two"
127.0.0.1:6379> LINDEX list 1
"one"
############################################
# LLEN 获取列表的长度（元素个数）
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LLEN list  # 返回列表的长度
(integer) 3
############################################
# 移除指定的值：取关 uid
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> LREM list 1 one  # 移除 list 集合中指定个数的 value
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
127.0.0.1:6379> LREM list 1 three
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LREM list 2 three
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
############################################
# LTRIM 截断 list
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> RPUSH mylist hello
(integer) 1
127.0.0.1:6379> RPUSH mylist hello1
(integer) 2
127.0.0.1:6379> RPUSH mylist hello2
(integer) 3
127.0.0.1:6379> RPUSH mylist hello3
(integer) 4
127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "hello1"
3) "hello2"
4) "hello3"
127.0.0.1:6379> LTRIM mylist 1 2  # 通过下标，截取 mylist 指定的长度，这个 mylist 已经被改变了，截断了只剩下截取的元素
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello1"
2) "hello2"
############################################
# rpoplpush 移除列表的最后一个元素，并将它移动到新的列表中
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> RPUSH mylist "hello"
(integer) 1
127.0.0.1:6379> RPUSH mylist "hello1"
(integer) 2
127.0.0.1:6379> RPUSH mylist "hello2"
(integer) 3
127.0.0.1:6379> RPOPLPUSH mylist myotherlist
"hello2"
127.0.0.1:6379> LRANGE mylist 0 -1  # 查看原来的列表
1) "hello"
2) "hello1"
127.0.0.1:6379> LRANGE myotherlist 0 -1  # 查看目标列表中，确实存在
1) "hello2"
############################################
# lset 将列表中指定下标的值替换为另外一个值，更新操作
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> EXISTS list  # 判断这个列表是否存在，存在返回 1， 不存在返回 0
(integer) 0
127.0.0.1:6379> LSET list 0 item  # 如果不存在列表，去更新就会报错
(error) ERR no such key
127.0.0.1:6379> LPUSH list value1
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "value1"
127.0.0.1:6379> LSET list 0 item  # 如果存在，更新当前下标的值
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "item"
127.0.0.1:6379> LSET list 1 other  # 如果当前下标不存在值，更新则会报错
(error) ERR index out of range
############################################
# linsert 将某个具体的 value 插入到列表中某个元素的前面或后面
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> RPUSH mylist hello
(integer) 1
127.0.0.1:6379> RPUSH mylist world
(integer) 2
127.0.0.1:6379> LINSERT mylist before hello say
(integer) 3
127.0.0.1:6379> LRANGE mylist 0 -1
1) "say"
2) "hello"
3) "world"
127.0.0.1:6379> LINSERT mylist after world haha
(integer) 4
127.0.0.1:6379> LRANGE mylist 0 -1
1) "say"
2) "hello"
3) "world"
4) "haha
```

小结：

- List 实际上是一个链表，before、after、left、right 都可以插入值
- 如果 key 不存在，则创建新的链表
- 如果 key 存在，就新增内容
- 如果移除了所有值，空链表，也代表不存在
- 在两边插入或者改动值，效率最高！中间元素，相对来说效率会低一点

消息队列（LPUSH RPOP）、栈（LPUSH LPOP）