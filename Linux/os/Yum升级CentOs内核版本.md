## Yum升级CentOs内核版本
#### 概要
需要将一台线下测试机内核版本从目前的2版本升级到3以上版本，之前在自己虚拟机里做过编译升级，但是在其后使用中遇到很多问题，是因为在新的config文件中，默认有很多模块被注释掉，导致使用新的默认config文件编译后很多模块无法加载使用，比如itpables的NAT模块，Docker也就无法安装了，为了避免这个问题呢，本次升级希望采用yum方式自动升级。
#### 升级过程
在从来没有进行过类似操作的前提下呢，第一步当然需要google了，然而并没有找到描述清晰的示例，

     http://elrepo.org/tiki/tiki-index.php
* 先看一下本地yum仓库中各个kernel的版本
```bash    
    [root@localhost ~]# yum list | grep kernel
    abrt-addon-kerneloops.x86_64            2.0.8-34.el6.centos         @anaconda-CentOS-201508042137.x86_64/6.7
    dracut-kernel.noarch                    004-388.el6                 @anaconda-CentOS-201508042137.x86_64/6.7
    kernel.x86_64                           2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-devel.x86_64                     2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-firmware.noarch                  2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-headers.x86_64                   2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    libreport-plugin-kerneloops.x86_64      2.0.9-24.el6.centos         @anaconda-CentOS-201508042137.x86_64/6.7
    abrt-addon-kerneloops.x86_64            2.0.8-43.el6.centos         base        
    dracut-kernel.noarch                    004-409.el6_8.2             base        
    kernel.x86_64                           2.6.32-696.6.3.el6          updates     
    kernel-abi-whitelists.noarch            2.6.32-696.6.3.el6          updates     
    kernel-debug.x86_64                     2.6.32-696.6.3.el6          updates     
    kernel-debug-devel.i686                 2.6.32-696.6.3.el6          updates     
    kernel-debug-devel.x86_64               2.6.32-696.6.3.el6          updates     
    kernel-devel.x86_64                     2.6.32-696.6.3.el6          updates     
    kernel-doc.noarch                       2.6.32-696.6.3.el6          updates     
    kernel-firmware.noarch                  2.6.32-696.6.3.el6          updates     
    kernel-headers.x86_64                   2.6.32-696.6.3.el6          updates     
    libreport-plugin-kerneloops.x86_64      2.0.9-33.el6.centos         base        
    [root@localhost ~]#
```

* 当然不出所料都和目前已运行的kernel版本相差无几
```bash  
    [root@localhost ~]# uname -r
    2.6.32-573.el6.x86_64
    [root@localhost ~]#
```
* 根据官网提示，安装新的yum仓库，已获取官方目前提供的较新版本的安装包
```bash
    #导入KEY，必要的步骤
    [root@localhost ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    #安装elrepo，根据CentOS版本进行选择
    [root@localhost ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
    Retrieving http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
    Preparing...                ########################################### [100%]
       1:elrepo-release         ########################################### [100%]
    [root@localhost ~]#
```
* 这个仓库里包含了[elrepo]、[elrepo-testing]、[elrepo-kernel]、[elrepo-extras]四个仓库，默认仅启用了[elrepo]，这里我们需要启用[elrepo-kernel]仓库
```bash
    #这里我们直接编辑elrepo配置文件，将[elrepo-kernel]模块中的enable置为1，或直接使用yum --ebable即可
    [root@localhost ~]# vi /etc/yum.repos.d/elrepo.repo
    [elrepo-kernel]
    name=ELRepo.org Community Enterprise Linux Kernel Repository - el6
    baseurl=http://elrepo.org/linux/kernel/el6/$basearch/
            http://mirrors.coreix.net/elrepo/kernel/el6/$basearch/
            http://mirror.rackspace.com/elrepo/kernel/el6/$basearch/
            http://repos.lax-noc.com/elrepo/kernel/el6/$basearch/
            http://mirror.ventraip.net.au/elrepo/kernel/el6/$basearch/
    mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-kernel.el6
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
    protect=0
```

