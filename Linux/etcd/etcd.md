## Etcd
* Etcd构建高可用集群
  * 静态发现：预先已知Etcd集群中的哪些节点，在启动时指定各个node节点地址
  * 动态发现：通过已有的Etcd集群作为数据交互点，在扩展新的集群时实现通过已有集群进行服务发现的机制
  * DNS动态发现：通过DNS查询方式获取其他节点地址信息
* 静态发现

```bash
[root@k8s-etcd1 ~]# grep -v "#" /etc/etcd/etcd.conf
# 数据存放位置
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
# 监听其他 Etcd 实例的地址
ETCD_LISTEN_PEER_URLS="http://192.168.1.111:2380"
# 监听客户端地址
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
# 节点名称
ETCD_NAME="k8s-etcd1"
# 通知其他 Etcd 实例地址
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://k8s-etcd1:2380"
# 通知客户端地址
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.111:2379"
# ETCD_INITIAL_CLUSTER 用于静态发现，初始化集群内节点地址
ETCD_INITIAL_CLUSTER="k8s-etcd1=http://k8s-etcd1:2380,k8s-etcd2=http://k8s-etcd2:2380,k8s-etcd3=http://k8s-etcd3:2380"
# 初始化集群 token
ETCD_INITIAL_CLUSTER_TOKEN="soyoung-etcd-cluster"
# 初始化集群状态，new 表示新建
ETCD_INITIAL_CLUSTER_STATE="new"

[root@k8s-etcd1 ~]# etcdctl member list
20ec712b782c1b29: name=k8s-etcd2 peerURLs=http://k8s-etcd2:2380 clientURLs=http://192.168.1.112:2379 isLeader=false
b2d88f1c3737ef8e: name=k8s-etcd3 peerURLs=http://k8s-etcd3:2380 clientURLs=http://192.168.1.113:2379 isLeader=false
cd35558c27f496b0: name=k8s-etcd1 peerURLs=http://k8s-etcd1:2380 clientURLs=http://192.168.1.111:2379 isLeader=true
```
