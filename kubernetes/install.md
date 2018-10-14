## Kubernetes v1.11安装过程
#### 环境说明
  * master
    * IP :
  * node
    * IP :
  * master和node间基于主机名通信(dns or hosts)
  * 各节点之间时间必须同步(ntpd)
  * 各节点disabled firewalld和iptables
  * yum仓库指向extras，centos7.3自带
  * 1.11版本负载均衡调度从iptables替换为ipvs，需要自行安装ipvs组件，否则还会使用iptables来完成
  * 网络通信
    * 同一个Pod内的多个容器间：eth lookback
    * 各Pod之间的通信：overlay network 叠加网络
    * Pod与Server之间的通信
  * 五个私有CA保证安全性
    * etcd 集群内部通信，单独证书端口
    * etcd 与客户端通信（apiserver），单独证书端口
    * apiserver向客户端通信，需要单独的证书
    * apiserver与内部集群通信(kubelet ,proxy)，需要两个单独的证书

#### 1.11.1版本的k8s集群安装过程
  1. 设置yum源
    * docker-ce源  
    ```
    wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```  
    * k8s源  
    ```bash
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=0
  ```  

  2. 安装docker-ce , kubeadm ,kubelet ,kubectl（node可选安装）
    * 目前k8s支持到docker-ce的17.03版本，如果安装更新版本的docker，安装k8s时会提示告警信息
```bash  
  yum install docker-ce kubectl kubelet kubeadm
```

  3. 设置docker的代理
```bash
  vim /usr/lib/systemd/system/docker.service
  [Service]
  Environment="HTTPS_PROXY=http://www.ik8s.io:10080"
```
  4. 设置kubelet忽略swap的使用(如果使用swap的话)
```bash
  vim /etc/sysconfig/kubelet
  KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```  
  5. 设置docker/kubelet开机自启动，并启动docker，dubelet不需要手动启动，kubeadm init时会启动该进程
  6. 确保以下配置为1,(因为网桥工作于数据链路层，在iptables没有开启bridge-nf时,数据会直接经过网桥转发，结果就是对FORWARD链的设置失效)
```bash   
  cat /proc/sys/net/bridge/bridge-nf-call-iptables
  cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
```  
  7. kubeadm init & join
  ```bash
  master:
  kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=swap
  mkdir -p HOME/.kubecp−i/etc/kubernetes/admin.confHOME/.kube/config
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  node:
  kubeadm join 192.168.1.115:6443 --token 6kduqc.23s178z3sct2rmrx --discovery-token-ca-cert-hash sha256:720cc878f4d06b3f51b698e5320cab5a8a1262a9443de443ea9874a137e9b08c --ignore-preflight-errors=Swap
  ```
