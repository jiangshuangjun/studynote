# docker 常用命令

- 查看容器内部 ip

  ```shell
  # 方法一：通过 docker inspect
  [root@k8s-node-01 ~]
  $ docker inspect --format='{{.NetworkSettings.IPAddress}}' 51514162a76b
  172.17.0.4
  
  # 方法二：通过 docker exec -it
  [root@k8s-node-01 ~]
  $ docker exec -it 51514162a76b hostname -i
  172.17.0.4
  ```

  