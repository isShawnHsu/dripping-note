# Linux [Docker](https://docs.docker.com/engine/install/centos/)安装

## 一、安装

#### 1、设置仓库

安装 `yum-utils`包（其中包含 `yum-config-manager`）并设置稳定版仓库

```shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
    
    #使用阿里云镜像源
    
    
    
    #https://download.docker.com/linux/centos/docker-ce.repo
```

#### 2、安装 `Docker Engine`

安装最新版本的`Docker Engine` 和`containerd`

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

####  3、启动镜像

```shell
sudo systemctl start docker
```

#### 4、卸载

```shell
sudo yum remove docker-ce docker-ce-cli containerd.io
```

#### 5、删除镜像及容器

```shell
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

#### 6、基础命令

```shell
docker ps -a #显示所有contianer列表
docker rm <CONTAINER ID> #删除contianer
docker images #显示所有image列表
docker rmi <IMAGE ID> #删除image
docker pull <软件名> #下载镜像
docker exec -it <CONTAINER ID> bash #进入容器
docker logs -f <CONTAINER ID> #查看日志
```



## 二、[mysql](https://hub.docker.com/_/mysql) 安装

#### 1、下载[mysql](https://hub.docker.com/_/mysql)镜像

```shell
docker pull mysql
```

#### 2、启动镜像

```
docker run -itd --restart=always --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
3306:3306 外部端口:内部端口
```

## 三、项目部署

#### 1、Dockerfile

```shell
# 该镜像需要依赖的基础镜像
FROM java:8
# 将当前目录下的jar包复制到docker容器的/目录下
COPY hsu-wxa-0.0.1-SNAPSHOT.jar app.jar
# 指定docker容器启动时运行jar包
ENTRYPOINT [ "sh", "-c", "java -Djava.security.egd=file:/dev/./urandom -jar -Duser.timezone=GMT+8 -Dspring.profiles.active=prod /app.jar" ]
# 指定维护者的名字
MAINTAINER shawn
```

#### 2、把`jar`包和`Dockerfile` 放在同一级目录下

#### 3、构建镜像

```docker build -t wxa:v1 ./```

#### 4、运行容器

```docker run  -it -d  -p 5555:5556 wxa:v1```

## 四、目录挂载

### 1、挂载方式

- `bind mount` 直接把宿主机目录映射到容器内，适合挂代码目录和配置文件。可挂到多个容器上
- `volume` 由容器创建和管理，创建在宿主机，所以删除容器不会丢失，官方推荐，更高效，Linux 文件系统，适合存储数据库数据。可挂到多个容器上
- `tmpfs mount` 适合存储临时文件，存宿主机内存中。不可多容器共享。

### 2、挂载演示

```
bind mount` 方式用绝对路径 `-v D:/code:/app
volume` 方式，只需要一个名字 `-v db-data:/app
```

示例：
`docker run -p 8080:8080 --name test-hello -v D:/code:/app -d test:v1`

## 五、多容器通信

### 1、演示

创建一个名为`test-net`的网络：

```
docker network create test-net
```

运行 Redis 在 `test-net` 网络中，别名`redis`

```
docker run -d --name redis --network test-net --network-alias redis redis:latest
```

修改代码中访问`redis`的地址为网络别名

```java
Config config = new Config();
SingleServerConfig singleServerConfig = config.useSingleServer()
        .setAddress("redis://" + redis:6379)
```

## 六、docker-compose

```如果项目依赖更多的第三方软件，我们需要管理的容器就更加多，每个都要单独配置运行，指定网络。
我们使用 docker-compose 把项目的多个服务集合到一起，一键运行。
```

### 1、 安装 Docker Compose

- 如果你是安装的桌面版 Docker，不需要额外安装，已经包含了。

- 如果是没图形界面的服务器版 Docker，你需要单独安装 [安装文档](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)

- 运行`docker-compose`检查是否安装成功

  

### 2、编写脚本

要把项目依赖的多个服务集合到一起，我们需要编写一个`docker-compose.yml`文件，描述依赖哪些服务
参考文档：https://docs.docker.com/compose/

```
version: "3.7"

services:
  app:
    build: ./
    ports:
      - 80:8080
    volumes:
      - ./:/app
    environment:
      - TZ=Asia/Shanghai
  redis:
    image: redis:5.0.13
    volumes:
      - redis:/data
    environment:
      - TZ=Asia/Shanghai

volumes:
  redis:
```

### 3、基础命令

在`docker-compose.yml` 文件所在目录，执行：`docker-compose up`就可以跑起来了。
命令参考：https://docs.docker.com/compose/reference/up/

在后台运行只需要加一个 -d 参数`docker-compose up -d`
查看运行状态：`docker-compose ps`
停止运行：`docker-compose stop`
重启：`docker-compose restart`
重启单个服务：`docker-compose restart service-name`
进入容器命令行：`docker-compose exec service-name sh`
查看容器运行log：`docker-compose logs [service-name]`

## 七、备份和迁移数据

### 1、 备份和导入 Volume 的流程

备份：

- 运行一个 ubuntu 的容器，挂载需要备份的 volume 到容器，并且挂载宿主机目录到容器里的备份目录。
- 运行 tar 命令把数据压缩为一个文件
- 把备份文件复制到需要导入的机器

导入：

- 运行 ubuntu 容器，挂载容器的 volume，并且挂载宿主机备份文件所在目录到容器里
- 运行 tar 命令解压备份文件到指定目录

### 2、备份 MongoDB 数据演示

- 运行一个 mongodb，创建一个名叫`mongo-data`的 volume 指向容器的 /data 目录
  `docker run -p 27018:27017 --name mongo -v mongo-data:/data -d mongo:4.4`
- 运行一个 Ubuntu 的容器，挂载`mongo`容器的所有 volume，映射宿主机的 backup 目录到容器里面的 /backup 目录，然后运行 tar 命令把数据压缩打包
  `docker run --rm --volumes-from mongo -v d:/backup:/backup ubuntu tar cvf /backup/backup.tar /data/`

最后你就可以拿着这个 backup.tar 文件去其他地方导入了。

### 3、恢复 Volume 数据演示

- 运行一个 ubuntu 容器，挂载 mongo 容器的所有 volumes，然后读取 /backup 目录中的备份文件，解压到 /data/ 目录
  `docker run --rm --volumes-from mongo -v d:/backup:/backup ubuntu bash -c "cd /data/ && tar xvf /backup/backup.tar --strip 1"`

> 
