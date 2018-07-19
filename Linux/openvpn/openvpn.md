## OpenVpn

#### 简介

VPN虚拟专用网络，可以让研发在家在任何地方登陆VPN之后都能愉快的连接公司内网的各个服务器。公司研发GG都需要这个，万一哪天出什么问题了在啥都不会就不好了，要未雨绸缪啊。
此处只介绍openvpn的部署，关于虚拟子网路由和NAT规则有网络基础很容易想明白。（无非就是将虚拟子网的网段nat成openvpn服务器的一个物理接口或者是虚拟接口而已）
#### 需要的组件
* openvpn
*	easy-rsa
*	Linux转发功能

#### 部署详细

```bash
	#打开Linux数据包转发功能
	[root@node1 ~]# more /etc/sysctl.conf
	# Controls IP packet forwarding
	net.ipv4.ip_forward = 1
	[root@node1 ~]# sysctl -p
	#安装openvpn和easy-rsa.其中openvpn提供vpn服务，easy-rsa为vpn服务提供CA
	yum -y install openvpn easy-rsa
	#制作CA，服务端证书和客户端证书
	[root@node1 ~]# cd /usr/share/easy-rsa/2.0/
	#修改默认参数值，CA和证书都可使用该默认参数进行制作
	[root@node1 2.0]# vi vars
	export KEY_COUNTRY="CN"
	export KEY_PROVINCE="BJ"
	export KEY_CITY="BJ"
	export KEY_ORG="so"
	export KEY_EMAIL="armo@so.com"
	export KEY_OU="armo"
	[root@node1 2.0]# source vars 	#参数生效
	NOTE: If you run ./clean-all, I will be doing a rm -rf on /usr/share/easy-rsa/2.0/keys
	[root@node1 2.0]# ./clean-all 	#清楚之前所有记录
	[root@node1 2.0]#
	[root@node1 2.0]# ls keys/
	index.txt  serial
	#制作CA
	[root@node1 2.0]# ./build-ca 	#一路回车使用默认值创建即可
	[root@node1 2.0]# ls keys/
	ca.crt  ca.key  index.txt  serial
	#使用CA颁发server端证书
	[root@node1 2.0]# ./build-key-server armo_server     #回车到下面两项选择Y，继续。
	Certificate is to be certified until Aug 14 08:15:42 2027 GMT (3650 days)
	Sign the certificate? [y/n]:y


	1 out of 1 certificate requests certified, commit? [y/n]y
	Write out database with 1 new entries
	Data Base Updated
	[root@node1 2.0]#
	[root@node1 2.0]# ls keys/
	01.pem  ca.crt  ca.key  index.txt  index.txt.attr  index.txt.old  serial  serial.old  armo_server.crt  armo_server.csr  armo_server.key
	#使用CA颁发客户端证书
	[root@node1 2.0]# ./build-key armo_client			#回车到下面两项选择Y，继续。
	Certificate is to be certified until Aug 14 08:29:40 2027 GMT (3650 days)
	Sign the certificate? [y/n]:y的


	1 out of 1 certificate requests certified, commit? [y/n]y
	Write out database with 1 new entries
	Data Base Updated
	[root@node1 2.0]#
	[root@node1 2.0]# ls keys/
	01.pem  02.pem  ca.crt  ca.key  armo_client.crt  armo_client.csr  armo_client.key  index.txt  index.txt.attr  index.txt.attr.old  index.txt.old  serial  serial.old  armo_server.crt  armo_server.csr  armo_server.key
	#创建dh，生成2048pm
	[root@node1 2.0]# ./build-dh
	[root@node1 2.0]# ll keys/dh2048.pem
	-rw-r--r-- 1 root root 424 Aug 16 16:49 keys/dh2048.pem
	#将所有证书移动到openvpn配置目录下
	[root@node1 2.0]# cp -a keys/ /etc/openvpn/
	#修改openvpn服务端配置文件
	[root@node1 2.0]# rpm -ql openvpn			#查询openvpn安装所有生成的文件
	/usr/share/doc/openvpn-2.4.3/sample/sample-config-files/server.conf
	[root@node1 2.0]# cp /usr/share/doc/openvpn-2.4.3/sample/sample-config-files/server.conf /etc/openvpn/
	[root@node1 2.0]# cp /etc/openvpn/server.conf /etc/openvpn/server.conf.bak
	[root@node1 ~]# vi /etc/openvpn/server.conf
	local 192.168.52.129
	port 1194
	proto tcp
	dev tun
	ca /etc/openvpn/keys/ca.crt			#根CA
	cert /etc/openvpn/keys/server.crt	#服务器证书
	key /etc/openvpn/keys/server.key 	#服务器证书key
	dh /etc/openvpn/keys/dh2048.pem		#pem
	server 1.1.1.0 255.255.255.0		#客户端地址池，所有登录VPN的客户端，都会获得该范围中的一个IP地址
	ifconfig-pool-persist ipp.txt		#防止openvpn重新启动后“忘记”Client曾经使用过的IP地址
	push "route 1.1.1.0 255.255.255.0"	#需要下发给客户端的路由信息，根据内网服务器网段而定
	push "dhcp-option DNS 8.8.8.8"		#需要下发给客户端的DNS信息
	client-to-client					#允许client之间通信
	keepalive 10 120					#10s一次keepalive，120s内无信息断开
	comp-lzo							#开启连接压缩，客户端和服务端必须同时开始或者同时关闭
	user nobody
	group nobody
	username-as-common-name 			#多用户同时登陆时可用同一账户
	status openvpn-status.log			#以登录的客户端信息
	log         openvpn.log
	log-append  openvpn.log
	verb 5								#日志等级
	#客户端下载地址：https://openvpn.net/index.php/open-source/downloads.html
	#配置windows客户端配置文件,将一下内容保存为armo.ovpn，后缀格式固定不可变
	client
	dev tun
	proto tcp
	remote  192.168.52.129 1194
	resolv-retry infinite				#断线自动重连
	nobind								#不绑定本地端口
	persist-key							
	persist-tun
	ca ca.crt							#指定CA文件路径
	cert armo_clent.crt					#指定客户端证书路径
	key armo_client.key					#指定客户端证书key路径
	ns-cert-type server					#使用服务器校验方式
	verb 3								#日志级别
	#从服务器中下载ca.crt、armo_clent.crt、armo_client.key	以及armo.ovpn保存至客户端的config文件夹中
	#此时可采用客户端证书进行登录VPN，一个用户需要制作一个证书十分的不方便，Openvpn在提供了openvpn-auth-pam.so模块可以协助认证，默认可使用非root的系统账号，客户端将账号密码打包传递给openvpn-auth-pam.so，由该模块验证提供的账号密码是否是正确的系统账号，完成登录过程。
	#配置项:
	#server端配置项，新增两行
	[root@node1]# vi /etc/openvpn/server.conf
	client-cert-not-required
	plugin /etc/openvpn/openvpn-auth-pam.so login
	#client端配置项，删除cert和key的配置，仅保留ca即可，然后新增以下参数
	auth-user-pass
```

###### 重启服务后，此时打开客户端登录VPN时则需要输入服务器系统的账号密码进行登录
###### 关于路由和NAT问题根据各个环境不同需要单独进行配置，需要注意的是NAT转换后的IP，以及openvpn服务端口的放通，有需要的话也可以将内网服务器的网关全部指向openvpn服务器
###### 最偷懒的登录方式，将客户端的账号密码保存到文件，让程序自动读取后登录
###### 修改客户端配置文件
	auth-user-pass pass.txt

在conf目录下创建pass.txt文件，将用户名写到第一行，密码写在第二行，就可以实现自动登录了，再也不用每次都输入账号密码，达到记住用户名密码的效果啦
