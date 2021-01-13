# 查找 docker 镜像的所有 tag

## 建议阅读方式

可前往语雀阅读，体验更好：[查找 docker 镜像的所有  tag](https://www.yuque.com/jiangshuangjun-upt1l/xve9g7/bn13gw)

## 环境说明

centos7 阿里云主机一台：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610470499334-adddbf8f-8933-4863-81bf-ff3145606a19.png)

docker 相关信息如下：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610471412273-098982b0-d57d-4b9b-9a36-097be41233bc.png)

测试镜像 `hello-world` 的 tags 情况见官网：[docker-hub#hello-world#tags](https://hub.docker.com/_/hello-world?tab=tags&page=1&ordering=last_updated)

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610471716671-2976c9c7-56fd-4184-b7d7-f772de54fe74.png)

`curl` 安装相关信息：

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472039213-306cb3b4-dde2-4c69-9b06-53e19bb68efd.png)

## 查看方式

### 方法一：利用 v1 版 api

命令如下，其中 `hello-world` 为镜像名字：

```shell
curl -L -s https://registry.hub.docker.com/v1/repositories/hello-world/tags | json_reformat | grep -i name | awk '{print $2}' | sed 's/\"//g' | sort -u
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472188199-2b2ec3d1-0532-4cea-a995-46504dbfc01a.png)

封装成 `list_image_tags_v1.sh` ：

```sh
#!/bin/bash

repo_url=https://registry.hub.docker.com/v1/repositories
image_name=$1

curl -L -s ${repo_url}/${image_name}/tags | json_reformat | grep -i name | awk '{print $2}' | sed 's/\"//g' | sort -u
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472342279-93dc4d63-e06c-4263-a823-8a320ab5b2cf.png)

注意：用到了 `json_reformt` 命令，系统中必须安装 `yajl` rpm 包

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472482484-805b51a3-1613-4ea0-a6c1-0ee7f36b7dd0.png)

### 方法二：利用 v2 版 api

命令如下，其中 `hello-world` 为镜像名字：

```shell
curl -L -s 'https://registry.hub.docker.com/v2/repositories/library/hello-world/tags?page_size=1024' | jq '.results[]["name"]' | sed 's/\"//g' | sort -u
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472585765-7a6da84c-1b3b-48a9-8a1f-647e80e430f5.png)

封装成 `list_image_tags_v2.sh` ：

```sh
#!/bin/bash

repo_url=https://registry.hub.docker.com/v2/repositories/library
image_name=$1

curl -L -s ${repo_url}/${image_name}/tags?page_size=1024 | jq '.results[]["name"]' | sed 's/\"//g' | sort -u
```

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472699168-a4e0aa95-fd9d-4edd-9faf-91ac10e41b9a.png)

注意：用到了 `jq` 命令，系统中必须安装 `jq` rpm 包

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610472789819-fcce065c-eddd-43b5-a597-c75cf3d2581d.png)

### 对比两种方法得到的结果

![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610473225059-2a6a5232-352a-4e16-8d26-6e082d00270b.png)

### 其他方法合集

测试镜像： `hello-world` 

- wget v1

  ```shell
  # 写法一
  wget -q https://registry.hub.docker.com/v1/repositories/hello-world/tags -O -  | sed -e 's/[][]//g' -e 's/"//g' -e 's/ //g' | tr '}' '\n'  | awk -F: '{print $3}' | sort -u
  
  # 写法二
  wget -q https://registry.hub.docker.com/v1/repositories/hello-world/tags -O - | tr -d '[]{, ' | tr '}' '\n' | awk -F: '{print $3}' | sed 's/"//g' | sort -u
  ```

  ![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610523617023-22577664-5bcf-4b3e-8b2e-1e5ae91c4ce1.png)

- wget v1 jq（简洁）

  ```shell
  wget -q https://registry.hub.docker.com/v1/repositories/hello-world/tags -O - | jq -r '.[].name' | sort -u
  ```

  ![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610510120727-6ca22563-6eac-480f-bc98-2c3f0ac5006c.png)

- skopeo（简洁，需安装 skopeo rpm 包，该包依赖项比较多）

  > 安装 skopeo rpm 包方法：
  >
  > yum search skopeo
  >
  > yum install -y skopeo

  ```shell
  # --override-os linux 参数只在非 Linux 宿主机上需要，如 MacOS
  skopeo --override-os linux inspect docker://hello-world | jq '.RepoTags' | tr -d '[]", ' | sort -u
  
  # 在 centos 上，上述命令还可以被简写成如下形式
  skopeo inspect docker://hello-world | jq '.RepoTags' | tr -d '[]", ' | sort -u
  ```

  ![](https://cdn.nlark.com/yuque/0/2021/png/489387/1610522924719-b3d76085-e4dc-47d2-a313-d645ff9064ba.png)

## 参考资料

- [How can I list all tags for a Docker image on a remote registry?](https://stackoverflow.com/questions/28320134/how-can-i-list-all-tags-for-a-docker-image-on-a-remote-registry)

- [https://github.com/stedolan/jq](https://github.com/stedolan/jq)

- [https://jqplay.org](https://jqplay.org)

