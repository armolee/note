## 基础知识
* k8s cluster中的几个名词
  * master
    * 负责管理集群,提供集群的资源数据访问入口（并非node节点提供的服务访问入口）
    * 包含几个关键组件
      * API Server : 提供master的restful风格的api接口
      * Scheduler : 调度器，负责创建pod，管理各控制器的健康状态等
      * Controller-Manager : 管理Scheduler的健康状态，该组件使用master节点冗余的方式达到高可用的效果
      * docker : 容器实体，运行k8s addons附件容器，例如etcd，dns等
      * kubectl : 连接API Server接口，执行各类k8s指令
  * node
    * 运行容器的实际载体，能够安装运行k8s的各类操作系统都可加入至node节点中
    * kubectl 、 docker
  * registry
    * 提供各类image镜像，公有或私有
  * pod
    * k8s中最小的运行单位，一个pod可以运行多个容器
    * 同一个pod中存储卷可共享挂载至多个容器中
    * Laber 、LaberSelector : 标签及标签选择器，在service中做分类等作用
  * pod管理器的类型
    * Replication Controller : 副本控制器
    * ReplicaSet : 副本集控制器，下一代的Replication Controller，主要用于Deployment
    * Deployment : 无状态控制器，支持二级控制器HPA(可根据目前PodSet的状态进行自动扩容)
    * StatefulSet : 有状态控制器
    * DaemonSet : 每个node仅运行一个副本
    * Job/CronJob : 作业类型pod的控制器
  * 服务发现
  ![zidongfaxian](http://www.8i88.cn/static/zidongfaxian.png)
    1. 养鸡场将自己的信息注册在交易中心，交易中心定期检查
    2. 想吃鸡的人在不知道去哪里买鸡的情况下，直接查询交易中心，交易中心告知养鸡场的机器位置
    3. 得到买鸡的途径去买鸡
    * 服务发现过程与上述过程类似，k8s中以service(微服务)的概念做到交易中心的作用，其借助kube-dns服务来实现具体pod信息的注册及变更，借助iptables或ipvs实现流量的负载(1.11版本的k8s开始将转发规则转移至ipvs以达到更好的转发效果，较早的版本中一直使用iptables来实现)；过程中使用不同的Pod Laber识别不同的service，之后再动态的探测其IP、PORT及hostname信息
