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
## 2. 升级系统内核 [教程](http://elrepo.org/tiki/tiki-index.php)
```bash
  rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
  yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
  yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel
  grub2-set-default 0
```
## 3. 安装 runtime (docker)
 - [install docker](https://docs.docker.com/install/linux/docker-ce/centos/)
 
## 4. 安装 kubelet
 - [03-install-kublet.md](03-install-kublet.md)


## 5. 其他参考配置

/etc/sysctl.d/k8s.conf
```bash
vm.swappiness=0

vm.overcommit_memory=1
vm.panic_on_oom=0

net.bridge.bridge-nf-call-iptables=1
net.ipv4.tcp_keepalive_time=600
net.ipv4.tcp_keepalive_intvl=30
net.ipv4.tcp_keepalive_probes=10
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv4.neigh.default.gc_stale_time=120
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.default.arp_announce=2
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_announce=2
net.ipv4.ip_forward=1
net.ipv4.tcp_max_tw_buckets=5000
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_max_syn_backlog=1024
net.ipv4.tcp_synack_retries=2
net.netfilter.nf_conntrack_max=2310720
fs.inotify.max_user_watches=89100
fs.may_detach_mounts=1
fs.file-max=52706963
fs.nr_open=52706963
net.bridge.bridge-nf-call-arptables=1
```