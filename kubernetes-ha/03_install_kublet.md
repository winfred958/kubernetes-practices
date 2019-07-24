## 1.配置仓库
>> vim /etc/yum.repos.d/kubernetes.repo
>> aliyun 镜像
```
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
```
## 2. 安装
>>yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes