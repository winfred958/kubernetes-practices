# kubernetes kubelet install [官网](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/#%E5%AE%89%E8%A3%85-kubeadm-kubelet-%E5%92%8C-kubectl)
## before start
 - [before begin](01-before-begin.md)

## 1.配置仓库
 > * vim /etc/yum.repos.d/kubernetes.repo   #(aliyun 镜像)

```bash
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
```

## 2. node 环境修改
 - 关闭 selinux
   - 临时: setenforce 0
   - 永久: vim /etc/selinux/config
      - SELINUX=disabled
 - 关闭 firewall
   - systemctl stop firewalld.service & systemctl disable firewalld.service

## 3. 安装依赖
 - Troubleshooting kubeadm
    - yum install ebtables ethtool
    
## 4. 安装 kubelet kubeadm kubectl
 - yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

## 5. 配置自启动
> * systemctl enable kubelet
> * systemctl start kubelet