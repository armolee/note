### 使用ldap将内部所有平台接入统一认证

### nginx需要使用第三方ldap模块可以接入ldap的认证
```bash  
  仓库地址：git clone https://github.com/kvspb/nginx-auth-ldap.git
  nginx -V   #查看nginx版本和目前编译的参数

  [admin_soyoung@al-bj2c-dev.web-allsite-01 bin]$ nginx -V
  nginx version: nginx/1.10.3
  built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC)
  built with OpenSSL 1.0.1e-fips 11 Feb 2013
  TLS SNI support enabled
  configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --add-module=/usr/local/src/lua-nginx-module --add-module=/usr/local/src/ngx_devel_kit --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-file-aio --with-threads --with-ipv6 --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'

  http://nginx.org/en/download.html  #nginx官网下载对应版本，解压后使用目前的编译参数加上以下参数进行再次编译，安装
  --add-module=/usr/local/src/nginx-auth-ldap   #https://github.com/kvspb/nginx-auth-ldap.git解压的地址

  nginx配置文件：
  http {   #http段配置
    ldap_server openldap {

      url ldap://ldap.ip:389/dc=xxxxx,dc=com?uid?sub?(&(objectClass=inetOrgPerson));
      # uid使用该属性为用户名，objectClass配置为uid对应的Class(inetOrgPerson)
      binddn "cn=username,dc=soyoung,dc=com";
      # ldap管理员用户账号，建议使用只读账号
      binddn_passwd "password";
      # 管理员账号密码
      group_attribute uid;

      group_attribute_is_dn on;

      require valid_user;

     }

  }

  location / {   #location段
     auth_ldap "use ldap account";  #提示信息
     auth_ldap_servers openldap;    #认证方式为openldap
  }

  # 重启服务，nginx的error日志中包含报错信息，如果认证失败可以查看错误日志定位
```
