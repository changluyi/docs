# Cluster Inter-Connection with Submariner

[Submariner](https://submariner.io/) 作为可以打通多个 Kubernetes 集群 Pod 和 Service 网络的开源网络组件，能够帮助
 Kube-OVN 实现多集群互联。

相比通过 [OVN-IC](./with-ovn-ic.md) 打通多集群网络的方式，Submariner 可以打通 Kube-OVN 和非 Kube-OVN 的集群网络，并
能提供 Service 的跨集群能力。但是 Submariner 目前只能实现默认子网的打通，无法实现多子网选择性打通。

## 前提条件
1. 两个集群的 Service CIDR 和默认子网的 CIDR 不能重叠。

## 部署 Submariner

下载 `subctl` 二进制文件，并部署到相应路径：

```bash
curl -Ls https://get.submariner.io | bash
export PATH=$PATH:~/.local/bin
echo export PATH=\$PATH:~/.local/bin >> ~/.profile
```

切换 `kubeconfig` 至希望部署 `submariner-broker` 的集群进行部署：

```bash
subctl deploy-broker
```

在本文档中 `cluster0` 的默认子网 CIDR 为 `10.16.0.0/16`，`cluster1` 的默认子网 CIDR 为 `11.16.0.0/16`。

切换 `kubeconfig` 至 `cluster0` 注册集群至 broker，并注册网关节点:

```bash
subctl  join broker-info.subm --clusterid  cluster0 --clustercidr 10.16.0.0/16  --natt=false --cable-driver vxlan --health-check=false
kubectl label nodes cluster0 submariner.io/gateway=true
```

切换 `kubeconfig` 至 `cluster1` 注册集群至 broker，并注册网关节点:

```bash
subctl  join broker-info.subm --clusterid  cluster1 --clustercidr 11.16.0.0/16  --natt=false --cable-driver vxlan --health-check=false
kubectl label nodes cluster1 submariner.io/gateway=true
```

接下来可以在两个集群内分别启动 Pod 并尝试使用 IP 进行相互访问。

如果出现网络互通问题可通过 `subctl` 命令进行诊断：

```bash
subctl show all
subctl diagnose all
```

更多 Submariner 相关操作请查看 [Submariner 用户手册](https://submariner.io/operations/usage/)。