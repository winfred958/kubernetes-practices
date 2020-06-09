# kubernetes 安装前准备 [before-you-begin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)
## 1. 操作系统要求
 - centOS 7/ RHEL7/ 其他...(其他系统没玩过)
 - RAM 2G+
 - CPU core 2+
 - 所有node互通, 并且可以访问外网(或使用私有yum仓库)
 - 所有node的 hostname, mac, ip 唯一, [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node)
 - 开放指定端口, [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)
 - 关闭内存swap 
   - swapoff -a (临时)
   - swap 永久关闭: 
     - vim /etc/fstab #注释swap行
   - echo "vm.swappiness = 0" >> /etc/sysctl.d/k8s.conf
   - systctl -p # 刷新 

## 2. 升级系统内核 (非必须) [教程](http://elrepo.org/tiki/tiki-index.php)
```bash
  rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
  yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
  yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel
  grub2-set-default 0
```
## 3. 安装 runtime (docker) [官网](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)
 - [Install Docker CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
 - 国内镜像配置
    - vim /etc/docker/daemon.json
        - ```json
          {
            "registry-mirrors": ["https://registry.docker-cn.com"]
          }
          ```
### 3.1 remove 已经存在的老版本
  - ```bash
    sudo yum remove docker \
                      docker-client \
                      docker-client-latest \
                      docker-common \
                      docker-latest \
                      docker-latest-logrotate \
                      docker-logrotate \
                      docker-engine
    ``` 
### 3.2 Install Docker Engine - Community
#### 3.2.1 Install required packages
  - ```bash
    sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2
    ```
#### 3.2.2 set up repository
  - ```bash
    # 官方镜像
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    
    # 阿里云镜像(推荐)
    sudo yum-config-manager \
        --add-repo \
        https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```
#### 3.2.3 Install Docker Engine - Community
 - ```bash
    sudo yum install docker-ce docker-ce-cli containerd.io
    # OR
    sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
   ```
#### 3.2.4 auto start
  - ```bash
    systemctl enable docker
    systemctl start docker
    systemctl status docker
    ```

## 4. 安装 kubelet
 - [03-install-kublet-kubeadm.md](03-install-kublet-kubeadm.md)

## 5. 其他参考配置 & 注意事项
 - 注意事项
   - RHEL/CentOS 7上的一些用户报告了由于iptables被绕过而导致流量路由错误的问题。你应该确保net.bridge.bridge-nf-call-iptables = 1
     - vim /etc/sysctl.d/k8s.conf
         - net.bridge.bridge-nf-call-ip6tables = 1
         - net.bridge.bridge-nf-call-iptables = 1
         - 刷新: sysctl --system

   - 确保br_netfilter模式加载, 查看lsmod | grep br_netfilter
     - modprobe br_netfilter

/etc/sysctl.d/k8s.conf 其他配置(非必须)
  - ```bash
    vm.swappiness=0
    
    vm.overcommit_memory=1
    vm.panic_on_oom=0
    
    net.bridge.bridge-nf-call-iptables=1
    net.bridge.bridge-nf-call-ip6tables=1
    
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
    ```
