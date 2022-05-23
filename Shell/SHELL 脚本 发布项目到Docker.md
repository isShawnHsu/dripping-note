# SHELL 脚本 发布项目到Docker

```shell
#! /bin/bash

##author:shawn
#
#
#
# 记得给文件加权限  chmod 777 ./greens-shopping-api.sh 
#
#
#
#
echo "当前执行文件......$0"
##################################################################
#名称
DOCKER_CONTAINER_NAME="greens-shopping-api"
#版本
DOCKER_CONTAINER_VERSION="${DOCKER_CONTAINER_NAME}:v1"
#端口
DOCKER_PORT="8080:8080"

function buildGreensShoppingApiImage(){

    #停止服务
    stopDockerService

    echo "开始构建镜像$DOCKER_CONTAINER_VERSION..."
    docker build -t $DOCKER_CONTAINER_VERSION ./

    echo "开始启动镜像..."
    docker run -it -d --name $DOCKER_CONTAINER_NAME -p ${DOCKER_PORT} ${DOCKER_CONTAINER_VERSION} 

    echo "开始日志输出..."
    DOCKER_CONTAINER_ID=$(docker ps -aqf "name=$DOCKER_CONTAINER_NAME")
    echo "容器ID: ${DOCKER_CONTAINER_ID} ..."
    docker logs -f ${DOCKER_CONTAINER_ID}
}


#停止服务
function stopDockerService(){
     # 停止容器
     DOCKER_CONTAINER_ID=$(docker ps -aqf "name=$DOCKER_CONTAINER_NAME")
     echo "开始停止容器${DOCKER_CONTAINER_ID}..........."
     if [[ -n "${DOCKER_CONTAINER_ID}" ]]; then
            docker stop ${DOCKER_CONTAINER_ID}
        else
            echo "容器$DOCKER_CONTAINER_NAME不存在"
     fi
     # 删除容器
     echo "开始删除容器$DOCKER_CONTAINER_ID..........."
     if [[ -n "${DOCKER_CONTAINER_ID}" ]]; then
            docker rm $DOCKER_CONTAINER_ID
        else
            echo "容器$DOCKER_CONTAINER_NAME不存在"
     fi
     # 删除镜像
    IMAGE_ID=$(docker images -q --filter reference=$DOCKER_CONTAINER_NAME)
    echo "开始删除镜像$IMAGE_ID..........."
    if [[ -n "${IMAGE_ID}" ]]; then
            docker rmi $IMAGE_ID
        else
            echo "镜像$DOCKER_CONTAINER_NAME不存在"
     fi
}

#执行构建 function 要写在前面
buildGreensShoppingApiImage
```

