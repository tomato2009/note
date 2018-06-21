#### windows 配置主从redis

##### 1，运行两个redis实例

* 在redis的安装目录下，复制redis.windows.conf 为 redis.windows6380.conf
* 修改redis.windows6380.conf 目录中的端口问 port 6380
* 修改 logfile "server_log6380.txt"
* 可选：把 redis-server.exe redis.windows6380.conf 配置为服务 



#####2 ，更改**从redis**的配置文件：redis.windows6380.conf 

* 更改**主从配置**的参数：

*  slaveof <masterip> <masterport>  把#去掉:slaveof 127.0.0.1 6379  注意，slaveof前不能有空格

  其中6379 为主redis端口

* 一切更改完成后，启动从redis服务



这时，在主redis中修改 key value，会自动同步到从redis上



