![mysqltmp]http://www.8i88.cn/static/mysql-tmp.md

当事务开始时，它将缓冲区语句分配一个binlog_cache_size大小的缓冲区（我这里设置的是16777216bytes，即16MB）。 如果一个语句大于此，线程将打开一个临时文件来存储事务(默认是存放在/tmp/目录下）。 当线程结束时，临时文件会自动被删除。

  上面就是因为事务里面的临时文件超过16MB了，被放到/tmp目录下了，但是这个临时文件实在太大了，导致磁盘空间不足告警了。



  解决方法：

  等上面的查询结束后，我们先关闭mysqld。（条件能允许的话，当然是让查询自己结束。如果直接kill掉的话，估计回滚也要话挺长时间的）



  然后调整mysql的tmpdir到其他更大的磁盘去。

  mkdir /bdata/mysql_tmp

  chown mysql.mysql /bdata/mysql_tmp -R

  chown 1777 -R /bdata/mysql_tmp -R

  vim /etc/my.cnf

  [mysqld]

  tmpdir = /bdata/mysql_tmp



  然后启动mysql即可
