# 单master kubernetes 集群安装(本地测试)
## before start
 - [before begin](01-before-begin.md)
 - [install kubelet kubeadm kubectl](03-install-kublet-kubeadm.md)
## 1. 获取默认初始化参数文件 [kubeadm config文档(kubeadm/v1beta2)](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2)
 - 获取 kubeadm config init 默认参数文件 
    - kubeadm config print init-defaults > kubeadm-init-config.yaml
 - 获取 kubeadm config join 默认参数文件
    - kubeadm config pring join-defaults > kubeadm-join-config.yaml
 - 获取更多命令
    - kubeadm config --help 

## 3. 使用 kubeadm init 初始化集群 
 - - (推荐)kubeadm init --config kubeadm-init-config.yaml
 - [kubeadm init create cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#installing-kubeadm-on-your-hosts)
 - [kubeadm init 命令介绍](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)
 - [kube.init.default.yaml 文档](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2)
    - ```text
        kubeadm init
        网络插件安装需要指定特殊参数
        例如: calico 需要指定 --pod-network-cidr=192.168.0.0/16 (默认 CALICO_IPV4POOL_CIDR)
        
        注意: 也可以指定其他无冲突网段, 例如: 172.20.0.0/16, 与calico.yaml 中 CALICO_IPV4POOL_CIDR 保持一致
        
        --config 和 --pod-network-cidr 如果不能同时存在可以使用命令行方式: 
          kubeadm init \
            --apiserver-advertise-address=0.0.0.0 \
            --apiserver-bind-port 6443 \
            --pod-network-cidr=192.168.0.0/16 \
            --kubernetes-version=v1.17.0
            
        注意:
          命令行参数 --pod-network-cidr=192.168.0.0/16
          等价于配置文件中: 
          networking:
            podSubnet: 192.168.0.0/16
        
        注意:
          镜像pull失败, 可以使用阿里云镜像仓库:
            --image-repository registry.aliyuncs.com/google_containers
       ```
安装成功会提示
 - ```text
    
     ......
    Your Kubernetes master has initialized successfully!
    
    To start using your cluster, you need to run (as a regular user):
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      
    You can now join any number of machines by running the following on each node as root:
      kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx
   ```

## 4. kubectl 环境变量配置
  - 非root:
    - mkdir -p ~/.kube
    - sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
    - sudo chown -R xx:xx ~/.kube/config
  - root:
    - export KUBECONFIG=/etc/kubernetes/admin.conf

## 5. node 加入集群
> * 在需要加入集群的node执行: kubeadm join xxx --token xxx --discovery-token-ca-cert-hash sha256:xxx

## 6. 遇到的问题
### 6.1 kubelet 版本与配置版本不一致
   - ```text
        [ERROR KubeletVersion]: the kubelet version is higher than the control plane version. This is not a supported version skew and may lead to a malfunctional cluster. Kubelet version: "1.15.0" Control plane version: "1.14.0"
     ```
> * 解决方式1: 修改kubeadm-config.yaml文件 kubernetesVersion: v1.15.0
> * 解决方式2: kubeadm init 参数 --kubernetes-version=v1.15.0

### 6.2 无法获取国外kube镜像
 - 1.手动阿里云获取镜像
 - ```bash
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.0
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.0
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.0
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1
    
    docker images
    ```
 - 2.docker tag
  - ```bash
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.0           k8s.gcr.io/kube-apiserver:v1.15.0
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0  k8s.gcr.io/kube-controller-manager:v1.15.0
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.0           k8s.gcr.io/kube-scheduler:v1.15.0
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.15.0               k8s.gcr.io/kube-proxy:v1.15.0
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1                        k8s.gcr.io/pause:3.1
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10                      k8s.gcr.io/etcd:3.3.10
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1                    k8s.gcr.io/coredns:1.3.1
    ```

## 7. 附录: kubeadm config init
 - kubeadm-init-config.yaml
 - ```yaml
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: InitConfiguration
    bootstrapTokens:
    - token: "9a08jv.c0izixklcxtmnze7"
      description: "kubeadm bootstrap token"
      ttl: "24h"
    - token: "783bde.3f89s0fje9f38fhf"
      description: "another bootstrap token"
      usages:
      - authentication
      - signing
      groups:
      - system:bootstrappers:kubeadm:default-node-token
    nodeRegistration:
      name: "ec2-10-100-0-1"
      criSocket: "/var/run/dockershim.sock"
      taints:
      - key: "kubeadmNode"
        value: "master"
        effect: "NoSchedule"
      kubeletExtraArgs:
        cgroup-driver: "cgroupfs"
      ignorePreflightErrors:
      - IsPrivilegedUser
    localAPIEndpoint:
      advertiseAddress: "10.100.0.1"
      bindPort: 6443
    certificateKey: "e6a2eb8581237ab72a4f494f30285ec12a9694d750b9785706a83bfcbbbd2204"
    ---
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    etcd:
      # one of local or external
      local:
        imageRepository: "k8s.gcr.io"
        imageTag: "3.2.24"
        dataDir: "/var/lib/etcd"
        extraArgs:
          listen-client-urls: "http://10.100.0.1:2379"
        serverCertSANs:
        -  "ec2-10-100-0-1.compute-1.amazonaws.com"
        peerCertSANs:
        - "10.100.0.1"
      # external:
        # endpoints:
        # - "10.100.0.1:2379"
        # - "10.100.0.2:2379"
        # caFile: "/etcd/kubernetes/pki/etcd/etcd-ca.crt"
        # certFile: "/etcd/kubernetes/pki/etcd/etcd.crt"
        # keyFile: "/etcd/kubernetes/pki/etcd/etcd.key"
    networking:
      serviceSubnet: "10.96.0.0/12"
      podSubnet: "10.100.0.1/24"
      dnsDomain: "cluster.local"
    kubernetesVersion: "v1.12.0"
    controlPlaneEndpoint: "10.100.0.1:6443"
    apiServer:
      extraArgs:
        authorization-mode: "Node,RBAC"
      extraVolumes:
      - name: "some-volume"
        hostPath: "/etc/some-path"
        mountPath: "/etc/some-pod-path"
        readOnly: false
        pathType: File
      certSANs:
      - "10.100.1.1"
      - "ec2-10-100-0-1.compute-1.amazonaws.com"
      timeoutForControlPlane: 4m0s
    controllerManager:
      extraArgs:
        "node-cidr-mask-size": "20"
      extraVolumes:
      - name: "some-volume"
        hostPath: "/etc/some-path"
        mountPath: "/etc/some-pod-path"
        readOnly: false
        pathType: File
    scheduler:
      extraArgs:
        address: "10.100.0.1"
    extraVolumes:
    - name: "some-volume"
      hostPath: "/etc/some-path"
      mountPath: "/etc/some-pod-path"
      readOnly: false
      pathType: File
    certificatesDir: "/etc/kubernetes/pki"
    imageRepository: "k8s.gcr.io"
    useHyperKubeImage: false
    clusterName: "example-cluster"
    ---
    apiVersion: kubelet.config.k8s.io/v1beta1
    kind: KubeletConfiguration
    # kubelet specific options here
    ---
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    kind: KubeProxyConfiguration
    # kube-proxy specific options here
   ```

## 8. 附录: calico.yaml

 - ```text
   https://docs.projectcalico.org/manifests/calico.yaml
   ```
