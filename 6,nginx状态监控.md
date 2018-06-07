监控

　　在Nginx的插件模块中有一个模块stub_status可以监控Nginx的一些状态信息，默认安装可能没有这个模块，手动编译的时候加一下即可。

**1. 模块安装**

　　先使用命令查看是否已经安装这个模块：



```
[root@ihxb123Z nginx]# ./nginx -V (V大写会显示版本号和模块等信息、v小写仅显示版本信息。
```



### 问题解决

没有安装的话，可以在tar包安装编译的时候添加如下参数：

\# ./configure --prefix=/usr/local/nginx **--with-http_stub_status_module**

已经安装过的nginx，可以手动添加此莫款：

先查看安装Nginx时候添加的参数，

\# /usr/local/nginx/sbin/nginx -V
nginx: nginx version: nginx/1.0.4
nginx: built by gcc 4.1.2 20080704 (Red Hat 4.1.2-52)
nginx: configure arguments: --prefix=/usr/local/nginx

保证原有参数的前提下，重新./config,添加新模块

\# cd /usr/local/src/nginx-1.0.4

\# ./configure --prefix=/usr/local/nginx **--with-http_stub_status_module**

\# make 

×× 千万不要make install ，不然会覆盖原有的nginx

然后备份原有的/usr/local/nginx/sbin/nginx

\# mv /usr/local/nginx/sbin/nginx{，.bak}

将新的拷贝过来

\# cp /usr/local/src/nginx-1.0.4/objs/nginx /usr/local/nginx/sbin/

然后重启nginx即可

这里需要注意，有时候Nginx平滑重启，模块会家不上，可以尝试kill掉之后，重新启动即可



**2. Nginx配置**

　　安装后只要修改nginx配置即可，配置如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
location /hxbcdnstatus {
            stub_status            on;
            access_log             off;
        　  allow 127.0.0.1;
            deny all;
            #auth_basic              "NginxStatus";
            #auth_basic_user_file  conf/nginxstaus;
}
```



此处默认只有本地访问，如果远程可以查看需要加相关的IP或者干脆去掉Deny all即可。加密文件可以使用#htpasswd -c /usr/nginx/conf hxb 命令来创建。配置完成后需要重启Nginx服务。

　　状态配置只能是针对某个Nginx服务。目前Nginx还无法做到针对单个站点进行监控。

　　**3. 状态查看**

　　配置完成后在浏览器中输入http://127.0.0.1/hxbcdnstatus查看，显示信息如下：

```
Active connections: 100 
server accepts handled requests
 1075 1064 6253 
Reading: 0 Writing: 5 Waiting: 95 
```

　　**4. 参数说明**

　　active connections – 活跃的连接数量

　　server accepts handled requests — 总共处理了107520387个连接 , 成功创建107497834次握手, 总共处理了639121056个请求

　　每个连接有三种状态waiting、reading、writing

　　reading —读取客户端的Header信息数.这个操作只是读取头部信息，读取完后马上进入writing状态，因此时间很短。

　　writing — 响应数据到客户端的Header信息数.这个操作不仅读取头部，还要等待服务响应，因此时间比较长。

　　waiting — 开启keep-alive后等候下一次请求指令的驻留连接.

　　正常情况下waiting数量是比较多的，并不能说明性能差。反而如果reading+writing数量比较多说明服务并发有问题。

 

　　**补充：**

　　查看Nginx并发进程数：ps -ef | grep nginx | wc -l

　　查看Web服务器TCP连接状态：netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'