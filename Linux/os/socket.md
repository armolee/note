####说明  
unix套接字文件，若生成在/tmp目录下，则无法被其他进程访问。  
####原因  
在/tmp下，生成的临时文件带有namespaced，所以仅有生产者能够访问到该临时文件，其他进程则无法访问该文件  
####以nginx调用uwsgi套接字为例  
* 启动uwsgi进程  

        [uwsgi]
        chdir=/usr/local/ops/
        module=ops.wsgi:application
        socket=/tmp/uwsgi.sock  
        pidfile=/tmp/uwsgi.pid  
        daemonize=/tmp/uwsgi.log   
        chmod-socket = 666
        http=192.168.80.81:8000
        uid=nginx
        gid=nginx
        master=true
        vacuum=true
        thunder-lock=true
        enable-threads=true
        harakiri=30
        post-buffering=4096
        [root@localhost]# uwsgi --ini uwsgi.ini


* nginx调用uwsgi.sock  

        location / {
            include uwsgi_params;
            uwsgi_connect_timeout 30;
            uwsgi_pass unix:/tmp/uwsgi.sock;
        }

* 访问提示404，查看日志提示unix:/tmp/uwsgi.sock failed (2: No such file or directory)   

        2018/02/07 19:45:57 [crit] 61999#0: *1 connect() to unix:/tmp/uwsgi.sock failed (2: No such file or directory) while connecting to upstream, client: 192.168.80.14, server: www.armo.com, request: "GET / HTTP/1.1", upstream: "uwsgi://unix:/tmp/uwsgi.sock:", host: "192.168.80.81"

* 将sock文件变更路径后访问正常

        location / {
            include uwsgi_params;
            uwsgi_connect_timeout 30;
            uwsgi_pass unix:/test/uwsgi.sock;
        }