* 这个时候再次查看一下我们yum源中存在的kernel版本，需要的新版本已经可以使用了
```bash
    [root@localhost ~]# yum list | grep kernel
    abrt-addon-kerneloops.x86_64            2.0.8-34.el6.centos         @anaconda-CentOS-201508042137.x86_64/6.7
    dracut-kernel.noarch                    004-388.el6                 @anaconda-CentOS-201508042137.x86_64/6.7
    kernel.x86_64                           2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-devel.x86_64                     2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-firmware.noarch                  2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    kernel-headers.x86_64                   2.6.32-573.el6              @anaconda-CentOS-201508042137.x86_64/6.7
    libreport-plugin-kerneloops.x86_64      2.0.9-24.el6.centos         @anaconda-CentOS-201508042137.x86_64/6.7
    abrt-addon-kerneloops.x86_64            2.0.8-43.el6.centos         base       
    dracut-kernel.noarch                    004-409.el6_8.2             base       
    kernel.x86_64                           2.6.32-696.6.3.el6          updates     
    kernel-abi-whitelists.noarch            2.6.32-696.6.3.el6          updates     
    kernel-debug.x86_64                     2.6.32-696.6.3.el6          updates     
    kernel-debug-devel.i686                 2.6.32-696.6.3.el6          updates     
    kernel-debug-devel.x86_64               2.6.32-696.6.3.el6          updates     
    kernel-devel.x86_64                     2.6.32-696.6.3.el6          updates     
    kernel-doc.noarch                       2.6.32-696.6.3.el6          updates     
    kernel-firmware.noarch                  2.6.32-696.6.3.el6          updates     
    kernel-headers.x86_64                   2.6.32-696.6.3.el6          updates     
    kernel-lt.x86_64                        3.10.107-1.el6.elrepo       elrepo-kernel
    kernel-lt-devel.x86_64                  3.10.107-1.el6.elrepo       elrepo-kernel
    kernel-lt-doc.noarch                    3.10.107-1.el6.elrepo       elrepo-kernel
    kernel-lt-firmware.noarch               3.10.107-1.el6.elrepo       elrepo-kernel
    kernel-lt-headers.x86_64                3.10.107-1.el6.elrepo       elrepo-kernel
    kernel-ml.x86_64                        4.12.8-1.el6.elrepo         elrepo-kernel
    kernel-ml-devel.x86_64                  4.12.8-1.el6.elrepo         elrepo-kernel
    kernel-ml-doc.noarch                    4.12.8-1.el6.elrepo         elrepo-kernel
    kernel-ml-firmware.noarch               4.12.8-1.el6.elrepo         elrepo-kernel
    kernel-ml-headers.x86_64                4.12.8-1.el6.elrepo         elrepo-kernel
    libreport-plugin-kerneloops.x86_64      2.0.9-33.el6.centos         base       
    perf.x86_64                             4.12.8-1.el6.elrepo         elrepo-kernel
    python-perf.x86_64                      4.12.8-1.el6.elrepo         elrepo-kernel
    [root@localhost ~]#
```

* 这里我们选择我们需要的3.0版本进行安装，命名格式name+version，那么我们直接使用yum进行指定版本安装
```bash  
    #如果不确定可以先不使用-y选项，在结果中查看对应版本后在确认安装即可
    [root@localhost ~]# yum install kernel-lt-3.10.107-1.el6.elrepo
    Dependencies Resolved

    ========================================================================================================================================================================================================================
     Package               Arch                     Version                    Repository               Size
    ============================================================================================================================================================================================================================================
    Installing:
     kernel-lt            x86_64             3.10.107-1.el6.elrepo           elrepo-kernel              33 M

    Transaction Summary
    ============================================================================================================================================================================================================================================
    Install       1 Package(s)

    Total download size: 33 M
    Installed size: 154 M
    Is this ok [y/N]: y
```

* 坐等安装成功即可。安装完成后需要我们进行最后一步神圣的操作，选择默认启动的grub为新版本。

```bash
    #编辑grub启动文件，这款需要注意两个值，default和title
    [root@localhost ~]# vi /etc/grub.conf
    #default选择默认启动的title标号，自上而下从0开始计数，那么我们一般新安装的kernel呢会出出现在第一个title中，所以我们将default的值改为0，即默认选择第一个title后的kernel进行启动
    default=0
    #每个title后跟一个以安装的kernel版本信息
    title CentOS (3.10.107-1.el6.elrepo.x86_64)
        root (hd0,0)
        kernel /vmlinuz-3.10.107-1.el6.elrepo.x86_64 ro root=UUID=97b15044-9109-48d4-bf6c-a3e87e46ad3c rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb
    quiet
        initrd /initramfs-3.10.107-1.el6.elrepo.x86_64.img

    title CentOS 6 (2.6.32-573.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-573.el6.x86_64 ro root=UUID=97b15044-9109-48d4-bf6c-a3e87e46ad3c rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.32-573.el6.x86_64.img
```

* 重启系统，让系统使用新版本的kernel进行启动，启动后查看内核版本，KO~！
```bash
    [root@localhost ~]# uname -r
    3.10.107-1.el6.elrepo.x86_64
    [root@localhost ~]#
```
