## 1. etcd 安装
> * yum install -y etcd
```
三节点:
h1-k8s-local
h2-k8s-local
h3-k8s-local
```
## 2. 修改配置
> * vim /etc/etcd/etcd.conf

h1-k8s-local
```
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""

# 集群内使用的url
ETCD_LISTEN_PEER_URLS="http://h1-k8s-local:2380"

# 外部客户端使用的url
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379,http://h1-k8s-local:2379"

#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="default"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]

# 广播给集群内其他成员的url
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://h1-k8s-local:2380"

# 广播给外部客户端的url
ETCD_ADVERTISE_CLIENT_URLS="http://h1-k8s-local:2379"

#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""

# 初始集群成员列表
ETCD_INITIAL_CLUSTER="etcd01=http://h1-k8s-local:2380,etcd02=http://h2-k8s-local:2380,etcd03=http://h3-k8s-local:2380"

# 集群名称
ETCD_INITIAL_CLUSTER_TOKEN="my-etcd-cluster"

# 初始集群状态, new为新建集群
ETCD_INITIAL_CLUSTER_STATE="new"

#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
#ETCD_CERT_FILE=""
#ETCD_KEY_FILE=""
#ETCD_CLIENT_CERT_AUTH="false"
#ETCD_TRUSTED_CA_FILE=""
#ETCD_AUTO_TLS="false"
#ETCD_PEER_CERT_FILE=""
#ETCD_PEER_KEY_FILE=""
#ETCD_PEER_CLIENT_CERT_AUTH="false"
#ETCD_PEER_TRUSTED_CA_FILE=""
#ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
```

> * h2-k8s-local
> * h3-k8s-local

## 3. etcd  自启动
### 3.1 修改 /usr/lib/systemd/system/etcd.service
> * vim /usr/lib/systemd/system/etcd.service
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors

ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd \
 --name=\"${ETCD_NAME}\" \
 --data-dir=\"${ETCD_DATA_DIR}\" \
 --listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" \
 --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\" \
 --advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" \
 --initial-advertise-peer-urls=\"${ETCD_INITIAL_ADVERTISE_PEER_URLS}\" \
 --initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" \
 --initial-cluster=\"${ETCD_INITIAL_CLUSTER}\" \
 --initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\""

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
### 3.2 添加自启动
> * systemctl enable etcd
> * systemctl start etcd
> * systemctl status etcd

## 4.验证etcd
### 4.1 查看节点
> * etcdctl member list
```
[root@host-01 ~]# etcdctl member list
39c503e55d72d28: name=etcd3 peerURLs=http://host-03:2380 clientURLs=http://127.0.0.1:2379,http://host-03:2379 isLeader=false
db08cbcec431402d: name=etcd1 peerURLs=http://host-01:2380 clientURLs=http://host-01:2379 isLeader=false
f843f53469625da2: name=etcd2 peerURLs=http://host-02:2380 clientURLs=http://127.0.0.1:2379,http://host-02:2379 isLeader=true
```
### 4.2 数据写入验证
> * @host-01 运行: etcdctl set /test/test-path "test"
> * @host-03 运行: etcdctl get /test/test-path
```
[root@host-01 ~]# etcdctl set /test/test-path "test"
test
```
```
[root@host-03 ~]# etcdctl get /test/test-path 
test
```

## 5.遇到的问题
### 5.1 hostname 无法识别
```
error verifying flags, expected IP in URL for binding (http://host-01:2380). See 'etcd --help'.
```
> * 解决方案: client url 使用 ip

### 5.2 etcd 重启, 无法启动
#### 5.2.1 解决方案1:
- 目标节点作为**新节点重新加入**集群
    - etcdctl member add ${ETCD_NAME} http://xx.xx.xx.xx:2380
    - 目标节点请空data-dir
    - 目标节点启动etcd, **需要修改 --initial-cluster-state existing**
