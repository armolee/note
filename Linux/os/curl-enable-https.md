## curl支持https

* 安装openssl，https://www.openssl.org/source/

```bash
  tar xf openssl-1.0.2g.tar.gz  
  #解压时不能使用-z参数，否则会以下产生错误，z：表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压
  /usr/bin/ld: libcrypto.a(rsaz_exp.o): relocation R_X86_64_32 against `.rodata' can not be used when making a shared object; recompile with -fPIC
  libcrypto.a(rsaz_exp.o): could not read symbols: Bad value

  collect2: ld returned 1 exit status


  #安装，编译时需要启动动态库，--shared，否则curl无法启用ssl
  cd openssl-1.0.2g && .configure --shared && make  && make test && make install

  # 创建链接
  mv /usr/bin/openssl /usr/bin/openssl.bak
  ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
  ln -s /usr/local/ssl/include/openssl /usr/include/openssl
  echo  "/usr/local/ssl/lib" >>  /etc/ld.so.conf
  ldconfig
```  

* 安装curl  

```bash
  ./configure –with-ssl=/usr/local/ssl
  make
  make install
  ldconfig
  [root@al-bj2c-prod.opencv~]# curl -V
  curl 7.55.1 (x86_64-pc-linux-gnu) libcurl/7.55.1 OpenSSL/1.0.2j zlib/1.2.7
  Release-Date: 2017-08-14
  Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp
  Features: AsynchDNS IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP UnixSockets HTTPS-proxy

```
