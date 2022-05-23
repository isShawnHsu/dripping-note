

# Dockerfile 及构建

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

## 