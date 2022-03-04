# 🏭部署应用到集群中

### 部署应用 YAML 文件

##### 直接命令运行

```
kubectl run testapp --image=ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1
```

##### Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  # 定义容器，可以多个
  containers:
    - name: test-k8s # 容器名字
      image: ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1 # 镜像
```

##### Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  # 部署名字
  name: test-k8s
spec:
  replicas: 2
  # 用来查找关联的 Pod，所有标签都匹配才行
  selector:
    matchLabels:
      app: test-k8s
  # 定义 Pod 相关数据
  template:
    metadata:
      labels:
        app: test-k8s
    spec:
      # 定义容器，可以多个
      containers:
      - name: test-k8s # 容器名字
        image: ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v1 # 镜像
```

##### Deployment 通过 label 关联起来 Pods

![deployment.png](https://sjwx.easydoc.xyz/46901064/files/kwpt8p8o.png)

### 部署应用演示

部署一个 nodejs web 应用，源码地址：[Github](https://github.com/gzyunke/test-k8s)

```
# 部署应用
kubectl apply -f app.yaml
# 查看 deployment
kubectl get deployment
# 查看 pod
kubectl get pod -o wide
# 查看 pod 详情
kubectl describe pod pod-name
# 查看 log
kubectl logs pod-name
# 进入 Pod 容器终端， -c container-name 可以指定进入哪个容器。
kubectl exec -it pod-name -- bash
# 伸缩扩展副本
kubectl scale deployment test-k8s --replicas=5
# 把集群内端口映射到节点
kubectl port-forward pod-name 8090:8080
# 查看历史
kubectl rollout history deployment test-k8s
# 回到上个版本
kubectl rollout undo deployment test-k8s
# 回到指定版本
kubectl rollout undo deployment test-k8s --to-revision=2
# 删除部署
kubectl delete deployment test-k8s
```

##### Pod 报错解决

如果你运行 `kubectl describe pod/pod-name` 发现 Events 中有下面这个错误

```
networkPlugin cni failed to set up pod "test-k8s-68bb74d654-mc6b9_default" network: open /run/flannel/subnet.env: no such file or directory
```

在每个节点创建文件`/run/flannel/subnet.env`写入以下内容即可解决

```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```

##### 更多命令

```
# 查看全部
kubectl get all
# 重新部署
kubectl rollout restart deployment test-k8s
# 命令修改镜像，--record 表示把这个命令记录到操作历史中
kubectl set image deployment test-k8s test-k8s=ccr.ccs.tencentyun.com/k8s-tutorial/test-k8s:v2-with-error --record
# 暂停运行，暂停后，对 deployment 的修改不会立刻生效，恢复后才应用设置
kubectl rollout pause deployment test-k8s
# 恢复
kubectl rollout resume deployment test-k8s
# 输出到文件
kubectl get deployment test-k8s -o yaml >> app2.yaml
# 删除全部资源
kubectl delete all --all
```

更多官网关于 [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) 的介绍

将 Pod 指定到某个节点运行：[nodeselector](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)
限定 CPU、内存总量：[文档](https://kubernetes.io/zh/docs/concepts/policy/resource-quotas/#计算资源配额)

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

### 工作负载分类

- Deployment
  适合无状态应用，所有pod等价，可替代
- StatefulSet
  有状态的应用，适合数据库这种类型。
- DaemonSet
  在每个节点上跑一个 Pod，可以用来做节点监控、节点日志收集等
- Job & CronJob
  Job 用来表达的是一次性的任务，而 CronJob 会根据其时间规划反复运行。

[文档](https://kubernetes.io/zh/docs/concepts/workloads/)