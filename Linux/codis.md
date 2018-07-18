#### codis

1、简介  
redis是目前使用广泛的中间件，从3.0版本开始官方支持了redis cluster。理解codis cluster首先需要理解官方的redis cluster。  
redis cluster去中心化，也就是说集群中每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态，换句话来说集群中每个节点内存储的数据内容均不相同。  
redis cluster采用哈希槽（hash slot）的方式来分配数据，集群默认分配了16384个slot，当client进行set key时，会调用CRC16算法对key取模（CRC16(key)%16384），然后将对应的key存储分配至对应的节点。以三个节点为例，A节点的slot范围为0-5460，B节点的slot范围为5461-10922，C节点的slot范围为10923-16384。假设CRC16(key)%16384=6666，该槽位在B节点中，那边集群会将该key分配至B节点进行存储，查询时也相同。  
在codis cluster的架构中，将slot分为1024份，CRC32的取模算法，引入了Group的概念，每个Group包括了一个master和至少一个slave，当master有问题时，可通过Dashboard切换到slave中。Codis也采用了预先分片机制，分成了1024个Slot，将这些路由信息保存在Zookeeper中。同时codis也支持了数据热迁移。  

	![三种集群方案对比](/static/threeredis.png)

2、架构
	* Codis-Proxy:cluster服务入口，供Client连接
	* Codis-Group:Codis-Server的组容器，实现水平扩展
	* Codis-Server:redis实例的实现
	* Zookeeper:存放Codis-Proxy节点的元信息，Cluster管理命令通过Zookeeper同步给各个存活的Codis-Proxy

	* 实验环境：
		* Zookeeper集群:192.168.52.129、192.168.52.130、192.168.52.131
		* Codis-Proxy:192.168.52.129、192.168.52.130
		* Codis-Server:192.168.52.129、192.168.52.130、192.168.52.131
	![codis-cluster逻辑结构](/static/codis-cluster.png)

3、部署过程
	1、安装zookeeper cluster。
	```bash
	#hosts文件三台主机保持一致
	[root@localhost ~]# more /etc/hosts 
	127.0.0.1  localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1        localhost localhost.localdomain localhost6 localhost6.localdomain6
	192.168.52.129	node1
	192.168.52.130	node2
	192.168.52.131	node3
	[root@localhost ~]#
	#准备java环境
	[root@localhost ~]# yum list java*
	[root@localhost ~]# yum -y install java-1.8.0-openjdk*
	[root@localhost ~]# java -version
	openjdk version "1.8.0_141"
	OpenJDK Runtime Environment (build 1.8.0_141-b16)
	OpenJDK 64-Bit Server VM (build 25.141-b16, mixed mode)
	[root@localhost ~]#
	# zookeeper下载地址http://ftp.jaist.ac.jp/pub/apache/zookeeper/
	[root@localhost ~]# cd /opt/
	[root@localhost opt]# wget http://ftp.jaist.ac.jp/pub/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
	[root@localhost opt]# tar xf zookeeper-3.4.10.tar.gz
	[root@localhost opt]# mv zookeeper-3.4.10 zookeeper
	[root@localhost opt]# cd zookeeper/conf/
	[root@localhost conf]# cp zoo_sample.cfg zoo.cfg
	[root@localhost conf]# vi zoo.cfg
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/tmp/zookeeper/data	#该目录在程序启动前需要事先创建
	dataLogDir=/tmp/zookeeper/log	#该目录在程序启动前需要事先创建
	clientPort=2181
	server.1=node1:2888:3888
	#server.1/2/3表示对应主机在集群中的标识，例中node1为server.1，则需要在data目录下创建myid文件，并且文件内容为2。data路径在上面定义为/tmp/zookeeper/data
	server.2=node2:2888:3888
	server.3=node3:2888:3888
	#nodeX是集群中成员IP或HOSTNAME，可直接使用IP，使用hostname需要事先在/etc/hosts中定义。其中2888端口为leader和follower的通信端口；3888为集群刚启动时或leader挂掉之后重新进行新的选举的通信端口
	在node1主机中:
	echo 1 > /tmp/zookeeper/data/myid
	在node2主机中:
	echo 2 > /tmp/zookeeper/data/myid
	在node3主机中:
	echo 3 > /tmp/zookeeper/data/myid
	修改环境变量或直接运行启动脚本
	[root@localhost ~]# export PATH=/opt/zookeeper/bin:$PATH
	[root@localhost ~]# zkServer.sh start
	ZooKeeper JMX enabled by default
	Using config: /opt/zookeeper/bin/../conf/zoo.cfg
	Starting zookeeper ... STARTED
	[root@localhost ~]# zkServer.sh status
	ZooKeeper JMX enabled by default
	Using config: /opt/zookeeper/bin/../conf/zoo.cfg
	Mode: follower
	[root@localhost ~]#
	```
![zookeeper集群状态](/static/zookeeprestatus.png) 

