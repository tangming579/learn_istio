# istio多集群安装

## 主从架构安装

<div>
    <image src="img/arch.svg" width="700"></image>
</div>

- 左侧cluster1为Primary：
  1. 其中的istiod为整个网格的控制面，可以连接两两个集群 API Server 监听两个集群的k8s资源
  2. 通过eastwest gateway暴露出istiod和业务的服务，供其他网络上的服务来连接xds和业务服务
- 右侧cluster2为Remote：
  1. 没有pilot（实际上也有istiod，不过只用作injector和citadel，不使用其pilot能力）
  2. `cluster2` 中的服务将通过专用的 EastWestGateway 流量访问 `cluster1` 的控制平面。
  3. api-server 必须提供一个可供Primary上控制面访问的地址

### 将 `cluster1` 设为主集群

为 `cluster1` 创建 Istio 配置文件：

```
$ cat <<EOF > cluster1.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster1
      network: network1
EOF
```



将配置文件应用到 `cluster1`：

```
$ istioctl install --set values.pilot.env.EXTERNAL_ISTIOD=true --context="${CTX_CLUSTER1}" -f cluster1.yaml
```



需要注意的是，当 `values.pilot.env.EXTERNAL_ISTIOD` 被设置为 `true` 时， 安装在 `cluster1` 上的控制平面也可以作为其他从集群的外部控制平面。 当这个功能被启用时，`istiod` 将试图获得领导权锁，并因此管理将附加到它的并且带有 [适当注解的](https://istio.io/latest/zh/docs/setup/install/multicluster/primary-remote/#set-the-control-plane-cluster-for-cluster2)从集群 （本例中为 `cluster2`）。