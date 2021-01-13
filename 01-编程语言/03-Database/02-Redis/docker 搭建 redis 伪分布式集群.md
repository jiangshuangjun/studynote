# docker 搭建 redis 伪分布式集群

## 建议阅读方式

可前往语雀阅读，体验更好：[docker 搭建 redis 伪分布式集群](https://www.yuque.com/jiangshuangjun-upt1l/xve9g7/hskrkx)

## 背景介绍

该实验主要来源于《Docker 容器与容器云 第2版》一书的 2.3 节：“搭建你的第一个 Docker 应用栈”中的一小步，搭建一个 redis 的一主双从的伪分布式集群



但书中的示例，在配置主、从 redis 实例的配置文件 redis.conf 时，漏掉了一个参数：bind 0.0.0.0，该参数在配置文件中的默认值为：bind 127.0.0.1，导致按书中步骤使用 docker 搭建 redis 的伪分布式集群时，主从之间无法同步数据



本文将按照，我优化过的实验步骤，介绍如何使用 docker 搭建 redis 伪分布式集群

## 环境说明

阿里云 centos 云主机一台：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610546786614-9ef12cac-36fa-4164-a233-13867e049b1d.png)

docker 相关信息：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610546915411-fe5bdd55-6e08-46e1-8d44-38ed4762a8c0.png)

## 搭建架构

使用 docker 搭建一主双从的 redis 伪分布式集群，使用 redis 的 `slaveof <master_ip> <master_port>`  命令完成 redis 的主从复制，结构图如下：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610550434614-c864b2c0-1dc7-402f-8953-aa7b1affe680.png)

如果是一个真正的分布式架构集群，还需要处理容器的跨主机通信问题，这里不做介绍

鉴于是在同一个宿主机完成 redis 伪分布式集群的搭建，只需要完成容器互联来实现容器间的通信即可，这里采用 `docker run`  命令的 `--link`  选项来建立容器间的互联关系



介绍下 `--link` 选项：

- 通过 `--link` 选项能够进行容器间安全的相互通信
- 使用格式为 `name:alias` 
- 可在一个 `docker run` 命令中重复使用该参数

> 通过 `--link` 选项，可以**避免容器的 IP 和端口暴露到外网所导致的安全问题**，还可以**防止容器在重启后 IP 地址变化导致的访问失效**
>
> 它的原理类似 DNS 服务器的域名和地址映射，当容器的 IP 地址发生变化时，Docker 将自动维护映射关系中的 IP 地址，在容器内的 `/etc/hosts` 文件中会体现上述映射关系

## 搭建步骤

### 1. 下载 redis 镜像，并下载镜像对应版本的 redis 压缩包

下载最新版 redis 镜像：

```shell
docker pull redis
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610548014803-021f1655-fb83-4841-b81e-03582a384688.png)

找到 redis 镜像对应的 redis.tar.gz 压缩包：

```shell
docker history --no-trunc redis | grep -i tar.gz
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610548108637-74a5db97-26ed-40ae-9999-896a3973fc20.png)

将 redis-6.0.10.tar.gz 下载到指定目录 `/usr/local/jsj` 中：

```shell
mkdir -p /usr/local/jsj && cd /usr/local/jsj && curl -O http://download.redis.io/releases/redis-6.0.10.tar.gz
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610549302271-6d6df59a-d3dd-410c-a980-125ba8677286.png)

解压 redis-6.0.10.tar.gz，得到与 redis 镜像匹配的 redis.conf 配置文件：

```shell
tar xzf redis-6.0.10.tar.gz
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610549266685-02ab196e-b964-4c82-a5e8-59b8141eac42.png)

### 2. 复制两份 redis.conf，作为 master, slave 的配置文件

复制 redis-master.conf：

```shell
cp redis-6.0.10/redis.conf ./redis-master.conf
```

redis-master.conf 修改如下参数：

```shell
daemonize yes
pidfile /var/run/redis.pid
bind 0.0.0.0
```

---

复制 redis-slave.conf（两台 redis）:

```shell
cp redis-6.0.10/redis.conf ./redis-slave.conf
```

redis-slave.conf 修改如下参数：

```shell
daemonize yes
pidfile /var/run/redis.pid
bind 0.0.0.0
slaveof master 6379
```

### 3. 先后分别启动 1 台 redis-master, 2 台 redis-slave 容器

