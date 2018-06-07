#### 1，nginx安装目录介绍及负载均衡配置

conf：所有配置文件，如nginx.conf，每个文件都有一个.default 的备份文件，方便还原

html：访问nginx默认的html

~~~shell
[root@localhost nginx]# cat html/index.html 
~~~

内容

~~~ht
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~



logs：access.log、error.log、nginx.pid

sbin：目前只有一个文件，Nginx服务的主程序

#### 2，nginx服务的启停

2.1 查看进程id

方式1：直接查看logs目录下的nginx.pid文件

~~~shell
[root@localhost nginx]# cat logs/nginx.pid 
2549
~~~

方式二：ps查看

~~~shel
[root@localhost nginx]# ps -ef|grep nginx
root      2549     1  0 04:03 ?        00:00:00 nginx: master process ./../sbin/nginx
nobody    2607  2549  0 05:27 ?        00:00:00 nginx: worker process
root      3151  2073  0 22:47 pts/0    00:00:00 grep --color=auto nginx
~~~

可以看到包含一个nginx的主进程 master process和一个个工作进程 worker process，其中pid为 2549

和nginx.pid 文件中的一致



##### 2.1，启动

查看nginx的命令帮助

~~~shell
[root@localhost nginx]# ./sbin/nginx -h
nginx version: nginx/1.10.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
~~~



查看nginx版本

~~~shell
[root@localhost nginx]# ./sbin/nginx -V
nginx version: nginx/1.10.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) 
configure arguments:
~~~



查看nginx的配置文件是否有语法错误

~~~shel
[root@localhost nginx]# ./sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
~~~

启动nginx服务

~~~shel
[root@localhost nginx]# ./sbin/nginx
~~~

##### 2.2，停止

~~~she
[root@localhost nginx]# ps -ef | grep nginx    
root      2549     1  0 04:03 ?        00:00:00 nginx: master process ./../sbin/nginx
nobody    2607  2549  0 05:27 ?        00:00:00 nginx: worker process
root      3218  2073  0 23:02 pts/0    00:00:00 grep --color=auto nginx
[root@localhost nginx]# 
[root@localhost nginx]# 
[root@localhost nginx]# 
[root@localhost nginx]# kill -9 2549
~~~

##### 2.3，重启

更新了nginx的配置文件和加入新模块后，如果希望当前的nginx服务应用型的配置或是新模块生效，就需要重启nginx服务，当然也可以停止nginx服务后再启动nginx服务。但是这块主要介绍平滑重启

~~~shel
[root@localhost nginx]# ./sbin/nginx -s reload
~~~



#### 3，nginx配置

查看配置文件

~~~shel
[root@localhost nginx]# cat conf/nginx.conf

#user  nobody;
worker_processes  3;							#全局配置

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {										#只在events生效
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    #tomcat 8080
    upstream tomcats {
        server 192.168.1.150:8081;
        server 192.168.1.150:8082;
        }

    #upstream tomcatserver2 {
    #   server 192.168.1.150:8082;
    #   }


    server {
        listen       80;
        #server_name  8081.tomato.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcats;
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    #server {
        #listen       80;
        #server_name  8082.tomato.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #location / {
            #add_header backendIP $upstream_addr;
            #proxy_pass http://tomcatserver2;
            #root   html;
            #index  index.html index.htm;
        #}
    #}

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
~~~

nginx.conf 一共由三个部分组成，分别问全局块、evnets块和http块

http又包含http全局块，多个server块

server块中又包含server全局块和多个location块



##### 3.1，全局块

有文件开始到evnets结束

主要设置影响nginx服务器整体运行配置指令

通常包括nginx服务用户组、运行生成的worker process数、nginx进程pid存放路径、日志的存放路径和类型已经配置文件引入等

##### 3.2，events块

主要影响nginx服务器与用户的网络连接，常用设置包括是否开启多worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，worker process可以同时支持最大连接数

这一部分的指令对nginx的性能影响较大

##### 3.3，http块

代理、缓存、日志定义等功能和第三方模块的配置都可以放在这个模块中

##### 3.4 server模块

server模块和虚拟主机的概念有密切联系

每个http块都可以包含多个server块，而每个server块就相当于一台虚拟主机，他内部可以有多台主机联合提供服务

##### 3.5，location模块

每个server块可以包含多个location块

主要作用是记忆nginx服务器接收到的请求字符串(例如server_name/url_string)，对除虚拟主机名称(也可以是IP别名)之外的字符串(嵌入url_string部分)进行匹配，对特定的请求进行处理，地址定向，数据缓存和应答控制等功能都是在这部分实现，许多第三方块的配置也是在location块中实现



#### 4，配置

##### 4.1，nginx服务器用户组

~~~shell
user [user] [group]
~~~



##### 4.2，配置worker process 数

~~~shell
worker_processes  3;
或worker_processes  auto；  #自动检测
~~~

 配置 reload后，发现有三个worker process

~~~shel
[root@localhost nginx]# ps ax|grep nginx
 2549 ?        Ss     0:00 nginx: master process ./../sbin/nginx
 3235 ?        S      0:00 nginx: worker process
 3236 ?        S      0:00 nginx: worker process
 3237 ?        S      0:00 nginx: worker process
 3265 pts/0    R+     0:00 grep --color=auto nginx
~~~



##### 4.3，负载均衡配置

###### 4.3.1 对所有资源配置轮询负载均衡

~~~shell
upstream tomcats {
    server 192.168.1.150:8081;
    server 192.168.1.150:8082;
}


server {
        listen       80;
        #server_name  8081.tomato.com;

        location / {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcats;
        }
}       
~~~



###### 4.3.2 对所有资源实现加权轮询负载均衡

~~~shell

upstream tomcats {
    server 192.168.1.150:8081 weight=3;  #3/5概率命中
    server 192.168.1.150:8082 weight=2;  #2/5概率命中
}


server {
        listen       80;
        #server_name  8081.tomato.com;

        location / {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcats;
        }
}  
~~~



###### 4.3.3，按访问资源类型实现负载均衡

~~~shell
upstream tomcat01 {
    server 192.168.1.150:8081;  
    server 192.168.1.150:8082;
}

upstream tomcat02 {
    server 192.168.1.150:8083;
    server 192.168.1.150:8084;
}


server {
        listen       80;
        #server_name  8081.tomato.com;

        location /video {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcat01;
        }
        
        location /file {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcat02;
        }
} 
~~~



###### 4.3.4，按访问域名实现负载均衡

~~~shel
upstream tomcat01 {
    server 192.168.1.150:8081;  
    server 192.168.1.150:8082;
}

upstream tomcat02 {
    server 192.168.1.150:8083;
    server 192.168.1.150:8084;
}


server {
        listen       80;
        server_name  video.tomato.com;

        location / {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcat01;
        }
}

server {
        listen       80;
        server_name  file.tomato.com;
         
        location / {
            add_header backendIP $upstream_addr;
            proxy_pass http://tomcat02;
        }
}
~~~



###### 4.3.5，URL重写的负载均衡

