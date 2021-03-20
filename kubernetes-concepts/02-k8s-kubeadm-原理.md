# kubeadm 详细流程

## 1. kubeadm init 的工作流程

1. 首先会进行一系列检查, 用来确认node是否可以用来部署 kublet. 这一步检查我们称为 --- Preflight Checks
    - Linux 内核版本检查, 是否 3.10+
    - Linux Cgroup 模块是否可用?
    - hostname是否标准? 必须使用标准DNS命名(RFC 1123)
    - 用户安装的kubeadm和kubelet版本是否匹配?
    - kubernetes 的工作端口 10250/10251/10252 端口是否已经被占用?
    - ip, mount 等 Linux 指令是否存在?
    - Container Runtime 是否已经准备好?
2. 通过Preflight Checks后, kubeadm要做的, 是生成Kubernetes对外提供服务所需的各种证书和对应的目录.
    - 除非专门开启不安全模式, 否则都要通过HTTPS功能访问kube-apiserver, 这就需要为Kubernetes集群配置好证书文件.
    - 证书生成后, 在master节点 /etc/kubernetes/pki 目录下, 主要证书文件是ca.crt和对应的私钥ca.key.
    - kubctl获取容器日志等streaming操作时, 需要通过kube-apiserver向kubelet发起请求, 这个证书是apiserver-kubelet-client.crt,
      和对应的私钥这个证书是apiserver-kubelet-client.key
3. 生成kube-apiserver需要的配置文件, 路径为: /etc/kubernetes/xxx.conf
    - /etc/kubernetes/admin.conf
    - /etc/kubernetes/controller-manager.conf
    - /etc/kubernetes/kubelet.conf
    - /etc/kubernetes/scheduler.conf
4. kubeadm 会为master组件生成pod配置文件, kube-apiserver, kube-controller-manager, kube-scheduler都会被以pod的方式部署起来
    - master组件的yaml文件会被生成在 /etc/kubernetes/manifests路径下
    - 允许用户把pod文件放置在 /etc/kubernetes/manifests 路径下, 在机器上启动
    - /etc/kubernetes/manifests
        - /etc/kubernetes/manifests/etcd.yaml
        - /etc/kubernetes/manifests/kube-apiserver.yaml
        - /etc/kubernetes/manifests/kube-controller.yaml
        - /etc/kubernetes/manifests/kube-scheduler.yaml
    - Master容器启动后, kubeadm会通过检查localhost:6443/healthz这个健康检查URL, 等待Master组件完全运行起来.
5. kubeadm 会为集群生成一个 bootstrap token
    - 只要持有这个token, 网络中联通环境下, 任何一个安装了kubelet和kubeadm的节点都可以通过kubeadm join到这个集群中.
6. token 生成后, kubeadm 会将 ca.crt等Master节点的重要信息, 通过ConfigMap的方式保存在etcd中名字为cluster-info, 供后续的node使用
7. 安装默认插件
    - kube-proxy
        - 服务发现
    - DNS
        - DNS功能

## 2. kubeadm join 的工作流程

## 3. Linux 证书生成工具

1. OpenSSL
   