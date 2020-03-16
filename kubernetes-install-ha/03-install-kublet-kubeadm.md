# kubernetes install kubelet, kubeadm, kubectl [官网](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
## before start
 - [before begin](01-before-begin.md)

## 1.配置仓库
 > * vim /etc/yum.repos.d/kubernetes.repo
```bash
# 官方镜像
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*

# 阿里云镜像(推荐)
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
 - 注意项
   - selinux 关闭是必须的, 不然pod无法访问文件系统
   - 部分用户在RHEL/CentOS报告iptables被绕过, 错误路由的问题, 需要配置网桥
     ```bash
     cat <<EOF > /etc/sysctl.d/k8s.conf
     net.bridge.bridge-nf-call-ip6tables = 1
     net.bridge.bridge-nf-call-iptables = 1
     EOF
     
     # 刷新配置
     sysctl --system
     ```
 - 确保 lsmod 是 br_netfilter 模式,在安装之前
   - ```bash
      # 查看
      lsmod | grep br_netfilter
     
      # 设置
      modprobe br_netfilter
      ```

## 3. 安装依赖
 - Troubleshooting kubeadm
    - yum install ebtables ethtool
    
## 4. 安装 kubelet kubeadm kubectl
 - yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
 
## 5. 配置自启动
 > * systemctl enable --now kubelet

## 6. 配置 被kubelet control-plan 节点使用的 cgroup driver, 然后重启kubelet
 - CentOS/RHEL
   - vim /etc/sysconfig/kubelet
     - KUBELET_EXTRA_ARGS=--cgroup-driver=<value>
     ```bash
     systemctl daemon-reload
     systemctl restart kubelet
     ```

## 7. [Troubleshooting](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#troubleshooting)
 - If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#troubleshooting).

