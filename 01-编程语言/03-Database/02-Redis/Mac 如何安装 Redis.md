# Mac 如何安装 Redis

## 1. 下载源码包

去官网 https://redis.io 下载，当前最新版为 6.0.7，安装包为：`redis-6.0.7.tar`

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599149605043-8d97e7f6-d5ef-415e-a0ad-b330b28d6844.png?x-oss-process=image%2Fresize%2Cw_1492)

## 2. 解压 Reids 源码包

```shell
# 1. 先将下载好的 redis-6.0.7.tar 拷贝到合适的目录，示例如下
cd /Users/dengjinhua/xiaolang/software && cp /Users/dengjinhua/Downloads/redis-6.0.7.tar .

# 2. 解压 redis-6.0.7.tar，如下两命令任选其一即可
# 将 redis-6.0.7.tar 解压到当前文件夹，并显示压缩包包含的文件明细
tar xvf redis-6.0.7.tar
# 将 redis-6.0.7.tar 解压到当前文件夹，不显示压缩包包含的文件明细
tar xzf redis-6.0.7.tar
```

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599149890059-2cdc1ffa-5f14-4a0e-9c64-cf7b75755fff.png)

## 3. 进入 Redis 解压目录

进入 Redis 解压目录 `/Users/dengjinhua/xiaolang/software/redis-6.0.7` ，可看到 Reids 的配置文件： `redis.conf` 

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599149953747-12607737-ac72-4311-8f52-37708f8e91ff.png)

## 4. 基本环境安装

需有 gcc 编译工具包，gcc -v 需显示有如下信息（Centos Linux 环境下可执行 yum install gcc-c++ 来安装），Mac 下安装 gcc 工具可参考 [如何在 Mac 上安装 GCC](https://www.zhihu.com/question/20588567)

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599150230194-5818d26e-4b46-47a3-91a9-17a1fa3cb370.png?x-oss-process=image%2Fresize%2Cw_1492)

```shell
# 在 `/Users/dengjinhua/xiaolang/software/redis-6.0.7` 目录下依次执行如下命令
make
make test
make install
```

正常会如下所示：

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599150320700-5da70bd2-1280-4b65-89be-bda92d96cf27.png)

## 5. Redis 默认安装路径： `/usr/local/bin`

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599150417738-a62cf0a6-bf8a-4ae8-9374-1a0bb3b37c1c.png)

## 6. 复制一份 redis.conf 配置文件

为防止覆盖默认的 `redis.conf` 配置文件，在 Redis 解压目录 `/Users/dengjinhua/xiaolang/software/redis-6.0.7` 下，新建一个 `conf` 文件夹，将安装目录下的 `redis.conf` 文件复制一份到 conf 文件夹下，我们后续就使用这个配置文件进行启动

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599150911821-13c25c9f-ea7d-4bcc-b770-5aed8afcff39.png)

## 7. 修改 redis.conf 配置文件，启动方式改为后台启动

Redis 默认不是后台启动的，修改配置文件，将 daemonize 改为 yes

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599150968648-56d6c272-31ad-453f-b595-252c97e2b544.png)

## 8. 启动 Redis 服务

```shell
# 以我们复制好的并修改了 daemonize = yes 的 redis.conf 配置文件后台启动 Redis 服务
/usr/local/bin/redis-server /Users/dengjinhua/xiaolang/software/redis-6.0.7/conf/redis.conf
```

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599151067879-ee7cb0b9-4939-4d5c-8e07-268254127c28.png?x-oss-process=image%2Fresize%2Cw_1492)

## 9. 使用 redis-cli 进行连接测试

```shell
/usr/local/bin/redis-cli -p 6379
```

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599151112378-f9a769ce-60d4-4de4-9902-9dbf63be36c8.png?x-oss-process=image%2Fresize%2Cw_1492)

## 10. 查看 Redis 进程是否开启

```shell
# 单独开启一个窗口执行如下指令查看
ps aux | grep -v grep | grep -i redis
```

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599151195608-7be466f7-61ac-4215-be60-197e222e1953.png?x-oss-process=image%2Fresize%2Cw_1492)

## 11. 如何关闭 Redis 服务： `shutdown`

```shell
# 在之前用 redis-cli 连接上 redis-server 的窗口上依次执行如下操作
shutdown
exit
```

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599151321680-56b4610c-73c8-459d-bf93-c379a1c5c8c7.png)

## 12. 再次查看 Redis 进程是否开启

![](https://cdn.nlark.com/yuque/0/2020/png/489387/1599151374861-bebfd4d9-6f8f-4f53-912f-baf7349914ae.png)



