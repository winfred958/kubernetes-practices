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
  token: abcdef.0123456789abcdefwinfred
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
  name: h1.k8s.local
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
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.14.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

# 3. kubeadm init master
> * kubeadm init --config=kubeadm-config.yaml --pod-network-cidr=192.168.0.0/16
```
网络插件安装需要指定特殊参数
例如 calico 需要指定 --pod-network-cidr
kubeadm init --config=kubeadm-config.yaml --pod-network-cidr=192.168.0.0/16
```
```
安装成功会提示
kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx
```
> * mkdir $HOME/.kube
> * sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> * sudo chown currentUser:currentUserGroup  $HOME/.kube/config

# 4. node 加入集群
> * 在需要加入集群的node执行: kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx