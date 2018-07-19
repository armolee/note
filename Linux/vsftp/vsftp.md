## vsftp

#### 认证模式

* 匿名用户
* 系统用户
* 虚拟用户

#### 配置操作  
```      
    /etc/logrotate.d/vsftpd  日志轮转配置文件  
    /etc/pam.d/vsftpd  pam配置文件  
    /etc/vsftpd/ftpusers  pam模块设置的ftp用户名单，默认为黑名单  
    /etc/vsftpd/user_list  vsftp白名单或黑名单用户列表，需要在配置文件中开启  
    /etc/vsftpd/vsftpd.conf  配置文件   
    /var/ftp  该文件夹为ftp用户的家目录，属主属组为root，不能更改，否则无法正常启动服务，其中子目录可随意更改属主属组  
    [root@armo ~]# more /etc/passwd | grep ftp  
    ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin  
```

#####  匿名用户  

```
    匿名用户映射为ftp用户，共享资源位置为ftp用户家目录：/var/ftp
    anonymous_enable=YES  启用匿名用户登录功能  
    anon_upload_enable=YES  匿名用户上传权限  
    anon_mkdir_write_enable=YES  匿名用户创建写权限  
    anon_other_write_ enable=YES  匿名用户删除等权限  
```

##### 系统用户

```
    系统用户通过ftp访问的资源的位置：用户自己的家目录
    local_enable=YES  启用系统用户认证  
    write_enable=YES  系统用户写权限  
    限制所有的本地用户仅能访问各自家目录：
    chroot_local_user=YES
    限制文件中指定的本地用户仅能访问其家目录：
    chroot_list_enable=YES
    chroot_list_file=/etc/vsftpd/chroot_list
    新建的目录权限是755，文件的权限是644：

    local_umask=022
    ps: 目录 777-022=755， 666-022=644
    使用pam完成用户认证，其用到的pam配置文件：
    pam_service_name=vsftpd
```

##### 虚拟用户(mysql)

```
    虚拟用户通过ftp访问的资源的位置：给虚拟用户指定的映射成为的系统用户的家目录
    建立虚拟用户映射的系统用户及对应的目录，并且让其他用户对该目录有操作权限
    useradd -s /sbin/nologin -d /var/ftproot vuser
    chmod go+rx /var/ftproot

    pam_service_name=vsftpd.mysql  调用文件名为vsftpd.mysql 的pam配置
    anonymous_enable=YES
    guest_enable=YES  
    guest_username=vuser  虚拟用户映射为vuser，访问其家目录
    virtual_use_local_privs=YES  虚拟用户拥有与本地用户相同的权限，默认为NO
    user_config_dir=/etc/vsftpd/user_config (如果需要实现不同用户拥有不同的权限,可使用该参数定义用户配置文件夹，在此文件夹中以用户名创建配置文件即可。配置文件可包含以下选项anon_upload_enable={YES|NO}
    anon_mkdir_write_enable={YES|NO}
    anon_other_write_enable={YES|NO})
    需要事先从epel源中安装pam.mysql模块，centos7需要编译安装
    mysql中创建表包含登陆名和密码即可，密码可使用明文或password函数加密
    在/etc/pam.d/文件夹下创建vsftpd.mysql文件:
    auth required /usr/lib64/security/pam_mysql.so user=vsftpd passwd=centos host=127.0.0.1 db=vsftpd table=users usercolumn=user passwdcolumn=passwordcrypt=2

    account required /usr/lib64/security/pam_mysql.so user=vsftpd passwd=centos host=127.0.0.1 db=vsftpd table=users usercolumn=user passwdcolumn=password crypt=2
    user为数据库用户，passwd为数据库密码。db库tables表。usercolum为保存用户名的列名，passwdcolumn为保存密码的列名。crypt值为0则保存密码为明文，1为密文，2为password函数加密的密文
```

##### 基础配置

```    
    日志：
        xferlog_enable=YES
        xferlog_std_format=YES
        xferlog_file=/var/log/xferlog

    改变上传文件的属主：
        chown_uploads=YES
        chown_username=whoever

    是否启用控制用户登录的列表文件
        userlist_enable=YES
        userlist_deny=YES|NO
        默认文件为/etc/vsftpd/user_list

    连接限制：
        max_clients: 最大并发连接数；
        max_per_ip: 每个IP可同时发起的并发请求数；

    传输速率：
        anon_max_rate: 匿名用户的最大传输速率, 单位是“字节/秒”;
        local_max_rate: 本地用户。。。
```
