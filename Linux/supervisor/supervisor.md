## Supervisor
* 概要
    发现在各个线上的环境中，有个目录中存了大量的日志信息，非常的占用磁盘空间，经过查看发现是由于之前在建立由Supervison管理的任务时，各个进程的日志管理配置不统一导致。有的日志200M有的500M，有的日志数量5个也有30个的。为了便于管理并且节省磁盘空间，准备将所有的日志设置为每200M分隔一次，共5次。
    在完成该工作的过程中需要了解以下几点：
    1、supervisor的使用
    2、sed替换匹配模式之后的一整行
    3、awk和xargs的联合使用
* Supervisor简单介绍
    Linux的后台进程运行有好几种方法，例如nohup，screen等，但是，如果是一个服务程序，要可靠地在后台运行，我们就需要把它做成daemon，最好还能监控进程状态，在意外结束时能自动重启。Supervisor就是用Python开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启。
    在使用Supervisor时，可以随时使用Supervisor的客户端Supervisorctl来管理进程，查看状态，重启，停止等。
```bash
    supervisor在EPEL源中，使用yum安装需要先安装EPEL源
    [root@localhost ~]# yum install supervisor
    [root@localhost ~]# rpm -ql supervisor
    /etc/logrotate.d/supervisor
    /etc/rc.d/init.d/supervisord
    /etc/supervisord.conf     
    /usr/bin/supervisorctl
    /usr/bin/supervisord
```

* 修改配置文件并且创建被其管理的进程，可以直接在主配置文件中定义各个进程，但是在生产环境中我们经常要有许多进程需要被管理，那么通常会在一个子配置文件夹中，为各个进程分别创建各自的配置文件，那么我们需要在主配置文件中定义子配置文件夹目录：
```bash
    [root@localhost ~]# vi /etc/supervisord.conf    
    [include]
    files = /etc/supervisor/*.ini
    新增以上两行，文件可以使用ini结尾或者是conf结尾的文件
    [root@localhost ~]#  vi /etc/supervisor/program1.ini
    [program:program1]                                   #定义该进程的在supervisor中的名称
    command=/www/site/program/program_name arg1 arg2     #定义该进程启动的脚本命令
    autostart = true                                     #在supervisord 启动的时候也自动启动
    user = nginx                                         #启动进程的用户
    process_name = -%(process_num)d                      #进程的序号命名
    numprocs = 5                                         #一共启动多少个进程
    numprocs_start = 1                                   #进程序号从1开始
    startsecs = 5                                        #启动 5 秒后没有异常退出，就当作已经正常启动了
    autorestart = true                                   #程序异常退出后自动重启
    startretries = 3                                     #启动失败自动重试次数，默认是 3
    redirect_stderr = true                               #把 stderr 重定向到 stdout，默认 false
    stdout_logfile_maxbytes = 200MB                      #stdout 日志文件大小，默认 50MB
    stdout_logfile_backups = 5                           #stdout 日志文件备份数
    stdout_logfile = /www/site/logs/program-%(process_num)d.log         #stdout 日志文件
```
* 定义完进程之后，启动supervisor，若autostart = true，启动supervisor时该进程也会自动启动,启动后就可以使用supervisorctl进程管理各个进程。
```bash   
    [root@localhost ~]# service supervisord start
    [root@localhost ~]# supervisorctl status
    program1                      RUNNING   pid 24128, uptime 0:01:56
```
* 在日常工作中，需要用到的最多的就是需要新增一个supervisor管理的进程时，我们在增加完进程的配置文件之后，不需要重启supervisor，使用supervisorctl即可完成上线工作。
```bash
    [root@localhost ~]# supervisorctl update new_program_name
    [root@localhost ~]# supervisorctl start new_program_name
```
* 当然也会遇到批量重启任务的时候，比如说某个研发GG改了最底层的某个进程内容，那么需要重启所有的进程才会恢复正常，可以使用all来完成
```bash
    [root@localhost ~]# supervisorctl restart all   
    [root@localhost ~]# supervisorctl
    supervisor> restart
    Error: restart requires a process name
    restart <processname>			Restart a process.                          #重启单个进程
    restart <processname> <processname>	Restart multiple processes              #重启多个进程
    restart all				Restart all processes                               #重启所有进程
```
* 像我们这次需要完成的任务是将所有进程存留的日志格式规范化，需要更新所有进程的配置文件，那边我们需要先对所有进程进程update，然后在start一次，指的注意的是，我们在update已经started的任务时，supervisor会先停止该进程，然后进行更新，更新完之后并不会自动启动进程。所以我们就需要了解到进程什么时候可以update，执行update停止进程时不会影响到线上业务时才可以执行，要不然就要上演一场从更新进程到跑路的大戏了。。。
```bash
    [root@localhost ~]# supervisorctl update all
    [root@localhost ~]# supervisorctl start all
```
* 使用sed规范所有进程配置文件的日志配置
    1、所有日志均存在 /etc/supervisor/*.ini中
    2、定义日志大小的参数stdout_logfile_maxbytes
    3、定义日志数量的参数stdout_logfile_backups
    综合以上三点来看虽然有几十个进程配置文件，所幸还是有规律可选，所以完成起来也算简单，这时候就体现了sed的强大之处，基础知识不扎实这个时候就要求助于google了。
```bash
    [root@localhost ~]# sed -i "s/^.*stdout_logfile_maxbytes.*$/stdout_logfile_maxbytes = 200MB/g" /etc/supervisor/*.ini     
    [root@localhost ~]# sed -i "s/^.*stdout_logfile_backups.*$/stdout_logfile_backups = 5/g" */etc/supervisor/.ini 
    [root@localhost ~]# supervisorctl update all
    [root@localhost ~]# supervisorctl start all
    #两次整行替换，然后更新配置文件后启动所有进程，大功告成~！
```
* 更新完进程之后，还有一个问题就是处理目前已经产生的日志了，在之前有很多的日志备份最多的存在30份，虽然更新了进程将所有的日志设置为5份，但是在不确定supervisor会不会回滚之前已经产生的多余日志的情况下还是手动删除为好，秉持着能用一条命令的情况下绝不使用两条命令，执行以下删除命令即可：
```bash
    [root@localhost ~]# ls /www/site/logs/*log.* | awk -F . '{if (NR>1){if ($NF>4)print $0}}' | xargs -t -n 1 rm -rf
    #日志命名格式为
    #program1.log
    #program1.log.1
    #program1.log.2
    ...
    #program1.log.30
```