## 由anacrontab管理的定时任务，例如logrotate

```bash  
  #anacrontab的配置文件
  [root@localhost log]# cat /etc/anacrontab   
  # /etc/anacrontab: configuration file for anacron
  # See anacron(8) and anacrontab(5) for details.
  SHELL=/bin/sh
  PATH=/sbin:/bin:/usr/sbin:/usr/bin
  MAILTO=root
  # the maximal random delay added to the base delay of the jobs
  RANDOM_DELAY=45                 #最大延迟时间
  # the jobs will be started during the following hours only
  START_HOURS_RANGE=3-22   #只在03到22点之间执行

  #period in days   delay in minutes   job-identifier   command
  1	5	cron.daily		nice run-parts /etc/cron.daily  
  7	25	cron.weekly		nice run-parts /etc/cron.weekly
  @monthly 45	cron.monthly		nice run-parts /etc/cron.monthly

  # 解释
  1	5	cron.daily		nice run-parts /etc/cron.daily  
  #每天都执行/etc/cront.daily/目录下的脚本文件，真实的延迟RANDOM_DELAY+delay。这里的延迟是5分钟，加上上面的RANDOM_DELAY，所以实际的延迟时间是5-50之间，开始时间为03-22点，如果机器没关，那么一般就是在03:05-03:50之间执行。nice命令将该进程设置为nice=10，默认为0，即低优先级进程。

  #如果RANDOM_DELAY=0，那么表示准确延迟5min，即03:05执行cron.daily

  #整个逻辑流为：
  crontd进程每小时的01分执行/etc/cront.hourly/0anacron --->执行anacron -->根据/etc/anacrontab的配置执行/etc/cron.daily，/etc/cron.weekly，/etc/cron.monthly

```
