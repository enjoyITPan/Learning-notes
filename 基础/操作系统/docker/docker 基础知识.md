## 容器使用

### 获取镜像

```dockerfile
docker pull 镜像名:tag
例如:docker pull nginx:latest
```

### 删除镜像

```
删除单个：docker rmi [image]
批量删除：docker  image   rm   $(docker  image  ls   -a  -q)
```



### 启动容器

```
 docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
 例如：docker run -it nginx /bin/bash
```

OPTIONS参数说明：

- **-d**：后台运行，并返回容器id
- **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
- **--name="nginx-lb":** 为容器指定一个名称；
- **`--rm`**：停止运行后，自动删除容器文件。
- **`--volume`**： 格式：本地目录:容器目录，例如："$PWD/":/var/www/html：将当前目录（`$PWD`）映射到容器的`/var/www/html`（Apache 对外访问的默认目录）。因此，当前目录的任何修改，都会反映到容器里面，进而被外部访问到。

- **-i**: 交互式操作，在容器内执行命令
- **-t**: 终端，进入容器终端
- **nginx**: nginx 镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

要退出终端，直接输入 **exit**

### 查看所有的容器

```
$ docker ps -a
```

### 启动一个已停止的容器：

```
docker start 容器id 
```

### 停止一个容器

```
docker stop <容器 ID>
```

### 停止的容器可以通过 docker restart 重启：

```
$ docker restart <容器 ID>
```

### 进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐大家使用 docker exec 命令，因为后续退出容器终端，不会导致容器的停止。

**exec 命令**

下面演示了使用 docker exec 命令。

```
docker exec -it 243c32535da7 /bin/bash
```

### 删除容器

```
$ docker rm -f 容器id
```

下面的命令可以清理掉所有处于终止状态的容器。

```
docker container prune
```

### 查看端口映射

```
docker port 容器ID
```

### 查看容器日志

```
docker logs 容器ID
```

### 查看容器内的进程

```
docker top 容器ID
```

