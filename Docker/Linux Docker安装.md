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
docker run -itd --restart=always --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
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

