## glibc的升级

centos6.8系统自带的glibc版本为2.12
```
  /lib64/libc.so.6 -> libc-2.12.so

  [root@soyoung-dev ~]# strings /lib64/libc.so.6 |grep GLIBC_
  GLIBC_2.2.5
  GLIBC_2.2.6
  GLIBC_2.3
  GLIBC_2.3.2
  GLIBC_2.3.3
  GLIBC_2.3.4
  GLIBC_2.4
  GLIBC_2.5
  GLIBC_2.6
  GLIBC_2.7
  GLIBC_2.8
  GLIBC_2.9
  GLIBC_2.10
  GLIBC_2.11
  GLIBC_2.12
  GLIBC_PRIVATE
  [root@soyoung-dev ~]# ldd --version
  ldd (GNU libc) 2.12
  Copyright (C) 2012 Free Software Foundation, Inc.
  This is free software; see the source for copying conditions.  There is NO
  warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
  Written by Roland McGrath and Ulrich Drepper.

```

* 版本下载地址
  http://ftp.gnu.org/gnu/glibc/

* 这次升级到2.17
```
  tar xf glibc-2.17.tar.gz
  cd glibc-2.17
  mkdir build
  cd build/
  ../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
  make && make install
  strings /lib64/libc.so.6 | grep GLIBC
  GLIBC_2.2.5
  GLIBC_2.2.6
  GLIBC_2.3
  GLIBC_2.3.2
  GLIBC_2.3.3
  GLIBC_2.3.4
  GLIBC_2.4
  GLIBC_2.5
  GLIBC_2.6
  GLIBC_2.7
  GLIBC_2.8
  GLIBC_2.9
  GLIBC_2.10
  GLIBC_2.11
  GLIBC_2.12
  GLIBC_2.13
  GLIBC_2.14
  GLIBC_2.15
  GLIBC_2.16
  GLIBC_2.17
  GLIBC_PRIVATE
  [root@soyoung-dev ~]# ldd --version
  ldd (GNU libc) 2.17
  Copyright (C) 2012 Free Software Foundation, Inc.
  This is free software; see the source for copying conditions.  There is NO
  warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
  Written by Roland McGrath and Ulrich Drepper.
```

* 版本升级会遇到的问题，crontab无法执行，rsync无法执行，会出现以下报错
```
  #crontab
  Aug  9 14:22:01 soyoung-dev crond[22328]: (armo) FAILED to authorize user with PAM (Module is unknown)
  Aug  9 14:22:01 soyoung-dev crond[22327]: (armo) FAILED to authorize user with PAM (Module is unknown)
  Aug  9 14:22:01 soyoung-dev crond[22322]: (armo) FAILED to authorize user with PAM (Module is unknown)
  Aug  9 14:22:01 soyoung-dev crond[22319]: (armo) FAILED to authorize user with PAM (Module is unknown)
  Aug  9 14:22:01 soyoung-dev crond[22326]: (armo) FAILED to authorize user with PAM (Module is unknown)

  # rsync
  @ERROR: invalid uid root
  rsync error: error starting client-server protocol (code 5) at main.c(1516) [sender=3.0.9]
  Build step 'Execute shell' marked build as failure
  Finished: FAILURE
```

* 版本回退
  使用系统自带的rpm包重新覆盖安装  
  yum install glibc
