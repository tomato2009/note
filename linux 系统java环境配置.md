### 1， linux下java环境配置



#### 1.1，jdk、tomcat安装包备份

https://pan.baidu.com/s/1RpxgBXRMny04EOvO7lMjHA



#### 1.2，jdk安装及环境变量配置

/usr/local/app目录下 解压jdk

~~~shell
tar -zxvf  jdk-8u161-linux-x64.tar.gz
~~~

后可以看到 目录：jdk1.8.0_161 

配置环境变量：最好先备份/etc/profile文件

~~~shell
vi /etc/profile
~~~

在文件最开始增加以下几行

~~~shell
# java env
export JAVA_HOME=/usr/local/app/jdk1.8.0_161
export CLASSPATH=.:$JAVA_PATH/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
~~~



#### 1.3，tomcat 解压及运行

同样解压 Tomcat

~~~shell
tar -zxvf apache-tomcat-8.5.30.tar.gz
~~~

运行 apache-tomcat-8.5.30/bin/start.sh文件

在bin目录下输入

~~~shell
./start.sh
~~~



#### 1.4 nginx 安装及运行

nginx的一些模块依赖一些lib库，在安装nginx之前，须先安装这些lib库，
依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装

~~~shel
依赖环境准备
1. 	yum install gcc-c++ 
2.	yum install pcre pcre-devel  
3.	yum install zlib zlib-devel 
4.	yum install openssl openssl--devel  
5.  wget http://nginx.org/download/nginx-1.9.14.tar.gz

Nginx编译安装
6. 	tar xvzf nginx-1.9.14.tar.gz 
	cd 解压后的Nginx目录
7. 	./configure 
8.	make && make install
9. 	cd /usr/local/nginx/sbin
10. ./nginx 
~~~



停止Nginx命令

~~~shel
#查询nginx主进程号  

ps -ef | grep nginx 

#停止进程  

kill -QUIT 主进程号  

#快速停止  

kill -TERM 主进程号  

#强制停止  

pkill -9 nginx 

重启nginx命令
[root@admin local]# /usr/local/nginx/sbin/nginx -s reload 
~~~



3.1.5将nginx配置成服务
通过上面的操作，nginx可以通过命令行启动，但nginx服务未被加入到开机自启动列表，重启服务器后，未发现nginx服务，我们需要手动加入开机自启动

~~~shel
。
通过vim /lib/systemd/system/nginx.service来添加nginx.service文件，并输入如下内容：
[plain] view plain copy
[Unit]  
Description=nginx 1.12.0  
After=network.target  
  
[Service]  
Type=forking  
ExecStart=/usr/local/nginx-1.12.0/sbin/nginx  
ExecReload=/usr/local/nginx-1.12.0/sbin/nginx -s reload  
ExecStop=/usr/local/nginx-1.12.0/sbin/nginx -s quit  
PrivateTmp=true  
  
[Install]  
WantedBy=multi-user.target  
~~~

