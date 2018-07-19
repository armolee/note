## SVN

#### 概要
（本文介绍过程，SVN部署和使用，并且将研发通过SVN更新的代码进行自动上线）
搭建完openvpn之后，研发GG还需要通过SVN来管理迭代自己的代码，那么在我们的环境中，SVN搭建在开发环境中，但是SVN存储的代码是以二进制的方式存储，所以类似于我们站点PHP代码的环境，在开发环境中并不会真正存储上PHP代码，那么也就需要将开发环境也作为SVN的客户端，将更新的PHP文件代码，实时的更新在开发环境的存储中，达到需要的效果就是研发在SVN提交了新的代码之后，开发环境会直接上线迭代的最新代码。
CS架构的SVN（subversion）用来管理和进行代码的迭代，此处介绍在Linux平台提供服务，Windows平台使用客户端进行连接。

#### 安装
```bash
    yum install httpd httpd-devel subversion mod_dav_svn mod_auth_mysql -y
    #查看版本
    svnserve --version
    #创建库
    #创建所有库的父目录
    mkdir /opt/svn
    #创建库1和库2（不同的项目，例如两个不同站点项目）
    [root@localhost opt]# mkdir /opt/svn
    [root@localhost opt]# svnadmin create /opt/svn/repo1
    [root@localhost opt]# svnadmin create /opt/svn/repo2
    #为了方便使用，可以让所有用户使用同一份账号密码登录不同的库中，（由于公司很多项目很多人都有使用，权限管理并不是很好。。。）将repo1中生成的
    config文件夹中authz文件和passwd文件复制一份到/opt/svn/conf文件夹下，使用该目录下的配置文件为所有库提供认证的账号密码
    [root@localhost ~]# mkdir /opt/svn/conf
    [root@localhost ~]# cp /opt/svn/repo1/conf/authz /opt/svn/conf/
    [root@localhost ~]# cp /opt/svn/repo1/conf/passwd /opt/svn/conf/
    [root@localhost ~]# ls /opt/svn/conf/
    authz  passwd
    #添加svn用户登录账号密码
    [root@localhost ~]# cat /opt/svn/conf/passwd
    ### This file is an example password file for svnserve.
    ### Its format is similar to that of svnserve.conf. As shown in the
    ### example below it contains one section labelled [users].
    ### The name and password for each user follow, one account per line.
    [users]
    # harry = harryssecret
    # sally = sallyssecret
    armo = passwd
    [root@localhost ~]#
    #给新用户armo授权，定义用户armo和armo1都属于dev组，在repo的根“[/]”下，dev组有读写权限
    [root@localhost ~]# cat /opt/svn/conf/passwd
    [groups]
    # harry_and_sally = harry,sally
    # harry_sally_and_joe = harry,sally,&joe
    dev = armo,armo1
    [/]
    @dev = rw
    #将所有库配置文件调用的账号文件和权限文件指向统一配置文件，该配置文件中所有配置必须在行首开始，否则无法识别
    [root@localhost ~]# vi /opt/svn/repo1/conf/svnserve.conf
    [root@localhost ~]# vi /opt/svn/repo2/conf/svnserve.conf
    [root@localhost ~]# more /opt/svn/repo1/conf/svnserve.conf | grep =
    #匿名账号权限
    anon-access = none
    #认证账号权限
    auth-access = write
    #指向全局密码文件
    password-db = /opt/svn/conf/passwd
    #指向全局授权文件
    authz-db = /opt/svn/conf/authz
    #启动服务
    [root@localhost ~]# svnserve -d -r /opt/svn
    -d以守护模式启动进程
    -r /opt/svn 指向所有库的父目录
```

#### 下载安装客户端
    https://tortoisesvn.net/
    #下载对应32位或64位客户端进行安装
    #安装过程中选择will be installed on local hard drive 
 ![](http://upload-images.jianshu.io/upload_images/6328743-33b4adf8886121c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 客户端连接SVN服务，开始第一个项目
在客户端桌面，右键，选择SVN checkout
   ![](http://upload-images.jianshu.io/upload_images/6328743-cc12a318a1d0edcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进行账号认证 armo 密码 passwd，第三步设置的账号密码
   ![](http://upload-images.jianshu.io/upload_images/6328743-bf0a8c4d32a0c69f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ![](http://upload-images.jianshu.io/upload_images/6328743-89dcfb022bbc3597.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打开本地对应目录：D:\svntest\repo1，开始项目，测试创建文件并提交。
新建文件后，在D:\svntest\repo1文件夹中，右键，选择SVN commit进行提交代码。

    SVN update从服务器上更新代码到本地
    SVN commit将本地代码提交至服务器
   ![](http://upload-images.jianshu.io/upload_images/6328743-c3a51f929d7c9e41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
提交到服务器的代码已二进制格式保存在/opt/svn/repo1/db/revs/0中，此时如果有新客户端进行check out，则会从服务器上同步已经提交的product文件夹和1.txt到新客户端本地
#### 设置HOOKS，实现提交到SVN服务器的代码同步到目标目录当中
    在/opt/svn/repo1/hooks/中将post-commit.tmpl复制一份为post-commit，并且赋予执行权限，该hook在客户端执行commit之后会执行一次
```bash
    [root@localhost ~]# cp post-commit.tmpl post-commit
    [root@localhost ~]# vi /opt/svn/repo1/hooks/post-commit
    #增加以下几行
    export LANG=en_US.UTF-8
    #将提交的文件在/tmp/svn_up文件夹中进行更新同步
    /usr/bin/svn update --username armo --password passwd /tmp/svn_up
    #使用rsync同步从SVN中下载的代码到研发环境对应的文件夹，不包含.svn文件
    rsync -av --delete /tmp/svn_up/ /www/site/ --exclude=.svn
```  
设置完该HOOKS之后，客户端提交的代码，会先使用svn update更新一份到/tmp/svn_up中，在使用rsync同步一份到需要的站点目录下，完成自动上线。
