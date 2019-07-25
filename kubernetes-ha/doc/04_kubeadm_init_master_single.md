# 1. 获取默认初始化参数文件
 > *  kubeadm config print init-defaults > kubeadm-config.yaml

# 2. 修改 kubeadm-config.yaml

 > *  修改镜像仓库 imageRepository: local.xxx.repository
 > *  配置 external etcd
 
 举例
```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
  - groups:
      - system:bootstrappers:kubeadm:default-node-token
    token: abcdef.0123456789abcdef
    ttl: 24h0m0s
    usages:
      - signing
      - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 0.0.0.0
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: h1-k8s-local
  taints:
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  external:
    endpoints:
      - https://h1-k8s-local:2379
      - https://h2-k8s-local:2379
      - https://h3-k8s-local:2379
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/aipserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 192.168.0.0/16
scheduler: {}
```

# 3. kubeadm init master
> * kubeadm init --config=kubeadm-config.yaml
```
网络插件安装需要指定特殊参数
例如 calico 需要指定 --pod-network-cidr=192.168.0.0/16

--config 和 --pod-network-cidr不能同时存在
所以需要使用命令行方式: 
kubeadm init \
  --apiserver-advertise-address=0.0.0.0 \
  --apiserver-bind-port 6443 \
  --pod-network-cidr=192.168.0.0/16 \
  --service-cluster-ip-range=10.96.0.0/12 \
  --kubernetes-version=v1.15.0
```
--cluster-cidr=192.168.0.0/16
```
安装成功会提示
kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx
```
> * mkdir $HOME/.kube
> * sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> * sudo chown currentUser:currentUserGroup  $HOME/.kube/config

# 4. node 加入集群
> * 在需要加入集群的node执行: kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx

# 5. 遇到的问题
## 5.1 kubelet 版本与配置版本不一致
```
[ERROR KubeletVersion]: the kubelet version is higher than the control plane version. This is not a supported version skew and may lead to a malfunctional cluster. Kubelet version: "1.15.0" Control plane version: "1.14.0"
```
> * 解决方式1: 修改kubeadm-config.yaml文件 kubernetesVersion: v1.15.0
> * 解决方式2: kubeadm init 参数 --kubernetes-version=v1.15.0