启动 redis-master 容器（单独开一个 terminal 操作）:

```shell
# 启动 redis-master 容器，并进入该容器
docker run -it --name redis-master -v /usr/local/jsj/redis-master.conf:/usr/local/bin/redis.conf redis /bin/bash

# 在容器内，启动 redis-server 进程
/usr/local/bin/redis-server /usr/local/bin/redis.conf
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610551542558-3a135b65-0dd6-4542-ab29-dd57e32ae7ee.png)

启动 redis-slave1 容器（单独开一个 terminal 操作）:

```shell
# 启动 redis-slave1 容器，并进入该容器
docker run -it --name redis-slave1 -v /usr/local/jsj/redis-slave.conf:/usr/local/bin/redis.conf --link redis-master:master redis /bin/bash

# 在容器内，启动 redis-server 进程
/usr/local/bin/redis-server /usr/local/bin/redis.conf
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610551564013-d4c3f772-258f-4732-a62b-f5d22a38c125.png)

启动 redis-slave2 容器（单独开一个 terminal 操作）:

```shell
# 启动 redis-slave1 容器，并进入该容器
docker run -it --name redis-slave2 -v /usr/local/jsj/redis-slave.conf:/usr/local/bin/redis.conf --link redis-master:master redis /bin/bash

# 在容器内，启动 redis-server 进程
/usr/local/bin/redis-server /usr/local/bin/redis.conf
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610551586061-b2cb90f1-ff11-4481-bae4-a2aacd1391d1.png)

### 4. 数据测试，验证主从复制

在 redis-master 容器中，连接上 redis-server，进行数据写入：

```shell
root@14ab3c0d0f9f:/data# redis-cli 
127.0.0.1:6379> set master jsj
OK
127.0.0.1:6379> get master
"jsj"
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.6,port=6379,state=online,offset=372,lag=1
slave1:ip=172.17.0.7,port=6379,state=online,offset=372,lag=1
master_replid:7eb1e8150916a477ff171bd0ab79bdff6bd002fe
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:372
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:372
127.0.0.1:6379> 
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610551836680-4113f09e-2d3b-44be-9027-c3c903e1223d.png)

在两台 redis-slave 容器中，连接上 redis-server，进行数据查询（这里展示 redis-slave1）：

```shell
root@7d830a49430a:/data# redis-cli 
127.0.0.1:6379> get master
"jsj"
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:master
master_port:6379
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:372
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:7eb1e8150916a477ff171bd0ab79bdff6bd002fe
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:372
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:372
127.0.0.1:6379> 
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610551903350-ee7dd455-b76a-47de-a794-22bad22605b4.png)

从数据测试结果得知，一主双从的 redis 伪分布式集群已搭建完毕

## 环境复原

为了方便操作 json 数据，主机上已安装 jq rpm，jq 是一个操作 json 数据非常方便的命令行工具

### 1. 删除 redis-slave1 容器

```shell
# 删除 redis-slave1
a=$(docker inspect redis-slave1 | jq -r '.[].Mounts[1].Source') && echo ${a}
docker rm -f redis-slave1

# 删除 redis-slave1 的宿主机默认 bind 目录
cd ~ && rm -rfv ${a} && unset a
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610553904664-397b09cd-8da3-49f5-a8e6-205d2299a031.png)

### 2. 删除 redis-slave2 容器

```shell
# 删除 redis-slave2
a=$(docker inspect redis-slave2 | jq -r '.[].Mounts[1].Source') && echo ${a}
docker rm -f redis-slave2

# 删除 redis-slave2 的宿主机默认 bind 目录
cd ~ && rm -rfv ${a} && unset a
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610554024838-c14c746f-ce78-4be4-bd66-3f0655c3e1d4.png)

### 3. 删除 redis-master 容器

```shell
# 删除 redis-master
a=$(docker inspect redis-master | jq -r '.[].Mounts[1].Source') && echo ${a}
docker rm -f redis-master

# 删除 redis-master 的宿主机默认 bind 目录
cd ~ && rm -rfv ${a} && unset a
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610554122398-f0e7856c-cea5-4d1a-a29a-00560ad0a1fe.png)

### 4. 删除 redis 资源目录：/usr/local/jsj

```shell
rm -rfv /usr/local/jsj
```

### 5. 删除 redis 镜像

```shell
docker rmi redis
```