2、安装Go环境，Codis由Go语言编写，EPEL源中可直接yum安装
```bash
	[root@localhost ~]# wget https://mirrors.tuna.tsinghua.edu.cn/epel/6/x86_64/epel-release-6-8.noarch.rpm
	[root@localhost ~]# rpm -ivh epel-release-6-8.noarch.rpm
	[root@localhost ~]# yum -y install golang
	[root@localhost conf]# go version
	go version go1.7.6 linux/amd64
```
3、安装Codis
```bash
	[root@localhost ~]# mkdir -p /usr/lib/golang/src/github.com/CodisLabs
	[root@localhost ~]# cd /usr/lib/golang/src/github.com/CodisLabs
	[root@localhost CodisLabs]# git clone https://github.com/CodisLabs/codis.git -b release3.2
	[root@localhost CodisLabs]# cd codis
	[root@localhost codis]# make
	make完成之后安装就竣工了，相对来使用git clone安装比较简单。当然必须要git到java的路径下
```
4、再次介绍一下codis的各个组件，以明确在集群中应该如何启动
	* Codis Server：基于 redis-3.2.8 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。具体的修改可以参考文档 redis 的修改。在集群中充当redis实例。
	* Codis Proxy：客户端连接的 Redis 代理服务, 实现了 Redis 协议。 除部分命令不支持以外(不支持的命令列表)，表现的和原生的 Redis 没有区别（就像 Twemproxy）。对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例；不同 codis-proxy 之间由 codis-dashboard 保证状态同步。
	* Codis Dashboard：集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。在集群状态发生改变时，codis-dashboard 维护集群下所有 codis-proxy 的状态的一致性。
	* Codis FE：集群管理界面  

5、在node1启动FE和Dashboard用来管理集群；node1和node3启动codis-proxy；node1,node2,node3启动codis-server
首先启动Dashboard，关联zookeeper集群，将dashboard信息保存在zookeeper集群中，FE通过读取zookeeper中保存的dashboard信息来连接需要被管理的集群。
```bash
	#启动dashboard，将coordinator修改为zookeeper模式，自定义product_name和auth
	[root@localhost codis]# vi config/dashboard.toml
	#coordinator_name = "filesystem"
	#coordinator_addr = "/tmp/codis"
	coordinator_name = "zookeeper"
	coordinator_addr = "node1:2181,node2:2181,node2:2181"
	# Set Codis Product Name/Auth.
	product_name = "armo1-demo"
	product_auth = ""
	[root@localhost codis]# /usr/lib/golang/src/github.com/CodisLabs/codis/admin/codis-dashboard-admin.sh start
	#启动FE，将FE启动模式更改为zookeeper模式，由于3.2版本默认的FE启动模式为filesystem，所以我们需要使用二进制文件手动启动
	[root@localhost codis]# ./bin/codis-fe
	Usage:
	codis-fe [--ncpu=N] [--log=FILE] [--log-level=LEVEL] [--assets-dir=PATH] [--pidfile=FILE] (--dashboard-list=FILE|--zookeeper=ADDR|--etcd=ADDR|--filesystem=ROOT) --listen=ADDR
	codis-fe  --version
	[root@localhost codis]#
	#(--dashboard-list=FILE|--zookeeper=ADDR|--etcd=ADDR|--filesystem=ROOT)默认为filesystem，需要更改为zookeeper，或者从zookeeper中拉取到本地文件中（ ./bin/codis-admin  --dashboard-list --zookeeper=XXX.XXX.XXX.XXX:2181 | tee codis.json），使用--dashboard-list=FILE指定对应信息
	[root@localhost codis]# nohup /usr/lib/golang/src/github.com/CodisLabs/codis/bin/codis-fe --ncpu=1 --log=/usr/lib/golang/src/github.com/CodisLabs/codis/log/fe.log --log-level=WARN --zookeeper=node1:2181 --listen=0.0.0.0:9090 &
	#登录监听的9090端口查看FE管理界面，接下来集群操作可在FE中完成
```

	![FE管理界面](/static/codisfe.png)
```bash	  
	#启动两个codis-proxy,分别修改node1和node3的proxy.toml配置文件
	[root@localhost codis]# vi config/proxy.toml
	# Set Codis Product Name/Auth.
	#对应dashboard的product_name
	product_name = "armo1-demo"
	product_auth = ""
	[root@localhost codis]# ./admin/codis-proxy-admin.sh start
	#因为node1上同时存在proxy和dashboard，启动proxy后，proxy会自动连接127.0.0.1的dashboard，node3的proxy需要在FE中添加到dashboard。使用admin_addr配置参数连接。启动proxy后，尽快执行new proxy操作，默认30秒超时，超时后退出proxy进程。
```

	！[新增Codis-Proxy](/static/newcodisproxy.png)
```bash
	#分别启动三台codis-server，并在FE中添加进组，此处不添加master和slave，每天服务器仅启动一个进程，如果需要启动m/s，使用slaveof配置即可。
	#codis-server配置文件默认监听在127.0.0.1地址上，修改该地址后再启动实例
	[root@localhost codis]# vi config/redis.conf
	bind 0.0.0.0
	[root@localhost codis]# ./admin/codis-server-admin.sh start
	#三台启动完成后再FE中先添加三个组，在将每个实例添加到组中，然后进行slot初始化。完成集群搭建。
```

	![新增Group和组成员](/static/newcodisgroup.png)
#初始化Slot，会在FE中看到slot迁移的整个过程




	![初始化Slot](/static/initcodisslot.png)

#### 参考文档
```
	https://github.com/CodisLabs/codis
```
