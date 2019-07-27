# kubernetes 安装前准备
## 1. 操作系统要求
 - centOS 7/ RHEL7/ 其他...(其他系统没玩过)
 - RAM 2G+
 - CPU core 2+
 - 所有node互通, 并且可以访问外网(或使用私有yum仓库)
 - 所有node的 hostname, mac, ip 唯一, [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node)
 - 开放指定端口, [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)
 - 关闭内存swap 
   - swapoff -a (临时)
   - echo "vm.swappiness = 0" >> /etc/sysctl.d/k8s.conf
## 2. 升级系统内核
```bash
  rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
  yum install -y https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
  yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel
  grub2-set-default 0
```
## 3. 安装 runtime (docker)
 - [install docker](https://docs.docker.com/install/linux/docker-ce/centos/)
 
## 4. 安装 kubelet
 - [03-install-kublet.md](03-install-kublet.md)
