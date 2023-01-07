## 概念

官网：https://istio.io/latest/zh/docs

Istio的核心思想就是将服务治理的功能从业务服务中独立出来，作为一个sidecar容器，解耦的同时也能够兼容不同语言，无需和业务服务使用同一套语言。公共的治理能力独立后，所有组件都可以接入，而且采用sidecar的方式让业务无需任何修改即可接入。

Istio主要提供四个特性：

- 流量管理：在实现服务连接的基础上，通过控制服务间的流量和调用，实现请求路由、负载均衡、超时、重试、熔断、故障注入、重定向等功能
- 安全：提供证书配置管理，以及服务访问认证、授权等安全能力
- 策略控制：提供访问速率限制能力。
- 观测：获取服务运行指标和输出，提供调用链追踪和日志收集能力。

## 安装

### 下载安装包

手动安装：

```
wget https://github.com/istio/istio/releases/download/1.13.4/istio-1.13.4-linux-amd64.tar.gz
tar -xf istio-1.13.4-linux-amd64.tar.gz 
cd istio-1.13.4/
cp bin/istioctl /usr/local/bin/
```

脚本安装：

```
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.16.1 TARGET_ARCH=x86_64 sh -
cd istio-1.16.1
export PATH=$PWD/bin:$PATH
```

### 通过 istioctl 安装 istio

1. 安装 demo 配置的 istio

```
$ istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete
```

2. 给命名空间添加标签，指示 Istio 在部署应用的时候，自动注入 Envoy SideCar 代理：

```
kubectl label namespace default istio-injection=enabled
```

- 错误提示：Istiod encountered an error: failed to wait for resource: resources not ready

  如果k8s是只有master节点的话，要开启允许master节点调度

  ```
  # 查看node
  kubectl get nodes 
  
  # 查看污点
  kubectl describe node k8s-master |grep Taints
  Taints:    node-role.kubernetes.io/master:NoSchedule
  
  # 删除污点
  kubectl taint nodes --all node-role.kubernetes.io/master
  
  # 让master节点参与调度，#如果想删除，把=换成-
  kubectl label nodes k8s-master node-role.kubernetes.io/worker=
  ```

  